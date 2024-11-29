
　　[Deep Variational Information Bottleneck (VIB) 变分信息瓶颈](https://github.com) 论文阅读笔记。本文利用变分推断将信息瓶颈框架适应到深度学习模型中，可视为一种正则化方法。


# 1  变分信息瓶颈


　　假设数据输入输出对为$(X,Y)$，假设判别模型$f\_\\theta(\\cdot)$有关于$X$的中间表示$Z$，本文旨在优化$\\theta$以最小化互信息$I(Z;X)$ ，同时最大化互信息$I(Z;Y)$，即：


$\\max\\limits\_{\\theta}I(Z;Y\|\\theta)\-\\beta I(Z;X\|\\theta)$


　　其中$\\beta\>0$为平衡系数。直觉理解，上式期望$Z$能保留更少$X$信息的同时能较好用于预测$Y$。那么如何构造这个模型以及相应的优化方案？下面推导上式的下界，使其下界变大，上式即可变大。为了简化，下面去掉$\\theta$进行推导。


## 1\.1  下界1


　　首先，$I(Z;Y)$展开为：


$\\displaystyle I(Z; Y) \= \\int \\int p(y, z) \\log \\frac{p(y\|z)}{p(y)} \\, dy \\, dz$


　　其中$p(y)$是数据的标签分布，已知。未知而需要进行处理的是其中的$p(y,z)$和$p(y\|z)$，也就是模型需要拟合的分布。对于$p(y\|z)$，可以用一个解码器$q(y\|z)$来拟合，即文中所谓的变分估计。利用KL散度的大于零性质，有以下不等式：


\\begin{align\*} \&\\text{KL}(p(Y\|Z),q(Y\|Z))\\geq 0\\\\ \\implies \&\\int \\, p(y\|z) \\log \\frac{p(y\|z)}{q(y\|z)} dy\\geq 0\\\\ \\implies \&\\int \\, \\frac{p(y,z)}{p(z)} \\log \\frac{p(y\|z)}{q(y\|z)} dy\\geq 0\\\\ \\implies \&\\int \\, p(y,z) \\log p(y\|z) dy\\geq \\int \\, p(y,z) \\log q(y\|z)dy\\\\ \\end{align\*}


　　则有


\\begin{align\*} \\displaystyle I(Z; Y) \&\= \\int \\int p(y, z) \\log p(y\|z) \- p(y, z) \\log p(y) \\, dy \\, dz\\\\ \&\\geq \\int \\int p(y, z) \\log q(y\|z) \- p(y, z) \\log p(y) \\, dy \\, dz\\\\ \&\= \\int \\int \\, p(y, z) \\log q(y\|z) dy \\, dz \- \\int \\, p(y) \\log p(y) dy \\\\ \&\= \\int \\int \\, p(y, z) \\log q(y\|z)dy \\, dz \+ H(Y) \\end{align\*}


　　对于其中的$p(y,z)$，本文基于马尔科夫假设：$Y\\leftrightarrow X\\leftrightarrow Z$。这个假设表明，$Y$和$Z$在$X$的条件下独立（那在优化时呢？$Z$是关于$X$和$Y$的联合分布进行更新的）。有：


$\\displaystyle p(y,z)\=\\int p(x,y,z)dx\=\\int p(x,y)p(z\|x,y)dx\= \\int p(x,y)p(z\|x)dx$


　　此外，由于$H(Y)$已知且固定，可忽略。则有


$\\displaystyle I(Z; Y) \\geq \\int \\int \\int  \\, p(x,y)p(z\|x) \\log q(y\|z)dx \\, dy \\, dz$


　　其中，$p(x,y)$是真实数据分布，$p(z\|x)$是原始模型关于$x$对中间表示$z$的推理分布。


## 1\.2  上界2


　　$I(Z;X)$展开为：


$\\displaystyle I(Z;X)\=\\int \\int p(x,z)\\log \\frac{p(z\|x)}{p(z)}dx\\,dz$


　　对于其中的$p(z)$，作者用另一个变分估计$r(z)$来拟合。由于有


\\begin{align\*} \&\\text{KL}(p(Z),r(Z))\\geq 0\\\\ \\implies\&\\int p(z)\\log p(z)dz\\geq\\int p(z)\\log r(z)dz\\\\ \\implies\&\\int\\int p(x,z)\\log p(z)dx\\,dz\\geq\\int \\int p(x,z)\\log r(z)dx\\,dz \\end{align\*}


　　则有


\\begin{align\*} I(Z;X)\\leq \\int\\int p(x)p(z\|x)\\log \\frac{p(z\|x)}{r(z)}dx\\,dz \\end{align\*}


## 1\.3  总体下界和优化


　　结合下界1和上界2，有：


\\begin{align\*} \&I(Z; Y) \- \\beta I(Z; X) \\\\ \\geq\&\\int \\int \\int \\, p(x,y)p(z\|x) \\log q(y\|z)dx \\, dy \\, dz \- \\beta \\int\\int p(x)p(z\|x)\\log \\frac{p(z\|x)}{r(z)}dx\\,dz \= L \\end{align\*}


　　针对上式，用经验分布来代替真实分布。即用$\\frac{1}{N}\\sum\_{n\=1}^N\\delta\_{x\_n}(x)$代替$p(x)$，用$\\frac{1}{N}\\sum\_{n\=1}^N\\delta\_{y\_n}(y)$代替$p(y)$，用$\\frac{1}{N}\\sum\_{n\=1}^N\\delta\_{(x\_n,y\_n)}(x,y)$代替$p(x,y)$。其中$\\delta\_{x\_n}(x)$表示狄拉克函数，其空间内积分为1，且仅在$x\_n$上非零。假设经验分布有$N$各样本$\\{(x\_n,y\_n)\\}\_{n\=1}^N$。文中额外引入所谓狄拉克函数让人看不懂，实际上直接把概率积分改成离散样本的求和取平均即可。则上式可被估计为：


$\\displaystyle L\\approx \\frac{1}{N}\\sum\\limits\_{n\=1}^N\\int p(z\|x\_n)\\log q(y\_n\|z)\-\\beta\\, p(z\|x\_n)\\log\\frac{p(z\|x\_n)}{r(z)}\\, dz$


　　文中将$z$视为隐变量，利用VAE的重参数技巧将$p(z\|x\_n)$实现为一个关于$x\_n$的正态分布$\\mathcal{N}(f\_e^\\mu(x\_n),f\_e^{\\Sigma}(x\_n))$，其中$f\_e^\\mu(x\_n),f\_e^\\Sigma(x\_n)$分别为基于$x\_n$生成的均值和协方差矩阵。将$z$抽样表示为$f(x\_n,\\epsilon)\= f\_e^{\\Sigma}(x\_n))\\epsilon \+ f\_e^\\mu(x\_n) $，其中$\\epsilon\\sim \\mathcal{N}(0,1\)$。则最大化$L$可表示为最小化：


