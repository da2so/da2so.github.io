---
layout: post
title: TFLite 뽀개기 (1)
tags: [Style Transfer]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-12-23-Master_TFlite/post.png
---

## TFLite 란?

{: .box-note}
**Setting:** 제가 다룰 내용은 python (api), Tensorflow-gpu 2.x, keras model, mobile 에 한정되 있음을 알려드립니다. :hand_over_mouth:

*Tensorflow Lite (TFLite)*는 mobile, embedding, IoT device에서 모델을 사용할 수 있도록 Tensorflow(keras) 모델을 변환해주는 tool입니다.
뿐만 아니라 모델의 크기와 성능을 최적화하는 도구를 제공하기도 합니다. (이건 다음에 할게요..ㅎ)


## keras model to TFLite model

본격적으로 Keras model을 TFLite model을 변경하는 tool인 <span style="color:#C70039">>Tensorflow Lite convert</span> 소개할게요.

### Tensorflow Lite convert

Tensorflow Lite convert는 Tensorflow model을 입력으로 받아 Tensorflow Lite model을 출력합니다.
저는 keras model을 입력으로 주기 때문에 다음과 같은 함수를 쓰게 됩니다. 

- `(tf.lite.TFLiteConverter.from_keras.model())