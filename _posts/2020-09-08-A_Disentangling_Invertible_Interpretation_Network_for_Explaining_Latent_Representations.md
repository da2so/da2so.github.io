---
layout: post
title: A Disentangling Invertible Interpretation Network for Explaining Latent Representations
tags: [Explainable AI, Interpretability]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-09-08-A_Disentangling_Invertible_Interpretation_Network_for_Explaining_Latent_Representations/post.PNG
---

## 1. Interpreting hidden representations

### <span style="color:gray"> 1.1 Invertible transformation of hidden representations </span>


![2](https://da2so.github.io/assets/post_img/2020-09-08-A_Disentangling_Invertible_Interpretation_Network_for_Explaining_Latent_Representations/1.png){: .mx-auto.d-block width="80%" :}

- Input image: <span style="color:DodgerBlue">$x \in \mathbb{R}^{H \times W \times 3}$</span>
- Sub-network of <span style="color:DodgerBlue">$f$</span> including hidden layers: <span style="color:DodgerBlue">$E$</span>
- Latent (original) representation: <span style="color:DodgerBlue">$z=\mathbb{E} (x) \in \mathbb{R}^{H \times W \times 3 = N}$</span> 
- Sub-network after the hidden layer: <span style="color:DodgerBlue">$G$</span>
	- <span style="color:DodgerBlue">$f(x)=G \cdot E(x)$</span>


The goal is to translate an original representation <span style="color:DodgerBlue">$z$</span> to an equivalent yet interpretable one. 

In order to <span style="color:DodgerBlue">$z$</span> into an interpretable representation, the following is introduced.

![2](https://da2so.github.io/assets/post_img/2020-09-08-A_Disentangling_Invertible_Interpretation_Network_for_Explaining_Latent_Representations/2.png){: .mx-auto.d-block width="90%" :}



So, what can we obtain from interpretable representations??

<span style="color:#CC16CF">**(i)**</span> We can comprehend the meaning of the latent variable from the value of interpretable representation and get more natural images from GAN.

![2](https://da2so.github.io/assets/post_img/2020-09-08-A_Disentangling_Invertible_Interpretation_Network_for_Explaining_Latent_Representations/3.png){: .mx-auto.d-block width="80%" :}

<span style="color:#CC16CF">**(ii)**</span> Semantic image modifications and embeddings. As we manipulate the semantic factor (=interpretable representation) for digit, we can get '3' digit image from '9' digit image.

![2](https://da2so.github.io/assets/post_img/2020-09-08-A_Disentangling_Invertible_Interpretation_Network_for_Explaining_Latent_Representations/4.png){: .mx-auto.d-block width="80%" :}



### <span style="color:gray"> 1.2 Distangling interpretable concepts </span>

What properties of <span style="color:DodgerBlue">$\widetilde{z}_k$</span> we should satisfy??

<span style="color:#BBAE31">**A.**</span> It must represent a specific interpretable concepts. (ex: color, class ...)  
<span style="color:#BBAE31">**B.**</span> It is independent to each other.  
     <span style="color:DodgerBlue">$\quad \Rightarrow P(\widetilde{z})=\Pi^K P(\widetilde{z}_k) $</span>  
<span style="color:#BBAE31">**C.**</span> Distribution <span style="color:DodgerBlue">$P(z_k)$</span> of each factor must be easy to sample from to gain insights into variability of a factor.  
<span style="color:#BBAE31">**D.**</span> Interpolation between two samples of a factor must be valid samples to analyze changes along a path


Satisfy <span style="color:#BBAE31">**C.**</span> and <span style="color:#BBAE31">**D.**</span> <span style="color:black">$\Rightarrow$</span> <span style="color:DodgerBlue">$P(\widetilde{z})=\Pi^K N(P(\widetilde{z}_k | 0,1) \quad \cdots Eq. (1)$</span>



**The way to supply additional constraints for Eq. (1).**

Let there be training image pairs <span style="color:DodgerBlue">$(x^a x^b)$</span> which specify semantics through their similarity.  
Each semantic concept <span style="color:DodgerBlue">$F \in \\{ 1, \cdots , K \\}$</span> defined by such pairs is represented by corresponding factor <span style="color:DodgerBlue">$\widetilde{z}_F$</span> and we write <span style="color:DodgerBlue">$(x^a, x^b) ~ P(x^a,x^b|F)$</span> to emphasize that <span style="color:DodgerBlue">$(x^a,x^b)$</span> is a training pair for factor <span style="color:DodgerBlue">$\widetilde{z}_F$</span>.


However, we cannot expect to have examples of image pairs for every semantic concept relevant in <span style="color:DodgerBlue">$z$</span>.
<span style="color:gray">$\quad \Rightarrow $</span> So, let introduce <span style="color:DodgerBlue">$z_0$</span> to act as a residual concept


For a given training pair <span style="color:DodgerBlue">$(x^a, x^b)$</span> factorized representation <span style="color:DodgerBlue">$\widetilde{z}^a=T(X(x^a))$</span> and <span style="color:DodgerBlue">$\widetilde{z}^b=T(X(x^b))$</span> must <span style="color:#95F056">$(i)$</span> mirror the semantic similarity of <span style="color:DodgerBlue">$(x^a, x^b)$</span> in its  <span style="color:DodgerBlue">$F$</span>-th factor and <span style="color:#95F056">$(ii)$</span> be invariant in the remaining factors.


<span style="color:#95F056">$\quad (i)$</span> a positive correlation factor <span style="color:DodgerBlue">$\sigma_{ab} \in (0,1)$</span> for the <span style="color:DodgerBlue">$F$</span>-th factors between pairs  
<span style="color:gray">$\quad \quad \Rightarrow$</span> <span style="color:DodgerBlue">$widetilde{z}^b_F ~ N (widetilde{z}^b_F | \sigma_{ab} widetilde{z}^a_F , (1- \sigma_\{ab\} ) \mathbf{1} ) \quad \cdots Eq. (2)$</span>

<span style="color:#95F056">$\quad (ii)$</span> no correlation for the remaining factors between pairs  
<span style="color:gray">$\quad \quad \Rightarrow$</span> <span style="color:DodgerBlue">$widetilde{z}^b_k ~ N (widetilde{z}^b_k | 0,1  ) \; k \in \\{ 0, \cdots , K \\} \\\ \\{ F \\} \quad \cdots Eq. (3)$</span>