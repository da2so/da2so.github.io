---
layout: post
title: TFLite 뽀개기 (1) - TFLite 기본 이해 및 사용하기
tags: [TFLite, Mobile, Tensorflow]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-12-23-Master_TFlite/post.png
---

## 1. TFLite 란?  
내용은 python, Tensorflow-gpu 2.x, keras model, mobile 에 한정되어 있음을 알려드립니다.  
{: .box-note}

*Tensorflow Lite (TFLite)*는 mobile, embedding, IoT device에서 모델을 사용할 수 있도록 Tensorflow(keras) 모델을 변환해주는 tool입니다.


### 1.1 Tensorflow Lite convert

본격적으로 Keras model을 TFLite model을 변경하는 tool인 <span style="color:#C70039">**Tensorflow Lite convert**</span> 소개할게요.

- <span style="color:DodgerBlue">**converter=tf.lite.TFLiteConverter.from_keras_model('your keras model')**</span>

위의 함수가 TFLite로 만드는 준비 compile과정이라고 생각하시면 될거예요. 그래서 compile setting을 converter변수에 할당해 놓는 거죠.
뿐만 아니라 위 함수의 중요한 attribute가 있어요.

- Attributes
	- **representative_dataset** : int8 quantization할 때 필요한 데이터셋이라고 일단 생각하세요
	- **target_spec**: [float32, float16, int8] 중에 어떤 data type(bit)로 표현되는 모델을 출력할지 정함
	- **inference_input_type**: input layer의 data type
	- **inference_output_type**: output layer의 data type

지금은 이러한 Attribute가 있다고 알고만 넘어가시죠... 나중에 자세히 설명드릴게여!  
그리고 compile이 진행 되었다면 실제로 convert는 다음과 같은 함수로 진행됩니다.

-  <span style="color:DodgerBlue">**tflite_model=converter.convert()**</span>

compile정보를 담고 있는 converter object에서 ```convert()``` 함수를 호출하여 ```tflite_model```변수에 tflite model을 할당하게 됩니다.

TFLiteConverter의 입출력 관계는 다음 그림과 같습니다.

![relation_input_output](https://da2so.github.io/assets/post_img/2020-12-23-Master_TFlite/1.png){: .mx-auto.d-block width="60%" :}

(저의 경우) Keras model을 입력으로 받고 TFLiteConverter를 통과해 tflite확장자를 가진 Flatbuffer형식의 파일을 얻게 됩니다.
Serialized인 Flatbuffer형식이므로 언어(i.e. C, Java, Python)에 dependency 영향을 받지 않는 모델이 되는 것이고 이는 다양한 device, mobile에 deploy할 수 있음을 암시하겠죠!?

그럼 이제 train해놓은 keras model을 이용해서 예제 코드를 실행해보죠. 
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


예제 모델은 제가 resnet18모델을 pruning시킨 거에요. Test acc는 85%정도인데 parameter는 8만개밖에 안쓰였습니다!  
Pruning은 [EagleEye code](https://github.com/da2so/Eagleeye_Tensorflow)를 사용했습니다.

|Model|File size|
|-----|--------|
|pruned_resnet18|507KB|
|float32_resnet18|329KB|

TFLite model로 변환했는데 model size가 줄었네요! 역시 TFLite model이 더 최적화되어 있는 듯하네요.
그리고 keras model와 tflite model을 netron을 이용해서 visualization한 모습을 비교해 드릴게요.

![2](https://da2so.github.io/assets/post_img/2020-12-23-Master_TFlite/2.png){: .mx-auto.d-block width="60%" :}


### 1.2 Operator compatibility

Tensorflow 에서는 지원하지만 Tensorflow Lite에서는 지원하지 않는 operator가 있습니다.  

**type측면으로 보면** 대부분의 TFLite의 operator들은 float32, uint8, int8 을 대상으로 한다고 합니다. 그래서 float16또는 string위한 많은 operator들은 아직이라네요..ㅠ  


오늘 보여드린 예제 코드와 실험 모델들은[Conquer_TFLite](https://github.com/da2so/Conquer_TFLite/)에서 사용가능합니다.
다음 장에서는 TFLite 모델로 inference하는 방식과 여유가 된다면 mobile로 작동하는 것을 보여드리도록 하는 게 목표입니다.  
**BYE!**

