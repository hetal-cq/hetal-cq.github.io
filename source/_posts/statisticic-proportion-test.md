---
title: 比例检验
date: 2020-01-04 21:04:04
categories: 
	- 统计
	- 假设检验
tags:
	- 比例检验
	- 卡方检验
---


总体比例的假设检验实际上是业界最常用也是最需要的检验，例如在ABtest中，检验两个实验的转化率是否有显著差异，则需要用到比例检验。

## 单总体比率的假设检验
前提条件：
- 样本取自两点分布 $X \sim B(1,p)$
- 样本量$n$很大，能够满足 $np>5$ 且 $n(1-p)>5$

记要检验的原假设为为 $H_0: p=p_0$,则样本比例 $\widetilde{\mathrm{p}}$服从方差为$p(1-p)/n$的正态分布，对应标准化的检验统计量近似服从$N(0,1)$ ：

$$
\mathrm{u}=\frac{\sqrt{\mathrm{n}}\left(\widetilde{\mathrm{p}}-\mathrm{p}_{0}\right)}{\sqrt{\mathrm{p}_0\left(1-\mathrm{p}_0\right)}}
$$

> 实际上，当样本量很少时，需要采用精确的比率检验，即直接使用二项分布来检验，具体实现见下文的R代码。



## 两个总体比率的假设检验

检验前提条件：
- 两总体互相独立
- 变量都取自两点分布，即两总体服从二项分布
- 两总体且每类的样本量满足大于5的要求，从而能用正态分布来近似

那么可知：

$$
\frac{\left(\widetilde{\mathrm{p}}_1-\widetilde{\mathrm{p}}_2\right)-\left(\mathrm{p}_1-\mathrm{p}_2\right)}{\frac{\mathrm{p}_1\left(1-\mathrm{p}_1\right)}{\mathrm{n}_1}+\frac{\mathrm{p}_2\left(1-\mathrm{p}_2\right)}{\mathrm{n}_2}} \approx \frac{\left(\widetilde{\mathrm{p}}_1-\widetilde{\mathrm{p}}_2\right)-\left(\mathrm{p}_1-\mathrm{p}_2\right)}{\frac{\mathbb{p}_1\left(1-\widetilde{\mathrm{p}}_1\right)}{\mathrm{n}_1}+\frac{\widetilde{\mathrm{p}}_2\left(1-\widetilde{\mathrm{p}}_2\right)}{\mathrm{n}_2}} \sim \mathrm{N}(0,1)
$$

因此对应的标准化检验统计量如下：
- 1)当原假设为$H_0:p_1-p_2=0$时，最佳估计量为联合两组样本的比例$\hat{\mathrm{p}}=\frac{\mathrm{x}_1+\mathrm{x}_2}{\mathrm{n}_1+\mathrm{n}_2}=\frac{\mathrm{p}_1 \mathrm{n}_1+\mathrm{p}_2 \mathrm{n}_2}{\mathrm{n}_1+\mathrm{n}_2}$，于是检验统计量如下：

$$
z=\frac{\left(\widetilde{p}_1-\widetilde{p}_2\right)-0}{\frac{\hat{p}(1-\hat{p})}{n_1}+\frac{\hat{p}(1-\hat{p})}{n_2}}=\frac{\left(\tilde{p}_1-\tilde{p}_2\right)}{\hat{p}(1-\hat{p})\left(\frac{1}{n_1}+\frac{1}{n_2}\right)}
$$

- 2)当原假设为$H_0:p_1-p_2=d_0(d_0 \neq 0)$时，直接用两个样本的比例$\widetilde{p}_1$和$\widetilde{p}_2$相应估计两个总体的的比例$p_1$和$p_2$，于是检验统计量入下：
$$
z=\frac{\left(\bar{p}_1-\bar{p}_2\right)-d_0}{\frac{\bar{p}_1\left(1-\bar{p}_1\right)}{n_1}+\frac{\bar{p}_2\left(1-\widetilde{p}_2\right)}{n_2}}
$$


