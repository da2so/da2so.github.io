---
layout: post
title: Zero-Shot Knowledge Distillation in Deep Networks
tags: [Model Compression, Knowledge Distillation, Data-free]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-08-12-Zero-Shot_Knowledge_Distillation_in_Deep_Networks/post.PNG
---

## 1. What is data-free knowledge distillation??

{: .box-note}
**Knowledge distillation:** Dealing with the problem of training a smaller model (Student) from a high capacity source model (Teacher) so as to retain most of its performance.

As the word itself, We perform knowledge distillation when there are no original dataset on which the Teacher network has been trained. It is because, in real world, most datasets are proprietary and not shared publicly due to privacy or confidentiality concerns. 

In order to perform data-free knowledge distillation, it is a necessary to reconstruct a dataset for training Student network. Then, in this paper, we propose "Zero-Shot Knowledge Distillation" (ZSKD), which perform pseudo data synthesis from the Teacher model that act as the transfer set to perform the distillation without even using any meta-data.

![2](https://da2so.github.io/assets/post_img/2020-08-12-Zero-Shot_Knowledge_Distillation_in_Deep_Networks/1.png){: .mx-auto.d-block :}


## 2. Method

### 2.1 Knowledge Distillation

Transferring the generalization ability of a large, complex _Teacher_ (<span style="color:DodgerBlue">$T$</span>) deep neural network to a less complex _Student_ (<span style="color:DodgerBlue">$S$</span>) network can be achieved using the class probabilities produced by a _Teacher_ as "soft targets" for training the _Student_.


Let <span style="color:DodgerBlue">$T$</span> be the _Teacher_ network with learned parameters <span style="color:DodgerBlue">$\theta_T$</span> and <span style="color:DodgerBlue">$S$</span> be the _Student_ with parameters <span style="color:DodgerBlue">$\theta_S$</span>, note that in general <span style="color:DodgerBlue">$\vert \theta_S \vert \ll \vert \theta_T \vert$</span>.


Knowledge distillation methods train the _Student_ by minimizing the following objective (<span style="color:DodgerBlue">$L$</span>).

<span style="color:DodgerBlue">\\[
L=\sum_{(x,y) \in \mathbb{D}} L_K (S(x,\theta_S,\tau), T(x,\theta_T, \tau))+\lambda L_C( \widehat{y}_S,y)
\\] </span>

,where <span style="color:DodgerBlue">$D$</span> is training dataset, <span style="color:DodgerBlue">$L_C$</span> is the cross-entropy loss computed on the labels <span style="color:DodgerBlue">$ \widehat{y}_S $</span> predicited by the *Student* and ground truth <span style="color:DodgerBlue">$y$</span>. <span style="color:DodgerBlue">$L_K$</span> is the distillation loss (e.g. cross-entropy or MSE), <span style="color:DodgerBlue">$ (T(x,\theta_T,\tau)$</span> indicates the softmax output of the *Teacher* and <span style="color:DodgerBlue">$S(x,\theta_S, \tau)$</span> denotes the softmax output of the *Student*. Note that, unless it is mentioned, we use a temperature (<span style="color:DodgerBlue">$\tau$</span>) of 1.



###<span style="color:gray"> 2.2 Modelling the Data in Softmax Space</span>

In this paper, we deal with the scenario where we have no access to **(i)** any training data samples (either from the target distribution or different) **(ii)** meta-data extracted from it.

To tackle this, our approach taps the learned parameters of the *Teacher* and produce synthesized input representations, named as *Data Impressions* ,from the underlying data distribution on which it is trained. These can be used as a transfer set in order to perform knowledge distillation to a *Student* model.


In order to craft the *Data impressions*, we model output space of the *Teacher* model. Let <span style="color:DodgerBlue">$s \sim p(s)$</span>, be the random vector that represents the softmax outputs of the *Teacher*, <span style="color:DodgerBlue">$T(x, \theta_T)$</span>. We model <span style="color:DodgerBlue">$p(s^k)$</span> belonging to each class <span style="color:DodgerBlue">$k$</span>, using Dirichlet distribution.


{: .box-note}
** Dirichlet distribution:**  <span style="color:DodgerBlue">$Dir(x_1, \cdots x_K, \alpha_1, \cdots, \alpha_K) s.t. \sum^K_i x_i =1 and x_i \geq$0 \forall i</span>

The distribution to represent the softmax output <span style="color:DodgerBlue">$s^k$</span> of class <span style="color:DodgerBlue">$k$</span> would be modelled as, <span style="color:DodgerBlue">$Dir(K,\alpha^k)$</span> where <span style="color:DodgerBlue">$k \in {1 \cdots K}$</span> is the class index, <span style="color:DodgerBlue">$K$</span> is the dimension of the output probability vector and <span style="color:DodgerBlue">$\alpha^k$</span> is the concentration parameter of the distribution modelling class <span style="color:DodgerBlue">$k$</span>. (<span style="color:DodgerBlue">$\alpha^k=\[\alpha^k_1, \cdots \alpha^k_K\] and \alpha^k_i>0, \forall i $</span>)


###<span style="color:gray"> 2.2.1 Concentration Parameter ($\alpha$)</span>

<span style="color:DodgerBlue">$\alpha$</span> can be thought of as determining how "concentrated" the porbability mass of a sample from a Dirichlet distribution is likely to be.


| If <span style="color:DodgerBlue">$\alpha \ll 1$</span> $\rightarrow$ the mass is highly concentrated in only a few components|
| Elif <span style="color:DodgerBlue">$\alpha \gg 1 $</span> $\rightarrow$ the mass is dispersed almost equally among all the components|

So, it is important to determine right <span style="color:DodgerBlue">$\alpha$</span>. We make its values to reflect the similarties across the components in the softmax vector. Since these components denote the underlying categories in the recognition problem, <span style="color:DodgerBlue">$\alpha$</span> should reflect the visual similarities among them.


Thus, we resort to the *Teacher* network for extracting this information. We compute a normalized class similarity matrix (<span style="color:DodgerBlue">$C$</span>) using the weights <span style="color:DodgerBlue">$W$</span> connecting the final (softmax) and the pre-final layers. The element <span style="color:DodgerBlue">$C(i,j)$</span> of this matrix denotes the visual similarity between the categories <span style="color:DodgerBlue">$i$</span> and <span style="color:DodgerBlue">$j$</span> in [0,1].


![2](https://da2so.github.io/assets/post_img/2020-08-12-Zero-Shot_Knowledge_Distillation_in_Deep_Networks/2.PNG){: .mx-auto.d-block :}

###<span style="color:gray"> 2.2.2 Class Similarity Matrix ($C$)</span>

The weights <span style="color:DodgerBlue">$w_k$</span> can be considered as the template of the class <span style="color:DodgerBlue">$k$</span> learned by the *Teacher* network. This is because the predicted class probability is proportional to the alignment of the pre-final layer’s output with the template <span style="color:DodgerBlue">$w_k$</span>.

|If pre-final layer's output is positive scaled version of <span style="color:DodgerBlue">$w_k$</span> $\rightarrow$ predicted probability for class <span style="color:DodgerBlue">$k$</span>peaks|
|Elif pre-final layer's ouput is misaligned with the <span style="color:DodgerBlue">$w_k$</span> $\rightarrow$ predicted probability for class <span style="color:DodgerBlue">$k$</span> is reduced|

Therefor, , wetreat the weights wk as the class template for class <span style="color:DodgerBlue">$k$</span> and compute the similarity between classes <span style="color:DodgerBlue">$i$</span> and <span style="color:DodgerBlue">$j$</span> as:

\\[
C(i,j)=\frac{w^T_i w_j}{\Vert w_i \Vert \Vert w_j \Vert}.
\\]

Since the elements of the concentration parameter have to be positive real numbers, we perform a min-max normalization over each row of the class similarity matrix.