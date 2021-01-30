---
layout: post
title: EfficientDet Scalable and Efficient Object Detection
tags: [Detection, EfficientNet]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2021-01-30-EfficientDet_Scalable_and_Efficient_Object_Detection/post.PNG
---

## 1. Introduction   
### <span style="color:gray"> 1.1 Motivation </span>

The existing methods for object detection mainly have two problems.  
**(i)** Most previous works have developed network structures for cross-scale feature fusion. However, they usally contribute to the fused output feature unequally.  **(ii)** While previous works mainly rely on bigger backbone networks or larger input image sizes for higher accuracy, scaling up feature network and box/class prediction network is also critical when taking into account both accuracy and efficiency. 

### <span style="color:gray"> 1.2 Goal </span>

For (i), they propose **a weighted bi-directional feature pyramid network (BiFPN)**, which allows easy and fast multi-scale feature fusion. And, in order to solve (ii), they propose a **compound scaling method** that uniformly scales the resolution, depth, and width for all backbone, feature network, and box/class prediction networks at the same time.  
Then, they call object detector that have these optimization methods as **EfficientDet**.


## 2. BiFPN

### <span style="color:gray"> 2.1 Problem Formulation </span>

- <span style="color:DodgerBlue">$\vec{P}^\{ in \} = \( P^\{in\}_\{l_1\} ,  P^\{in\}_\{l_2\} , \dots \)$</span>: A list of multi-scale features
	- <span style="color:DodgerBlue">$ P^\{ in \}_\{ l_i \}$</span>: The feature at level $l_i$
- <span style="color:DodgerBlue">$ f $</span>: Transformation
	- (1) which is effectively aggregates different features
	- (2) which outputs a list of new features <span style="color:DodgerBlue">$ \vec{P}^\{ out \} = f(\vec{P}^\{ in \}) $</span>
