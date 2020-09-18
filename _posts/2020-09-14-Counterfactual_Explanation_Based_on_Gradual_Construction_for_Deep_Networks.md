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


<span style="color:DodgerBlue">
\\[
i^\ast = argmax_i f_\{c_t\} (X+\delta e_i), \quad \cdots Eq. (1)
\\]
</span>

where <span style="color:DodgerBlue">$e_i$</span> is a one-hot vector whose value is 1 only for the $i$-th element, <span style="color:DodgerBlue">$\delta$</span> is a non-zero real value and <span style="color:DodgerBlue">$f_{c_t}$</span> is the classification score for the target class <span style="color:DodgerBlue">$c_t$</span>.


Suppose <span style="color:DodgerBlue">$\delta = \bar{\delta} h $</span> where <span style="color:DodgerBlue">$h$</span> is a non-zero and infinitesimal value and <span style="color:DodgerBlue">$\bar{\delta}$</span> is a proper scalr to match the equality. Then, the Eq. (2) is approximated as the directional derivative with respect to  <span style="color:DodgerBlue">$X$</span>.

<span style="color:DodgerBlue">
\\[
\\begin{array}{l}
f_\{c_t\} (X+\delta e_i) = f_\{c_t\} (X+\delta e_i) - f_\{c_t\} (X)+f_\{c_t\} (X) \cr
\quad \quad \quad \quad \quad \; \,= f_\{c_t\} (X+ \bar{\delta} h e_i) - f_\{c_t\} (X)+f_\{c_t\} (X) \cr
\quad \quad \quad \quad \quad \; \,= \frac{ f_\{c_t\} (X+ \bar{\delta} h e_i) - f_\{c_t\} (X) }{h}h+ f_\{c_t\} (X) \cr
\quad \quad \quad \quad \quad \; \, \approx \bigtriangledown f_\{c_t\} (X) \bar{\delta}  e_i h + f_\{c_t\} (X) \cr
\quad \quad \quad \quad \quad \; \, = \bigtriangledown f_\{c_t\} (X) \delta e_i  + R.
\\end{array} \quad \cdots Eq .(2)
\\]
</span>

Since the  <span style="color:DodgerBlue">$\delta$</span> is a real value, we separately consider positive and negative cases in order to find an optimal  <span style="color:DodgerBlue">$i^\ast$</span>.

<span style="color:DodgerBlue">\\[ 
i^\ast= \\left\\{ \\begin{array}{ll} max( \bigtriangledown f_t (X))_\{i\}, & if \; \delta >0, \cr
											 min( \bigtriangledown f_t  (X))_i, & otherwise.
\\end{array} \quad \cdots Eq .(3) \\right. 
\\]</span>


The <span style="color:DodgerBlue">$t$</span> means <span style="color:DodgerBlue">$c_t$</span>. The <span style="color:DodgerBlue">$max(\cdot)_i$</span> function returns an index that has a maximum value in the input vector and <span style="color:DodgerBlue">$min(\cdot)_i$</span> is similarly defined.


Thus, we choose a sub-optimal idndex as 


<span style="color:DodgerBlue">
\\[
\hat{i}^\ast = max ( | \bigtriangledown f_\{c_t\} (X) | )_i. \quad \cdots Eq. (4)
\\]
</span>

In summary, each masking step selects an index in the descending order by calculating Eq. (5) and changes the zero value of mask <span style="color:DodgerBlue">$M$</span> into one.


### <span style="color:gray"> 2.3 Composite step </span>

After selecting the input feature to be modified, the composition step otpimizes the feature value to ensure that the deep network classifies the perturbed data <span style="color:DodgerBlue">$X'$</span> as the target class <span style="color:DodgerBlue">$c_t$</span>. To achieve this, the conventional approaches have proposed an objective function to improce the output score of <span style="color:DodgerBlue">$c_t$</span> as follow:

<span style="color:DodgerBlue">
\\[
argmax_\epsilon f_\{ c_t \} (X+\epsilon) +R_\epsilon , \quad \cdots Eq. (5)
\\]
</span>

where <span style="color:DodgerBlue">$\epsilon= \\{ \epsilon_1, \cdots , \epsilon_d \\}$</span> is a perturbation variable and <span style="color:DodgerBlue">$R_\epsilon$</span> is a regularization term.

However, this obejctive function causes an adversarial attack such as Failure images in Fig. 3. Then, we compared the tributions of logit scores (before the softmax layer) for each failure case and the training images that are classified as <span style="color:DodgerBlue">$c_t$</span> from a pre-trained network. And, we discovered that there exist a notable difference between the two distributions as depicted in Fig 3. 

![2](https://da2so.github.io/assets/post_img/2020-09-14-Counterfactual_Explanation_Based_on_Gradual_Construction_for_Deep_Networks/4.png){: .mx-auto.d-block width='70%' :}

Thus, we regard failure cases as the result of an n inappropriate objective function that maps the perturbed data onto a different logit space from the training data. To solve this problem, we instead force the logit space of <span style="color:DodgerBlue">$X'$</span> to belong to the space of training data as follows:

<span style="color:DodgerBlue">
\\[
argmax_\epsilon f_\{ c_t \} (X+\epsilon) +R_\epsilon , \quad \cdots Eq. (6)
\\]
</span>

where <span style="color:DodgerBlue">$K$</span> is the number of classes, <span style="color:DodgerBlue">$f'_k$</span> represents a logit score for a class <span style="color:DodgerBlue">$k$</span>, <span style="color:DodgerBlue">$X_\{i, c_t \}$</span> denotes $i$-th training data that is classified into a target classes <span style="color:DodgerBlue">$c_t$</span>. <span style="color:DodgerBlue">$N$</span> denotes the number of randomly sampled training data. In additon, we add a regularizer <span style="color:DodgerBlue">$\lambda$</span> to encourage the values of <span style="color:DodgerBlue">$X'$</span> to close to the input data <span style="color:DodgerBlue">$X$</span>.  
As a result, Eq. (6) makes the composite <span style="color:DodgerBlue">$C$</span> to improve the probability of <span style="color:DodgerBlue">$c_t$</span> and also pushes the perturbed data towards belonging to the logit score distribution of a training data.


Overall, gradual construction iterates over the masking and composition steps until the classification probability of a target class is reached to a hyperparameter <span style="color:DodgerBlue">$\tau$</span>.  
We present a pseudo-code in Algorithm 1.


![2](https://da2so.github.io/assets/post_img/2020-09-14-Counterfactual_Explanation_Based_on_Gradual_Construction_for_Deep_Networks/5.png){: .mx-auto.d-block width='60%' :}



## 3. Experiment

![2](https://da2so.github.io/assets/post_img/2020-09-14-Counterfactual_Explanation_Based_on_Gradual_Construction_for_Deep_Networks/6.png){: .mx-auto.d-block width='100%' :}


![2](https://da2so.github.io/assets/post_img/2020-09-14-Counterfactual_Explanation_Based_on_Gradual_Construction_for_Deep_Networks/7.png){: .mx-auto.d-block width='100%' :}





### <span style="color:#C70039 ">Reference </span>
*Kang, Sin-Han, et al. "Counterfactual Explanation Based on Gradual Construction for Deep Networks." arXiv preprint arXiv:2008.01897 (2020).*


**Github Code: [Here](https://github.com/da2so/Counterfactual-Explanation-Based-on-Gradual-Construction-for-Deep-Networks)**