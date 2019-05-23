---
title: data.table小技巧
date: 2019-05-23
tags: 
	- R 
categories: 
	- 技术语言
	- 小技巧系列
comments: on
---


data.table 作为R中数据处理的利器，本文记录常用的代码。


- 操控数据
	- dt[i,j,k]
	- i：where 选择
	- j：条件，更新，计算；直接对j处的向量进行（自定义）函数操作得到新列
	- k：group by，聚合

	
- 基础操作

``` R
# 根据(x,y)排序
dt[i,j,k][order(x,y)]
# 按照s_id和date_time排序的数据
d1 <- dt[,.(t_id,date_time,s_id)][order(s_id,date_time)]
# 取列和取行方式和data.frame基本通用，但如果直接传入 index向量/列名向量，需要加 with=FALSE
idx = 1:5 
data[,idx,with=FALSE] 
```


- 加列
``` R
# 多加一列，get_day是自定义对单个字符串操作的函数，所以还需要group by 这个向量，就可以对每个字符串操作了
dat[,date_day3:=get_day(as.character(date_time)),date_time]
# 生成新的汇总数据
dat0108[m_type==0,.(t1=.N),mon]
```

- 加行
``` R
re <- rbind(re,cbind('mon'='all',t(colSums(re[,2:4]))))
```

- 设置列名：
``` R
setnames(DT, c("b","b") )
```

- 一次加多列，赋值：
``` R
dat[,`:=`(r_buy_call=a/b,
          x=1)]
```

- 一次删除多列，根据列名：
``` R
x <- x[,colnames(x)[grep('^y_',colnames(x))]:=NULL]
re5 <- re5[,paste0('ft_cnt_',c(1:5)):=NULL]
```

- 去重
``` R
# 直接unique会默认选择之前的key，id进行去重，所以不行
dim(unique(re1))
# 需要加入变量的unique
dim(unique(re1,by=c('id','call_time','call_bridge_duration')))
```

- 一次计算多列
``` R
# 写一个返回list的函数，即可返回多列
myfun <- function (a) {
re <-  strsplit(a,',')[[1]]
return(list(re[1],re[2],re[3],re[4]))
}
tmp1 <- dat[,c('num','A','B','C'):=myfun(V1),V1]
```

- 一次改多列类型
``` R
thecol = c('last6_ftclass','last6_paycnt')
ftpay[, (thecol) := lapply(.SD, as.numeric), .SDcols = thecol]
```

- 分组计算 （使用`.SD`）
``` R
# 按列计算
data[,ldply(.SD,function…)] 
# 按行计算
data[,apply(.SD,1,function…)] 
data[,sum(unlist(.SD)),by=1:nrow(data)]
# 取每组的第一行
tmp[order(subject_no, -time)][, head(.SD, 1), by = list(user_id, subject_no)]
```

- 根据列名group by（传入的可以直接是字符串向量）
``` R
tmp <- re[,.N,by=c('x1','x2','x3')]
```

- 返回值说明
``` R
options(datatable.verbose = TRUE)
# 不显示赋值的返回结果
DT[ , 5] # data.table
DT[2, 5] # data.table
DT[,"region"] # data.table
DT[["region"]] # vector
DT$region # vector
DT[,columnRed:columnViolet]
# column name range to get data.table
DT[,c("columnRed","columnOrange","columnYellow")]
# column name to get data.table
DT[,.(columnRed,columnOrange,columnYellow)]
# column name to get data.table
DT[, { x<-colA+10; x*x/2 }]
# 对X进行计算
DT[ , mycol, with = FALSE]
# mycol = "x"
NEWDT = DT[0]
```


- 变换
``` R
dcast(tmp,user_id~status,value.var ='N')
```


