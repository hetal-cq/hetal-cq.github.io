---
title: 变量分箱
date: 2020-01-04 20:49:18
categories: 
	- 特征处理
tags:
	- 分箱
---

特征处理实际是整个建模过程中耗时最多，实际也是最重要最有意义的部分，所谓巧妇难为无米之炊。当我们有了米该如何淘呢？那么变量分箱则是实现'模型稳健性'+'业务实用性'的一大杀器。

我最喜欢的分箱方法：对x直接建决策树，参考其分割点，结合业务背景微调成为想要的cutoff。

示例R代码如下：

```{r}
library(partykit)
get_bins <- function(df,x,y,p=0.05){
  print(paste('----- x: ',x,' ---------'))
  df=as.data.frame(df)
  ctree = ctree(formula(paste(y, "~", x)), data = df, na.action = na.exclude, 
                control = ctree_control(minbucket = ceiling(round(p * nrow(df)))))
  bins = width(ctree)
  if (bins < 2) {
    return(list('ctree'="No significant splits",
                'cutvct'="No significant splits"))
  }
  cutvct = data.frame(matrix(ncol = 0, nrow = 0))
  n = length(ctree)
  for (i in 1:n) {
    cutvct = rbind(cutvct, ctree[i]$node$split$breaks)
  }
  cutvct = cutvct[order(cutvct[, 1]), ]
  return(list('ctree'=ctree,'cutvct'=cutvct))
}

get_var_cut <- function(cut_name = 'eg',
                        the_breaks = c(-1,2,3),
                        dt=dat_model_data,
                        the_labels=NA){
  # 根据传入的指标和分割点，给数据框新增cut后的一列，命名以_cut结尾
  if(is.na(the_labels)){
    the_labels  <- paste0(the_breaks[-length(the_breaks)],'~',the_breaks[-1])
  }
  # 注意，cut是左开右闭类型( ]
  re_cut <- cut(dt[[cut_name]],breaks = the_breaks,labels = the_labels)
  dt[[paste0(cut_name,'_cut')]] <- re_cut
  return(dt)
}
```

以上为工作中总结出的代码，屡试不爽，具体原理和和科学分析，且听下回分解...