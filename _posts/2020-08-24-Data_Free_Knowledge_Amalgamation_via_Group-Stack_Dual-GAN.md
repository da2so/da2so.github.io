---
layout: post
title: Data-Free Knowledge Amalgamation via Group-Stack Dual-GAN
tags: [Knowledge distillation, Data-free, Synthesized image, Multi-]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-08-24-Data_Free_Knowledge_Amalgamation_via_Group-Stack_Dual-GAN/post.png
---

## 1. Goal

The goal is to peform **Data-free Knowledge distillation**.

{: .box-note}
**Knowledge distillation:** Dealing with the problem of training a smaller model (Student) from a high capacity source model (Teacher) so as to retain most of its performance.

As the word itself, We perform knowledge distillation when there are no original dataset on which the Teacher network has been trained. It is because, in real world, most datasets are proprietary and not shared publicly due to privacy or confidentiality concerns. 

To tackle this problem, it is a necessary to reconstruct a dataset for training Student network. Thus, in this paper, <ul> the authors propose a data free knowledge amalgamate strategy to craft a well-behaved multi-task student network from multiple singl/multi-task teachers. </ul>


For this, the main idea is to construct the group-stack generative adversarial networks (GANs) which have two dual generators. **First** one generator is trained to collect the knwoledge by reconstructing the images approximating the original dataset. Then, a **dual generator (** is trained by taking the output from the former generator as input. Finally, we treat the dual part generator as the **TargetNet (Student network)** and regroup it.

The architecture of Dual-GAN is shown in Fig 1.


![1](https://da2so.github.io/assets/2020-08-24-Data_Free_Knowledge_Amalgamation_via_Group-Stack_Dual-GAN/1.png){: .mx-auto.d-block :}


## 2. Problem Definiton

The authors aim to explore a more effective approach to train the student network (TargetNet), only utilizing the knowledge amalgamated from the pre-trained teachers. The TargetNet is designed to deal with multiple tasks and learns a customized multi-branch network that can recognize all labels selected from separate teachers.

* The number of the customized categories  <span style="color:DodgerBlue">$C$</span>
* Label vector  <span style="color:DodgerBlue">$Y_{cst}= \\{ y_1, \cdots , y_C \\}  \subsete \\{ 0 , 1 \\}^C$</span>


Given a trained model <span style="color:DodgerBlue">$p_T$</span> and a dataset <span style="color:DodgerBlue">$\mathcal{X}$</span>, the parameters of the student model, <span style="color:DodgerBlue">$W_S$</span>, can be learned by 


<span style="color:DodgerBlue">
\\[
min_{\, W_S} \sum_\{ x \in \mathcal{X} \} KL( p_T(x), p_S(x) ), \quad \cdots Eq. (1)
\\]
</span>
