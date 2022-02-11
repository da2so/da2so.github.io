---
layout: post
title: EagleEye Fast Sub-net Evaluation for Efficient Neural Network Pruning
tags: [Pruning]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-10-25-EagleEye_Fast_Sub_net_Evaluation_for_Efficient_Neur_Network_Pruning/post.png
---

## 1. Introduction  
**Pruning:** Eliminating the computational redundant part of a trained DNN and then getting a smaller and more efficient pruned DNN.  
{: .box-note}


### <span style="color:gray"> 1.1 Motivation </span>

The important thing to prune a trained DNN is to obtain the sub-net with highest accuracy swith reasonably small searching efforts. Existing methods to solve this problem mainly focus on evaulation process. **Evaluation process** aims to unveil the potential of sub-nets so that best pruning candidate can be selected to deliver the final pruning startegy such as Fig 1.

![general_pruning](https://da2so.github.io/assets/post_img/2020-10-25-EagleEye_Fast_Sub_net_Evaluation_for_Efficient_Neur_Network_Pruning/1.png){: .mx-auto.d-block width="90%" :}


However, the existing methods for evaluation process are either **(i)** inaccurate or **(ii)** complicated.

For **(i)**, the winner sub-nets from the evaluation process do not deliver high accuracy.  
For **(ii)**, evaluation proccess relies on computationally intensive components or is higly sensitive to some hyperparameters. 


### <span style="color:gray"> 1.2 Goal </span>

To solve these problems, the authors propose a pruning algorithm called EagleEye that is a faster and more accurate evaluation process. The EagleEye adopt the technique of adaptive batch normalization.


## 2. Method

![pipeline](https://da2so.github.io/assets/post_img/2020-10-25-EagleEye_Fast_Sub_net_Evaluation_for_Efficient_Neur_Network_Pruning/2.png){: .mx-auto.d-block width="80%" :}

A typical pruning pipeline is shown in Fig 2. In this pipeline, the authors aim to structured filter prining approaches:

<span style="color:DodgerBlue">
\\[
(r_1, \cdots, r_L)^*= argmin_\{ r_1, \cdots, r_L \} \mathcal{L} (\mathcal{A} (r_1, \cdots , r_L ; w )), s.t. \mathcal{C} constraints, \quad \cdots Eq. (1)
\\]
</span>

where <span style="color:DodgerBlue">$\mathcal{L}$</span> is the loss function and <span style="color:DodgerBlue">$\mathcal{A}$</span> is the neural network model. <span style="color:DodgerBlue">$r_l$</span> is the pruning ratio applied to the $l^th$ layer. Given some constraints <span style="color:DodgerBlue">$\mathcal{C}$</span>, (e.g. latency, total parameters) a combination of pruning ratios <span style="color:DodgerBlue">$(r_1, \cdots , r_L)$</span> is applied. the authors consider pruning task as finding the optimal pruning strategy, denoted as  <span style="color:DodgerBlue">$(r_1, \cdots , r_L)^*$</span>, that achieves the higest accuracy of the pruned model.




### <span style="color:gray"> 2.1 Motivation </span>



The authors found that the existing evaluation processes, called vanilla evaluation, do not satisfy that the sub-nets with higher evaluation accuracy are expected to also deliver high accuracy after fine-tuning as shown in Fig 3. In Fig 3 (a). left, the red bars form the histogram of accuracy collected from doing vanilla evaluation with the 50 pruned candiates. And the gray bars show the situation after fine-tuning these 50 pruned networks.

However, there is a huge difference in accuracy distribution between two results. In addition, Fig 3 (b) indicates that it might not be the weights that mess up the accuracy at the evaluation stage as only a gentle shift in weight distribution is observed during fine-tuning for the 50 networks.

![histogram_distribution](https://da2so.github.io/assets/post_img/2020-10-25-EagleEye_Fast_Sub_net_Evaluation_for_Efficient_Neur_Network_Pruning/3.png){: .mx-auto.d-block width="100%" :}


Interestinly, the authors found that it is the batch normalization layer that largely affects the evaluation. More specifically, the sub-networks use moving mean and moving variance of Batch Normlaization (BN) inherited from the full-size model. The outdated statistical values of BN layers eventually drag down the evaluation accuracy and then break the correlation between evaluation accuracy and the final converged accuracy of the pruning candidates.

To quantitatively demonstrate the problem of vanilla evaluation, let's symbolize the orignal BN as follows:

<span style="color:DodgerBlue">
\\[
y=\gamma \frac{x- \mu}{\sigma^2 + \epsilon} +\beta, \quad \cdots Eq. (2)
\\]
</span>

where <span style="color:DodgerBlue">$\beta$</span> and <span style="color:DodgerBlue">$\gamma$</span> are trainable scale and bias terms. <span style="color:DodgerBlue">$\epsilon$</span> is a term with small value to avoid zero division. For a mini-batch with size <span style="color:DodgerBlue">$N$</span>, the statistical values of <span style="color:DodgerBlue">$\mu$</span> and <span style="color:DodgerBlue">$\sigma^2$</span> are calculated as below:

<span style="color:DodgerBlue">
\\[
\mu_B= E[ x_B] = \frac{1}{N} \sum^N_i x_i, \quad \sigma^2_B= Var[ x_B]= \frac{1}{N-1} \sum^N_i (x_i- \mu_B)^2. \quad \cdots Eq. (3)
\\]
</span>

During training, <span style="color:DodgerBlue">$\mu$</span> and <span style="color:DodgerBlue">$\sigma^2$</span> are calculated with the moving mean and variacne:

<span style="color:DodgerBlue">
\\[
\mu_t= m \mu_z+ (1-m) \mu_B, \quad \sigma^2_t= m \sigma^2_z+ (1-m) \sigma_B, \quad \cdots Eq. (4)
\\]
</span>

where <span style="color:DodgerBlue">$z=t-1$</span>, <span style="color:DodgerBlue">$m$</span> is the momentum coeffcient and subscript <span style="color:DodgerBlue">$t$</span> refers to the number of training iterations. And if the total number of training iteration is <span style="color:DodgerBlue">$T$</span>,  <span style="color:DodgerBlue">$\mu_T$</span> and <span style="color:DodgerBlue">$\sigma^2_T$</span> are used in testing phase.

These two items are called global BN statistics, where "global" referes to the full-size model.


### <span style="color:gray"> 2.2 Adaptive Batch Normalization </span>

Because the global BN statistics are out-dated to the sub-nets from vanilla evaluation, we should re-calculate <span style="color:DodgerBlue">$\mu_T$</span> and <span style="color:DodgerBlue">$\sigma^2_T$</span> with adaptive values by conducting a few iterations of inference on part of the training sets. Concretely, the authors freeze all the network parameters while resetting the moving average statistics. Then, do update the moving statistics by a few iterations of only forward-propagation, using Eq. (4). The authors denote **adaptive BN statistics** as <span style="color:DodgerBlue">$\hat{\mu}_T$</span> and <span style="color:DodgerBlue">$\hat{\sigma}^2_T$</span>


To validate the effectiveness of proposed method, Fig 4. shows that adaptive BN delivers evaluation accuracy with a stronger correlation, compared to the vailla evaluation. The correlation measurement is conducted by Pearson Correlation Coefficient (PCC).

![correlation](https://da2so.github.io/assets/post_img/2020-10-25-EagleEye_Fast_Sub_net_Evaluation_for_Efficient_Neur_Network_Pruning/4.png){: .mx-auto.d-block width="100%" :}

 As another evidence, the authors compare the distance of BN statistical values between true statistics. They consider the true statistics as <span style="color:DodgerBlue">$\mu_{val}$</span>,  <span style="color:DodgerBlue">$\sigma_{val}$</span> sampled from validation data. As you expect, the adaptive BN provides closer statistical values to the true values while global BN is way further as shown in Fig 5. (each pixel in the heatmaps represents a distance.)

![visualization_distance](https://da2so.github.io/assets/post_img/2020-10-25-EagleEye_Fast_Sub_net_Evaluation_for_Efficient_Neur_Network_Pruning/5.png){: .mx-auto.d-block width="100%" :}


### <span style="color:gray"> 2.3 EagleEye pruning algorithm </span>

The overall procedure of EagleEye is described in Fig 6. The procedure contains three parts, (1) pruning strategy generation, (2) filter pruning, and (3) adpative BN-basesd evaluation.

![procedure_eagleeye](https://da2so.github.io/assets/post_img/2020-10-25-EagleEye_Fast_Sub_net_Evaluation_for_Efficient_Neur_Network_Pruning/6.png){: .mx-auto.d-block width="100%" :}


**1. Strategy generation**  
Generation randomly samples <span style="color:DodgerBlue">$L$</span> real numbers from a given range <span style="color:DodgerBlue">$[0, R]$</span> to form a pruning strategy <span style="color:DodgerBlue">$(r_1, \cdots , r_L)^*$</span>. <span style="color:DodgerBlue">$R$</span> is the largest pruning ratio applied to a layer. This is a Monte Carlo sampling process with a uniform distribution for all layer-wise pruning rates.


**2. Filter pruning process**  
Similar to a normal filter pruning method, the filters are firstly ranked according to their $L1$-norm and the <span style="color:DodgerBlue">$r_l$</span> of the least important filters are trimmed off.

**3. The adaptive-BN based candidate evaluation module**  
Given a pruned network, it freezes all learnable parameters and traverses through a small amount of data in the training set to calculate the adaptive BN statistics. In practice, authors sampled 1/30 of the total training set for 100 iterations in ImageNet dataset. Next, the module evaluates the performance of the candidate networks on a small part of training set data, called sub-validation set, and picks the top ones in the accuracy ranking as a winner candidate.

**4. Fine tuning**  
The fine-tuning will be applied to the winner candidate network.


## Experiments


![experiment_result](https://da2so.github.io/assets/post_img/2020-10-25-EagleEye_Fast_Sub_net_Evaluation_for_Efficient_Neur_Network_Pruning/7.png){: .mx-auto.d-block width="100%" :}



## <span style="color:#C70039 ">Reference </span>
*Li, Bailin, et al. "Eagleeye: Fast sub-net evaluation for efficient neural network pruning." European Conference on Computer Vision. Springer, Cham, 2020.*


**Github Code: [Eagleeye](https://github.com/da2so/Eagleeye_Tensorflow)**