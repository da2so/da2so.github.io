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

To tackle this problem, it is a necessary to reconstruct a dataset for training Student network. Thus, in this paper, <u> the authors propose a data free knowledge amalgamate strategy to craft a well-behaved multi-task student network from multiple single/multi-task teachers. </u>


For this, the main idea is to construct the group-stack generative adversarial networks (GANs) which have two dual generators. First, **one generator** is trained to collect the knwoledge by reconstructing the images approximating the original dataset. Then, a **dual generator (** is trained by taking the output from the former generator as input. Finally, we treat the dual part generator as the **TargetNet (Student network)** and regroup it.

The architecture of Dual-GAN is shown in Fig 1.


![1](https://da2so.github.io/assets/post_img/2020-08-24-Data_Free_Knowledge_Amalgamation_via_Group-Stack_Dual-GAN/1.png){: .mx-auto.d-block width="70%" :}


## 2. Problem Definiton

The authors aim to explore a more effective approach to train the student network (TargetNet), only utilizing the knowledge amalgamated from the pre-trained teachers. The TargetNet is designed to deal with multiple tasks and learns a customized multi-branch network that can recognize all labels selected from separate teachers.

* The number of the customized categories:  <span style="color:DodgerBlue">$C$</span>
* Label vector:  <span style="color:DodgerBlue">$Y_{cst}= \\{ y_1, \cdots , y_C \\}  \subseteq \\{ 0 , 1 \\}^C$</span>
* TargetNet: <span style="color:DodgerBlue">$\mathcal{T}$</span>
	* Handling multiple tasks on the <span style="color:DodgerBlue">$Y_{cst}$</span>
* Pre-trained teachers: <span style="color:DodgerBlue">$\mathcal{A} = \\{ \mathcal{A}_1, \cdots, \mathcal{A}_M \\}$</span>
* For each teacher <span style="color:DodgerBlue">$m$</span>, a <span style="color:DodgerBlue">$T_m$</span>-label classification task :<span style="color:DodgerBlue">$Y_m= \\{ y^1_m, \cdots , y^{T_m}_m \\}$</span>
* Feature maps in the <span style="color:DodgerBlue">$b$</span>-th block of the <span style="color:DodgerBlue">$m$</span>-th pre-trained teacher: <span style="color:DodgerBlue">$F^b_m$</span>


The teacher networks are in the constraint: <span style="color:DodgerBlue">$Y_{cst} \subseteq \cup^M_\{ m=1 \} Y_m $</span>, which reveals that either the full or the subset of classification labels is alternative for making up the customized task set.


## 3. Method


The process of obtaining the well-behaved TargetNet with the proposed data-free framework contains three steps.  
1. The generator <span style="color:DodgerBlue">$G(z): z \rightarrow \mathcal{I}$</span> is trained with knowledge amalgamation in the adversarial way, where the images in the same distribution of the original dataset can be manufactured. Note that <span style="color:DodgerBlue">$z$</span> is the random noise and <span style="color:DodgerBlue">$\mathcal{I}$</span> denotes the image.

2. The dual generator <span style="color:DodgerBlue">$\mathcal{T} (\mathcal{I}): \mathcal{I} \rightarrow Y_{cst}$</span> is trained with generated samples from <span style="color:DodgerBlue">$G$</span> in the block-wise way to produce multiple predict labels. Note tha <span style="color:DodgerBlue">$Y_{cst}$</span> is the predicted labels.

3. After training the whole dual-GAN, The dual-generator is modified as TargetNet for classifying the customized label sete <span style="color:DodgerBlue">$Y_{cst}$</span>

### <span style="color:gray"> 3.1 Amalgamating GAN </span>


First, introduce the arbitrary vanilla GAN. The GAN performs a minmax game between a generator <span style="color:DodgerBlue">$G(z): z \rightarrow \mathcal{I} $</span> and a discriminator <span style="color:DodgerBlue">$D (x) : \mathcal{I} \rightarrow [0,1]$</span>, and the objective function can be defined as:

<span style="color:DodgerBlue">
\\[
L_{GAN}= \mathbb{E}_x [ log \mathcal{D} ( \mathcal{I})] +\mathbb{E} [log(1-D(G(z)))]. \quad \cdots Eq. (1)
\\]
</span>

However, since the absence of the real data, we can not perfrom training by Eq. (1). Then, several modification have been made as follows.


#### <span style="color:gray"> 3.1.1 Group-stack GAN </span>

The first modification is the group-stack architecture. The generator is designed to generate not only synthesized images but also the intermediated activations aligned with the teachers. 

Thus, we set <span style="color:DodgerBlue">$B$</span> as the total group number of generator, which is the same as the block numbers of the teacher and the student networks. In this way, the generator can be denoted as a stack of <span style="color:DodgerBlue">$B$</span> groups <span style="color:DodgerBlue">$\\{G^1, \cdots, G^B \\}$</span>, from which both the image <span style="color:DodgerBlue">$\mathcal{I}_{gan}$</span> and the consequent activation <span style="color:DodgerBlue">$F^j_{gan}$</span> at group <span style="color:DodgerBlue">$j$</span> are synthesized from a random noise <span style="color:DodgerBlue">$z$</span>:

<span style="color:DodgerBlue">
\\[
\\begin{array}{l}
F^1_{gan}=G^1(z) \cr
F^j_\{gan\}=G^j(F^\{ j-1 \}_\{gan\}) \; 1 < j \leq B,
\\end{array} \quad \cdots Eq .(2)
\\]
</span>


