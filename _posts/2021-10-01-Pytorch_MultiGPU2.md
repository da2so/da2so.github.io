---
layout: post
title: PyTorch MultiGPU (2) - Single-GPU vs Multi-GPU (DistributedDataParallel)
tags: [PyTorch, MultiGPU, DistributedDataParallel]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2021-09-23-Pytorch_MultiGPU/post.PNG
---

## 1. Introduction

게재된 모든 실험은 python 3.6, Pytorch 1.7.0 에서 진행되었음을 알려드립니다. 
{: .box-note}


해당 글은 Pytorch에서 [이전 글](https://da2so.github.io/2021-09-23-Pytorch_MultiGPU/)에서 말씀드린 Pytorch의 DistributedDataParallel에 대해 설명드립니다.
**이전 글과 Experiment setting은 동일하니 궁금하시면 이전 글에서 참고 하십시오!**
오늘 설명드릴 목차는 다음과 같습니다.
 
- DistributedDataParallel 이란?
- DistributedDataParallel 사용 방법
- DistributedDataParallel 결과 비교 (with Single-GPU and DataParallel)

### 2. DistributedDataParallel (DDP) 이란?

DataParallel(DP)과 비교했을 때 머가 다르냐!? 가 중요한데요. 일단 기본적으로 DDP는 <span style="color:#C70039">**multi-process parallelism**</span>을 사용합니다.
이 차이점이 왜 중요하냐면 [이전 글](https://da2so.github.io/2021-09-23-Pytorch_MultiGPU/)에서도 말씀드렸듯이 GIL때문에 multi-thread가 성능효과를 못내자나요. 그래서 
multi-process로 가버리자! 라는 마인드입니다. 뿐만 아니라 DDP는 performance optimization 기술도 들어가 있다고 하네요. (자세한건 이 [논문](http://www.vldb.org/pvldb/vol13/p3005-li.pdf)을 참고하라고 하네요.)


즉, DDP를 통해 Multi-GPU를 쓸 경우 각 GPU가 자신을 위한 하나의 process를 갖게됩니다(DP는 각 GPU가 thread를 가졋죠). 각 process가 어떤 과정을 거치면 실행되는 지 알아보기 전에 만족해야 할 조건이 있습니다. Process가 여러개이므로 서로가 통신이 되어야 하는 건데요. 그래야 weight update 공유도 하고 model을 parallel하게 학습시킬 수 있겠죠. 그래서 Pytorch에서는 process간의 통신을 위해 torch.distributed.init_process_group() 함수를 사용하도록 요구합니다. 이 함수는 사용되는 process들을 grouping하여 초기화시켜준다고 생각하시면 됩니다.

그럼 이제 process간의 통신이 가능해졌으니 DDP가 어떻게 작동하는 지 보시죠.

1. DDP constructor는 각기 다른 GPU를 가진 process에 model을 복제하고 rank 0의 process가 다른 process들에게 weight를 broadcast하여 같은 weight값을 갖게 한다.
    - 기본적으로 각 processs는 고유한 숫자의 rank를 가지는 데 process의 고유 id라고 생각하면 편하다.
2. 각 process는 disk에서 서로 다른 mini-batch data를 불러오게 됩니다.
    - Distributed data sampler를 사용함으로써 각 process에 data가 overlapping되지 않도록 해줍니다. (How는 좀 이따가...)
3. 각 process의 GPU에서 forward pass와 loss의 계산이 이루어지게 됩니다.
4. 매 iterationd마다 Backward시에 각 GPU에서 gradient구하고 서로 all-reduced됩니다.
    - All-reduced: 각 GPU에서 mini-batch에 대한 graident를 구하고 통신을 통해 gradient의 평균을 구하여 각 process에서 model이 동일하게 weight update하도록 함.


![1](https://da2so.github.io/assets/post_img/2021-10-01-Pytorch_MultiGPU2/1.png){: .mx-auto.d-block width="100%" :}

### 3. DistributedDataParallel 사용 방법

이제부터 코드를 따라 하나씩 이해해가면서 사용방법을 익혀 보자.








```python
os.environ['CUDA_VISIBLE_DEVICES'] = '1, 2, 3, 4'  # what GPUs you will use
device = torch.device('cuda:0' if cuda else 'cpu') 

model = torch.nn.DataParallel(model) # convert model to DataParallel (DP) model
model.to(device)

...

outputs = model(images)
loss = nn.CrossEntropyLoss()(outputs, labels)
loss.backward() 
optimizer.step() 
```

Multi-GPU를 사용하기 위해 torch.nn.DataParallel function인자에 Multi-GPU로 training하고 싶은 model만 넣어주면 됩니다.
torch.nn.DataParallel를 사용하였을 때 model의 forward와 backward는 다음과 같이 작동합니다.

![1](https://da2so.github.io/assets/post_img/2021-09-23-Pytorch_MultiGPU/1.png){: .mx-auto.d-block width="100%" :}

1. **Forward process**s
    1. GPU 1가 master GPU로써 batch를 사용하는 GPU 개수(4개)로 나누어 줍니다(scatter).
    2. Model를 GPU 수만큼 복제(replicate)합니다.
    3. 나누어진 batch를 각각 복제된 model에 forwarding해줍니다.
    4. forward된 outputs을 master GPU에 다시 모아줍니다(gather). 
2. **Backward process**
    1. Master GPU에서 loss를 compute합니다.
    2. 계산된 loss를 GPU 개수(4개)로 나누어 보내줍니다(scatter).
    3. loss로부터 각 GPU에서 backwarding를 통해 gradient를 구해줍니다.
    4. 각 GPU에서 계산된 gradient를 master GPU에 모아줍니다(reduce).
3. **Master GPU에서 weight를 update해줍니다.**


실제로 위의 과정이 일어나는 지 확인하기 위해 다음과 같은 변수 value 및 size를 print 해보겠습니다.
먼저 mobilenetv2의 forward 함수에서 input과 output의 size를 출력해보겠습니다.

```python
    def forward(self, x): # in mobilenetv2.py
        input = x
        print(f'Input size: {input.size()}')
        x = self.features(x)
        x = self.conv(x)
        x = self.avgpool(x)
        x = x.view(x.size(0), -1)
        x = self.classifier(x)
        output = x 
        print(f'Output size: {output.size()}')
        return output
```

**Out:**

![1](https://da2so.github.io/assets/post_img/2021-09-23-Pytorch_MultiGPU/2.PNG){: .mx-auto.d-block width="60%" :}


총 4개의 GPU를 사용하였고 Batch size가 256개 이므로 각 GPU에서 64개 batch단위로 forward하게 되는것을 확인가능합니다.
그리고 다음과 같이 위 코드에서 output자체를 출력해볼경우에도 각 device에서 서로 다른 batch image를 처리하는 것을 알 수 있습니다.

**Out:**


![1](https://da2so.github.io/assets/post_img/2021-09-23-Pytorch_MultiGPU/3.png){: .mx-auto.d-block width="80%" :}


다음으로는 나누어진 outputs들이 합쳐지는 지 다음과 같은 코드로 확인해보면

```python
outputs = model(images) 
print(f'Output: {outputs}')
print(f'Output size: {outputs.size()}')
```

**Out:**

![1](https://da2so.github.io/assets/post_img/2021-09-23-Pytorch_MultiGPU/4.PNG){: .mx-auto.d-block width="75%" :}

각 GPU에서 forwards된 outputs이 master GPU(cuda:0)에 합쳐지는 것을 볼 수 있습니다.


### 4. Single-GPU vs Multi-GPU (DataParallel) 결과 비교

Mobilenetv2 model에서 총 10epoch을 돌려 Single-GPU와 Multi-GPU의 elapse time를 비교하겠습니다.


![1](https://da2so.github.io/assets/post_img/2021-09-23-Pytorch_MultiGPU/5.png){: .mx-auto.d-block width="100%" :}


위의 그림에서 볼 수 있듯이 같은 batch일 경우는 Multi-GPU의 효과를 보기 힘드네요. 추측해본건데 같은 batch일경우
Dataparallel사용에 따른 scatter, replicate, gather 과정이 추가되므로 속도가 저하되는 것 같습니다.

MobileNetv2 모델하나로 판단하는 게 generality가 떨어진다고 생각되어 ResNet50으로도 돌려보았습니다.

![1](https://da2so.github.io/assets/post_img/2021-09-23-Pytorch_MultiGPU/6.png){: .mx-auto.d-block width="80%" :}

분석하자면 다음과 같습니다.

- 상대적으로 모델이 작은 MobileNetv2에서 Multi-GPU의 time computation효과를 보기 위해서는 batch size를 늘려야한다.
- 반대로, 모델이 큰 ResNet50에서는 Single-GPU와 비교했을 때 같은 batch size이더라도 Multi-GPU로 나누어 training하는 게 더 빠르다.
- Multi-GPU를 사용한 모델의 test inference time이 Single-GPU보다 높다.


### 5. Multi-GPU (DataParallel) 의 문제점

이 Section에서는 Dataparallel을 썼을 때의 문제점을 정리해보고 그에 대한 해결방안이 무엇이 있는지 알아보겠습니다.
먼저 Dataparallel의 문제점은 크게 2가지입니다.

1. **추가적인 resource사용이 요구된다.**

위에서 언급드린 (a) 매 iteration마다 model를 replicate하고 forward시켜야 한다. (b) gather, reduce 작업이 추가로 요구된다.
라는 process때문에 training time을 더 효율적으로 줄일 수 없다고 합니다.

2. **single-process multi-thread parallelism이 GIL에 의한 방해를 받는다**

먼저 GIL이란 **Global Interpreter Lock** 의 약자인데 여러개의 쓰레드간의 동기화를 위해 사용되는 기술인데 동시에 하나 이상의 쓰레드가 실행되지 않게 합니다.
예를 들어 3개의 thread가 동시에 cpu를 점유할 수 없습니다. 즉, 멀티 쓰레드 프로그래밍해도 성능이 향상되지 않는다는 말이다.
이 기술을 쓰는 이유는 만약 parallel 한 multi-threads 구현체들을 사용할수 있도록 할 경우(GIL을 쓰지 않을 경우) single thread만 사용하는 경우의 성능 저하를 만들어낸다.

그래서 정리하자면 GIL때문에 DataParallel사용 시 multi-thread paralleslism이 잘 작동하지 않는 것이다. 그렇다면 이를 해결하기 위해 Pytorch에서는 <span style="color:#C70039">**DistributedDataParallel**</span>을 사용하기를 
권장하는데 이는 다음 글에서 만나보자.


### <span style="color:#C70039 "> Reference </span>

[Training Neural Nets on Larger Batches: Practical Tips for 1-GPU, Multi-GPU & Distributed setups](https://medium.com/huggingface/training-larger-batches-practical-tips-on-1-gpu-multi-gpu-distributed-setups-ec88c3e51255)

[Pytorch docs](https://pytorch.org/tutorials/beginner/blitz/data_parallel_tutorial.html)