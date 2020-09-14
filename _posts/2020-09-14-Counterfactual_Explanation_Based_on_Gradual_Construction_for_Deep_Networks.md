---
layout: post
title: Counterfactual Explanation Based on Gradual Construction for Deep Networks
tags: [Explainable AI, Interpretability]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-09-14-Counterfactual_Explanation_Based_on_Gradual_Construction_for_Deep_Networks/post.PNG
---

## 1. Counterfactual Explanation


**Counterfactual Explanation:** Given an input data that are classified as a class from a deep network, it is to perturb the subset of features in the input data such that the model is
forced to predict the perturbed data as a target class.  
The Framework for counterfactual explanation is described in Fig 1. 


![2](https://da2so.github.io/assets/post_img/2020-09-14-Counterfactual_Explanation_Based_on_Gradual_Construction_for_Deep_Networks/1.png){: .mx-auto.d-block width="90%" :}

From perturbed data, we can **interpret** that the pre-trained model thinks the the perturbed parts(regions) as the discriminative features between the original and target classes, such as Fig 2. 

![2](https://da2so.github.io/assets/post_img/2020-09-14-Counterfactual_Explanation_Based_on_Gradual_Construction_for_Deep_Networks/2.png){: .mx-auto.d-block width="90%" :}


For this, the perturbed data for counterfactual explanation should satisfy two desirable properties.

1. **Explainability**
	- A generated explanation should be naturally understood by humans.
2. **Minimality**
	- Only a few features should be pertrubed.


## 2. Method

To generate counterfactual explanations, we propose a counterfactual explanation method based on gradual construction that considers the statistics learned from training data. We particularly generate counterfactual explanation by iterating over masking and composition steps.

![2](https://da2so.github.io/assets/post_img/2020-09-14-Counterfactual_Explanation_Based_on_Gradual_Construction_for_Deep_Networks/3.png){: .mx-auto.d-block :}


### <span style="color:gray"> 2.1 Problem definition </span>


* Input (original) data: <span style="color:DodgerBlue">$X \in \mathbb{R}^d$</span>
	* Its predicted class : <span style="color:DodgerBlue">$c_0$</span> under a pre-trained model <span style="color:DodgerBlue">$f$</span>
* Perturbed data <span style="color:DodgerBlue">$X'=(1-M) \circ X + M \circ C$</span>
	* Binary mask: <span style="color:DodgerBlue">$M = \\{ 0,1 \\}^d$</span>
	* Composite: <span style="color:DodgerBlue">$C$</span>
* Target class: <span style="color:DodgerBlue">$c_t$</span>
* Desired classification score for the target class: <span style="color:DodgerBlue">$\tau$</span>

The mask <span style="color:DodgerBlue">$M$</span> indicates wheter to replace subset features of <span style="color:DodgerBlue">$X$</span> with the composite <span style="color:DodgerBlue">$C$</span> or to preserve the features of <span style="color:DodgerBlue">$X$</span>. The <span style="color:DodgerBlue">$C$</span> represents newly generated feature values that will be replaced into a perturbed data <span style="color:DodgerBlue">$X'$</span>.


To prduce a perturbed data <span style="color:DodgerBlue">$X'$</span> whose prediction will be a target class <span style="color:DodgerBlue">$c_t$</span>, we progressively search for an optimal mask and a composite. To this end, our method builds gradual construction that iterates over the maskingand composition steps until the desired classification score <span style="color:DodgerBlue">$\tau$</span> is obtained.


### <span style="color:gray"> 2.2 Masking step </span>

The goal of the masking step is to select the most influential feature to
produce a target class from a pre-trained network as follows:


