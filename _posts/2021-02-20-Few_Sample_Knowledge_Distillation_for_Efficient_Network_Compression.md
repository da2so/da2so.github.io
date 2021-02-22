---
layout: post
title: Few Sample Knowledge Distillation for Efficient Network Compression
tags: [Knowledge distillation, Model Compression]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2021-02-20-Few_Sample_Knowledge_Distillation_for_Efficient_Network_Compression/post.png
---

## 1. Introduction   
**Knowledge distillation:** Dealing with the problem of training a smaller model (Student) from a high capacity source model (Teacher) so as to retain most of its performance.  
{: .box-note}

### <span style="color:gray"> 1.1 Motivation </span>

The compression methods such as pruning and filter decompostion require the fine-tuning to recover performance. However, fine-tuning suffers from the requirement of a large training set and the time-consuming training procedure.

### <span style="color:gray"> 1.2 Goal </span>

For realizing both data efficiency and training efficiecny, this paper proposes a novel method, namely **few-sample knowledge distillation (FSKD)** with none of labels. The authors of this paper treat the original network as *teacher-net* and the compressed network as *student-net*. A 1x1 convolution layer is added at the end of each layer block of the studnet-net and then align the block-level outputs of the student-net with the teacher-net by estimating the parameters of the added layer.


## 2. Method

### <span style="color:gray"> 2.1 Overview </span>

![2](https://da2so.github.io/assets/post_img/2021-02-20-Few_Sample_Knowledge_Distillation_for_Efficient_Network_Compression/1.png){: .mx-auto.d-block width="90%" :}

FSKD method consists of three steps:

1. Obtaining a student-net either by pruning or by decomposing the teacher net.
2. Adding a 1x1 conv-layer at the end of each block of student-net and align the block-level outputs between teacher and student by estimating the parameters of the added layer.
3. Merging the added 1x1 conv-layer into the previous conv-layer so that extra parameters and computation cost is not required on the studnet-net.

Three reasons make the idea works efficiently.

1. Adding 1x1 conv-layer is enough to calibrate studnet-net and recover performance.
2. The 1x1 conv-layers have relatively fewer parameters.
3. The block-level output from teacher-net provides rich information as shown in FitNet.

### <span style="color:gray"> 2.2 Block-level Alignment </span>
 
- <span style="color:DodgerBlue">$ X^s , X^t $</span>: the block-level output in matrix form for the student and teacher, respectively.
	- <span style="color:DodgerBlue">$ X^s , X^t \in \mathcal{R}^\{n_o \times d \}$</span>: supposing both network's output sizes are same.
		- <span style="color:DodgerBlue">$ d $</span>: the per-channel feature map resolution size.
		- <span style="color:DodgerBlue">$ n_o $</span>: the number of output channels 

- <span style="color:DodgerBlue">$ Q $</span>: 1x1 conv-layer

They add a 1x1 conv-layer <span style="color:DodgerBlue">$ Q $</span> at the end of each block of student-net before non-linear activation.
As <span style="color:DodgerBlue">$ Q $</span> is degraded to the matrix form, it can be estimated with least squared regression as:

 <span style="color:DodgerBlue">
\\[
Q^* = argmin_Q  \sum^N_i \lVert Q * X^s_i - X^t_i \rVert , \quad \cdots (1)
\\] 
</span>

where <span style="color:DodgerBlue">$ N $</span> is the number of label-free samples used, and <span style="color:DodgerBlue">$ *$</span> means matrix product. The number of parameters of <span style="color:DodgerBlue">$ Q $</span> is <span style="color:DodgerBlue">$ n_o \times n_o $</span>.


Suppose there are <span style="color:DodgerBlue">$ M $</span> corresponding blocks in the teacher and student required to alian. To achieve our goal, minimize the following loss function

 <span style="color:DodgerBlue">
\\[
L(Q_j) = \sum^M_j  \sum^N_i \lVert Q * X^s_\{ij\} - X^t_\{ij\} \rVert_F , \quad \cdots (2)
\\] 
</span>

where <span style="color:DodgerBlue">$ Q_j $</span> is the tensor for the added 1x1 conv-layer of the $j$-th block.
 
In practice, the authors optimize this loss with a **block coordinate descent (BCD) algorithm**, which greedily handles each of the <span style="color:DodgerBlue">$ M$</span> blocks in the student sequentially. They also uses FSKD with standard SGD but, they experimentally showed that FSKD with BCD is more efficient: **(1)** Each <span style="color:DodgerBlue">$ Q $</span> can be solved with the same set of few samples by aligning the block-level outputs between student and teacher. **(2)** The BCD can be done in less than a minute.

### <span style="color:gray"> 2.3 Mergeable 1x1 conv-layer </span>

In this section, They prove that the added 1x1 conv-layer can be merged into the previous conv-layer without introducing additional parameters and computation cost.


#### Theorem 1

A pointwise convolution with tensor <span style="color:DodgerBlue">$ Q \in \mathcal{R}^\{n'_o \times n'_i \times 1 \times 1 \}$</span> can be merged into the previous convolution layer with tensor <span style="color:DodgerBlue">$ W' \in \mathcal{R}^\{ n_o \times n_i \times k \times k \} $</span> to obtain the merged tensor <span style="color:DodgerBlue">W' = Q \circ W $</span>, where <span style="color:DodgerBlue">$ \circ $</span> is merging operator and <span style="color:DodgerBlue">$ W' \in \mathcal{R}^\{ n'_o \times n_i \times k \times k \} $</span> if the following conditions are statisfied.