$\\displaystyle J\_{IB} \= \\frac{1}{N}\\sum\\limits\_{n\=1}^N \\mathbb{E}\_{\\epsilon\\sim \\mathcal{N}(0,1\)}\\left\[\-\\log q(y\_n\|f(x\_n,\\epsilon))\\right] \+ \\beta\\,\\text{KL}\\left\[p(Z\|x\_n);r(z)\\right]$


　　其中$r(z)$利用某一特定分布实现，文中使用标准正态分布实现。


# 2  直觉理解


**直觉上理解：**模型要把每个$x\_n$分别映射到特定的分布，这些分布既不能偏离标准正态分布太远，又需要让模型后续能根据这些分布的抽样来预测$x\_n$的标签。那么这种做法为什么能从$x\_n$中抽取对预测$y\_n$有效的关键信息而忽略无关信息呢（即信息瓶颈）？我的理解是，模型被惩罚以使不同$x\_n$得到的$z\_n$分布靠近同一分布，但为了有效预测$y\_n$，又必须产生一定的不一致。不同$x\_n$对应的$z$分布越一致，通过$z$而流向$y$的差异性信息将越少，导致$q$更难利用采样的$z$预测$y$，从而促使模型忽略$x$中的冗余信息而保留预测$y$所需的关键信息。$\\beta$则用于控制$z$保留$x$信息的程度，越大保留信息越少。


**相较于一般的判别模型：**当不把$z $视为隐变量，而变成关于$x$唯一确定的中间表示时，就是一般的判别模型。这种方式隐式地假定了表示的连续性，然而无法确保所有$z$都不是被离散地分散在表示空间中。最坏的过拟合情况下，每个$(x\_n,y\_n)$都孤立地确定了一个中间表示$z\_n$来实现一一映射，导致无泛化。而对于使用了信息瓶颈$z$的判别模型，由于$x$仅仅确定$z$的生成分布，不同的$x\_i,x\_j$依然可能抽样出同一个$z$，这种模式强制这个抽样出的$z $必须共享这两个样本的相似信息并忽略不同的信息，从而表示语义的相似性被强制由线性距离控制，实现表示语义的连续性，从而显式地确定了模型的泛化。


# 3  实验


　　表1：信息瓶颈加成的模型和各种正则化后模型的对比。


　　图1：不同$\\beta$、$z$维度$K$下VIB模型在MNIST上的错误率，以及两个互信息的平衡。


　　图2：$z$维度$K\=2$时，1000张图片的$z$分布的可视化。


　　后续是一些对抗鲁棒的实验，不记录


 本博客参考[wgetcloud全球加速器服务](https://wgetcloud6.org)。转载请注明出处！