## R语言实现
除了直接用代码写上面的公式，实际上可以直接使用现成的检验函数：`chisq.test` 和 `prop.test`，这两个函数默认都会加Yates 校正（修正小样本的影响，具体见文末），但要得到和上文公式一样的结果，则注意限制`correct = F` 。

- 检验单总体比例是否等于特定的值：
	- `prop.test`，大样本才适用的近似检验
		- 不加Yates 校正和上文公式结果的平方一致（小样本一般都需要加Yates 校正） 
	- `binom.test`，精确的比例检验，即直接使用二项分布检验是否是该比例
- 检验两总体比例是否相等:
	- `chisq.test`，大样本才适应的近似检验
		- 不加Yates 校正和上文公式结果的平方一致（小样本一般都需要加Yates 校正）
		- 正态分布平方，服从自由度为1的卡方分布
		- 注意卡方检验的传参：是两个样本分别取0和1的数量（而不是取1的量和整体量n）
	- Fisher 精确检验，适用于小样本量（此处暂无示例）


### 单总体比率检验

```
x=60
n=2000
p_real=x/n
p0=0.02
u=sqrt(n)*(p_real-p0)/sqrt(p0*(1-p0))
u^2
# 不加校正，得到和u^2一致的结果
prop.test(x, n,p0, correct = F)

# > u^2
# [1] 10.20408
# > prop.test(x, n,p0, correct = F)$statistic
# X-squared 
# 10.20408 
```

实际上单比例检验，推荐直接使用精确的二项分布的假设检验即可。可以看到p值远小于0.05，因此拒绝比率是 $p_0$ 的原假设，认为改总体比率不为$p_0$ 。
```{r}
binom.test(x, n, p0)
# Exact binomial test
# 
# data:  x and n
# number of successes = 60, number of trials =
#     2000, p-value = 0.002974
# alternative hypothesis: true probability of success is not equal to 0.02
# 95 percent confidence interval:
#     0.02296955 0.03844886
# sample estimates:
#     probability of success 
# 0.03 


```


### 两总体比率相等的假设检验

```{r}
# 检验的样本如下：
a=19;b=22;c=52;d=39
pA=a/(a+c);pB=b/(b+d)
nA=a+c;nB=b+d

# 借助chisq.test检验两总体比率是否一致
a1 <- rbind(c(nA*(1-pA),nA*pA), c(nB*(1-pB), nB*pB))
# 此处为了保持一致未加Yates 校正，但实际应用最好设置 correct=T
chisq.test(a1,correct=F)

# 直接计算统计量，和上面卡方检验的X-squared一致，都是 1.326693
p_fit=(a+b)/(a+b+c+d)
z2=(pA-pB)/sqrt((1/nA+1/nB)*p_fit*(1-p_fit))
z2^2
```

```
# > chisq.test(a1,correct=F)
# 
# Pearson's Chi-squared test
# 
# data:  a1
# X-squared = 1.3267, df = 1, p-value = 0.2494
# 
# > z2^2
# [1] 1.326693
```

可以看到p值大于0.05，认为无法拒绝原假设，即$p_1=p_2$ 

> 注意只是在这次样本里找不到拒绝的理由，不代表这两个比例就一定相等了


### Yates 校正

小样本情况时Yates 校正后的卡方统计量可信度更高（大样本时对其影响很微弱），因此我们最常用加了Yates 校正后的比例检验，对应Yates 校正实际上是对卡方分布的统计量的修正，取平方之前将正偏差(观测-期望)减0.5，负偏差时加0.5，对应公式如下：

$$
\mathrm{X}^2((n_1-1)*(n_2-1)) = \sum_{i=1}^{n_1}\sum_{j=1}^{n_2} \frac{\left(\left|O_{ij}-E_{ij}\right|-0.5\right)^2}{\mathrm{E}_{\mathrm{ij}}}
$$