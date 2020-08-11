---
layout: post
title: Interpretable Explanations of Black Boxes by Meaningful Perturbation
tags: [Explainable AI(XAI), Perturbation, Model Induction]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-08-11-GradCAM/post.png
---

## 1. How to explain the decision of black-box model??

<img src="https://da2so.github.io/assets/post_img/2020-08-11-Meaningful_Perturbation/1.png" width="500" height="300" style="float: left">

Given those things from left figure, we wonder that why the deep network predicts the image as "dog".

<br />

To gratify this curiosity, we aim to find <span style="background-color: #A4FF21">the important regions of an input image</span> to classify it as "dog" class.

\\[downarrow \text{<span style="color:red">The idea</span>}\\]

If we find and remove <span style="background-color: #A4FF21">the regions</span>, the probability of the prediction significantly gets lower.


{: .box-note}
**Note:** In this paper, removing the region represents that the regions are replaced with reference data (e.g. blurred, noise)