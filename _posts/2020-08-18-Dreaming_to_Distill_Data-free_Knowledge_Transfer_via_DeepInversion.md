---
layout: post
title: Dreaming to Distill Data-free Knowledge Transfer via DeepInversion
tags: [Knowledge distillation, Data-free, Synthesized image]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-08-18-Dreaming_to_Distill_Data-free_Knowledge_Transfer_via_DeepInversion/post.png
---

## 1. Goal

The goal is to peform **Data-free Knowledge distillation**.

{: .box-note}
**Knowledge distillation:** Dealing with the problem of training a smaller model (Student) from a high capacity source model (Teacher) so as to retain most of its performance.

As the word itself, We perform knowledge distillation when there are no original dataset on which the Teacher network has been trained. It is because, in real world, most datasets are proprietary and not shared publicly due to privacy or confidentiality concerns. 

To tackle this problem, it is a necessary to reconstruct a dataset for training Student network. Thus, in this paper, we prospose a new method, which synthesizes images from the image distribution used to train a deep neural network. Further, we aim to improve the diversity of synthesized images.

## 2. Method

* Proposing *DeepInversion*, a new method for synthesizing class-conditional images from a CNN trained for iamge classification.
	* Introducing a regularization term for intermediate layer activations of synthesized images based on just the two layer-wise statistics: mean and variance from teacher network.

<br />
* Improving synthesis diversity via application-specific extension of *DeepInversion*, called *Adaptive DeepInversion*.
	* Exploiting disagreements between the pretrained teacher and the in-training student network to expand the coverage of the training set.


The overall procedure of our method is described in Fig. 1.

