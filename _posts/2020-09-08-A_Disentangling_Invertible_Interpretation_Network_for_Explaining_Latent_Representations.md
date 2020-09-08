---
layout: post
title: A Disentangling Invertible Interpretation Network for Explaining Latent Representations
tags: [Explainable AI, Interpretability]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-09-08-A_Disentangling_Invertible_Interpretation_Network_for_Explaining_Latent_Representations/post.png
---

## 1. Interpreting hidden representations

### <span style="color:gray"> 1.1 Invertible transformation of hidden representations </span>


![2](https://da2so.github.io/assets/post_img/2020-09-08-A_Disentangling_Invertible_Interpretation_Network_for_Explaining_Latent_Representations/1.png){: .mx-auto.d-block width="80%" :}

- Input image: <span style="color:DodgerBlue">$x \in \mathbb{R}^{H \times W \times 3}$</span>
- Sub-network of <span style="color:DodgerBlue">$f$</span> including hidden layers: <span style="color:DodgerBlue">$E$</span>
- Latent (original) representation: <span style="color:DodgerBlue">$z=\mathbb{E} (x) \in \mathbb{R}^{H \times W \times 3 = N}$</span> 
- Sub-network after the hidden layer: <span style="color:DodgerBlue">$G$</span>
	- <span style="color:DodgerBlue">$f(x)=G \dot E(x)$</span>
The goal is to translate an original representation <span style="color:DodgerBlue">$z$</span> to an equivalent yet interpretable one. 

In order to <span style="color:DodgerBlue">$z$</span> into an interpretable representation, the following is introduced.

![2](https://da2so.github.io/assets/post_img/2020-09-08-A_Disentangling_Invertible_Interpretation_Network_for_Explaining_Latent_Representations/2.png){: .mx-auto.d-block width="90%" :}
