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

이전 글로부터 만들어진 TFLite model로 이제 Inference을 해볼려고 합니다.  

그 전에, Inference를 어디서 할 수 있는 지 알아야겠죠?

- **Android**
    - Java 또는 C++ API를 사용하여 Inference
- **iOS**
    - Swift 또는 Objective-C를 사용하여 Inference
- **Linux**
    - C++ 또는 Python으로 제공되는 TFLite API를 사용하여 Inference

이번 글에서는 **Linux**부터 해볼게요..  
다시 돌아와서 Inference, 즉 prediction를 위해서는 TFLite API인 <span style="color:#C70039">**Interpreter**</span>을 주축으로 다음과 같은 단계를 진행합니다.

1. **Loading a model**
    - .tflite 확장자를 가진 모델을 메모리에 올림
2. **Transforming data**
    - tflite 모델과 호환되도록 입력 데이터 형식을 변경
3. **Running inference**
    - tflite api를 사용하여 모델 inference를 실행
4. **Interpreting output**
    - model의 output을 device에 맞게 의미있게 변경


예제를 통해 실행 및 분석해봐요.
(모델은 [전의 글](https://da2so.github.io/2020-12-23-Master_TFlite/)에서 생성한 tflite_resnet18을 사용)

```python
def TFLiteInference(model_path,x_test,y_test):

    #Step 1. Load TFLite model and allocate tensors.
    interpreter = tf.lite.Interpreter(model_path=model_path)
    interpreter.allocate_tensors()
    
    # Get indexes of input and output layers
    input_index = interpreter.get_input_details()[0]['index']
    output_index = interpreter.get_output_details()[0]['index']

    sum_correct=0.0
    sum_time=0.0
    for idx, data in enumerate(zip(x_test,y_test)):
        image=data[0]
        label=data[1]
        image=tf.expand_dims(image, axis=0) #shape will be [1,32,32,3]
        
        s_time=time.time()
        #Step 2. Transform input data
        interpreter.set_tensor(input_index,image)
        #Step 3. Run inference
        interpreter.invoke()
        #Step 4. Interpret ouput
        pred=interpreter.get_tensor(output_index)
        
        sum_time+=time.time()-s_time
        if np.argmax(pred)== np.argmax(label):
            sum_correct+=1.0
    
    mean_acc=sum_correct / float(idx+1)
    mean_time=sum_time / float(idx+1)

    print(f'Accuracy of TFLite model: {mean_acc}')
    print(f'Inference time of TFLite model: {mean_time}')

```

code와 Inference단계를 매칭시켜보면서 설명드릴게요.

1. **Loading a model**
    -  ```tf.lite.Interpreter()```: TFLite model을 memory에 올림.
    -  ```interpreter.allocate_tensors()```: tensor initialization

2. **Transforming data**
    - ```interpreter.get_input_details()```: input에 대한 다양한 정보 호출.
        - code내의 *index* element는 input layer에 대한 순서(index)를 GET함.
        - 뿐만 아니라, input의 shape, data type등 다양한 정보를 호출 가능.
    - ```interpreter.set_tensor(input_index,image)```: 









