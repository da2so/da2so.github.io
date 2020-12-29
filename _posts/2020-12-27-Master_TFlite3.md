---
layout: post
title: TFLite 뽀개기 (3) - Quantization
tags: [TFLite, Mobile, Tensorflow]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-12-27-Master_TFlite3/post.png
---

## 1. Quantization  
내용은 python, Tensorflow-gpu 2.x, keras model, mobile 에 한정되어 있음을 알려드립니다.  
{: .box-note}

**[이전 글](https://da2so.github.io/2020-12-24-Master_TFlite2/)**에서 TFLite model로 Inference까지 해봤습니다. 이번에는 TFLite model을 경량화 시키는 방법을 알려드릴게요. 

경량화 방법은 TFLite에서 제공하는 Quantization이며 경량화의 효과는 다음과 같습니다.

- Smaller storage size
    - 모델 사이즈를 줄여 user의 device에 적은 storage을 occupy함.
- Less memory usage
    - 작은 모델로 만들어 RAM memory를 적게 occupy함.
- Reducing latency
    - 좀 더 빠르게 inference할 수 있게 함.

하지만 마냥 장점만 가져갈 수는 없죠. 단점으로는

- Lower performance
    - model accuracy가 하락함.

이제 <span style="color:#C70039">**Quantization**</span>에 대해 알아 봅시다.  
기존 keras model의 parameters(i.e. weights, bias, ...)들은 각각이 **float32**로 표현되어 있습니다. float32은 
32bit의 memory를 가지니 Quantization에서는 32bit보다 낮은 bit로 표현시켜 경량화 시킵니다. 

그럼, 다음과 같은 2가지 궁금증이 생기실 겁니다. 질문과 함께 답변드려볼게요.

1. **몇 bit로 줄일 거냐?**
    - 기본적으로 TFLite에서 제공하는 타입 및 bit수는 float16(16bit), int8(8bit), uint8(8bit).
2. **어떻게 줄일 거냐?**
    - float32로 표현된 parameters들을 줄이고자 하는 bit에 맞춰 mapping시킴.

2번에 대해 좀더 설명하자면 아래 그림을 참고하십시오.  
해당 그림은 uint8로 Quantization하는 과정을 보여줍니다.

![2](https://da2so.github.io/assets/post_img/2020-12-27-Master_TFlite3/1.png){: .mx-auto.d-block width="80%" :}

그래서 TFLite에서는 다음과 같은 Quantization 방법론을 제공합니다.

![2](https://da2so.github.io/assets/post_img/2020-12-27-Master_TFlite3/2.png){: .mx-auto.d-block width="80%" :}

총 4가지의 Technique를 TFLite에서 제공합니다. 크게 보면 **Post-Training Quantization**과 **Quantization Aware Training**두 개로 나뉘는 것을 볼 수 있습니다.  
그럼 먼저 **Post-Training Quantization** 알아봐요.

## 2. Post-Training Quantization (PTQ)

PTQ는 해석 그대로 training 후에 Quantization하겠다는 말입니다. 그래서 already-trained된 float TF model을 대상으로 Quantization하게됩니다.  
TF Lite에서는 **[이전 글](https://da2so.github.io/2020-12-23-Master_TFlite/)**에서 설명드린 Tensorflow Lite Converter을 이용하여 PTQ를 합니다.

PTQ방법론에서는 위의 표와 같이 3가지의 option과 그에 대한 장점이 있어요.

![2](https://da2so.github.io/assets/post_img/2020-12-27-Master_TFlite3/3.png){: .mx-auto.d-block width="80%" :}


하나하나 example code도 돌려보며 자세히 알아봐요.

### 2.1 Dynamic range quantization

해당 technique은 **quantized kernels**을 이용해서 model의 weights들만 float32에서 int8로 바꾸게 됩니다. 그리고, inference시에만 **floating-point kernel**를 이용해여 weights를 int8을 float32로 convert됩니다.
Activation은 항상 floating point로 저장되어져 있습니다. 그래서 quantized kernels processing을 지원하는 operator의 경우에는 activation을 processing전에 **dynamic**하게 8bit로 quantized하고 processing후에 다시 dequantization하게 됩니다.

정리하자면, weights들은 training 후에 quantized되는 것이고 activations은 inference time에 dynamic하게 quantized되는 것입니다.

Example code는 다음과 같습니다.

```python
def keras2TFlite(model_path):
    #load a pre-trained model
    keras_model =tf.keras.models.load_model(model_path) #model_path is 'cifar10_resnet18_pruned.h5'

    #convert to tflite model
    converter = tf.lite.TFLiteConverter.from_keras_model(keras_model)
    converter.optimizations = [tf.lite.Optimize.DEFAULT] #dynamic range PTQ
    tflite_model = converter.convert()

    #save tflite model
    ext_idx=model_path.rfind('.')
    save_path=model_path[:ext_idx]+'_dynamic.tflite'
    with open(save_path, "wb") as f:
        f.write(tflite_model)

```

TFLite의 Converter를 통해서 quantization을 진행하게 되는데 ```converter.optimizations = [tf.lite.Optimize.DEFAULT]``` 를 추가하면 dynamic range quantization을 하도록 한뒤 TFLite model을 출력하게 됩니다.

이제 결과를 **[이전 글](https://da2so.github.io/2020-12-24-Master_TFlite2/)**에서 진행했던 keras model과 float32로 operation되는 tflite와 비교해 보죠.

|Model|Test Acc|Inference Time(seconds)|File size|Download|
|-----|--------|-----------------------|---------|--------|
|pruned_resnet18|85.65%|0.0133s [GPU]|507KB|[pruned.h5](https://drive.google.com/file/d/15fmEkZYk0bvi_9YbsBw5jZELuzoz7gym/view?usp=sharing)|
|float32_resnet18|85.65%|0.0023s [CPU]|329KB|[float32.tflite](https://drive.google.com/file/d/1IpjGsOwqaqBg3S7RqSxVR3aN0qOF_AMS/view?usp=sharing)|
|dynamic_tflite_resnet18|85.48%|0.0033s [CPU]|107KB|[dynamic.tflite](https://drive.google.com/file/d/1msiOxUmI7OfwOVSajP-ID17h_NuzhuqN/view?usp=sharing)|

TFLite파일을 기준으로 dynamic range quantization은 weights들을 모두 float32에서 int8로 줄이므로 File size는 1/4 (329KB-> 107KB)정도 줄어드는 정상입니다.
하지만, 위에서 설명드린 activation연산을 위한 내부 kernels을 쓰므로 Inference time은 늘어나는 것이라 추축하고 있습니다. (저의 지극한 개인 의견)

그리고 netron으로 network를 visualization했을 때 재밌는 발견이 있네요.

![2](https://da2so.github.io/assets/post_img/2020-12-27-Master_TFlite3/4.png){: .mx-auto.d-block width="90%" :}

어떤 conv layer는 float32로 표현되지만 어떤 conv layer의 weights는 int8로 표현되네요... (머지?)

Dynamic PTQ의 Example code는 [여기서](https://github.com/da2so/Conquer_TFLite/blob/main/3_dynamicPTQ.py) 사용가능 합니다.


### 2.2 Post-training integer quantization

위의 dynamic range quantization과 다르게 이 방법론은 static하게 activation까지 int8로 표현합니다. 하지만, activation까지 quantization하려면 위에서 설명드린 quantization과정과 같이 rmin/rmax를 구해야합니다. rmin/rmax를 구하려면 activation values가 어느 range에 속해있는 지 알아야하고 이는 모델을 통과시킬 data를 필요로 하게 됩니다. 그래서 integer quantization은 **representative dataset**이 필요합니다.

Example code로 자세히 알아보죠. 먼저 Converter 부분입니다.

```python
def keras2TFlite(model_path,x_train):
    #load a pre-trained model
    keras_model =tf.keras.models.load_model(model_path) #model_path is 'cifar10_resnet18_pruned.h5'

    #define a representative_dataset
    def representative_data_gen():
        for image in tf.data.Dataset.from_tensor_slices(x_train).batch(1).take(100):
            yield [image]

    
    #convert to tflite model
    converter = tf.lite.TFLiteConverter.from_keras_model(keras_model)
    
    converter.optimizations = [tf.lite.Optimize.DEFAULT]
    converter.representative_dataset = representative_data_gen
    converter.target_spec.supported_ops = [tf.lite.OpsSet.TFLITE_BUILTINS_INT8]
    converter.inference_input_type = tf.int8
    converter.inference_output_type = tf.int8
    
    tflite_model = converter.convert()

    #save tflite model
    ext_idx=model_path.rfind('.')
    save_path=model_path[:ext_idx]+'_int8.tflite'
    with open(save_path, "wb") as f:
        f.write(tflite_model)

```

Representative dataset을 function으로 정의한뒤 converter의 attribute인 ```representative_dataset```에 변수할당을 하였으며, **converter.target_spec.supported_ops**을 통해 support operator를 int8로 지정하였습니다. 마지막으로는 input과 output의 data type을 uint8 로 할당하였습니다.  
하지만 original float model처럼 input, output을 float32로 받고 출력하기 위해서는 다음의 code를 제외시키면 됩니다.

```python
    converter.target_spec.supported_ops = [tf.lite.OpsSet.TFLITE_BUILTINS_INT8] #remove
    converter.inference_input_type = tf.uint8 #remove
    converter.inference_output_type = tf.uint8 #remove
```

Input,output이 int8인 경우를 <span style="color:#C70039">**int8_all.tflite**</span>라고 지칭하고 float32인 경우를 <span style="color:#C70039">**int8_notall.tflite**</span>로 명칭하고 다음과 같은 inference를 진행하였습니다.


```python
def TFLiteInference(model_path,x_test,y_test):

    #Step 1. Load TFLite model and allocate tensors.
    interpreter=tf.lite.Interpreter(model_path=model_path)
    interpreter.allocate_tensors()
    
    input_details=interpreter.get_input_details()[0]
    output_details=interpreter.get_output_details()[0]
    
    # Get indexes of input and output layers
    input_index= input_details['index']
    output_index= output_details['index']

    sum_correct=0.0
    sum_time=0.0
    for idx, data in enumerate(zip(x_test,y_test)):
        image=data[0]
        label=data[1]

        # set for tranforming input data
        if input_details['dtype'] == np.uint8:
            input_scale, input_zero_point = input_details["quantization"]
            image = image / input_scale + input_zero_point
        image=np.expand_dims(image, axis=0).astype(input_details['dtype'])
        
        s_time=time.time()
        #Step 2. Transform input data
        interpreter.set_tensor(input_index,image)
        #Step 3. Run inference
        interpreter.invoke()
        #Step 4. Interpret output
        pred=interpreter.get_tensor(output_index)
        
        sum_time+=time.time()-s_time
        if np.argmax(pred)== np.argmax(label):
            sum_correct+=1.0
    
    mean_acc=sum_correct / float(idx+1)
    mean_time=sum_time / float(idx+1)

    print(f'Accuracy of TFLite model: {mean_acc}')
    print(f'Inference time of TFLite model: {mean_time}')
```

Int8_all.tflite인 경우에는 ```input_details['dtype'] == np.uint8```이므로 input data를 int8로 바꾸는 작업을 하게됩니다.  
위의 결과표에 더하여 int8_all.tflite 와 int8_notall.tflite를 추가한 결과를 보여드립니다.

|Model|Test Acc|Inference Time(seconds)|File size|Download|
|-----|--------|-----------------------|---------|--------|
|pruned_resnet18|85.65%|0.0133s [GPU]|507KB|[pruned.h5](https://drive.google.com/file/d/15fmEkZYk0bvi_9YbsBw5jZELuzoz7gym/view?usp=sharing)|
|float32_resnet18|85.65%|0.0023s [CPU]|329KB|[float32.tflite](https://drive.google.com/file/d/1IpjGsOwqaqBg3S7RqSxVR3aN0qOF_AMS/view?usp=sharing)|
|dynamic_tflite_resnet18|85.48%|0.0033s [CPU]|107KB|[dynamic.tflite](https://drive.google.com/file/d/1msiOxUmI7OfwOVSajP-ID17h_NuzhuqN/view?usp=sharing)|
|int8_all_resnet18|85.65%|0.0323s [CPU]|115KB|[int8_all.tflite](https://drive.google.com/file/d/1H7Lwg4Rbna4hX9025-9_jW7nmppXFpfu/view?usp=sharing)|
|int8_notall_resnet18|85.59%|0.0323s [CPU]|115KB|[int8_notall.tflite](https://drive.google.com/file/d/1tglks42aur_4y4q8PPv8Z7h4Ec81Y8mp/view?usp=sharing)|


Int8로 quantization하고 linux서버 환경에서 run하게되면 file size는 4배정도 줄지만 inference time이 엄청 많이 높아지네요...ㅠ 
그리고 int8_all.tflite과 int8_notall.tflite을 netron으로 visualization하면 다음과 같습니다.

![2](https://da2so.github.io/assets/post_img/2020-12-27-Master_TFlite3/5.png){: .mx-auto.d-block width="90%" :}

### 2.3 Post-training float16 quantization

Weights들을 float16을 quantization하는 방법론이다. Tensorflow Lite GPU deletegate가 기존보다 2배 빠르게 진행될 수 있도록 float16 operation을 한다.  
하지만, 추가적인 modification이 없으며 CPU에서 run될 경우, float16 weights들은 infernece되기 전에 float32으로 upsampling되어 계산된다고 한다.


```python
def keras2TFlite(model_path):
    #load a pre-trained model
    keras_model =tf.keras.models.load_model(model_path) #model_path is 'cifar10_resnet18_pruned.h5'

    #convert to tflite model
    converter = tf.lite.TFLiteConverter.from_keras_model(keras_model)
    converter.optimizations = [tf.lite.Optimize.DEFAULT] 
    converter.target_spec.supported_types = [tf.float16] # float16 PTQ

    tflite_model = converter.convert()

    #save tflite model
    ext_idx=model_path.rfind('.')
    save_path=model_path[:ext_idx]+'_float16.tflite'
    with open(save_path, "wb") as f:
        f.write(tflite_model)
```

dynamic range quantization과 다른점은 ```converter.target_spec.supported_types = [tf.float16]```이 추가된것 이외에는 없습니다. 


|Model|Test Acc|Inference Time(seconds)|File size|Download|
|-----|--------|-----------------------|---------|--------|
|pruned_resnet18|85.65%|0.0133s [GPU]|507KB|[pruned.h5](https://drive.google.com/file/d/15fmEkZYk0bvi_9YbsBw5jZELuzoz7gym/view?usp=sharing)|
|float32_resnet18|85.65%|0.0023s [CPU]|329KB|[float32.tflite](https://drive.google.com/file/d/1IpjGsOwqaqBg3S7RqSxVR3aN0qOF_AMS/view?usp=sharing)|
|dynamic_tflite_resnet18|85.48%|0.0033s [CPU]|107KB|[dynamic.tflite](https://drive.google.com/file/d/1msiOxUmI7OfwOVSajP-ID17h_NuzhuqN/view?usp=sharing)|
|int8_all_resnet18|85.65%|0.0323s [CPU]|115KB|[int8_all.tflite](https://drive.google.com/file/d/1H7Lwg4Rbna4hX9025-9_jW7nmppXFpfu/view?usp=sharing)|
|int8_notall_resnet18|85.59%|0.0323s [CPU]|115KB|[int8_notall.tflite](https://drive.google.com/file/d/1tglks42aur_4y4q8PPv8Z7h4Ec81Y8mp/view?usp=sharing)|
|float16_resnet18|85.64%|0.0022 [CPU]|181KB|[float16.tflite](https://drive.google.com/file/d/1s_o57wM7Yl33Gn1regO58MKYCtg3kf0W/view?usp=sharing)|

예상과 같이 float32_resnet18보다 float16_resnet18이 2배정도 File size는 줄었네요. Inference time은 비슷하고요. 

float16_resnet18의 visualization은 다음과 같습니다.

![2](https://da2so.github.io/assets/post_img/2020-12-27-Master_TFlite3/5.png){: .mx-auto.d-block width="80%" :}

## <span style="color:#C70039 "> Reference </span>

[TFLite document](https://www.tensorflow.org/lite/guide)