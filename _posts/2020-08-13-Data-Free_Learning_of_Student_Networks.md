---
layout: post
title: Data-Free Learning of Student Networks
tags: [Model Compression, Knowledge Distillation, Data-free, GAN]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-08-13-Data-Free_Learning_of_Student_Networks/post.png
---

## 1. Data-free Knowledge distillation


{: .box-note}
**Knowledge distillation:** Dealing with the problem of training a smaller model (Student) from a high capacity source model (Teacher) so as to retain most of its performance.

As the word itself, We perform knowledge distillation when there are no original dataset on which the Teacher network has been trained. It is because, in real world, most datasets are proprietary and not shared publicly due to privacy or confidentiality concerns. 


In order to perform data-free knowledge distillation, it is a necessary to reconstruct a dataset for training Student network. Thus, in this paper, we prospose a novel framework, named as *Data-Free Learning* (DAFL), for trainging efficient deep neural networks by exploiting generative adversarial networks (GANs).


![1](https://da2so.github.io/assets/post_img/2020-08-13-Data-Free_Learning_of_Student_Networks/1.png){: .mx-auto.d-block :}

## 2. Method

### <span style="color:gray">2.1 Teacher-Student Interactions </span>

Knowledge Distillation (KD) is a widely used apporach to transfer the output information from a heavy network to a smaller network for achieving higher performance. To describe this formally, let <span style="color:DodgerBlue">$\mathcal{N}_T$</span> and <span style="color:DodgerBlue">$\mathcal{N}_S$</span> denote the original pre-trained CNN (teacher network) and the desired portable network (student network). The student network can be optimized using the following loss function based on KD:

<span style="color:DodgerBlue">
\\[
L_{KD}=\frac{1}{n} \sum_i \mathcal{H}_c (y^i_S, y^i_T). \quad \quad \cdots Eq.(1)
\\]
</span>

where <span style="color:DodgerBlue">$\mathcal{H}_c$</span> is the cross-entropy loss, <span style="color:DodgerBlue">$y^i_T= \mathcal{N}_T(x^i)$</span> and <span style="color:DodgerBlue">$y^i_S=\mathcal{N}_S(x^i)$</span> are the outputs of the teacher network <span style="color:DodgerBlue">$\mathcal{N}_T$</span> and student network <span style="color:DodgerBlue">$\mathcal{N}_S$</span>, respectively.



### <span style="color:gray">2.2 GAN for Generating Training Samples </span>

In order to learn portable network without original data, we exploit GAN to generate training samples utilizing the available information of the given network.

GANs consist of a generator  <span style="color:DodgerBlue">$G$</span> and a discriminator  <span style="color:DodgerBlue">$D$</span>.  <span style="color:DodgerBlue">$G$</span> is expected to generates desired data while <span style="color:DodgerBlue">$D$</span> is trained to the differences between real images and the those produced by the generator as follows:

\\[
L_\{ GAN\}= \mathbb{E}_\{ ysadfad\}
\\]