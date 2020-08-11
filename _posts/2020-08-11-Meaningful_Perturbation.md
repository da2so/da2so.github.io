---
layout: post
title: Interpretable Explanations of Black Boxes by Meaningful Perturbation
tags: [Explainable AI(XAI), Perturbation, Model Induction]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-08-11-Meaningful_Perturbation/post.png
---

## 1. How to explain the decision of black-box model??

<img src="https://da2so.github.io/assets/post_img/2020-08-11-Meaningful_Perturbation/1.png" width="400" height="270" style="float: left">

Given those things from left figure, we wonder that why the deep network predicts the image as "dog".

<br />

To gratify this curiosity, we aim to find <span style="background-color: #A4FF21">the important regions of an input image</span> to classify it as "dog" class.

\\[downarrow \text{The idea}\\]

If we find and remove <span style="background-color: #A4FF21">the regions</span>, the probability of the prediction significantly gets lower.


{: .box-note}
**Note:** In this paper, removing the region represents that the regions are replaced with reference data (e.g. blurred, noise).


![2](https://da2so.github.io/assets/post_img/2020-08-11-Meaningful_Perturbation/2.png){: .mx-auto.d-block :}

<p align=center><b>Fig 1. Input and result for explanations </b></p>


## 2. How to find the important regions to be classified as a target class?

### 2.1 Problem definition

* Input image: <span style="color:DodgerBlue">$x_0$</span>
* Black-box model: <span style="color:DodgerBlue">$f$</span>
* Target class: <span style="color:DodgerBlue">$c$</span>
* Prediction score for the target class: <span style="color:DodgerBlue">$f_c(x_0)$</span>

### 2.2 Methods

The goal is to find deletion (perturbation) regions that are maximally informative to the decision.


Let <span style="color:DodgerBlue">$m:\;\lambda \rightarrow [0,1]$</span> be a mask, associating each pixel <span style="color:DodgerBlue">$u \in \lambda$</span> with a scalar value <span style="color:DodgerBlue">$m(u)</span>. 

Then, the perturbation operator is defined as follows:

\\[[\Phi(x_0: m)](u)= \lef\{ \begin{array}{rcl} m(u)x_0(u)+(1-m(u))u_0, \quad \text{constant}, & m(u)x_0(u)+(1-m(u))\eta(u), \quad \text{noise}, & \int g^{\sigma_0 m(u)} (v-u)x_0(v)dv, \quad blur \end{array}\right \\]

,where <span style="color:DodgerBlue">$u_0$</span> is an average color, <span style="color:DodgerBlue">$\eta(u)$</span> are i.i.d Gaussian noise samples for each pixel and <span style="color:DodgerBlue">$\sigma_0$</span> is the maximum isotropic standard deviation of the Gaussian blur kernel <span style="color:DodgerBlue">$g_\sigma$</span>


