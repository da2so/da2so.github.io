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


![CNN](https://da2so.github.io/assets/post_img/2020-09-08-A_Disentangling_Invertible_Interpretation_Network_for_Explaining_Latent_Representations/1.png){: .mx-auto.d-block width="80%" :}

- Input image: <span style="color:DodgerBlue">$x \in \mathbb{R}^{H \times W \times 3}$</span>
- Sub-network of <span style="color:DodgerBlue">$f$</span> including hidden layers: <span style="color:DodgerBlue">$E$</span>
- Latent (original) representation: <span style="color:DodgerBlue">$z=\mathbb{E} (x) \in \mathbb{R}^{H \times W \times 3 = N}$</span> 
- Sub-network after the hidden layer: <span style="color:DodgerBlue">$G$</span>
	- <span style="color:DodgerBlue">$f(x)=G \cdot E(x)$</span>


In A Disentangling Invertible Interpretation Network for Explaining Latent Representations, the goal is to translate an original representation <span style="color:DodgerBlue">$z$</span> to an equivalent yet interpretable one. 

In order to <span style="color:DodgerBlue">$z$</span> into an interpretable representation, the following is introduced.

![interpretable_representation](https://da2so.github.io/assets/post_img/2020-09-08-A_Disentangling_Invertible_Interpretation_Network_for_Explaining_Latent_Representations/2.png){: .mx-auto.d-block width="90%" :}



So, what can we obtain from interpretable representations??

<span style="color:#CC16CF">**(i)**</span> We can comprehend the meaning of the latent variable from the value of interpretable representation and get more natural images from GAN.

![comparison_experiment](https://da2so.github.io/assets/post_img/2020-09-08-A_Disentangling_Invertible_Interpretation_Network_for_Explaining_Latent_Representations/3.png){: .mx-auto.d-block width="80%" :}

<span style="color:#CC16CF">**(ii)**</span> Semantic image modifications and embeddings. As we manipulate the semantic factor (=interpretable representation) for digit, we can get '3' digit image from '9' digit image.

![control_experiment](https://da2so.github.io/assets/post_img/2020-09-08-A_Disentangling_Invertible_Interpretation_Network_for_Explaining_Latent_Representations/4.png){: .mx-auto.d-block width="80%" :}



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


Each semantic concept <span style="color:DodgerBlue">$F \in \\{ 1, \cdots , K \\}$</span> defined by such pairs is represented by corresponding factor <span style="color:DodgerBlue">$\widetilde{z}_F$</span> and we write <span style="color:DodgerBlue">$(x^a, x^b) \sim P(x^a,x^b|F)$</span> to emphasize that <span style="color:DodgerBlue">$(x^a,x^b)$</span> is a training pair for factor <span style="color:DodgerBlue">$\widetilde{z}_F$</span>.


However, we cannot expect to have examples of image pairs for every semantic concept relevant in <span style="color:DodgerBlue">$z$</span>.  
<span style="color:gray">$\quad \Rightarrow $</span> So, let introduce <span style="color:DodgerBlue">$z_0$</span> to act as a residual concept


For a given training pair <span style="color:DodgerBlue">$(x^a, x^b)$</span> factorized representation <span style="color:DodgerBlue">$\tilde{z}^a=T(X(x^a))$</span> and <span style="color:DodgerBlue">$\tilde{z}^b=T(X(x^b))$</span> must <span style="color:#84BD5D">$(i)$</span> mirror the semantic similarity of <span style="color:DodgerBlue">$(x^a, x^b)$</span> in its  <span style="color:DodgerBlue">$F$</span>-th factor and <span style="color:#84BD5D">$(ii)$</span> be invariant in the remaining factors.


<span style="color:#84BD5D">$\quad (i)$</span> a positive correlation factor <span style="color:DodgerBlue">$\sigma_{ab} \in (0,1)$</span> for the <span style="color:DodgerBlue">$F$</span>-th factors between pairs  
<span style="color:gray">$\quad \quad \Rightarrow$</span> <span style="color:DodgerBlue">$\widetilde{z}^b_F \sim N (\widetilde{z}^b_F | \sigma_{ab} \widetilde{z}^a_F , (1- \sigma_\{ab\} ) \mathbf{1} ) \quad \cdots Eq. (2)$</span>

<span style="color:#84BD5D">$\quad (ii)$</span> no correlation for the remaining factors between pairs  
<span style="color:gray">$\quad \quad \Rightarrow$</span><span style="color:DodgerBlue">$\widetilde{z}^b_k \sim N (\widetilde{z}^b_k | 0,1  ) \; k \in \\{ 0, \cdots , K \\} \backslash \\{ F \\} \quad \cdots Eq. (3)$</span>


![equation](https://da2so.github.io/assets/post_img/2020-09-08-A_Disentangling_Invertible_Interpretation_Network_for_Explaining_Latent_Representations/5.png){: .mx-auto.d-block width="90%" :}


To fit this model to data, we utilize the invertibility of <span style="color:DodgerBlue">$T$</span> to directly compute and **maximize the likelihood <span style="color:DodgerBlue">$(z^a, z^b)=(E(x^a), E(x^b))$</span>.**

Compute the likelihood with the absolute value of the Jaccobian determinant of <span style="color:DodgerBlue">$T$</span>, denoted <span style="color:DodgerBlue">$T'(\cdot)$</span>, as


![equation_explanation1](https://da2so.github.io/assets/post_img/2020-09-08-A_Disentangling_Invertible_Interpretation_Network_for_Explaining_Latent_Representations/6.png){: .mx-auto.d-block width="90%" :}


Build <span style="color:DodgerBlue">$T$</span> based on ActNorm, AffineCoupling and suffling layers. (detailed in original paper...)


For training, we use negative log-likelihood as our loss function.
Subsitituting Eq. (1) into Eq. (4), Eq. (2) and (3) into Eq.(5) leads to the per-example loss <span style="color:DodgerBlue">$l(z^a,z^b |F)$</span>.


![equation_explanation2](https://da2so.github.io/assets/post_img/2020-09-08-A_Disentangling_Invertible_Interpretation_Network_for_Explaining_Latent_Representations/7.png){: .mx-auto.d-block width="90%" :}


The loss function is optimized over training pairs <span style="color:DodgerBlue">$(x^a, x^b)$</span> for all semantic concepts <span style="color:DodgerBlue">$F \in \\{1, \cdots , K \\}$</span>


<span style="color:DodgerBlue">
\\[
L=\sum^K_{F=1} E_\{ (x^a, x^b) \sim P(x^a,x^b |F) \} l(E(x^a), E(x^b) | F), \quad \cdots Eq. (9)
\\]
</span>

where  <span style="color:DodgerBlue">$x^a$</span> and  <span style="color:DodgerBlue">$x^b$</span> share at least one semantic factor.



## 2. Obtaining semantic concepts

### <span style="color:gray"> 2.1 Estimating dimensionality of factors </span>

Given image pairs <span style="color:DodgerBlue">$(x^a, x^b)$</span> that define the  <span style="color:DodgerBlue">$F$</span>-th semantic concept, we must estimate the dimensionality of factor  <span style="color:DodgerBlue">$\tilde{z}_F \in \mathbb{R}^{N_F}$</span>  
 

Semantic concepts captured by the network  <span style="color:DodgerBlue">$E$</span> require a larger share of the overall dimensionality than those  <span style="color:DodgerBlue">$E$</span> is invariant to.  
$\quad \cdot$ the similarity of <span style="color:DodgerBlue">$(x^a, x^b)$</span> in the <span style="color:DodgerBlue">$F$</span>-th semantic concept $\uparrow \Rightarrow$ dimensionality of  <span style="color:DodgerBlue">$\tilde{z}_F$</span> $\uparrow$


So, Approximate their mutual information with their correlation for each component <span style="color:DodgerBlue">$i$</span>.

Summing over all components <span style="color:DodgerBlue">$i$</span> yields a relative score <span style="color:DodgerBlue">$s_F$</span> that serves as proxy for the dimensionality of <span style="color:DodgerBlue">$\tilde{z}_F$</span> in case of training images <span style="color:DodgerBlue">$(x^a, x^b)$</span> for concepts <span style="color:DodgerBlue">$F$</span>.


<span style="color:DodgerBlue">
\\[
s_F=\sum_i \frac{Cov (E(x^a)_i, E(x^b)_i)}{\sqrt{Var(E(x^a)_i Var(E(x^b)_i))}}. \quad \cdots Eq. (10)
\\]
</span>


Since, correlation is in $[-1,1]$, scores <span style="color:DodgerBlue">$s_F$</span> are in <span style="color:DodgerBlue">$[-1 \times N, 1 \times N] = [-N, N]$</span> for <span style="color:DodgerBlue">$N$</span> dimensional latent representations of <span style="color:DodgerBlue">$E$</span>.

![dimensionality](https://da2so.github.io/assets/post_img/2020-09-08-A_Disentangling_Invertible_Interpretation_Network_for_Explaining_Latent_Representations/8.png){: .mx-auto.d-block width="100%" :}


### <span style="color:gray"> 2.2 Sketch-based description of semantic concepts </span>

<span style="color:gray">**Problem: **</span> Most often, a sufficiently large number of image pairs is not easy to obtain.


<span style="color:gray">**Solution: **</span> A user only has to provide two sketches, <span style="color:DodgerBlue">$y^a$</span> and <span style="color:DodgerBlue">$y^b$</span> which demonstrate a change in concept.

![sketch_example](https://da2so.github.io/assets/post_img/2020-09-08-A_Disentangling_Invertible_Interpretation_Network_for_Explaining_Latent_Representations/9.png){: .mx-auto.d-block width="100%" :}



### <span style="color:gray"> 2.3 Unsupervised Interpretations </span>


Even without examples for changes in semantic factors, our approach can still produce disentangled factors, In this case, we minimized the negative log-likelihood of the marginal distribution of hidden representations <span style="color:DodgerBlue">$z=E(x)$</span>:

<span style="color:DodgerBlue">
\\[
L_{unsup}= - E_x \Vert T(E(x)) \Vert^2 - log \vert T'(E(x)) \vert. \quad \cdots Eq. (11)
\\]
</span>

![linear_vs_nonlinear](https://da2so.github.io/assets/post_img/2020-09-08-A_Disentangling_Invertible_Interpretation_Network_for_Explaining_Latent_Representations/10.png){: .mx-auto.d-block width="90%" :}


<br />


### <span style="color:#C70039 ">Reference </span>
*Esser, Patrick, Robin Rombach, and Bjorn Ommer. "A disentangling invertible interpretation network for explaining latent representations." Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2020.*



