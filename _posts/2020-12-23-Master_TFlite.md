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

- tf.lite.TFLiteConverter.from_keras_model('your keras model')

위의 함수가 TFLite로 만드는 준비 compile과정이라고 생각하시면 될거예요. 뿐만 아니라 위 함수의 중요한 attribute가 있어요.

- Attributes
	- representative_dataset: int8 quantization할 때 필요한 데이터셋이라고 일단 생각하세요
	- target_spec: [float32, float16, int8] 중에 어떤 data type(bit)로 표현되는 모델을 출력할지 정함
	- inference_input_type: input layer의 data type
	- inference_output_type output layer의 data type

위 함수를 이용해서 train해놓은 model로 실험해보죠.  
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

|Model|File size ||
|pruned
