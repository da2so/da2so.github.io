---
layout: post
title: Zero-Shot Knowledge Transfer via Aversarial Belif Matching
tags: [Model Compression, Knowledge Distillation, Data-free, GAN]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-08-14-Zero-Shot_Knowledge_Transfer_via_Adversarial_Belif_Matching/post.png
---

## 1. Data-free Knowledge distillation


{: .box-note}
**Knowledge distillation:** Dealing with the problem of training a smaller model (Student) from a high capacity source model (Teacher) so as to retain most of its performance.

As the word itself, We perform knowledge distillation when there are no original dataset on which the Teacher network has been trained. It is because, in real world, most datasets are proprietary and not shared publicly due to privacy or confidentiality concerns. 


In order to perform data-free knowledge distillation, it is a necessary to reconstruct a dataset for training Student network. Thus, in this paper, we train an adversarial generator to search for iamges on which the student poorly mathces the teacher, and then using them to train the student.

![1](https://da2so.github.io/assets/post_img/2020-08-14-Zero-Shot_Knowledge_Transfer_via_Adversarial_Belif_Matching/1.png){: .mx-auto.d-block :}

## 2. Zero-shot knowledge transfer Method

