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

1. 몇 bit로 줄일 거냐?
    - 기본적으로 TFLite에서 제공하는 타입 및 bit수는 float16(16bit), int8(8bit), uint8(8bit).
2. 어떻게 줄일 거냐?
    - float32로 표현된 parameters들을 줄이고자 하는 bit에 맞춰 mapping시킴.

2번에 대해 좀더 설명하자면 아래 그림을 참고하십시오.  
해당 그림은 uint8로 Quantization하는 과정을 보여줍니다.

![2](https://da2so.github.io/assets/post_img/2020-12-27-Master_TFlite3/1.png){: .mx-auto.d-block width="90%" :}

그래서 TFLite에서는 다음과 같은 Quantization 방법론을 제공합니다.

![2](https://da2so.github.io/assets/post_img/2020-12-27-Master_TFlite3/2.png){: .mx-auto.d-block width="70%" :}

총 4가지의 Technique를 TFLite에서 제공합니다. 크게 보면 **Post-Training Quantization**과 **Quantization Aware Training**두 개로 나뉘는 것을 볼 수 있습니다.  
그럼 먼저 **Post-Training Quantization** 알아봐요.

## 2. Post-Training Quantization (PTQ)

PTQ는 해석 그대로 training 후에 Quantization하겠다는 말입니다. 그래서 already-trained된 float TF model을 대상으로 Quantization하게됩니다.  
TF Lite에서는 [이전 글](https://da2so.github.io/2020-12-23-Master_TFlite/)에서 설명드린 Tensorflow Lite Converter을 이용하여 PTQ를 합니다.

PTQ방법론에서는 위의 표와 같이 3가지의 option과 그에 대한 장점이 있어요.


하나하나 example code도 돌려보며 자세히 알아봐요.

### 2.1 Dynamic range quantization






## <span style="color:#C70039 "> Reference </span>

[TFLite document](https://www.tensorflow.org/lite/guide)