**c1.** *The output channel number of* <span style="color:DodgerBlue">$ W $</span> *equals to the input channel number of* <span style="color:DodgerBlue">$ Q $</span>, *i.e.,* <span style="color:DodgerBlue">$ n_o=n'_i $</span> .
**c2.** *No non-linear activation layer like ReLu between* <span style="color:DodgerBlue">$ W$</span>  *and* <span style="color:DodgerBlue">$ Q $</span>.


The pointwise convolution can be viewed as a linear combination of the kernels in the previous convolution layer. This implies that the two layers are integrated one layer. However, the number of output channels of <span style="color:DodgerBlue">$ W' $</span> is <span style="color:DodgerBlue">$ n'_o $</span>, which is differenct from that of <span style="color:DodgerBlue">$ W $</span>. It is easy to have the following corollary.


**Corollary 1.** When the following condition is satisfied,

**c3.** *the number of input and output channels of * <span style="color:DodgerBlue">$ Q$</span>  *equals to the number of output channel of* <span style="color:DodgerBlue">$ W $</span>, *i.e.,* <span style="color:DodgerBlue">$ n'_i = n'_o = n_o $</span>, <span style="color:DodgerBlue">$ Q \in \mathcal{R}^\{ n_o \times n_o \times 1 \times 1 \} $</span>, *the merged convolution tensor* <span style="color:DodgerBlue">$ W' $</span> has the same parameters and computation cost as <span style="color:DodgerBlue">$ W $</span> *, i.e. both* <span style="color:DodgerBlue">$ W', W  \in \mathcal{R}^\{n_o \times n_i \times k \times k \}$</span>.


This condition is required not only for ensuring the same parameter size and computing cost, but also for ensuring the ouput size of current layer matching to the input of next layer.

The overall algorithm of FSKD is described in below figure.

![3](https://da2so.github.io/assets/post_img/2021-02-20-Few_Sample_Knowledge_Distillation_for_Efficient_Network_Compression/2.png){: .mx-auto.d-block width="60%" :}

## 3. Experiment Setting

For all experiments, the results are averaged over 5 trials of different randomly selected images. Considering the number of channels in teacher may be different from that in studnet, they only match the un-pruned part of feature maps in teacher-net to the feature maps in the student net as shown in Fig 2. And, Fig 3 represents how FSKD works for block-level alignment.

![3](https://da2so.github.io/assets/post_img/2021-02-20-Few_Sample_Knowledge_Distillation_for_Efficient_Network_Compression/3.png){: .mx-auto.d-block width="90%" :}


## 4. Experiment Result


![3](https://da2so.github.io/assets/post_img/2021-02-20-Few_Sample_Knowledge_Distillation_for_Efficient_Network_Compression/4.png){: .mx-auto.d-block width="100%" :}
![3](https://da2so.github.io/assets/post_img/2021-02-20-Few_Sample_Knowledge_Distillation_for_Efficient_Network_Compression/5.png){: .mx-auto.d-block width="100%" :}

