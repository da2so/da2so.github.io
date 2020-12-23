---
layout: post
title: TFLite 뽀개기 (1)
tags: [TFLite, Mobile, Tensorflow]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-12-23-Master_TFlite/post.png
---

## TFLite 란?  
내용은 python, Tensorflow-gpu 2.x, keras model, mobile 에 한정되어 있음을 알려드립니다.  
{: .box-note}

*Tensorflow Lite (TFLite)*는 mobile, embedding, IoT device에서 모델을 사용할 수 있도록 Tensorflow(keras) 모델을 변환해주는 tool입니다.


### Tensorflow Lite convert

본격적으로 Keras model을 TFLite model을 변경하는 tool인 <span style="color:#C70039">**Tensorflow Lite convert**</span> 소개할게요.

- <span style="color:DodgerBlue">**tf.lite.TFLiteConverter.from_keras_model('your keras model')**</span>

위의 함수가 TFLite로 만드는 준비 compile과정이라고 생각하시면 될거예요.  
뿐만 아니라 위 함수의 중요한 attribute가 있어요.

- Attributes
	- **representative_dataset** : int8 quantization할 때 필요한 데이터셋이라고 일단 생각하세요
	- **target_spec**: [float32, float16, int8] 중에 어떤 data type(bit)로 표현되는 모델을 출력할지 정함
		- Default: float16
	- **inference_input_type**: input layer의 data type
	- **inference_output_type**: output layer의 data type

지금은 이러한 Attribute가 있다고 대충 알고 넘어가시죠... 뒤에서 자세히 할게요!

TFLiteConverter의 입출력 관계는 다음 그림과 같습니다.

![2](https://da2so.github.io/assets/post_img/2020-12-23-Master_TFlite/1.png){: .mx-auto.d-block width="60%" :}

(저의 경우) Keras model을 입력으로 받고 TFLiteConverter를 통과해 tflite확장자를 가진 Flatbuffer형식의 파일을 얻게 됩니다.
Flatbuffer형식이므로 언어에 depency에 영향을 받지 않는 모델이 되는 것이고 이는 다양한 device, mobile에 deploy할 수 있음을 암시하겠죠!?

그럼 위 함수를 이용해서 train해놓은 keras model로 실험해보죠.  
(예제 모델:pruned resnet18 on cifar10 )


```python
def keras2TFlite(model_path):
    #load a pre-trained model
    keras_model =tf.keras.models.load_model(model_path) #model_path is 'cifar10_resnet18_pruned.h5'

    #convert to tflite model
    converter = tf.lite.TFLiteConverter.from_keras_model(keras_model)
    tflite_model = converter.convert()

    #save tflite model
    ext_idx=model_path.rfind('.')
    save_path=model_path[:ext_idx]+'.tflite'
    with open(save_path, "wb") as f:
        f.write(tflite_model)
```


조금 설명드리면 예제 모델은 제가 resnet18모델을 pruning시킨 모델이에요. Test acc는 85%정도인데 parameter는 8만개만 쓰였습니다!  
어떻게 pruning시킨지 알고싶으면 여기로 [HERE](https://github.com/da2so/Eagleeye_Tensorflow)

|Model|File size|Download|
|-----|--------|---------|
|pruned_resnet18|507KB|[pruned.h5](https://drive.google.com/file/d/15fmEkZYk0bvi_9YbsBw5jZELuzoz7gym/view?usp=sharing)|
|tflite_resnet18|329KB|[tflite.h5](https://drive.google.com/file/d/1IpjGsOwqaqBg3S7RqSxVR3aN0qOF_AMS/view?usp=sharing)|

왜 TFLite model로 변환했는데 model size가 줄었는 지 위의 attribute를 잘 보셨다면 이해가실겁니다.  
**target_spec** attribute가 default로 float16으로 되있기 때문입니다. 원래 keras model의 weight들은 모두 float32표현되어 있었지만 tflite로 되면서 float16으로 표현된 것이죠.
이렇게 간단하게 모델의 크기를 줄엿네요;;

이번 장의 마지막으로 keras model와 tflite model을 netron을 이용해서 visualization한 모습을 비교해 드릴게요.

![2](https://da2so.github.io/assets/post_img/2020-12-23-Master_TFlite/2.png){: .mx-auto.d-block width="60%" :}


다음 장에서는 TFLite 모델로 inference하는 방식과 여유가 된다면 mobile로 작동하는 것을 보여드리도록 하는 게 목표입니다.  
BYE!