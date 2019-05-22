---
title: 自适应测评
date: 2019-05-22
tags: 
	- IRT
	- CAT
categories: 
	- 测评
comments: on
---


## 目标
基于IRT理论，实现对题库参数的估计，根据新用户做题情况给出用户能力值估计及其标准差，再根据当前能力值给出最能够区分用户等级的下一道题，当能力值稳定在一定范围内即结束测试。实现根据学生能力动态出题的自适应测试。


## 基础知识
### IRT-3PL
IRT理论即项目反应理论(Item Response Theory, IRT)，又称题目反应理论、潜在特质理论（Item Response Theory）是一系列心理统计学模型的总称。在实际应用中，人们出于数值处理的简便，倾向于使用3参数Logistic模型（简称3PL模型，3-parameter Logistic model），3PL模型中，第$j$人做对第$i$题的概率如下：

$$ P_i(\theta _j)=c_i+\frac{(1-c_i)}{1+e^{-\alpha_i(\theta_j-\beta_i))}} $$

- ${\theta_j}$，第$j$人的能力值
- ${\alpha_i}$，第$i$题的区分度(item discrimination)
- ${\beta_j}$，第$i$题的难度(item difficulty)
- $c_i$，第$i$题的猜对概率(guessing parameter)

### MLE
极大似然估计方法（Maximum Likelihood Estimate，MLE）也称为最大概似估计或最大似然估计，是建立在极大似然原理的基础上的一个统计方法。求极大似然函数估计值的一般步骤：1） 写出似然函数；2）对似然函数取对数；3） 求导数 4） 解似然方程得到参数估计

- 本项目中，$u_{ij}$表示用户$j$是否做第$i$题的结果（0/1），则对应对数似然函数为：
$$L=log(P(U|\theta)) \\
=log[\coprod_{j=1}^{J}\coprod_{i=1}^{N} P_{ij}^{u_{ij}}Q_{ij}^{1-u_{ij}} ] \\
=\sum_{j=1}^{J}\sum_{i=1}^{N}[u_{ij}logP_{ij}+(1-u_{ij})logQ_{ij}] $$

- 结合3PL模型，可得用户$j$是否做第$i$题的对数似然值如下：
$$ ell_{ij}=u_{ij}log[c_i+\frac{(1-c_i)}{1+e^{-z_i}}] + (1-u_{ij})log[(1-c_i)\frac{e^{-z_i}}{1+e^{-z_i}}]$$
	- 项目代码`utl.clib.log_likelihood_2PL`
其中，$z_i=\alpha_i(\theta_j-\beta_i)$


### EM
EM算法指的是最大期望算法（Expectation Maximization Algorithm，又译期望最大化算法），是一种迭代算法，用于含有隐变量（latent variable）的概率参数模型的最大似然估计或极大后验概率估计。

