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


즉, DDP를 통해 Multi-GPU를 쓸 경우 각 GPU가 자신을 위한 하나의 process를 갖게됩니다(DP는 각 GPU가 thread를 가졋죠). 각 process가 어떤 과정을 거치면 실행되는 지 알아보기 전에 만족해야 할 조건이 있습니다. Process가 여러개이므로 서로가 통신이 되어야 하는 건데요. 그래야 weight update 공유도 하고 model을 parallel하게 학습시킬 수 있겠죠. 그래서 Pytorch에서는 process간의 통신을 위해 **torch.distributed.init_process_group()** 함수를 사용하도록 요구합니다. 이 함수는 사용되는 process들을 grouping하여 초기화시켜준다고 생각하시면 됩니다.

그럼 이제 process간의 통신이 가능해졌으니 DDP가 어떻게 작동하는 지 보시죠.

1. DDP constructor는 각기 다른 GPU를 가진 process에 model을 복제하고 rank 0의 process가 다른 process들에게 weight를 broadcast하여 동일한 weight값을 전달.
    - 각 processs는 고유한 숫자의 rank를 가지는 데 process의 고유 id.
2. 각 process는 disk에서 서로 다른 mini-batch data를 load.
    - **Distributed data sampler** 사용함으로써 각 process의 data가 overlapping되지 않게 함 (How는 좀 이따가...)
3. 각 process의 GPU에서 forward pass와 loss의 계산.
4. 매 iterationd마다 Backward시에 각 GPU에서 gradient구하고 서로 all-reduced됩니다.
    - All-reduced란 각 GPU에서 mini-batch에 대한 graident를 계산하고 통신을 통해 gradient 평균을 구하여 각 process의 model이 동일한 값으로 weight update.


