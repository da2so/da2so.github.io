---
layout: post
title: Interpretable And Fine-grained Visual Explanations For Convolutional Neural Networks
tags: [Explainable AI(XAI), Perturbation, Interpretable, Fine-grained]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-08-17-Interpretable_And_Fine-grained_Visual_Explanations_For_Convolutional_Neural_Networks/post.png
---

## 1. Goal

We propose an optimization based visual explanation method, which highlights the evidence in the input images for a specific prediction.


### <span style="color:gray">1.1 Sub-goal </span>

<span style="color:gray">*A.*</span> Defend against adversarial evidence (i.e. faulty evidence due to artifacts).
<span style="color:gray">*B.*</span> Provide the explanations which are both fine-grained and preserve the characteristics of images, such as edges and colors.

![1](https://da2so.github.io/assets/post_img/2020-08-17-Interpretable_And_Fine-grained_Visual_Explanations_For_Convolutional_Neural_Networks/1.png){: .mx-auto.d-block :}


## 2. Method

We provide *local explanations*, which focus on an individual input.

&nbsp;* Given one data point, our method highlights the evidence on which a model bases its deciion.


### <span style="color:gray">2.1 Perturbation based visual explanations </span>

Perturbatuion based explanations can be defined as:
 <span style="color:#5256BC">Explanation by preservation: </span> The smallest region of the image which must be retained to preserve the original model output. 

 <span style="color:#5256BC">Explanation by deletion: </span> The smallest region of the image which must be deleted to change the model output.


