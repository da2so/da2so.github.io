---
layout: post
title: Interpretable Explanations of Black Boxes by Meaningful Perturbation
tags: [Explainable AI(XAI), Perturbation, Model Induction]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-08-11-Meaningful_Perturbation/post.png
---

## 1. How to explain the decision of black-box model??

<img src="https://da2so.github.io/assets/post_img/2020-08-11-Meaningful_Perturbation/1.png" width="370" height="230" style="float: left">

Given the left figure, we wonder that why the deep network predicts the image as "dog".


To gratify this curiosity, we aim to find <span style="background-color: #A4FF21">the important regions of an input image</span> to classify it as "dog" class.

<span style="color: red">\\[\downarrow \text{The idea}\\] </span>

If we find and remove <span style="background-color: #A4FF21">the regions</span>, the probability of the prediction significantly gets lower.


{: .box-note}
**Note:** In this paper, removing the region represents that the regions are replaced with reference data (e.g. blurred, noise).


![2](https://da2so.github.io/assets/post_img/2020-08-11-Meaningful_Perturbation/2.png){: .mx-auto.d-block :}

<p align=center><b>Fig 1. Input and result for explanations </b></p>


## 2. How to find the important regions to be classified as a target class?


### <span style="color:gray">2.1 Problem definition </span>

* Input image: <span style="color:DodgerBlue">$x_0$</span>
* Black-box model: <span style="color:DodgerBlue">$f$</span>
* Target class: <span style="color:DodgerBlue">$c$</span>
* Prediction score for the target class: <span style="color:DodgerBlue">$f_c(x_0)$</span>


### <span style="color:gray">2.2 Methods </span>  
The goal is to find deletion (perturbation) regions that are maximally informative to the decision.


Let <span style="color:DodgerBlue">$m:\lambda \rightarrow [0,1]$</span> be a mask, associating each pixel <span style="color:DodgerBlue">$u \in \lambda$</span> with a scalar value <span style="color:DodgerBlue">$m(u)$</span>.
Then, the perturbation operator is defined as follows:

\\[ 
x = \left\{ \\begin{array}{ll}
         x & \mbox{if $x \geq 0$};\cr
        -x & \mbox{if $x < 0$}.
        \\end{array} \right. 
\\] 

\\[
\\begin{array}{cc}
  a & b \cr
  c & c
\\end{array}
\\]
,where <span style="color:DodgerBlue">$u_0$</span> is an average color, <span style="color:DodgerBlue">$\eta(u)$</span> are i.i.d Gaussian noise samples for each pixel and <span style="color:DodgerBlue">$\sigma_0$</span> is the maximum isotropic standard deviation of the Gaussian blur kernel <span style="color:DodgerBlue">$g_\sigma$</span>.


| If <span style="color:DodgerBlue">$m(u)=1 \quad $</span> $\rightarrow$ Preserve the original pixel |
| elif <span style="color:DodgerBlue">$m(u)=0 \quad $</span> $\rightarrow$ Replace the original pixel with a pixel of reference data|


<br />

#### <span style="color:gray"> 2.2.1 The objective function </span>
Find the smallest deletion mask <span style="color:DodgerBlue">$m$</span> that causes the score <span style="color:DodgerBlue">$f_c(\Phi (x_0:m)) \ll f_c(x_0)$</span> to drop significantly, where <span style="color:DodgerBlue">$c$</span> is the target class.

<span style="color:DodgerBlue">\\[ m^*= argmin_{m \in [0,1]^\Lambda} \lambda \Vert 1-m\Vert_1+ f_c(\Phi(x_0:m))\\] </span>

,where <span style="color:DodgerBlue">$\lambda$</span> encourages most of the mask to be turned off.


#### <span style="color:gray">2.2.2 Dealing with artifacts</span>  
By committing to finding a single representative perturbation, our approach incurs the risk of triggering artifacts of the black-box model, like below figures.

<img src="https://da2so.github.io/assets/post_img/2020-08-11-Meaningful_Perturbation/3.png" width="300" height="170" style="float: left">
<br />
<br />
<br />
$\rightarrow$ Explanations containing artifacts

<br />
<br />
To solve this problem, we suggest two approaches in generating explanations.


<span style="color:#5256BC"><b>First, generalization for the mask</b></span>  
This means not relying on the details of a singly-learned mask <span style="color:DodgerBlue">$m$</span>. Hence, we reformulate the problem to apply the mask <span style="color:DodgerBlue">$m$</span> stochastically, up to small random jitter.


<span style="color:#5256BC"><b>Second, Total Variation (TV) regularization and upsampling</b></span>  
By uisng TV regularization and upsampling mask, we can encourage the result to have a simple, regular structure which can not be co-adapted to artifacts.

With these two modifications, the final objective function is follows:

<span style="color:DodgerBlue">\\[min_{m \in [0,1]^\Lambda} \lambda_1 \Vert 1-m \Vert_1 + \lambda_2 \sum_u \Vert \bigtriangledown m(u) \Vert^\beta_\beta + \mathbb{E}_\tau \[ f_c(\Phi (x( \cdot -\tau), m)) \] \\] </span>

,where second term represents TV regularization and third term indicates Generalization for the mask.


## 3. Implement details

<img src="https://da2so.github.io/assets/post_img/2020-08-11-Meaningful_Perturbation/4.png" width="400" height="230" style="float: left">

<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />

### <span style="color:gray">3.1 Procedure of the upsampling method in the method</span>


![3](https://da2so.github.io/assets/post_img/2020-08-11-Meaningful_Perturbation/5.png){: .mx-auto.d-block :}