![1](https://da2so.github.io/assets/post_img/2021-10-01-Pytorch_MultiGPU2/1.png){: .mx-auto.d-block width="60%" :}

### 3. DistributedDataParallel 사용 방법

먼저 python script를 실행하는 방법부터 조금은 다릅니다.

```python
python -m torch.distributed.launch --nproc_per_node 4 3_trainer_DDP.py
```

실행하려는 python script는 *3_trainer_DDP.py* 이며 *--nproc_per_node*의 값은 몇 개의 GPU를 쓸 건지 정하는 것입니다.
이제 코드안에서 DDP를 위해 필요한 중요한 code가 무엇인지 **순차적**으로 살펴보죠.

```python
LOCAL_RANK = int(os.getenv('LOCAL_RANK', -1))  
WORLD_SIZE = int(os.getenv('WORLD_SIZE', 1))

if __name__=="__main__":

    parser = argparse.ArgumentParser(description='PyTorch DDP')
    ...
    parser.add_argument('--device', type = str, default = "1,2,3,4", help = 'cuda device, i.e. 0 or 0,1,2,3')
    parser.add_argument('--local_rank', type=int, default=-1, help='DDP parameter, do not modify')
```

LOCAL_RANK는 위에서 말씀드린 RANK와 비슷한 개념으로 이해하시면 됩니다. 그래서 4개의 GPU를 사용하므로 process도 4개이며 각 process는 각기 다른 LOCAL_RANK값을 갖게 됩니다.

![1](https://da2so.github.io/assets/post_img/2021-10-01-Pytorch_MultiGPU2/2.png){: .mx-auto.d-block width="100%" :}


LOCAL_RANK값이 0번인 process가 master가 되고 WORLD_SIZE는 전체 프로세스의 수를 의미하고 master가 얼마나 많은 워커(process)들을 기다릴지 알 수 있습니다. 그리고 argparse 모듈을 통해
local_rank라는 argument는 꼭 추가해주어야 작동합니다. 


```python
import torch.distributed as dist
...
torch.cuda.set_device(LOCAL_RANK)
dist.init_process_group(backend="nccl" if dist.is_nccl_available() else "gloo")
```

1번째 줄에 의해 각 process가 고유한 LOCAL_RANK를 가지므로 점유하는 GPU도 다르게 됩니다. <span style="color:#C70039">**dist.init_process_group**</span>은 
distributed process group을 initialize하여 process간의 통신을 가능토록 합니다. 그리고 통신을 위한 backend를 nccl이나 gloo로 선택가능합니다. 어떤 backend를 쓸 지에 따라
CPU또는 GPU device에서 사용가능한 통신관련 function이 다릅니다. 


기본적으로 nccl이 NVIDIA GPU에 최적화 된 다중 GPU 및 다중 노드 집단 통신 library이므로 CUDA사용 시 nccl를 쓰는 게 좋다라고 생각하시고 nccl쓰시면 됩니다. 

```python
from torch.nn.parallel import DistributedDataParallel as DDP
model = model.to(LOCAL_RANK)
model = DDP(model, device_ids=[LOCAL_RANK])
```

이제 process끼리 통신이 가능해 졋으니 각 model를 DDP로 wrapping시키고 각 process가 가지는 GPU를 model에 할당 해주는 것이 위의 코드입니다.
<span style="color:#C70039">**DDP**</span> function을 통해서 model을 paralleize하게 되고 그로 인해 각 process에 model이 replicate됩니다. 
각 process의 model의 input data을 dataloader가 다음과 같은 코드로 동일한 크기로 나누어 주게 됩니다. (단, 각 process에 할당하는 data는 모두 다르다.)

```python
train_dt = CIFAR10(self.data_dir, transform=self.train_trans, download=True)
test_dt = CIFAR10(self.data_dir, train=False, transform=self.test_trans, download=True)

tr_sampler = torch.utils.data.distributed.DistributedSampler(train_dt, num_replicas=WORLD_SIZE, rank=LOCAL_RANK) if LOCAL_RANK != -1 else None
te_sampler = torch.utils.data.distributed.DistributedSampler(test_dt, num_replicas=WORLD_SIZE, rank=LOCAL_RANK) if LOCAL_RANK != -1 else None

train_loader = DataLoader(train_dt, batch_size=bs // WORLD_SIZE, num_workers=nw, pin_memory=True, sampler=tr_sampler)
test_loader = DataLoader(test_dt, batch_size=bs // WORLD_SIZE, num_workers=nw, pin_memory=True, sampler=te_sampler)
```

기존에 사용하던 부분과 크게 다른 점은 <span style="color:#C70039">**torch.utils.data.distributed.DistributedSampler**</span> 함수를 사용한다는 것입니다.
즉, 이 함수를 통해서 각 process에게 서로 다른 data를 주게 됩니다. 하나 더 다른 점은 batch_size를 WORLD_SIZE로 나누어 주게 됩니다. 밑의 그림과 같이 사용하려 하는 batch size가 256이라면
각 process에서는 64개의 batch로 나누어 계산하고 각기 다른 64개 batch에 대한 output을 내게 됩니다.

![1](https://da2so.github.io/assets/post_img/2021-10-01-Pytorch_MultiGPU2/3.png){: .mx-auto.d-block width="80%" :}

또한 각기 다른 input data를 받고 있다는 것을 보여드리기 위해 model의 output을 출력해보면 서로 다른 output값을 내는 것을 확인 가능합니다. (편의를 위해 64 batch 중 1번째만 출력)

![1](https://da2so.github.io/assets/post_img/2021-10-01-Pytorch_MultiGPU2/4.png){: .mx-auto.d-block width="80%" :}


### 4. DistributedDataParallel 결과 비교 (with Single-GPU and DataParallel)

[이전글](https://da2so.github.io/2021-09-23-Pytorch_MultiGPU/)과 experiment setting은 동일하게 진행하였으며 한 epoch당 train/test 시간(Avg_train/test)을 batch단위로 나누어서 비교하였습니다.

![1](https://da2so.github.io/assets/post_img/2021-10-01-Pytorch_MultiGPU2/5.png){: .mx-auto.d-block width="100%" :}


결과적으로 DDP를 쓰게되면 single-GPU와 다르게 batch size도 크게 setting할 수 있으며 batch size가 증가할 수록 singe-GPU나 DP에 비해 효과적으로 빠른 training을 할 수 있음을 보여준다.

### <span style="color:#C70039 "> Reference </span>

[Training Neural Nets on Larger Batches: Practical Tips for 1-GPU, Multi-GPU & Distributed setups](https://medium.com/huggingface/training-larger-batches-practical-tips-on-1-gpu-multi-gpu-distributed-setups-ec88c3e51255)

[Pytorch docs](https://pytorch.org/tutorials/beginner/blitz/data_parallel_tutorial.html)