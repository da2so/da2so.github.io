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
**(i)** Most previous works have developed network structures for cross-scale feature fusion. However, they usally contribute to the fused output feature unequally.  
**(ii)** While previous works mainly rely on bigger backbone networks or larger input image sizes for higher accuracy, scaling up feature network and box/class prediction network is also critical when taking into account both accuracy and efficiency. 

### <span style="color:gray"> 1.2 Goal </span>

For (i), they propose **a weighted bi-directional feature pyramid network (BiFPN)**, which allows easy and fast multi-scale feature fusion. And, in order to solve (ii), they propose a **compound scaling method** that uniformly scales the resolution, depth, and width for all backbone, feature network, and box/class prediction networks at the same time.  
Then, they call object detector that have these optimization methods as **EfficientDet**.


## 2. BiFPN

First, they formulate the multi-scale feature fusion problem that aims to aggregate features at different resolutions. And then, introduce ideas of BiFPN.


### <span style="color:gray"> 2.1 Problem Formulation </span>

- <span style="color:DodgerBlue">$\vec{P}^\{ in \} = \( P^\{in\}_\{l_1\}$</span><span style="color:DodgerBlue">$,  P^\{in\}_\{l_2\} , \dots \)$</span>: A list of multi-scale features
	- <span style="color:DodgerBlue">$ P^\{ in \}_\{ l_i \}$</span>: The feature at level $l_i$
- <span style="color:DodgerBlue">$ f $</span>: Transformation
	- (1) which is effectively aggregates different features
	- (2) which outputs a list of new features <span style="color:DodgerBlue">$ \vec{P}^\{ out \} = f(\vec{P}^\{ in \}) $</span>


For example of a muti-scale feature fusion, FPN [*T. Y. Lin et al, 2017*] has the conventional top-down pathway. The way of fusion describes the below figure.

![2](https://da2so.github.io/assets/post_img/2021-01-30-EfficientDet_Scalable_and_Efficient_Object_Detection/1.PNG){: .mx-auto.d-block width="90%" :}

$Resize$ is usually a upsampling or downsampling op for resolution matching, and $Conv$ is a convolutional op for feature processing.


### <span style="color:gray"> 2.2 Cross-Scale Connections </span>

![2](https://da2so.github.io/assets/post_img/2021-01-30-EfficientDet_Scalable_and_Efficient_Object_Detection/2.png){: .mx-auto.d-block width="90%" :}

Above figure represents multi-scale feature fusion method of previous approaches.  
The problem of existing methods for feature fusion is: 
- **FPN**: one-way information flow
- **PANet**: better accuracy, but high cost of more parameters and computations
- **NAS-FPN**: requring thousands of GPU hours

To improve model efficiency, this paper propose several optimizations for cross-scale connections:

1. Remove those nodes that only have one input edge.
	- Why?
		- One input edge with no feature fusion has less contribution to feature network.
		- This leads a simplified bi-directional network.

2. Add an extra edge from the original input to output node if they're at the same level.
	- Why?
		- This makes to fuse more features without adding much cost.

3. Treat each bidirectional path as one feature network and repeat the same layer multiple times.
	- Why?
		- ...


### <span style="color:gray"> 3.2 Weighted Feature Fusion </span>

When fusing features with different resolutions, a common way is to first resize them to the same resolution and then sum them up. At this point,
all previous methods treat all input features equally without distinction. 

In this paper, in order to contribute to the ouput feature unequally from the input feature, the authors add an additional weight for each input and let the network to learn the importance of each input feature.

####  <span style="color:gray"> Fast normalized fusion </span>


<span style="color:DodgerBlue">
\\[
O = \sum_i \frac{w_i}{\epsilon + \sum_j w_j } \cdot I_i
\\] 
</span>


,where <span style="color:DodgerBlue">$ w_i $</span> is a learnable weight and <span style="color:DodgerBlue">$ w_i \geq 0 $</span> is ensured by appling a Relu after each <span style="color:DodgerBlue">$ w_i $</span>, and <span style="color:DodgerBlue">$ \epsilon = 0.0001 $</span> is a small value to avoid numerical instability. And the value of each normalized weight falls between 0 and 1.

The final BiFPN integrates both the bidirectional cross scale connections and the fast normalized fusion. As a concrete example, they describe the two fused features at level **6** for BiFPN shown in Fig 2 (d):


<span style="color:DodgerBlue">
\\[
\\begin{array}{l} P^ = m^\ast_\{ c_T \} \cdot x, \cr
				  m^\ast_\{ c_T \}= argmin_\{ m_\{ c_T \} \} \[ \varphi( y^{c_T}_x, y^{c_T}_e ) +\lambda \cdot R_m \] . 
\\end{array} 
</span>


![2](https://da2so.github.io/assets/post_img/2021-01-30-EfficientDet_Scalable_and_Efficient_Object_Detection/3.png){: .mx-auto.d-block width="90%" :}












