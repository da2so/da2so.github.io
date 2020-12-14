---
layout: post
title: Adjustable Real-time Style Transfer
tags: [Style Transfer]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-12-11-Adjustable_Real_time_Style_Transfer/post.PNG
---

## 1. Introduction

{: .box-note}
**Style transfer:** Synthesizing an image with content similar to a given image and style similar to another.

![2](https://da2so.github.io/assets/post_img/2020-12-11-Adjustable_Real_time_Style_Transfer/1.PNG){: .mx-auto.d-block width="60%" :}

### <span style="color:gray"> 1.1 Motivation </span>

There are two main problems in style transfer on the existing methods. <span style="color:#6495ED">(i)</span> The first weak point is that they generate only one stylization for a given content/style pair. <span style="color:#6495ED">(ii)</span>One other issue of them is their high-sensitivity to the hyper-parameters.

### <span style="color:gray"> 1.2 Goal </span>

To solve these problems, the authors provide a novel mechanism which allows adjustment of crucial hyper-parameters, after the training and in real-time, through a set of manually adjustable parameters.


## 2. Background

### <span style="color:gray"> 2.1 Style transfer using deep networks </span>

Style transfer can be formulated as generating a stylized image <span style="color:DodgerBlue">$p$</span> which its content is similar to a given content image <span style="color:DodgerBlue">$c$</span> and its style is close to another given style image <span style="color:DodgerBlue">$s$</span>.

<span style="color:DodgerBlue">
\\[
p= \Psi (c,s)
\\] </span>

The similarity in **style** can be vaguely defined as sharing the same spatial statistics in low-level features of a network (use VGG-16 in this paper), while similarity in **content** is roughly having a close Eculidean distance in high-level features.  
The main idea is that the features obtained by the network contain information about the content of the input image while the corrleation between these features represents its style.

In order to increase the similarity of between two images, minimize the following distnaces between their extracted features:

<span style="color:DodgerBlue">
\\[
\\begin{array}{l}
L^l_c (p) = \Vert \phi^l (p) - \phi^l (s) \Vert^2_2 \quad \cdots Eq .(1) \cr
L^l_c (p) = \Vert G(\phi^l (p)) - G(\phi^l (s)) \Vert^2_F \quad \cdots Eq .(2)
\\end{array}
\\]
</span>

where
- <span style="color:DodgerBlue">$\phi^l (x) $</span>: Activation of a pre-trained network at layer <span style="color:DodgerBlue">$l$</span>.
	- <span style="color:DodgerBlue">$x$</span>: Given the input image. 
- <span style="color:DodgerBlue">$L^l_c (p) and L^l_c (p) $</span>: Content and style loss at layer <span style="color:DodgerBlue">$l $</span> respectively.
- <span style="color:DodgerBlue">$ G(\phi^l (s)) $</span>: Gram matrix associated with <span style="color:DodgerBlue">$ \phi^l (p) $</span>.
	- Gram matrix <span style="color:DodgerBlue">$G^l_\{ij\} = \sum_k \phi^l_\{ik\}\phi^l_\{jk\} $</span>: Variance of RGB between image textures.


The total loss is calculated as a weighted sum of losses a set of *content layers* <span style="color:DodgerBlue">$C$</span> and *style layers* <span style="color:DodgerBlue">$S$</span>:


<span style="color:DodgerBlue">
\\[
L_c (p)= \sum_\{l \in C \} w^l_c L^l_c (p), \; L_s (p) = \sum_\{l \in S \} w^l_s L^l_s(p) \quad \cdots Eq .(3)
\\] </span>

where  <span style="color:DodgerBlue">$w^l_c, w^l_s$</span> are hyper-parameters to adjust the contribution of each layer to the loss. The problem is that these hyper-paremeters have to be manually fine-tuned through try and error.  
Finally, the objective of style transfer can be defined as:

<span style="color:DodgerBlue">
\\[
min_p (L_c (p) + L_s (p)) \quad \cdots Eq .(4)
\\] </span>


### <span style="color:gray"> 2.2 Real-time feed-forward style transfer </span>

We can solve the objective in Eq. (4) using iterative method but it can be very slow and has to be repeated for any given input.  
A much faster method is to directly train a deep network <span style="color:DodgerBlue">$T$</span> which maps a given content image <span style="color:DodgerBlue">$c$</span> to a stylized image <span style="color:DodgerBlue">$p$</span>. <span style="color:DodgerBlue">$T$</span> is a feed-forward CNN (parameterized by <span style="color:DodgerBlue">$\theta$</span>) with residual connections between down-sampling and up-sampling layers and is trained on content images like as:


<span style="color:DodgerBlue">
\\[
min_\theta (L_c (T(c))+L_s (T(c))) \quad \cdots Eq .(5)
\\] </span>

The second problem from this is that this generates only one stylization for a pair of style and content images.


## 3. Proposed Method

![2](https://da2so.github.io/assets/post_img/2020-12-11-Adjustable_Real_time_Style_Transfer/2.png){: .mx-auto.d-block width="100%" :}

To address the two issues that are mentioned in above, the authors condition the generated stylized image on additional input parameters where each parameter controls the share of the loss from a corresponding layer.

As for figural style, they enable the users to adjust <span style="color:DodgerBlue">$w^l_c, w^l_s $</span> without retraining the model by replacing them with input parameters and conditioning the generated style images on these parameters:

<span style="color:DodgerBlue">
\\[
p = \Phi (c,s,\alpha_c \alpha_s) 
\\] </span>

<span style="color:DodgerBlue">$ \alpha_c $</span> and <span style="color:DodgerBlue">$ \alpha_s $</span> are vectors of parameters where each element corresponds to a different layer in content layers <span style="color:DodgerBlue">$C$</span> and style layers <span style="color:DodgerBlue">$ S$</span> respectively. <span style="color:DodgerBlue">$ \alpha^l_c $</span> and <span style="color:DodgerBlue">$ \alpha^l_s $</span> replace the hyper-parameters <span style="color:DodgerBlue">$ w^l_c $</span> and <span style="color:DodgerBlue">$ w^l_s $</span>:     

<span style="color:DodgerBlue">
\\[
L_c (p)= \sum_\{ l \in C \} \alpha^l_c L^l_c (p), \; L_s (p) = \sum_\{ l \in S \} \alpha^l_s L^l_s (p) \quad \cdots Eq .(6)

\\] </span>

To learn the effect of <span style="color:DodgerBlue">$ \alpha_c $</span> and <span style="color:DodgerBlue">$ \alpha_s $</span> on the objective, the authors use a technique called **conditional instance normalization**.  
This method transforms the activations of a layer <span style="color:DodgerBlue">$ x $</span> in the <span style="color:DodgerBlue">$T$</span> to a normalized activation <span style="color:DodgerBlue">$ z $</span> which is conditioned on additional inputs <span style="color:DodgerBlue">$ \alpha_= [ \alpha_c , \alpha_s] $</span>:


<span style="color:DodgerBlue">
\\[
z= \gamma_\{ \alpha \} ( \frac{x- \mu}{ \sigma }) + \beta_\{ \alpha \} \quad \cdots Eq .(7)
\\] </span>

where <span style="color:DodgerBlue">$ \mu $</span> and <span style="color:DodgerBlue">$ \sigma $</span> are mean and standard deviation of activations at layer <span style="color:DodgerBlue">$ x $</span> across spatial axes and <span style="color:DodgerBlue">$ \gamma_\{ alpha \} $</span> , <span style="color:DodgerBlue">$ \beta_\{ \alpha \} $</span> are the trained mean and standard deviation this transformation. These parameters can be approximated using a second neural network (in here, MLP) which will be trained end-to-end with <span style="color:DodgerBlue">$ T$</span>:

<span style="color:DodgerBlue">
\\[
 \gamma_\{ \alpha \} , \beta_\{ \alpha \} = \Lambda ( \alpha_c, \alpha_s) \quad \cdots Eq .(8)
\\] </span>

Since <span style="color:DodgerBlue">$ L^l $</span> can be different in scale, do normalize them using their exponential moving average as a normalizing factor, i.e. each <span style="color:DodgerBlue">$ L^l $</span> will be normalized to:

<span style="color:DodgerBlue">
\\[
L^l (p) = \frac{ \sum_\{ i \in C \cup S \} \bar{L^i} (p) }{\bar{L^l} (p)} \quad \cdots Eq .(9)
\\] </span>

where <span style="color:DodgerBlue">$ \bar{L^l} (p) $</span> is the exponential moving average of <span style="color:DodgerBlue">$ L^l (p) $</span>.

## 4. Experiment

![2](https://da2so.github.io/assets/post_img/2020-12-11-Adjustable_Real_time_Style_Transfer/3.png){: .mx-auto.d-block width="100%" :}