- 输入：观测变量数据$Y$，隐变量数据$Z$，联合分布$P(Y,Z|\theta)$，条件分布$P(Z|Y,\theta)$；
- 输出：模型参数$\theta$
	- (1) 选择参数的初值$\theta^{(0)}$，开始迭代；
		- 参数的初值可以任意选择，但EM算法对初值是比较敏感的
	- (2) E步：记$\theta^{(i)}$为第$i$次迭代参数${\theta}$的估计值，在第$i+1$次迭代的E步，计算
	$$ Q(\theta,\theta^{(i)})=E_Z[logP(Y,Z|\theta)|Y,\theta^{(i)}]=\sum_{Z}logP(Y,Z|\theta)P(Z|Y,\theta^{(i)}) $$
	这里，$P(Z|Y,\theta^{(i)}$是在给定观测数据$Y$和当前参数估计$\theta^{(i)}$下隐变量数据$Z$的条件概率分布
		- $Z$是为观测数据，$Y$是观测数据，$Q(\theta,\theta^{(i)})$中第一个变元表示 要极大化的参数，第2个变元表示参数当前的估计值。每次迭代式实际是求使得$Q$函数取最大值的$\theta$
	- (3) M步：求是$Q(\theta,\theta^{(i)})$极大化的$\theta$，确定第$i+1$次迭代的参数的估计值$\theta^{(i+1)}$
	$$\theta^{(i+1)}=argmax_{\theta}Q(\theta,\theta^{(i)})$$
	- (4) 重复第(2)步和第(3)步，直到收敛
		- 给出迭代的停止条件，一般是对较小的正数$\varepsilon _1,\varepsilon _2
$，若满足
		$$\left \| \theta^{(i+1)}- \theta^{(i)}\right \| < \varepsilon _1 
		或 \left \| Q(\theta^{(i+1)},\theta^{(i)})-Q(\theta^{(i)},\theta^{(i)}) \right \| < \varepsilon _2$$
		则停止迭代
	
## 参数估计
采用EM算法实现MLE，得到对$\Delta ,\pi$的估计
- $\Delta$，题参数
	- 第$j$题的参数记为$\delta _j$，则 $\Delta = (\delta _1,...,\delta _N)$
- $\pi$，能力分布
	- 假设所有用户一共可分为$K$个等级水平，其中第$k$级水平记为$q_k$，对应出现概率为$\pi_k$，则$\pi=(\pi_1,...,\pi_K)$
此外，建模过程中需要用到的其他元素定义如下：
- $n_k^{(s)}$，所用用户中$\theta$（能力）在$k$等级($q_k$)的人数
	- 代码中为`self.item_expected_right_bytheta+self.item_expected_wrong_bytheta`
	- 记$n$为$n_k$的集合向量
- $r_{jk}^{(s)}$，能力$\theta$为$q_k$的并且做对第$j$题的人数
	- 代码中为`self.item_expected_wrong_bytheta`
	- 记$R$为$r_{jk}$的集合矩阵
于是，完全数据下的对数似然函数为(具体推导过程见参考文献)：
$$log[L(R^{(s)},n^{(s)})|\Delta,\pi]=\sum_{j=1}^{J}l(\delta_j)+l(\pi)$$
其中
$$l(\delta_j)=\sum_{k=1}^{K}r_{jk}^{(s)}log[P(q_k|\delta_j)]+(n_{k}^{(s)}-r_{jk}^{(s)})log[1-P(q_k|\delta_j)] $$
$$l(\pi)=\sum_{k=1}^{K}n_{k}^{(s)}log[\pi_k]$$
因此，分开对$l(\delta_j)$和$l(\pi)$求导，从而能使整体对数似然函数达到最大，得到相应的参数估计。
于是应用于本项目的EM算法实现如下：

### E step
在第$(s-1)$次迭代已计算出$\pi^{(s)}$与$\delta^{(s)}$，则此次迭代的E step带入计算

- (1)
$$n_k^{(s)}=\sum_{i=1}^{N}f(q_k|y_i,\Delta^{(s)},\pi^{(s)})$$

- (2)
$$r_{jk}^{(s)}=\sum_{i=1}^{N}u_{ij}f(q_k|y_i,\Delta^{(s)},\pi^{(s)})$$
即，在$n_k^{(s)}$的分子内乘以做第$j$题的结果(0/1)$u_{ij}$


### M step
根据$l(\delta_j)$和$l(\pi)$分开估计$\pi^{(s+1)}$与$\delta^{(s+1)}$
- (1) $$\pi_k^{(s+1)}=\frac{n_k^{(s)}}{N}$$
- (2) 求解 $\frac{\partial l(\delta_j) }{\partial\delta_{tj}} = 0 $，得到$\delta_j^{(s)}$的估计值
	- 代码中主要采用L-BFGS-G（拟牛顿法）求解
	- `from scipy.optimize import minimize , method='L-BFGS-B'`
	
重复以上E和M step直到达到收敛条件，即可得到题库的参数以及用户能力的估计值。


## CAT
计算机自适应考试（Computerized Adaptive Testing，简称CAT）是建构在20世纪50年代发展起来的现代测验理论——项目反应理论（简称IRT）基础上的一种考试方式。
它能根据考生答题的情况不断计算受试者的能力值及信息量，并实时地根据这些参数调整出题策略，最终给受试者一个恰当的评价。

- step1: 首先从题库中随机调取学生初始等级下的一道题
- step2: 根据学答题情况，以及相应题目的3个参数，估计出当前的能力值$\theta^{(s)}$
	- 项目代码`solver.theta_estimator`，目前只实现了MLE的$\theta$估计
- step3: 根据当前能力值，计算可选题库中对于该能力值$\theta^{(s)}$下的信息量
$$I_i(\theta)=\frac{[{P}'_i(\theta)]^2}{P_i(\theta)Q_i(\theta)} $$
选择信息量最大的一道题给予学生，继续作答，同时可计算下一步的$\theta^{(s+1)}$估计量的标准差
$$ SE = \frac{1}{\sqrt{\sum I_i(\theta)}} $$
	- 项目代码`utl.clib.info_item`
- step4: 重复step2-step3，直到标准差在给定的阈值内，或者满足其他停止条件（例如最多只做30题）则结束测试，并给出最后一次估计的能力值$\theta$


## Reference
- [IRT Parameter Estimation using the EM Algorithm](http://www.b-a-h.com/papers/note9801.html)
- https://github.com/17zuoye/pyirt
- https://github.com/xluo11/xxIRT
- https://cran.r-project.org/web/packages/catR/index.html



