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

- tf.lite.TFLiteConverter.from_keras.model()

위 함수를 이용해서 train해놓은 model로 실험해보죠.  
(모델 정보: pruned resnet18 on cifar10)

