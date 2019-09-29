# xgboost paper阅读

## 1. Introduction

xgboost主要有如下的贡献：

* 设计并构建了适用于大规模的 end-to-end 的Boosting系统（end-to-end指的是端到端，就是只关心输入和输出，中间过程都不care）
* 提出了理论证明的weighted quantile sketch（分布式加权直方图）来有效生成候选分裂点
* 介绍了树并行学习使用的稀疏感知算法
* 提出了cache-aware block结构来进行out of core（基于外存)学习



## 2. Tree Boosting in a nutshell

主要介绍gradient tree boosting算法，其中的想法来自在gradient boosting，尤其是Friedman的second order method。我们做了一些小的提升，并在实际中证明有效。

### 2.1 Regularized Learning Objective

$$
\tag{1}
\hat{y_i} = \phi(X_i) = \sum_{k=1}^{K}f_k(X_i), f_k \in F,    (1) \\

wheres F = {f(X) = \omega_{q(x)}} (q: R^m \rightarrow T, \omega \in R^T)
$$



 $q$表示每棵树的结构，可以将输入映射到叶子结点，$T$是叶子结点数量，$f_k$ 表示第k棵树，包含$q$和叶子结点$\omega$ 。与决策树不同，回归树的叶子结点是连续的分数，用$\omega_i$来表示第$i$个叶子结点的权重。我们$q$表示的规则来判断，将例子分配到叶子结点上，并累加所有树对应叶子结点的权重来进行最终的预测。
$$
\tag{2}
L(\theta) = \sum_il(\hat y_i,y_i) + \sum_k \Omega(f_k)\\
where \quad \Omega(f) = \gamma T = \frac{1}{2} \lambda\parallel\omega\parallel^2
$$


$l$ 是可微凸损失函数，表示预测值和目标值的差值。第二部分的正则项通过平滑权重来防止过拟合。正则项倾向于选择简单&预测准的模型。RGF(Regularized greedy forest)也使用了类似的正则技术，但是xgboost使用的正则项更简单并且可以更方便的并行。如果正则项参数配置为0，就变成了常规的梯度提升树(gradient tree boosting)。



### 2.2 Gradient Tree Boosting

公式$Eq.\tag{2}$ 不能用欧氏空间中传统的方法来优化，模型是用加法的方式来训练的。

令$\hat{y_i}^{(t)}$表示第$i$ 个例子在第$t$轮的预测值，我们添加$f_t$来最小化下面公式
$$
L^{(t)} = \sum_{i=1}^{n}l(y_i,\hat{y_i}^{(t-1)}) + f_t(x_i) + \Omega(f_t)
$$
 这意味着我们使用贪婪算法来选择使优化$Eq.\tag{2}$。二阶逼近可以用来更快的优化。
$$
L^{(t)} \approx\ \sum_{i=1}^{n}[l(y_i,\hat{y}^{t-1}) + g_if_t(x_i) + \frac{1}{2}h_if_t^2(x_i)] + \Omega(f_t) \\
where \: g_i =\delta_{\hat{y}^{t-1}}l(y_i,\hat{y}^{t-1}) \: and h_i = \delta^2_{\hat{y}^{t-1}}l(y_i,\hat{y}^{t-1})
$$
移除常数项后，得到如下简化的目标。
$$
\tag{3}
\tilde{L^t} = \sum_{i=1}^{n}[g_if_t(x_i) = \frac{1}{2}h_if_t^2(x_i)] + \Omega(f_t)
$$
