---
layout: post
title: Learning Low-Rank Approximation for CNNs
tags: [Filter decomposition, Model Compression]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2021-03-27-Learning_Low-Rank_Approximation_for_CNNs/post.png
---

## 1. Introduction   
**Filter Decomposition (FD):** Decomposing a weight tensor into multiple tensors to be multiplied in a consecutive manner and then get a compressed model.  
{: .box-note}

### <span style="color:gray"> 1.1 Motivation </span>

Well-known low-rank approximation (i.e. FD) methods, such as Tucker or CP decomposition, result in degraded model accuracy because decomposed layers hinder training convergence. 

### <span style="color:gray"> 1.2 Goal </span>

To tackle this problem, they propose a new training technique, called **DeepTwist**, that finds a flat minimum in the view of low-rank approximation without a decomposed structure during training.
That is, they retain the original model structure without considering low-rank approximation. Moreover, because the method preserves the original model structure during training, 2-dimensional low-rank approximation demanding lowering (such as im2col) is available.

## 2. Training Algorithm for Low-Rank Approximation


Note that FD methods incur the accuracy drop from the original network because of an approximation error given a rank. If so, in terms of loss surface, which position of parameters is appropreatie before applying FD in a below figure?

![optimal_point_for_FD](https://da2so.github.io/assets/post_img/2021-03-27-Learning_Low-Rank_Approximation_for_CNNs/1.png){: .mx-auto.d-block width="90%" :}


As you already  expect, the right position of parameters is less sensitive to accuracy drop (loss variance). That is, the underlying principle of proposed training algorithm is to find a particular local minimum of flat loss surface, robust to parameter errors induced by low-rank approximation.


The proposed scheme preserves the original model structure and searches for a local minimum well suited for low-rank approximation. The training procedure for low-rank approximation, called **DeepTwist**, is illustrated in Fig. 1.

![DeepTwist_procedure](https://da2so.github.io/assets/post_img/2021-03-27-Learning_Low-Rank_Approximation_for_CNNs/2.png){: .mx-auto.d-block width="90%" :}


- Step by Step
	1. Training <span style="color:DodgerBlue">$ S_D $</span> batches.
	2. At each weight distortion step, parameters are decomposed into multiple tensors following low-rank approximation (e.g. CP, tucker).
		- Those decomposed tensors are multiplied to reconstruct the original tensor form.
	3. If a particular local minimum is not flat enough, training procedure escapes such a local minimum through a weight distortion step (step 2).
	4. Otherwise, the optimization process is continued in the search space around a local minimum (step 4 and 5).



### <span style="color:gray"> 2.1 Determining the best distortion step $S_D$ </span>
	
Determining the best distortion step <span style="color:DodgerBlue">$ S_D $</span> is important in training procedure. The loss function of a model <span style="color:DodgerBlue">$ L(w)$</span> is approximated as: 

<span style="color:DodgerBlue">
\\[
L(w) \simeq L(w_0) + (w - w_0)^{\top} (H(w_0)/2) (w - w_0), \quad \cdots (1)
\\] 
</span>

using a local quadratic approximation (taylor series).

- Parameter set: <span style="color:DodgerBlue">$ w $</span>
- Set of parameters at a local minimum: <span style="color:DodgerBlue">$ w_0 $</span>
- Hessian of <span style="color:DodgerBlue">$ L $</span>: <span style="color:DodgerBlue">$ H $</span>
- Learning rate: <span style="color:DodgerBlue">$ \gamma $</span>

Then, <span style="color:DodgerBlue">$ w $</span> can be updated by gradient descent as follows:

<span style="color:DodgerBlue">
\\[
w_\{t+1\} = w_t - \gamma \frac{\partial L}{\partial w} |_\{ w = w_t \} \simeq w_t - \gamma H(w_0)(w_t - w_0) \quad \cdots (2)
\\] 
</span>


Thus, after <span style="color:DodgerBlue">$ S_D $</span> steps, you obtain <span style="color:DodgerBlue">$ w_\{ t+ S_D \}  = w_0 + (I - \gamma H(w_0))^\{S_D\} (w_t - w0)$</span>, where <span style="color:DodgerBlue">$ I $</span> is identity matrix. Suppose that <span style="color:DodgerBlue">$ H $</span> is positive semi-definite and all elements of <span style="color:DodgerBlue">$ I - \gamma H(w_0) $</span> are less than 1.0, large <span style="color:DodgerBlue">$ S_D $</span> converges to <span style="color:DodgerBlue">$ w_0 $</span>.

Correspondingly, large <span style="color:DodgerBlue">$ S_D $</span> is preferred for this proposed method. Large <span style="color:DodgerBlue">$ S_D $</span> is also desirable to avoid stochastic variance from the batch selection. However, Too large of a <span style="color:DodgerBlue">$ S_D $</span> yields fewer opportunities to escape local minima especially when the learning rate has decayed. In practice, <span style="color:DodgerBlue">$ S_D $</span> falls in the range of 100 to 5000 steps.

## 3. Tucker Decomposition Optimized by DeepTwist

First of all, Tucker decomposition is described as below:

<span style="color:DodgerBlue">
\\[
 K'_\{i, j, s, t\} = \sum^\{R_s\} \sum^\{R_t\} C P^S P^T  \quad \cdots (3)
\\] 
</span>

![tucker_decomposition](https://da2so.github.io/assets/post_img/2021-03-27-Learning_Low-Rank_Approximation_for_CNNs/3.png){: .mx-auto.d-block width="70%" :}


- Original 4D kernel tensor: <span style="color:DodgerBlue">$ K = \mathcal{R}^\{ d \times d \times S \times T \}$</span>.
	- Kernel dimension: <span style="color:DodgerBlue">$ d \times d $</span>.
	- Input feature map size: <span style="color:DodgerBlue">$ S $</span>.
	- Output feature map size: <span style="color:DodgerBlue">$ T $</span>.
- Approximated 4D kernel tensor: <span style="color:DodgerBlue">$ K' = \mathcal{R}^\{ d \times d \times S \times T \}$</span>.
- Reduce kernel tensor: <span style="color:DodgerBlue">$ C = C_\{ i, j, r_s, r_t\} $</span>.
- The rank for input feature map dimension: <span style="color:DodgerBlue">$ R_s $</span>.
- The rank for output feature map dimension: <span style="color:DodgerBlue">$ R_t $</span>.
- 2D filter matrices to map <span style="color:DodgerBlue">$ C_\{ i, j, r_s, r_t\} $</span> to  <span style="color:DodgerBlue">$ K_\{ i, j, r_s, r_t\} $</span>: <span style="color:DodgerBlue">$ P^S, P^T $</span>.

Each component is obtained to minimize the Frobenius norm of <span style="color:DodgerBlue">$ ( K_\{ i, j, r_s, r_t\} - K'_\{ i, j, r_s, r_t\} ) $</span>. As a result, one convolution layer is divided into three convolution layers, specifically, (1×1) convolution for <span style="color:DodgerBlue">$ P^S $</span>, (d × d) convolution for <span style="color:DodgerBlue">$ C_\{ i, j, r_s, r_t \} $</span>, and (1 × 1) convolution for <span style="color:DodgerBlue">$ P^T $</span>.


DeepTwist training algorithm is conducted for Tucker decomposition as follows:

1. Perform normal training for <span style="color:DodgerBlue">$ S_D $</span> steps (batches) without considering Tucker decomposition.
2. Calculate <span style="color:DodgerBlue">$ C $</span>, <span style="color:DodgerBlue">$ P^s $</span> and <span style="color:DodgerBlue">$ P^T $</span> using Tucker decomposition to obtain <span style="color:DodgerBlue">$ K' $</span>.
3. Replace <span style="color:DodgerBlue">$ K $</span> with <span style="color:DodgerBlue">$ K' $</span> (a weight distortion step in Fig. 1).
4. Go to Step 1 with updated <span style="color:DodgerBlue">$ K $</span>.

Using the pre-trained ResNet-32 model with CIFAR-10 dataset, compare two training methods for Tucker decomposition: 1) typical training with a decomposed model and 2) DeepTwist training (<span style="color:DodgerBlue">$ S_D $</span> = 200). The result is as follows.

![DeepTwist_test](https://da2so.github.io/assets/post_img/2021-03-27-Learning_Low-Rank_Approximation_for_CNNs/4.png){: .mx-auto.d-block width="70%" :}


## 4. 2-Dimensional SVD Enabled by DeepTwist


### <span style="color:gray"> 4.1 Issues of 2D SVD on Convolution Layers </span>

Convolution can be performed by matrix multiplication if an input matrix is transformed into a Toeplitz matrix with redundancy and a weight kernel is reshaped into a T × (S × d × d) matrix (i.e., a lowered matrix). Then, commodity computing systems (such as CPUs and GPUs) can use libraries such as Basic Linear Algebra Subroutines (BLAS) without dedicated hardware resources for convolution.

For BLAS-based CNN inference, reshaping a 4D tensor <span style="color:DodgerBlue">$ K $</span> and performing SVD is preferred for low-rank approximation rather than relatively inefficient Tucker decomposition. However, because of lowering steps, two decomposed matrices by SVD do not present corresponding (decomposed) convolution layers. As a result, the fine-tuning requiring a structurally modified model for training is not allowed. But, DeepTwist does not alter the model structure during training.


### <span style="color:gray"> 4.2 Tiling-Based SVD for Skewed Weight Matrices </span>

A reshaped kernel matrix <span style="color:DodgerBlue">$ K \in \mathcal{R}^\{ T \times (S \times d \times d)\} $</span> is usually a skewed matrix where row-wise dimension (<span style="color:DodgerBlue">$ n $</span>) is smaller than column-wise dimension (<span style="color:DodgerBlue">$ m $</span>) as shown in Fig. 4 (i.e., <span style="color:DodgerBlue">$ n \ll m $</span>). A range of available rank <span style="color:DodgerBlue">$ r $</span> for SVD, then, is constrained by small <span style="color:DodgerBlue">$ K $</span> and the compression ratio is approximated
to be <span style="color:DodgerBlue">$ n/r $</span>. If such a skewed matrix is divided into four tiles as shown in Fig. 4 and the four tiles do not share much common characteristics, then tiling-based SVD can be a better approximator and rank <span style="color:DodgerBlue">$ r $</span> can be further reduced without increasing approximation error. Moreover, fast matrix multiplication is usually implemented by a tiling technique in hardware to improve the weight reuse rate. 


![experiment_setting](https://da2so.github.io/assets/post_img/2021-03-27-Learning_Low-Rank_Approximation_for_CNNs/5.png){: .mx-auto.d-block width="90%" :}

They tested a (1024 × 1024) random weight matrix in which elements follow a Gaussian distribution. A weight matrix is divided by (1×1), (16×16), or (128×128)tiles (then, each tile is a submatrix of (1024×1024), (64×64), or (8×8) size). Each tile is compressed by SVD to achieve the same overall compression ratio of 4× for all of the three cases. As described in Fig. 4. (on the right side), increasing the number of tiles tends to increase the count of near-zero and large weights. 


They applied the tiling technique and SVD to the 9 largest convolution layers of ResNet-32 using the CIFAR-10 dataset. Weights of selected layers are reshaped into 64 × (64 × 3 × 3) matrices with the tiling configurations described in Table 1. DeepTwist performs training with the same learning schedule and SD(=200) used in Section 3.

![experiment_result](https://da2so.github.io/assets/post_img/2021-03-27-Learning_Low-Rank_Approximation_for_CNNs/6.png){: .mx-auto.d-block width="90%" :}

<br />

### <span style="color:#C70039 ">Reference </span>
*Lee, Dongsoo, et al. "Learning low-rank approximation for cnns." arXiv preprint arXiv:1905.10145 (2019).*

