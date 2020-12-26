---
layout: post
title: TFLite 뽀개기 (2) - TFLite Inference
tags: [TFLite, Mobile, Tensorflow]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-12-24-Master_TFlite2/post.png
---

## 1. TFLite Inference  
내용은 python, Tensorflow-gpu 2.x, keras model, mobile 에 한정되어 있음을 알려드립니다.  
{: .box-note}

전 장에서 얻은 TFLite model로 이제 Inference을 해볼려고 합니다.  

그 전에, Inference를 어디서 할 수 있는 지 알아야겠죠?

- **Android**
    - Java 또는 C++ API를 사용하여 Inference
- **iOS**
    - Swift 또는 Objective-C를 사용하여 Inference
- **Linux**
    - C++ 또는 Python으로 제공되는 TFLite API를 사용하여 Inference

이번 글에서는 **Linux**부터 해볼게요..  
다시 돌아와서 Inference, 즉 prediction를 위해서는 TFLite API인 <span style="color:#C70039">**Interpreter**</span>을 추죽으로 다음과 같은 단계를 진행합니다.

1. Loading a model 
    - .tflite 확장자를 가진 모델을 메모리에 올림
2. Transforming data
    - tflite 모델과 호환되도록 입력 데이터 형식을 변경
3. Running inference
    - tflite api를 사용하여 모델 inference를 실행
4. Interpreting output
    - model의 output을 device에 맞게 의미있게 바꿈


예제를 통해 실행 및 분석해봐요.
(모델은 [전의 글](https://da2so.github.io/2020-12-23-Master_TFlite/)에서 생성한 tflite_resnet18을 사용)













### 1.1 Tensorflow Lite convert

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

그리고 keras model와 tflite model을 netron을 이용해서 visualization한 모습을 비교해 드릴게요.

![2](https://da2so.github.io/assets/post_img/2020-12-23-Master_TFlite/2.png){: .mx-auto.d-block width="60%" :}


### 1.2 Operator compatibility

Tensorflow 에서는 지원하지만 Tensorflow Lite에서는 지원하지 않는 operator가 있습니다.  

**type측면으로 보면** 대부분의 TFLite의 operator들은 float32, uint8, int8 을 대상으로 한다고 합니다. 그래서 float16또는 string위한 많은 operator들은 아직이라네요..ㅠ  
**document상으로 보면** 해당 [site](https://www.tensorflow.org/mlir/tfl_ops)에서 지원하는 operator를 보실수 있습니다.


다음 장에서는 TFLite 모델로 inference하는 방식과 여유가 된다면 mobile로 작동하는 것을 보여드리도록 하는 게 목표입니다.  
BYE!


## <span style="color:#C70039 "> Reference </span>

[TFLite document](https://www.tensorflow.org/lite/guide)