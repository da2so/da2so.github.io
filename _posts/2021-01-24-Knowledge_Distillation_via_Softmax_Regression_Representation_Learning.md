---
layout: post
title: Knowledge Distillation via Softmax Regression Representation Learning
tags: [Knowlege Distillation]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2021-01-24-Knowledge_Distillation_via_Softmax_Regression_Representation_Learning/post.png
---

## 1. Introduction  
**Knowledge Distillation:** Dealing with the problem of training a smaller model (Student) from a high capacity source model (Teacher) so as to retain most of its performance. 
{: .box-note}


![2](https://da2so.github.io/assets/post_img/2021-01-24-Knowledge_Distillation_via_Softmax_Regression_Representation_Learning/1.png){: .mx-auto.d-block width="70%" :}


### <span style="color:gray"> 1.1 Motivation </span>

The authors of this paper try to make the outputs of student network be similar to the outputs of the teacher network. Then, they have advocate for a method that optimizes not only the output of softmax layer of the student network but also the output feature of its penultimate layer. Because optimizing the output feature of the penultimate layer is related to representation learning, they propose two approaches.


### <span style="color:gray"> 1.2 Goal </span>

The **first thing** is a direct feature matching approach which focuses on optimizing the student penultimate layer. Because the first approach detached from the classification task, they propose the **second approach** that is to decouple representation learning and classification and utilize the teacher's pre-trained classifier to train the student's penultimate layer feature.


## 2. Definition


- <span style="color:DodgerBlue">$T$</span> , <span style="color:DodgerBlue">$S$</span>: Teacher and student networks
- <span style="color:DodgerBlue">$ f^\{ Net \} $</span>: A convolutional feature extractor
    - <span style="color:DodgerBlue">$Net = { T, S }$</span>
- <span style="color:DodgerBlue">$ F^\{ Net \}_{i} \in R^\{ C_i \times H_i \times W_i \}$</span>: The output feature of i-th layer
    - <span style="color:DodgerBlue">$C_i$</span>: the output feature dimensionality
    - <span style="color:DodgerBlue">$H_i, \, W_i$</span>: the output spatial dimensions
- <span style="color:DodgerBlue">$h_\{ Net \} = \sum^M_h \sum^N_w F^\{ Net \}_\{L\} \in R^\{ C_L \}$</span>: the last layer feature learned by <span style="color:DodgerBlue">$f^\{ Net \}$</span>
    - <span style="color:DodgerBlue">$M = H_L, \, N = W_L$</span>
- <span style="color:DodgerBlue">$W^\{ Net \} \in R^\{ C_L \times K \}$</span>: Projects the feature representation <span style="color:DodgerBlue">$h^\{Net\}$</span>into <span style="color:DodgerBlue">$K$</span> class 
    - Class logits <span style="color:DodgerBlue">$ z^\{Net\}_i, \,\, i=1, \cdots, K $</span> followed by softmax function <span style="color:DodgerBlue">$s(z_i) =  \frac{exp(z_i / \tau)}{\sum_j exp(z_j / \tau)}$</span> with temperature <span style="color:DodgerBlue">$\tau$</span>



## 3. Method

![2](https://da2so.github.io/assets/post_img/2021-01-24-Knowledge_Distillation_via_Softmax_Regression_Representation_Learning/2.png){: .mx-auto.d-block width="90%" :}


In this work, the authors aim to minimize the discrepancy between the representations <span style="color:DodgerBlue">$h^T$</span> and <span style="color:DodgerBlue">$h^S$</span>. To do this, they propose to use two losses. 

The first one is an $L_2$ feature matching loss:


<span style="color:DodgerBlue">
\\[
L_\{ FM \} = \Vert h^T - r(h^S) \Vert^2, \quad \cdots Eq .(1)
\\] </span>

where <span style="color:DodgerBlue">$r(.)$</span> is a function for matching the feature tensor dimensions. The intuition for this is that this feature is directly connected to the classifier and hence imposing the student's feature to be similar to that of the teacher could have more impact on classification accuracy. The reason why they use only feature of the last layer will be explained by ablation studies.


They found <span style="color:DodgerBlue">$L_\{FM\}$</span> to be effective but only to limited extent. Its drawback is that it treats each channel dimension in the feature space independently, and ignores the inter-channel dependencies of the feature representations <span style="color:DodgerBlue">$h^S$</span> and <span style="color:DodgerBlue">$h^T$</span> for the final classification. (for notational simplicity they dropped the dependency on <span style="color:DodgerBlue">$r(.)$</span>). Therefore, they propose a second loss for optimizing <span style="color:DodgerBlue">$h^S$</span> which is directly linked with classification accuracy. To this end, they use the teacher's pre-trained **Softmax Regression (SR)** classifier.


Let us denote by <span style="color:DodgerBlue">$p$</span> the output of the teacher network when given some input image <span style="color:DodgerBlue">$x$</span>. And <span style="color:DodgerBlue">$h^S(x)$</span> represents the feature that is obtained from feeding the same image through the student network. Finally, let us pass <span style="color:DodgerBlue">$h^S(x)$</span> through the teacher's SR classifier to obtain output <span style="color:DodgerBlue">$q$</span>. The loss is defined as:

<span style="color:DodgerBlue">
\\[
L_\{ SR \} = - p \log q.  \quad \cdots Eq .(2)
\\] </span>

At this point, the following two observations is: **(1)** If <span style="color:DodgerBlue">$p=q$</span>, then this implies that <span style="color:DodgerBlue">$h^S(x) = h^T(x)$</span> which shows that Eq. (2) optimizes the student's feature representation <span style="color:DodgerBlue">$h^S$</span>. **(2)** The loss of Eq. (2) can be written as:

<span style="color:DodgerBlue">
\\[
L_\{ SR \} = - s ( W'_T h^T) \log s(W'_T h^S).  \quad \cdots Eq .(3)
\\] </span>

Now write Hinton KD loss in a similar way:

<span style="color:DodgerBlue">
\\[
L_\{ KD \} = - s ( W'_T h^T) \log s(W'_S h^S).  \quad \cdots Eq .(4)
\\] </span>


When comparing Eq. (3) and Eq. (4), because <span style="color:DodgerBlue">$W_S$</span> is also optimized in Eq. (4), this gives more degrees of freedom to the optimization algorithm. This has an impact on the learning of the student's feature representation <span style="color:DodgerBlue">$h^S$</span> that hinders the generalization capability of the student on the test set.

Overall, in this method, they train the student network using three losses:

<span style="color:DodgerBlue">
\\[
L= L_\{CE\} + \alpha L_\{FM\} + \beta L_\{SR\},  \quad \cdots Eq .(5)
\\] </span>

where <span style="color:DodgerBlue">$\alpha$</span> and  <span style="color:DodgerBlue">$\beta$</span> are the weights used to scale the losses. The teacher network is pretrained and fixed during training the student. <span style="color:DodgerBlue">$L_\{CE\}$</span> is the standard loss based on ground truth labels for the task in hand (cross-entropy loss for image classification).

The algorithm of this method is described on below.

![2](https://da2so.github.io/assets/post_img/2021-01-24-Knowledge_Distillation_via_Softmax_Regression_Representation_Learning/3.png){: .mx-auto.d-block width="90%" :}


## 4. Experiment Results

![2](https://da2so.github.io/assets/post_img/2021-01-24-Knowledge_Distillation_via_Softmax_Regression_Representation_Learning/4.png){: .mx-auto.d-block width="100%" :}



**Github Code: [HERE](https://github.com/da2so/Knowledge-Distillation-via-Softmax-Regression-Representation-Learning)**