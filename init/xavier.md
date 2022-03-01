

[TOC]

## 参考

https://ph0en1xgseek.github.io/2018/02/22/Xavier/

https://zhuanlan.zhihu.com/p/27919794



## Xavier 初始化



​		Xavier 初始化是一种模型训练中的 trick，其目的是为了**减小神经元输入和输出之间的数据分布的gap**，使得模型训练的过程能够更加稳定，收敛速度更快



​		如果是从头训练一个模型，我们一般要对神经网络的参数进行一个初始化，这时候如果无脑把所有参数都初始化成0，那模型就没法训练了（从前向传播的角度来说，全0的时候，每个神经元的输入和输出都是0，那信息全没了；从反向传播的角度来说，误差反向传播的时候乘个0全没了）。

​		所以神经网络的参数应该如何初始化呢？我们在上课的时候一般会听到“随机初始化”这个词，这个“随机”是怎么个随法呢？其实有很多种方法，可以在一个正太分布上随机，可以在一个范围内的均匀分布上随机......无非就是效果好不好的问题。

​		

​		在深度学习的训练中，我们一般会假设训练数据的分布和实际场景中的数据分布是一样的，这样我们训练出来的模型才能在真实场景下进行准确的预测。但是在神经网络内部，需要进行非常多的线性变化、非线性变化运算，我们肯定是希望在这个过程中，尽量**保持每层输入和输出的数据分布一致**。

​		**Xavier通过对限定条件的参数初始化，使得数据在经过模型每层的时候，能够保留均值和方差。**



## 原理



对于一个神经元（假设偏置项为0）
$$
z = w_1x_1 + w_2x_2 + ... + w_nx_n \\
z = \sum^n_{i=1}w_ix_i
$$
n 为输入的数量



根据两个随机变量的方差乘积公式
$$
Var(w_ix_i) = E(w_i)^2Var(x_i) + E(x_i)^2Var(w_i) + Var(w_i)Var(x_i)
$$


假设参数初始化均值为0，输入x均值为0（可以通过 batch normalization 实现）
$$
E(x_i) = E(W_i) = 0
$$
再假设 x 和 w 独立同分布
$$
Var(w_ix_i) = Var(w_i)Var(x_i) \\
Var(z) = \sum^n_{i=1}Var(w_i)Var(x_i) = nVar(w)Var(x)
$$


**整个模型结构，其实就是一个超级复杂的映射，将原始的输入样本稳定地映射到它的类别（或者数值），也就是将样本空间映射到类别空间。**

**那么如果这两个空间如果分布差异很大，比如说类别空间特别稠密，样本空间特别稀疏辽阔，那么在类别空间得到的用于反向传播的误差丢给样本空间后简直变得微不足道，也就是会导致模型的训练非常缓慢。同样，如果类别空间特别稀疏，样本空间特别稠密，那么在类别空间算出来的误差丢给样本空间后简直是爆炸般的存在，即导致模型发散震荡，无法收敛。**

**因此，我们要让样本空间与类别空间的分布差异（密度差别）不要太大，也就是要让它们的方差尽可能相等（或者不要差太多）**



那么既然要让
$$
Var(z) = Var(x)
$$
就要让
$$
nVar(w) = 1 \\
Var(w) = \frac{1}{n}
$$

$$
Var(w)=
\begin{cases}
\frac{1}{n_{in}}, & 前向传播 \\
\frac{1}{n_{out}}, & 反向传播 
\end{cases}
$$

但是输入和输出的数量 n 往往并不相同

所以就取均值
$$
Var(w) = \frac{2}{n_{in} + n_{out}}
$$


- 假设 w 满足 [a,b] 上的**均匀分布（uniform distribution）**

$$
方差 = \frac{(b-a)^2}{12}
$$

**w 和 x 同分布，均值为0，[a,b] -> [-a, a]**
$$
\frac{[(-a)-a]^2}{12} = \frac{2}{n_{in} + n_{out}} \\
a = \sqrt{\frac{6}{n_{in} + n_{out}}}
$$
w 服从均匀分布
$$
U[-\sqrt{\frac{6}{n_{in} + n_{out}}},  \sqrt{\frac{6}{n_{in} + n_{out}}}]
$$


- 假设 w 满足均值为0，方差为std平方的正态分布（normal distribution）

$$
方差 = std^2 = \frac{2}{n_{in} + n_{out}}
$$

w 服从正态分布
$$
N(0, \frac{2}{n_{in} + n_{out}}) \\
\mu = 0 , \sigma = \sqrt{\frac{2}{n_{in} + n_{out}}}
$$


## pytorch



- 文档 **torch.nn.init**

https://pytorch.org/docs/stable/nn.init.html

- 均匀分布

```
torch.nn.init.xavier_uniform_(tensor, gain=1.0)
```

- 正态分布

```
torch.nn.init.xavier_normal_(tensor, gain=1.0)
```
