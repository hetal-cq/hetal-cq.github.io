---
title: 异常处理-R与Python
date: 2019-09-08
tags: 
	- R 
	- Python
categories: 
	- 技术语言
comments: on
---

本文将简单介绍R和Python中常用的异常处理方式。


## R
R中常用`try`和`tryCatch`
### try
如果只是为了某段代码不影响后续脚本的执行，最简单的办法就是直接在外层套上`try()`

```{r}
# 如果该语句失败也没关系，那么可以直接用偷懒的写法:try
print('-- 开始 --')
try(o <- 'error'+2) # 会输出报错，但是程序不会停止，接下来的代码可以继续运行
print('-- 结束 --')
```

```
# [1] "-- 开始 --"
# Warning message:
# In strsplit(code, "\n", fixed = TRUE) :
  # input string 1 is invalid in this locale
# Error in "error" + 2 : non-numeric argument to binary operator
# [1] "-- 结束 --"
```

### trycatch
最常见的异常处理函数是`tryCatch`，能够让代码更加flexible

#### 简单示例:
- `'error'+2`报错
- 错误记录在`e`中 并打印
- 报错后，对应变量 `o` 被赋值为3

```{r}
tryCatch({o <- 'error'+2}, error = function(e) print(e), call = (o <- 3))
print(o)
# [1] 3
```

#### call的使用
为了防止一段代码出错，保证后续继续运行，并记录具体出错的位置，可以结合 `call` 使用

```{r}
error_ls <- c()
i <- 0
for(x in c(-2:2)){
    i <- i+1
    tryCatch(
        expr={x1 <- ifelse(x>0, x, "make error") + 1
        error_ls[i] <- 'success'
        },
        error = function(e){
            cat('Error: ', e$message, 'index: ',i,'\n')}, 
        call = (
            # 如果报错，则对 x1 和 error_ls 重赋值
            {x1 <- 100
            error_ls[i] <- 'error'}))
    cat('x1: ',x1,'\n')
}
```
`x<=0`时都会报错，并给x1重赋值为100，`x>0`时则正常运行
```
# Error:  non-numeric argument to binary operator index:  1 
# x1:  100 
# Error:  non-numeric argument to binary operator index:  2 
# x1:  100 
# Error:  non-numeric argument to binary operator index:  3 
# x1:  100 
# x1:  2 
# x1:  3 
```
`error_ls`已记录对应每次循环的正误
```
print(error_ls)
```

注：call函数会优先执行expr的内容，如果执行失败，则执行call内部的语句，但若call内部出现其它赋值语句，则始终会被执行（会认为expr中没有相应返回），例如:
```{r}
error_ls2 <- c()
i <- 0
for(x in c(-2:2)){
    i <- i+1
    tryCatch(
        expr={1+1},
        call = (
            # 对expr中不包含的变量赋值，每次都会执行
            {error_ls2[i] <- 'error'}))
}
```
即时没有任何异常，依然会执行`error_ls2[i] <- 'error'`
```{r}
print(error_ls2)
# [1] "error" "error" "error" "error" "error"
```

#### 包装成函数
包装成函数，不使用循环，转为在 data.table中快速运行

```{r}
# 写一个会遇到错误的函数
process <- function(x){
    # 显然，x<=0时，'make error'+1 字符串和数值不能相加，该函数会报错
    ifelse(x>0, x, "make error") + 1
}

# 嵌套异常处理后的函数
process_tryCatch <- function(x){
    tryCatch(expr=process(x), 
             error=function(e) {
                 # 如果报error，则输出对应报错，并直接返回 -1001
                 cat('Error: ', e$message, ". Input: ", x, "\n")
                 # 此处不再需要call，可以直接在error里写return
                 return(-1001)
             }
    )
}
```
构造个简单的data.table, 对`a`列的每个元素做process运算

```{r}
library(data.table)
df = data.table(a=-2:2)
df[, new_a := process_tryCatch(a),a]
# Error:  non-numeric argument to binary operator . Input:  -2 
# Error:  non-numeric argument to binary operator . Input:  -1 
# Error:  non-numeric argument to binary operator . Input:  0
```
对于`>0`的会正常返回，其他返回`tryCatch`设置的 `-1001`
```{r}
df
    # a new_a
# 1: -2 -1001
# 2: -1 -1001
# 3:  0 -1001
# 4:  1     2
# 5:  2     3
```



## Python
### try-except
`try-except`是python中常用异常处理方式，如果`try`语句块中发生异常，则执行`except`语句块（捕获异常并处理）。示例如下

```{python}
import sys 
try:
    re = "error" + 2
except Exception as e: 
    s = sys.exc_info()
    print('Error: %s; line:[%s]' % (str(e), s[2].tb_lineno))
```

- `sys.exc_info()` 记录错误出现的位置（问题回溯）
- `Exception as e` 记录具体的报错信息。

以上示例运行结果如下：

```
# Error: cannot concatenate 'str' and 'int' objects; line:[3]
```







