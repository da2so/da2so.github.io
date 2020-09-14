---
layout: post
title: Counterfactual Explanation Based on Gradual Construction for Deep Networks
tags: [Explainable AI, Interpretability]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-09-14-Counterfactual_Explanation_Based_on_Gradual_Construction_for_Deep_Networks/post.PNG
---

## 1. Counterfactual Explanation </span>

{: .box-note}
**Counterfactual Explanation:** Given an input data that are classified as a class from a deep network, it is to perturb the subset of features in the input data such that the model is
forced to predict the perturbed data as a target class.

The Framework for counterfactual explanation is described in Fig 1. 


![2](https://da2so.github.io/assets/post_img/2020-09-14-Counterfactual_Explanation_Based_on_Gradual_Construction_for_Deep_Networks/1.png){: .mx-auto.d-block :}

From perturbed data, we can **interpret** that the pre-trained model thinks the the perturbed parts(regions) as the discriminative features between the original and target classes, such as Fig 2. 

![2](https://da2so.github.io/assets/post_img/2020-09-14-Counterfactual_Explanation_Based_on_Gradual_Construction_for_Deep_Networks/2.png){: .mx-auto.d-block :}


For this, the perturbed data for counterfactual explanation should satisfy two desirable properties.

1. **Explainability**
	- A generated explanation should be naturally understood by humans.
2. **Minimality**
	- Only a few features should be pertrubed.


## 2. Method

To generate counterfactual explanations, we propose a counterfactual explanation method based on gradual construction that considers the statistics learned from training data. We particularly generate counterfactual explanation by iterating over masking and composition steps.

![2](https://da2so.github.io/assets/post_img/2020-09-14-Counterfactual_Explanation_Based_on_Gradual_Construction_for_Deep_Networks/3.png){: .mx-auto.d-block :}


### <span style="color:gray"> 2.1 Problem definition </span>