![2](https://da2so.github.io/assets/post_img/2020-08-18-Dreaming_to_Distill_Data-free_Knowledge_Transfer_via_DeepInversion/1.png){: .mx-auto.d-block :}

### <span style="color:gray">2.1 Background </span>

<br />

#### <span style="color:gray">2.1.1 Knowledge distillation </span>

Given a trained model <span style="color:DodgerBlue">$p_T$</span> and a dataset <span style="color:DodgerBlue">$\mathcal{X}$</span>, the parameters of the student model, <span style="color:DodgerBlue">$W_S$</span>, can be learned by 


<span style="color:DodgerBlue">
\\[
min_{\, W_S} \sum_\{ x \in \mathcal{X} \} KL( p_T(x), p_S(x) ), \quad \cdots Eq. (1)
\\]
</span>

where <span style="color:DodgerBlue">$KL( \cdot )$</span> refers to the Kullback-Leibler divergence and <span style="color:DodgerBlue">$p_T(x)= p(x, W_T)$</span> and <span style="color:DodgerBlue">$p_S(x)=p(x, W_S)$</span> are the output distributions produced by the teacher and student model, respectively, typically obtained using a high temperature on the softmax.

#### <span style="color:gray">2.1.2 DeepDream </span>

DeepDream synthesize a large set of images <span style="color:DodgerBlue">$\widehat{x} \in \widehat{\mathcal{X}}$</span> from noise that could replace <span style="color:DodgerBlue">$x \in \mathcal{X}$</span>.

Given a randomly initialized input <span style="color:DodgerBlue">$\widehat{x} \in \mathcal{R}^{H \times W \times C} $</span> and an arbitrary target label <span style="color:DodgerBlue">$y$</span>, the image is synthesized by optimizing

<span style="color:DodgerBlue">
\\[
min_{\widehat{x}} L(\widehat{x},y) + \mathcal{R} ( \widehat{x} ), \quad \cdots Eq. (2)
\\]
</span>

where <span style="color:DodgerBlue">$L(\cdot)$</span> is a classification loss (*e.g.* cross-entropy), and <span style="color:DodgerBlue">$\mathcal{R} ( \cdot ) $</span> is an image reularization term, which steers <span style="color:DodgerBlue">$\widehat{x}$</span> away from unrealistic images wit no discernible visual information:

<span style="color:DodgerBlue">
\\[
\mathcal{R}_p \( \widehat{x} \) = \alpha_T \mathcal{R}_T (\widehat{x}) + \alpha_L \mathcal{R}_L (\widehat{x}), \quad \cdots Eq. (3)
\\]
</span>

where <span style="color:DodgerBlue">$\mathcal{R}_T$</span> and <span style="color:DodgerBlue">$\mathcal{R}_L$</span> penalize the total variance and $l_2$ norm of <span style="color:DodgerBlue">$\widehat{x}$</span>.



### <span style="color:gray">2.2 DeepInversion (DI) </span>


We improve DeepDream's image quality by extending image regularization  <span style="color:DodgerBlue">$\mathcal{R} (\widehat{x}) $</span> with a new feature distribution regularization term.

To effectively enforce feature similarities between <span style="color:DodgerBlue">$x$</span> and  <span style="color:DodgerBlue">$\widehat{x}$</span> at all levels (layers), we propose to minimize the distance between feature map statistics for <span style="color:DodgerBlue">$x$</span> and  <span style="color:DodgerBlue">$\widehat{x}$</span>. We assume that feature statistics follow the Gaussian distribution across batches and then can be defined by mean  <span style="color:DodgerBlue">$\mu$</span> and variance  <span style="color:DodgerBlue">$\sigma^2$</span>. Therefore , the *feature distribution regularization* term can be formulated as:

<span style="color:DodgerBlue">
\\[
\mathcal{R}_{feature} (\widehat{x}) = \sum_l \Vert \mu_l (\widehat{x}) - \mathbb{E}(\mu_l (x) \| \mathcal{X}) \Vert_2 +\sum_l \Vert \sigma_l (\widehat{x}) - \mathbb{E}(\sigma_l (x) \| \mathcal{X}) \Vert_2, \quad \cdots Eq. (4)
\\]
</span>


where <span style="color:DodgerBlue">$\mu_l (\widehat{x}) $</span> and <span style="color:DodgerBlue">$\sigma^2_l (\widehat{x}) $</span> are the batch-wise mean and variance estimates of feature maps corresponding to the $l$-th convolutional layer. Obtaining <span style="color:DodgerBlue">$\mathbb{E} ( \mu_l (x) \| \mathcal{X} )$</span> and <span style="color:DodgerBlue">$\mathbb{E} ( \sigma^2_l (x) \| \mathcal{X} )$</span> is that we extract running average statistics stored in the widely-used BatchNorm (BN) layers. It implicitly captures the channel-wise means and variances during training, hence allows for estimation of the expectations in Eq. (4) by:

<span style="color:DodgerBlue">
\\[
\\begin{array}{l}
\mathbb{E} (\mu_l (x) \| \mathcal{X}) \simeq BN_l (running\\_mean), \quad \cdots Eq. (5) \cr
\mathbb{E} (\sigma^2_l (x) \| \mathcal{X}) \simeq BN_l (running\\_variance). \quad \cdots Eq. (6)
\\end{array}
\\]
</span>

We refer to this model inversion method as *DeepInversion*. <span style="color:DodgerBlue">$R(\cdot)$</span> can thus be expressed as 

<span style="color:DodgerBlue">
\\[
\mathcal{R}_D (\widehat{x}) = \mathcal{R}_p ( \widehat{x}) +\alpha_f \mathcal{R}_F (\widehat{x}). \quad \cdots Eq. (7)
\\]
</span>



### <span style="color:gray">2.3 Adaptive DeepInversion (ADI) </span>

Diversity also plays a crucial role in avoiding repeated and redundant synthetic images. For this, we propose *Adaptive DeepInversion*, an enhanced image generation scheme based on a iterative competition scheme between the image generation process and the student network. The main idea is to encourage the synthesized images to cause student-teacher disagreement.

Then, we introduce an additional loss <span style="color:DodgerBlue">$\mathcal{R}_{c}$</span> for image generation based on the Jensen-Shannon divergenece that penalizes output distribution similarities,


<span style="color:DodgerBlue">
\\[
\\begin{array}{l}
\mathcal{R}_{c} (\widehat{x}) = 1- JS(p_T(\widehat{x}), p_S (\widehat{x})), \cr
JS(p_T(\widehat{x}), p_S (\widehat{x}))= \frac{1}{2} ( KL (p_T (\widehat{x}),M)+KL (p_S (\widehat{x}),M)), 
\\end{array} \quad \cdots Eq. (8)
\\]
</span>

where <span style="color:DodgerBlue">$M=\frac{1}{2} \cdot ( p_T (\widehat{x} )+p_S (\widehat{x})) $</span> is the average of the teacher and student distributions.

During optimization, this new term leads to new images the student cannot easily classify whereas the teacher can. As illustrated in Fig 2. our proposal iteratively expands the distributional coverage of the image distribution during the learning process. The regularization <span style="color:DodgerBlue">$\mathcal{R} ( \cdot )$</span> from Eq.(7) is updated with an additional loss scaled by <span style="color:DodgerBlue">$\alpha_c$</span> as

<span style="color:DodgerBlue">
\\[
<span style="color:DodgerBlue">$\mathcal{R}_A (\widehat{x})= \mathcal{R}_D(\widehat{x}) +\mathcal{R}_c(\widehat{x})$</span> 
\\]
</span>


![2](https://da2so.github.io/assets/post_img/2020-08-18-Dreaming_to_Distill_Data-free_Knowledge_Transfer_via_DeepInversion/2.png){: .mx-auto.d-block :}