---
layout: post
title: Zero-Shot Knowledge Transfer via Aversarial Belief Matching
tags: [Model Compression, Knowledge Distillation, Data-free, GAN]
comments: true
use_math: true
thumbnail-img: /assets/thumbnail_img/2020-08-14-Zero-Shot_Knowledge_Transfer_via_Adversarial_Belif_Matching/post.png
---

## 1. Data-free Knowledge distillation


{: .box-note}
**Knowledge distillation:** Dealing with the problem of training a smaller model (Student) from a high capacity source model (Teacher) so as to retain most of its performance.

As the word itself, We perform knowledge distillation when there are no original dataset on which the Teacher network has been trained. It is because, in real world, most datasets are proprietary and not shared publicly due to privacy or confidentiality concerns. 


In order to perform data-free knowledge distillation, it is a necessary to reconstruct a dataset for training Student network. Thus, in Zero-Shot Knowledge Transfer via Aversarial Belief Matching, we train an adversarial generator to search for iamges on which the student poorly matches the teacher, and then using them to train the student.


## 2. Zero-shot knowledge transfer


### <span style="color:gray"> 2.1 Problem definition </span>


* Teacher network: <span style="color:DodgerBlue">$T(x)$</span> with an input image <span style="color:DodgerBlue">$x$</span>
	* Probability vector of teacher network: <span style="color:DodgerBlue">$t$</span>
* Student network: <span style="color:DodgerBlue">$S(x; \theta)$</span> with weigths <span style="color:DodgerBlue">$\theta$</span>
	* Probability vector of student network: <span style="color:DodgerBlue">$s$</span>
* Generator: <span style="color:DodgerBlue">$G(z; \phi) $</span> with weights <span style="color:DodgerBlue">$\phi$</span>
	* Pseudo data: <span style="color:DodgerBlue">$x_p$</span> from a noise vector <span style="color:DodgerBlue">$z \sim \mathcal{N} (0, I) $</span>


### <span style="color:gray"> 2.2 Method </span>

The goal is to produce pseudo data from generator and use them to train student network by knowledge distillation.

To do this, Our zero-shot training algorithm is described in Algorithm 1. For <span style="color:DodgerBlue">$N$</span> iterations we sample one batch of <span style="color:DodgerBlue">$z$</span>, and take <span style="color:DodgerBlue">$n_G$</span> gradient updates on the generator whi learning rate <span style="color:DodgerBlue">$\eta$</span>, such that it produces pseudo samples <span style="color:DodgerBlue">$x_p$</span> that maximize <span style="color:DodgerBlue">$D_{KL} (T(x_p) || S(x_p))$</span>.

{: .box-note}
**$D_{KL} (T(x_p) \| \| S(x_p))= \sum_i t_p^{(i)} log (t^{(i)}_p / s^{(i)}_p)$:** Kullback-Leibler (KL) divergence between outputs of the teacher and student netowkrs on pseudo data ($i$ is image classes)



| **If** maximize <span style="color:DodgerBlue">$D_{KL} (T(x_p) \| \| S(x_p))$</span> $\rightarrow$ <span style="color:DodgerBlue">$ t_p^{(i)}$</span> $\uparrow$, <span style="color:DodgerBlue">$\; s^{(i)}_p$</span> $\downarrow$|
| **Elif** minimize <span style="color:DodgerBlue">$D_{KL} (T(x_p) \| \| S(x_p))$</span> $\rightarrow$ <span style="color:DodgerBlue">$ t_p^{(i)}$</span> $\downarrow$, <span style="color:DodgerBlue">$\; s^{(i)}_p$</span> $\uparrow$|


We then take <span style="color:DodgerBlue">$n_S$</span> gradient steps on the student with <span style="color:DodgerBlue">$x_p$</span> fixed, such that it matches the teacher's predictions on <span style="color:DodgerBlue">$x_p$</span>. In practice, we use <span style="color:DodgerBlue">$n_S > n_G$</span>, which gives more time to the student to match the teacher on <span style="color:DodgerBlue">$x_p$</span> and encourages the generator to explore other regions of the input space at the next iteration.



### <span style="color:gray"> 2.2 Extra loss functions </span>

The high student entropy is a vital component to our method since it makes it hard for the generator to fool the student easily. Then, since many student-teacher pairs have similar block structures, we can add an attention term to the student loss as follows:

<span style="color:DodgerBlue">
\\[
L_s=D_{KL} (T(x_p) || S(x_p)) + \beta \sum_l^{N_L} \Vert \frac{f (A^{(t)}_l)}{ \Vert f (A^{(t)}_l) \Vert_2} - \frac{f (A^{(s)}_l)}{ \Vert f (A^{(s)}_l) \Vert_2}\Vert_2. \quad \cdots Eq. (1)
\\]
</span>


* Hyperparameter: <span style="color:DodgerBlue">$\beta$</span>
* Total layers: <span style="color:DodgerBlue">$N_L$</span>
* Teacher and student activation blocks: <span style="color:DodgerBlue">$A^{(t)}_l$</span> and <span style="color:DodgerBlue">$A^{(s)}_l$</span> for layer <span style="color:DodgerBlue">$l$</span>
	* Total channels of l-th layer: <span style="color:DodgerBlue">$N_{A_l}$</span>
* Spatial attention map: <span style="color:DodgerBlue">$f(A_l)= \frac{1}{N_{A_l}} \sum_c a^2_\{lc \}$</span>

 We take the sum over some subet of <span style="color:DodgerBlue">$N_L$</span> layers. The second term encourages both spatial attention maps between teacher and student networks to be similar. We don't use attention for the generator loss because it makes it too easy to fool the student. The training procdure is described in Fig. 1.

