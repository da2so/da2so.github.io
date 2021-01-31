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

![2](https://da2so.github.io/assets/post_img/2021-01-30-EfficientDet_Scalable_and_Efficient_Object_Detection/1.PNG){: .mx-auto.d-block width="100%" :}

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


### <span style="color:gray"> 2.3 Weighted Feature Fusion </span>

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


![2](https://da2so.github.io/assets/post_img/2021-01-30-EfficientDet_Scalable_and_Efficient_Object_Detection/3.png){: .mx-auto.d-block width="90%" :}


## 3. EfficientDet

Based on BiFPN, they developed a detection models named EfficientDet.


### <span style="color:gray"> 3.1 EfficientDet Architecture </span>


![2](https://da2so.github.io/assets/post_img/2021-01-30-EfficientDet_Scalable_and_Efficient_Object_Detection/4.png){: .mx-auto.d-block width="100%" :}


Fig 3. shows the overall architecture of EfficientDet, which is the one-stage detectors paradigm. They use ImageNet-pretrained EfficientNets as the backbone network. BiFPN
serves as the feature network and repeatedly applies top-down and bottom-up bidirectional feature fusion. These fused features are fed to a class and box network. The class and box network weights are shared across all levels of features.


### <span style="color:gray"> 3.2 Compound Scaling </span>

Aiming at optimizing both accuracy and efficiency, they would like to develop a family of models that can meet a wide spectrum of resource constraints. A key challenge here is how to scale up a baseline EfficientDet model.

Therefore, they propose a new compound scaling method for object detection, which uses a simple compound coefficient <span style="color:DodgerBlue">$\phi$</span> to jointly scale up all dimension of backbone, BiFPN, class/box network, and resolution.

####  <span style="color:gray"> Backbone network </span>

They reuse the same width/depth scaling coefficients of EfficientNet-B0 to B6.


####  <span style="color:gray"> BiFPN network </span>

They linearly increase BiFPN depth <span style="color:DodgerBlue">$D_\{ bifpn \}$</span> (#layer) since depth needs to be rounded to small integers. For BiFPN width <span style="color:DodgerBlue">$\W_\{bifpn\}$</span> (#channels), exponentially grow BiFPN width <span style="color:DodgerBlue">$\W_\{bifpn\}$</span> (#channels).

Specially, they perform a grid-search on a list of values {1.2, 1.25, 1.3, 1.35, 1.4, 1.45}, and pick the best value 1.35 as the BiFPN width scaling factor. Formally, BiFPN width and depth are scaled with the following equation:

<span style="color:DodgerBlue">
\\[
W_\{bifpn\} = 64 \cdot (1.35^\{ \phi \}), \quad D_\{bifpn\} = 3 + \phi  \quad \cdots (1)
\\] 
</span>


####  <span style="color:gray"> Box/class prediction network </span>

They fix their width to be always the same as BiPFN (i.e., $W_\{pred\} = W_\{ bifpn \}$), but linearly increase the depth (#layers) using equation:

<span style="color:DodgerBlue">
\\[
D_\{ box \} = D_\{class \} = 3 + \lfloor \phi / 3 \rfloor \quad \cdots (2)
\\] 
</span>


####  <span style="color:gray"> Input image resolution </span>

Since feature level 3-7 are used in BiFPN, the input resolution must be dividable by $2^7=128$, so the linearly increase resolutions using equation:


<span style="color:DodgerBlue">
\\[
R_\{input\} = 512 + \phi \cdot 128 \quad \cdots (3)
\\] 
</span>

Following Eq. (1), (2), (3) with different <span style="color:DodgerBlue">$\phi$</span>, they have developed EfficientDet-D0 (<span style="color:DodgerBlue">$\phi$</span>=0 ) to D7 (<span style="color:DodgerBlue">$\phi$</span>=7). 

![2](https://da2so.github.io/assets/post_img/2021-01-30-EfficientDet_Scalable_and_Efficient_Object_Detection/5.png){: .mx-auto.d-block width="70%" :}


## 4. Experiment Setting

- **Dataset**: COCO 2017 detection datasets. 
- **Optimizer**: SGD optimizer with momentum 0.9 and weight-decay 4e-5. 
	- **Learning rate**: lineraly increased from 0 to 0.16 in the first training epoch then annealed down using cosine decay rule. 
	- **Epoch size**: 300 (D7/D7x is 600)
	- **Batch size**: 128
- **Batch norm**: Synchronized batch norm is added after every convolution with batch norm decay 0.99 and epsilon 1e-3. 
- **Activation**: SiLU (Swish-1) activation  and exponential moving average with decay 0.9998. 
- **Loss**: commonly-used focal loss with $\alpha =0.25$ and $\gamma =1.5$ and aspect ratio {1/2, 1, 2}.
- **Data augmentation**: horizontal flipping and scale jittering [0.1, 2.0], which randomly resizes images between 0.1x and 2.0x of the orignal size before cropping. 
- **Evaluation method**: soft-NMS

## 5. Experiment Result

![2](https://da2so.github.io/assets/post_img/2021-01-30-EfficientDet_Scalable_and_Efficient_Object_Detection/6.png){: .mx-auto.d-block width="100%" :}



