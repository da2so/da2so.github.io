---
layout: post
title: Zero-Shot Knowledge Distillation in Deep Networks
tags: [Model Compression, Knowledge Distillation, Data-free]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-08-11-Zero-Shot_Knowledge_Distillation_in_Deep_Networks/post.png
---

## 1. What is data-free knowledge distillation??

{: .box-note}
**Knowledge distillation:** Dealing with the problem of training a smaller model (Student) from a high capacity source model (Teacher) so as to retain most of its performance.

As the word itself, We perform knowledge distillation when there are no original dataset on which the Teacher network has been trained. It is because, in real world, most datasets are proprietary and not shared publicly due to privacy or confidentiality concerns. 

In order to perform data-free knowledge distillation, it is a necessary to reconstruct a dataset for training Student network. Then, in this paper, we propose "Zero-Shot Knowledge Distillation" (ZSKD), which perform pseudo data synthesis from the Teacher model that act as the transfer set to perform the distillation without even using any meta-data.

![2](https://da2so.github.io/assets/post_img/2020-08-11-Zero-Shot_Knowledge_Distillation_in_Deep_Networks/1.png){: .mx-auto.d-block :}


## 2. Method

### 2.1 Knowledge Distillation

Transferring the generalization ability of a large, complex _Teacher_ (<span style="color:DodgerBlue">$T$</span>) deep neural network to a less complex _Student_ (<span style="color:DodgerBlue">$S$</span>) network can be achieved using the class probabilities produced by a _Teacher_ as "soft targets" for training the _Student_.


Let <span style="color:DodgerBlue">$T$</span> be the _Teacher_ network with learned parameters <span style="color:DodgerBlue">$\theta_T$</span> and <span style="color:DodgerBlue">$S$</span> be the _Student_ with parameters <span style="color:DodgerBlue">$\theta_S$</span>, note that in general <span style="color:DodgerBlue">$\vert \theta_S \vert \ll \vert \theta_T \vert$</span>.


Knowledge distillation methods train the _Student_ by minimizing the following objective (<span style="color:DodgerBlue">$L$</span>).

<span style="color:DodgerBlue">\\[
L=\sum_{(x,y) \in \mathbb{D}} L_K (S(x,\theta_S,\tau), T(x,\theta_T, \tau))+\lambda L_C( \widehat{y}_S,y)
\\] </span>

,where <span style="color:DodgerBlue">$D$</span> is training dataset, <span style="color:DodgerBlue">$L_C$</span> is the cross-entropy loss computed on the labels <span style="color:DodgerBlue">$\widehat{y}_S$</span> predicited by the _Student_ and ground truth <span style="color:DodgerBlue">$y$</span>. <span style="color:DodgerBlue">. $L_K$</span> is the distillation loss (e.g. cross-entropy or MSE), <span style="color:DodgerBlue">$ (T(x,\theta_T,\tau)$</span> indicates the softmax output of the _Teacher_ and <span style="color:DodgerBlue">$S(x,\theta_S, \tau)$</span> denotes the softmax output of the _Student_. Note that, unless it is mentioned, we use a temperature (<span style="color:DodgerBlue">$\tau$</span>) of 1.