![2](https://da2so.github.io/assets/post_img/2020-08-14-Zero-Shot_Knowledge_Transfer_via_Adversarial_Belif_Matching/1.png){: .mx-auto.d-block :}

 

### <span style="color:gray"> 2.3 Toy experiment </span>


The dynamics of our algorithm is illustrated in Fig. 2, where we use two layer MLPs for both teacher and student, and learn the pseudo points directly. These are initialized away from the real data manifold. 

During training, pseudo points can be seen to explore the input space, typically running along decision boundaries where the student is most likely to match the teacher poorly. At the same time, the student is trained to match the teacher on the pseudo points, and so they must keep changing locations. When the decision boundaries between student and teacher are well aligned, some pseudo points will naturally depart from them and search for new high teacher mismatch regions, which allows disconnected decision boundaries to be explored as well.


![1](https://da2so.github.io/assets/post_img/2020-08-14-Zero-Shot_Knowledge_Transfer_via_Adversarial_Belif_Matching/2.png){: .mx-auto.d-block :}



## 3. Experiments & Results


For each experiment, we run three seeds and report the mean with one standard deviation. Experiment setting is described in Fig. 3 (a).


### <span style="color:gray"> 3.1 CIFAR-10 and SVHN </span>

We focus our experiments on two common datasets, SVHN and CIFAR-10. For both datasets we use WideResNet (WRN) architecture. Our distillation results are shown in Fig. 3 (b). We include the few-shot performance of our method as a comparison, by naively finetuning our zero-shot model with <span style="color:DodgerBlue">$M$</span> samples per class. Our method reaches 83.69$\pm$0.58% without using any real data, and increase to 85.91$\pm$0.24% when finetuned with $M=100$ images per class.


### <span style="color:gray"> 3.2 Architecture  dependence </span>

We observe that some teacher-student pairs tend to work better than others, as in the case for few-shot distillation. The comparison results are shown in Fig. 3 (c). In zero-shot, deep students with more parameters don't neccessarily help: the WRN-40-2 teacher distills 3.1% better to WRN-16-2 than to WRN-40-1, even though WRN-16-2 has less than half number of layers and a similar parameter count than WRN-40-1.


### <span style="color:gray"> 3.3 Nature of the pseudo data </span>

Samples from generator during training are shown in Fig. 3 (d). We notice that early in training the samples look like coarse textures and are reasonly diverse. After about 10% of the training run, most images produced by generator look like high frequency patterns that have little meaning to humans.


![3](https://da2so.github.io/assets/post_img/2020-08-14-Zero-Shot_Knowledge_Transfer_via_Adversarial_Belif_Matching/3.png){: .mx-auto.d-block :}


### <span style="color:gray"> 3.4 Measuring belief match near decision boundaries </span>

We would like to verify that the student is implicitly trainied to match the teacher's predictions close to decision boundaries. For this, in Algorithm 2, we propose a way to probe the difference between beliefs of network <span style="color:DodgerBlue">$A$</span> and <span style="color:DodgerBlue">$B$</span> near the decision boundaries of <span style="color:DodgerBlue">$A$</span>. The procedure of Algorithm 2 is follows.

1. Sampling a real image: <span style="color:DodgerBlue">$x$</span> from the test set <span style="color:DodgerBlue">$X_{test}$</span> such that network network <span style="color:DodgerBlue">$A$</span> and <span style="color:DodgerBlue">$B$</span> both give the same class prediction <span style="color:DodgerBlue">$i$</span>.

2. For each class <span style="color:DodgerBlue">$j \neq i$</span> we update <span style="color:DodgerBlue">$x$</span> by taking <span style="color:DodgerBlue">$K$</span> adversarial steps on network <span style="color:DodgerBlue">$A$</span>, with learning rate <span style="color:DodgerBlue">$\xi$</span>, to go from class <span style="color:DodgerBlue">$i$</span> to class <span style="color:DodgerBlue">$j$</span>.

3. The proability <span style="color:DodgerBlue">$p^A_i$</span> of <span style="color:DodgerBlue">$x$</span> belonging to class <span style="color:DodgerBlue">$i$</span> according to network <span style="color:DodgerBlue">$A$</span> quickly reduces, with a concurrent increase in <span style="color:DodgerBlue">$p^A_j$</span>. 

4. During 3. of process, we also record <span style="color:DodgerBlue">$p^B_j$</span> and compare <span style="color:DodgerBlue">$p^A_j$</span> ad <span style="color:DodgerBlue">$p^B_j$</span>.

Consequently, we are asking the following question, *as we perturb <span style="color:DodgerBlue">$x$</span> to move from class <span style="color:DodgerBlue">$i$</span> to <span style="color:DodgerBlue">$j$</span> according to network <span style="color:DodgerBlue">$A$</span>, to what degree do we also move from class <span style="color:DodgerBlue">$i$</span> to <span style="color:DodgerBlue">$j$</span> according to network <span style="color:DodgerBlue">$B$</span>?*


We refer to <span style="color:DodgerBlue">$pj$</span> curves as transition curves. For a dataset of <span style="color:DodgerBlue">$C$</span> classes, we obtain <span style="color:DodgerBlue">$C-1$</span> transitioncurves for each image <span style="color:DodgerBlue">$x \in X_{test}$</span>, and for each network <span style="color:DodgerBlue">$A$</span> and <span style="color:DodgerBlue">$B$</span>. We show the average transition curves in Fig. 4 (a), in the case where network <span style="color:DodgerBlue">$B$</span> is the teacher, and network <span style="color:DodgerBlue">$A$</span> is either our zero-shot student or a standard student distilled with KD+AT. 



The result is particularly surprising because while updating images to move from class  <span style="color:DodgerBlue">$i$</span> to class  <span style="color:DodgerBlue">$j$</span> on zero-shot student also corresponds to moving from class  <span style="color:DodgerBlue">$i$</span> to class  <span style="color:DodgerBlue">$j$</span> according to the teacher, the KD+AT student have flat  <span style="color:DodgerBlue">$p_j=0$</span> curves for several images even though KD+AT student was trained on real data, and the transition curves are also calculated for real data.

We can more explicitly quantify the belief match between networks <span style="color:DodgerBlue">$A$</span> and <span style="color:DodgerBlue">$B$</span> as we take steps to cross the decision boundaries of network <span style="color:DodgerBlue">$A$</span>. We define the Mean Transition Error (MTE) as the absolute probability difference between <span style="color:DodgerBlue">$p^A_j$</span> and <span style="color:DodgerBlue">$p^B_j$</span>, averaged over <span style="color:DodgerBlue">$K$</span> steps, <span style="color:DodgerBlue">$N_{test}$</span> test images and <span style="color:DodgerBlue">$C-1$</span> classs: 

<span style="color:DodgerBlue">
\\[
MTE(net_A, net_B)= \frac{1}{N_{test}} \sum^{N_\{test\}}_n \frac{1}{C-1} \sum^{C-1}_c \frac{1}{K} \sum^K_k \vert p^A_j- p^B_j\vert \quad \cdots Eq. (2)
\\]
</span>


The mean transition errors are reported in Fig. 4 (b).


![4](https://da2so.github.io/assets/post_img/2020-08-14-Zero-Shot_Knowledge_Transfer_via_Adversarial_Belif_Matching/4.png){: .mx-auto.d-block :}




