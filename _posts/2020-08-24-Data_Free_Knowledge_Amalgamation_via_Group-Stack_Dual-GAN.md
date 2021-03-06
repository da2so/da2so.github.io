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

The architecture of Dual-GAN is shown in Fig 0.


![1](https://da2so.github.io/assets/post_img/2020-08-24-Data_Free_Knowledge_Amalgamation_via_Group-Stack_Dual-GAN/1.png){: .mx-auto.d-block width="50%", height="50%" :}


## 2. Problem Definiton

In Data-Free Knowledge Amalgamation via Group-Stack Dual-GAN, The authors aim to explore a more effective approach to train the student network (TargetNet), only utilizing the knowledge amalgamated from the pre-trained teachers. The TargetNet is designed to deal with multiple tasks and learns a customized multi-branch network that can recognize all labels selected from separate teachers.

* The number of the customized categories:  <span style="color:DodgerBlue">$C$</span>
* Label vector:  <span style="color:DodgerBlue">$Y_{cst}= \\{ y_1, \cdots , y_C \\}  \subseteq \\{ 0 , 1 \\}^C$</span>
* TargetNet: <span style="color:DodgerBlue">$\mathcal{T}$</span>
	* Handling multiple tasks on the <span style="color:DodgerBlue">$Y_{cst}$</span>
* Pre-trained teachers: <span style="color:DodgerBlue">$\mathcal{A} = \\{ \mathcal{A}_1, \cdots, \mathcal{A}_M \\}$</span>
* For each teacher <span style="color:DodgerBlue">$m$</span>, a <span style="color:DodgerBlue">$T_m$</span>-label classification task :<span style="color:DodgerBlue">$Y_m= \\{ y^1_m, \cdots , y^{T_m}_m \\}$</span>
* Feature maps in the <span style="color:DodgerBlue">$b$</span>-th block of the <span style="color:DodgerBlue">$m$</span>-th pre-trained teacher: <span style="color:DodgerBlue">$F^b_m$</span>


The teacher networks are in the constraint: <span style="color:DodgerBlue">$Y_{cst} \subseteq \bigcup^M_\{ m=1 \} Y_m $</span>, which reveals that either the full or the subset of classification labels is alternative for making up the customized task set.


## 3. Method


The process of obtaining the well-behaved TargetNet with the proposed data-free framework contains three steps.  
1. The generator <span style="color:DodgerBlue">$G(z): z \rightarrow \mathcal{I}$</span> is trained with knowledge amalgamation in the adversarial way, where the images in the same distribution of the original dataset can be manufactured. Note that <span style="color:DodgerBlue">$z$</span> is the random noise and <span style="color:DodgerBlue">$\mathcal{I}$</span> denotes the image.

2. The dual generator <span style="color:DodgerBlue">$\mathcal{T} (\mathcal{I}): \mathcal{I} \rightarrow Y_{cst}$</span> is trained with generated samples from <span style="color:DodgerBlue">$G$</span> in the block-wise way to produce multiple predict labels. Note tha <span style="color:DodgerBlue">$Y_{cst}$</span> is the predicted labels.

3. After training the whole dual-GAN, The dual-generator is modified as TargetNet for classifying the customized label sete <span style="color:DodgerBlue">$Y_{cst}$</span>


The overall training procedure of group-stack GAN is depicted in Fig 1. 


![1](https://da2so.github.io/assets/post_img/2020-08-24-Data_Free_Knowledge_Amalgamation_via_Group-Stack_Dual-GAN/5.PNG){: .mx-auto.d-block width="100%":}

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

Thus, we set <span style="color:DodgerBlue">$B$</span> as the total group number of generator, which is the same as the block numbers of the teacher and the student networks. In this way, the generator can be denoted as a stack of <span style="color:DodgerBlue">$B$</span> groups <span style="color:DodgerBlue">$\\{G^1, \cdots, G^B \\}$</span>, from which both the image <span style="color:DodgerBlue">$\mathcal{I}_g$</span> and the consequent activation <span style="color:DodgerBlue">$F^j_g$</span> at group <span style="color:DodgerBlue">$j$</span> are synthesized from a random noise <span style="color:DodgerBlue">$z$</span>:

<span style="color:DodgerBlue">
\\[
\\begin{array}{l}
F^1_g=G^1(z) \cr
F^j_g=G^j(F^\{ j-1 \}_g) \; 1 < j \leq B,
\\end{array} \quad \cdots Eq .(2)
\\]
</span>

![2](https://da2so.github.io/assets/post_img/2020-08-24-Data_Free_Knowledge_Amalgamation_via_Group-Stack_Dual-GAN/2.png){: .mx-auto.d-block width="60%" height="80%" :}


when <span style="color:DodgerBlue">$j=B$</span>, the output of the <span style="color:DodgerBlue">$B$</span>-th group <span style="color:DodgerBlue">$G^B$</span> is <span style="color:DodgerBlue">$F^B_g$</span>, which is also thought as the final generated image <span style="color:DodgerBlue">$\mathcal{I}_g$</span>.


Since both the architecture of generator <span style="color:DodgerBlue">$G^j$</span> and discriminator <span style="color:DodgerBlue">$D^jj$</span> are symmetrical, the group adversarial pair is presented as <span style="color:DodgerBlue">$[ \\{ G^1,D^1\\}, \cdots ,\\{ G^B,D^B\\} ]$</span>. In this way, the satisfying <span style="color:DodgerBlue">$G^{j \ast}$</span> can be acquired by:

<span style="color:DodgerBlue">
\\[
G^{j \ast }=argmin_\{ G^j \} \mathbb{E}_z [log(1-D^{j*} (G^j (F^{j-1}_g)))] \quad \cdots Eq. (3)
\\]
</span>

where <span style="color:DodgerBlue">$1 \leq j \leq B$</span> and <span style="color:DodgerBlue">$D^{j*}$</span> is the optimal <span style="color:DodgerBlue">$j$</span>-group discriminator. We transfer the discriminator to designing a plausible loss function to calculate the difference between the generated samples and the real ones. Thus, we take the teacher network <span style="color:DodgerBlue">$\mathcal{A}$</span> to constitute each <span style="color:DodgerBlue">$D^j$</span>:

<span style="color:DodgerBlue">
\\[
D^j \leftarrow \bigcup^j_{i=1} \\{ \mathcal{A}^{B-j+i} \\} \quad \cdots Eq. (4)
\\]
</span>

![3](https://da2so.github.io/assets/post_img/2020-08-24-Data_Free_Knowledge_Amalgamation_via_Group-Stack_Dual-GAN/3.png){: .mx-auto.d-block width="80%" :}

During the training for the group pair <span style="color:DodgerBlue">$\\{ G^j, D^j \\}$</span>, only <span style="color:DodgerBlue">$G^j$</span> is optimized with discriminator <span style="color:DodgerBlue">$D^j$</span> is fixed, whose output is for classifying multiple labels. We make use of several losses to constraint the output of <span style="color:DodgerBlue">$D^j$</span> to motivate the real data's response. 

The output for <span style="color:DodgerBlue">$D$</span> is <span style="color:DodgerBlue">$\mathcal{O} ( F_g) = \\{ y^1, \cdots , y^C \\} $</span>, with the predict label as <span style="color:DodgerBlue">$t^i$</span>:

<span style="color:DodgerBlue">
\\[ 
t^i= \\left\\{ \\begin{array}{ll} 1  & y^i \geq \epsilon, \cr
								  0  & y^i < \epsilon, 
\\end{array} \\right. \quad \cdots Eq. (5)
\\]
</span>

where <span style="color:DodgerBlue">$1 \leq i \leq C$</span> and <span style="color:DodgerBlue">$\epsilon$</span> is set to be 0.5 in the experiment. Then the one-hot loss function can be defined as:

<span style="color:DodgerBlue">
\\[
L_{oh}= \frac{1}{C}\sum_i l(y^i, t^i), \quad \cdots Eq. (6)
\\]
</span>

where <span style="color:DodgerBlue">$l$</span> is the cross-entropy and <span style="color:DodgerBlue">$L_{oh}$</span> enforces the outputs of the generated samples to be close to one-hot vectors.

In addition, the outputs need to be sparse, since an image in the real world can't be tagged with dense labels which are the descriptions for different situations. So, an extra discrete loss function is proposed:

<span style="color:DodgerBlue">
\\[
L_{dis}= \frac{1}{C} \sum_i \vert y^i \vert, \quad \cdots Eq. (7)
\\]
</span>

which is known as L1-norm loss function. Finally, combining all the losses, the final objective function can be obtained:

<span style="color:DodgerBlue">
\\[
L_{gan}= L_\{oh\} + \alpha L_a + \beta L_\{ie\} + \gamma L_\{dis \}, \quad \cdots Eq. (8)
\\]
</span>

where <span style="color:DodgerBlue">$\alpha, \beta $</span> and <span style="color:DodgerBlue">$\gamma $</span> are the hyperparameters for balancing different loss items. <span style="color:DodgerBlue">$L_a$</span> and <span style="color:DodgerBlue">$L_{ie}$</span> are the activation loss and infromation entropy loss functions and described in [here](https://da2so.github.io/2020-08-13-Data_Free_Learning_of_Student_Networks/).

#### <span style="color:gray"> 3.1.2 Multiple Targets </span>

Since the TargetNet is customized to perform multi-label classifications learning from multiple teachers, the generator should generate samples containing multiple targets. As a result, for the <span style="color:DodgerBlue">$j $</span>-th group generator <span style="color:DodgerBlue">$G^j$</span>, authors construct multiple group-stack discriminators <span style="color:DodgerBlue">$\\{ D^j_1, \cdots, D^j_M \\}$</span> in concert with teachers specializing in different task sets by Eq. (4).

In order to amalgamate multi-knowledge into the generator by <span style="color:DodgerBlue">$\\{ D^j_m \\}^M_{m=1} $</span>, the teacher-level filtering is applied to <span style="color:DodgerBlue">$F^j_g $</span>. And this filtering is conducted as:

<span style="color:DodgerBlue">
\\[
F^{f,m}_g=f^j_m (F^j_g), \quad \cdots Eq. (9)
\\]
</span>

where the filtering function <span style="color:DodgerBlue">$f^j_m$</span> is realized by a light learnable module consisting of a global pooling layer and two fully connected layers. <span style="color:DodgerBlue">$F^{j,m}_g$</span> is the filtered generated features that approaches the output feature distribution of <span style="color:DodgerBlue">$\mathcal{A}^{B-j}_m$</span>. The output of discriminator is donated as  <span style="color:DodgerBlue">$\mathcal{O}_m (F^j_g) = D^j_m (F^{j,m}_g$</span>.


![3](https://da2so.github.io/assets/post_img/2020-08-24-Data_Free_Knowledge_Amalgamation_via_Group-Stack_Dual-GAN/4.png){: .mx-auto.d-block width="100%" :}

The for the generated features  <span style="color:DodgerBlue">$F^j_g$</span> from  <span style="color:DodgerBlue">$G^j$</span>, we collect from the multi-discriminator the  <span style="color:DodgerBlue">$M$</span> prediction sets  <span style="color:DodgerBlue">$\\{ \mathcal{O}_1(F^j_g), \cdots , \mathcal{O}_M (F^j_g) \\}$</span>, which are:


<span style="color:DodgerBlue">
\\[
\mathcal{O}_g (F^j_g) = \bigcup^M_m \mathcal{O}_m ( F^j_g ), \quad \cdots Eq. (10)
\\]
</span>


which is treated as new input to the loss Eq. 8, then <span style="color:DodgerBlue">$L^i_{gan}$</span> is the adversarial loss for each <span style="color:DodgerBlue">$G^j$</span>. Despite the fact that the generated features should appear like the ones extracted
from the real data, they should also lead to the same predictions from the same input span style="color:DodgerBlue">$z$</span>. Thus, the stack-generator <span style="color:DodgerBlue">$\\{ G^1, \cdots, G^B \\}$</span> can be jointly optimized by the final loss:


<span style="color:DodgerBlue">
\\[
L_{joint}= L^B_\{gan\} + \frac{1}{B-1} \sum^{B-1}_\{ j=1 \} l(\mathcal{O}_g (F^j_g), \mathcal{O}_g (\mathcal{I}_g)), \quad \cdots Eq. (11)
\\]
</span>

where the adversarial loss <span style="color:DodgerBlue">$L_{gan}$</span> only calculates from the last group <span style="color:DodgerBlue">$\\{ G^B, D^B \\}$</span>. The rest part of the loss is the cross-entropy loss that restrains the intermediate features generated from <span style="color:DodgerBlue">$G^1$</span> to <span style="color:DodgerBlue">$G^{B-1}$</span> to make the same prediction as <span style="color:DodgerBlue">$\mathcal{I}_g$</span>, which offsets the adversarial loss <span style="color:DodgerBlue">$\\{ L^1_{gan}, \cdots$</span><span style="color:DodgerBlue">$, L^{B-1}_\{ gan \} \\}$</span>.


By minimizing <span style="color:DodgerBlue">$L_{joint}$</span>, the optimal generator <span style="color:DodgerBlue">$G$</span> can synthesis the images that have the similar activations as the real data fed to the teacher.



### <span style="color:gray"> 3.2 Dual-generator Training </span>

After training the group-stack GAN, a set of generated samples are obtained, which are in the form of <span style="color:DodgerBlue">$\mathcal{R}= \\{ F^1_g, \cdots, F^{B-1}_g\\} \cup \\{ \mathcal{I}_g\\}$</span> including both the generated intermediate features and the RGB images. Then, the next step is to train the dual-generator <span style="color:DodgerBlue">$\mathcal{T}$</span>. Also, the group-discriminator for each <span style="color:DodgerBlue">$\mathcal{T}^b$</span> is constructed as:


<span style="color:DodgerBlue">
\\[
D^{b,m}_d \rightarrow \bigcup^{B-b}_i \\{ \mathcal{A}^{b+i}_m \\}, \quad \cdots Eq. (12)
\\]
</span>


where the dual generator <span style="color:DodgerBlue">$\mathcal{T}$</span> is trained to synthesis the samples that <span style="color:DodgerBlue">$D_d$</span> can't distinguished from <span style="color:DodgerBlue">$\mathcal{I}_g$</span> produced by the trained <span style="color:DodgerBlue">$G$</span>.


We divide dual-generator into <span style="color:DodgerBlue">$B$</span> blocks as <span style="color:DodgerBlue">$\\{ \mathcal{T}^1, \cdots , \mathcal{T}^B \\}$</span>, and during the training process of <span style="color:DodgerBlue">$\mathcal{T}^b (1 < b \leq B)$</span>, the whole generator <span style="color:DodgerBlue">$G$</span> and the blocks of the dual-generator <span style="color:DodgerBlue">$\mathcal{T}$</span> from $1$ to $b-1$ keep fixed.


Taking training <span style="color:DodgerBlue">$\mathcal{T}^b$</span> for example, the problem is to learn features <span style="color:DodgerBlue">$F^b_u$</span> generated by the <span style="color:DodgerBlue">$b$</span>-th block of the dual-generator. As shown in Fig 5, the generated <span style="color:DodgerBlue">$F^b_u$</span> is treated as 'fake' to the discriminator with <span style="color:DodgerBlue">$\mathcal{I}_g$</span> as the 'real' data in this dual part GAN. To do this, a **two-level filtering method** is applied to <span style="color:DodgerBlue">$F^b_u$</span>. 


1. Teacher-level filtering for multi-target demand <span style="color:DodgerBlue">$f_m$</span>
	* Transforming <span style="color:DodgerBlue">$F^b_u$</span> to <span style="color:DodgerBlue">$m$</span> teacher streams <span style="color:DodgerBlue">$F^{b,m}_u=f^b_m(F^b_u)$</span>, as defined in Eq. (9).

2. Task-level filtering <span style="color:DodgerBlue">$g_m$</span>
	* Conducting after the last few fully connected layers of the corresponding discriminator, which is established for the constraint of <span style="color:DodgerBlue">$Y_{cst}  \subseteq \bigcup^M_m Y_m$</span>.


And, the authors feed the generated features <span style="color:DodgerBlue">$F^b_u$</span> to the corresponding discriminator <span style="color:DodgerBlue">$D^{b,m}_d$</span>, and derive the predictions <span style="color:DodgerBlue">$\mathcal{O}_m(F^b_u)$</span>. Then, the task-level filtering <span style="color:DodgerBlue">$g_m$</span> is applied to meet the task customizable demand ( <span style="color:DodgerBlue">$Y_{cst}  \subseteq \bigcup^M_m Y_m$</span> )and embedded into the <span style="color:DodgerBlue">$m$</span>-th branch block-wise adversarial loss:


<span style="color:DodgerBlue">
\\[
L^{b,m}_{d}=l(g_m (\mathcal{O}_m(F^b_u) ,\; g_m(\mathcal{O}_m(\mathcal{I}_g))  , \quad \cdots Eq. (13)
\\]
</span>


So, the block-wise loss for updating dual-generator <span style="color:DodgerBlue">$\mathcal{T}^b$</span> from multiple branches can be defined as:

<span style="color:DodgerBlue">
\\[
L^b_{d}=\sum_m \lambda_m \cdot L^{b,m}_\{d\}  , \quad \cdots Eq. (14)
\\]
</span>

where <span style="color:DodgerBlue">$\lambda=1$</span> for <span style="color:DodgerBlue">$m \in \\{ 1, \cdots , M \\}$</span>. Since the existence of the links between the generators <span style="color:DodgerBlue">$G$</span> and <span style="color:DodgerBlue">$\mathcal{T}$</span>, the input has two streams:

<span style="color:DodgerBlue">
\\[ 
 \\begin{array}{l} F^1_i=\mathcal{T}^{b-1} \mathcal{T}^{b-2} \cdots \mathcal{T}^1 (\mathcal{I}_g ), \cr
								  F^2_i = G^{B+1-b} G^{B-b} \cdots G^1 (z). 
\\end{array} \quad \cdots Eq. (15)
\\]
</span>


Then according to the different inputs to <span style="color:DodgerBlue">$\mathcal{T}^b$</span>, **the final loss** can be rewritten from Eq. (14) as:

<span style="color:DodgerBlue">
\\[
L^{b}_u=\lambda^1_i L^{b,m}_d (F^1_i) + \lambda^2_i L^{b,m}_d (F^2_i), \quad \cdots Eq. (16)
\\]
</span>

