<!-- TITLE: 支持向量机 -->
<!-- SUBTITLE:  -->

# Overview
在机器学习中，支持向量机（Support Vector Machine，SVM）是在分类与回归分析中分析数据的监督式学习模型与相关的学习算法。[1][1]从本质上来说是一种：**用一条线（方程）分类两种事物**。[2][2]其中，用于分类（Classification）的SVM被称为SVC，用于回归（Regression）的被称为SVR。

![SVM Overview](/uploads/2018/svm-overview.jpg "SVM Overview")

上图中，每一个圆形与正方形都代表一个样本数据。对SVC来说，SVM的任务是在两类样本数据中找到一条分界线（*超平面，hyperplane*），使得它到两边的*间隔（margin）*都最大。所谓支持向量机，分为两个部分：
- *支持向量（Support Vector）*：简单来说，就是支持或支撑超平面上把两异类划分开来的超平面的向量点。在Maximum Margin上的这些训练样本点距离超平面最近，他们被成为支持向量。具体来说就是最终分类器的表达式中只含有这些支持向量的信息，与其他的训练样本点无关。两个异类支持向量到超平面的距离之和称之为间隔$\gamma$。
- *机（Machine）*：一个算法。在机器学习领域，常把一些算法看做是一个机器，如分类机（也叫做分类器）。

**SVM的目标是找到具有最大间隔（Maximum Margin）划分的超平面。**

SVM是建立在统计学习理论的*VC维*理论和*结构风险*最小原理基础之上的，根据有限的样本信息在模型的复杂性（即对特定训练样本的学习精度，Accuracy）和学习能力（即无错误地识别任意样本的能力）之间寻求最佳折衷，以期获得最好的*推广能力（或称泛化能力）*。[4][4]

> Note：
>  - 超平面
>> 在数学中，超平面是$n$维度欧里几得空间中余维度等于1的线性子空间。这是平面中直线、空间中平面的推广。[8][8]
> - VC维
>> VC维可用来刻画分类系统的性能，往往说：VC维越大，学习能力就越强，问题的复杂度也越高。它的物理意义在于：如果我们将假设集合的数量|H|比作假设集合的自由度，那么VC维就是假设集合在做二元分类的有效的自由度，即这个假设空间能够产生多少Dichotomies的能力（VC维说的是，到什么时候，假设集合还能shatter，还能产生最多的Dichotomies）。
>> 如果VC维很小，那么发生预测偏差很大的可能性也就很小，但是假设空间的表达能力受到了限制，随机抽样得到的样本错误率就可能会很大；VC维很大，假设空间的容量也变得很大，尽管样本的错误率很低，但是样本之外的错误率很大。
>> SVM关注的是VC维，与样本的维数是无关的。[4][4][5][5][6][6][7][7]
> - 结构风险
>> 机器学习的本质是对真实模型的逼近，选择的模型与真实模型之间的误差叫做*风险*（更严格地说，积累的误差叫做风险）。使用分类器在样本数据上分类的结果与真实结果之间的差值叫做*经验风险（empirical risk）*。经验风险最小化会导致在样本集上轻易达到100%正确率，可真实分类时却一塌糊涂（即所谓的*推广能力*差，或*泛化能力*差）。统计学习因此引入了*泛化误差*的概念，即真实风险由经验风险与*置信风险*（代表了多大程度上可以信任分类器在未知文本上分类的结果）。置信风险与样本数量和分类函数的VC维有关。样本数量越大，置信风险越小；分类函数的VC维越高，置信风险越大。
>> 有公式：真实风险 $\leq$ 经验风险$+$置信风险
>> 统计学习的目标从经验风险最小化变为寻求经验风险与置信风险的和最小化，即*结构风险（structural risk）*最小化。[4][4]

# SVC
给定训练样本集$D = {(\boldsymbol {x}_1, y_1), (\boldsymbol {x}_2, y_2), ..., (\boldsymbol {x}_m, y_m); y_i \in \{-1, +1 \}}$。

![Hard Margin](/uploads/2018/hard-margin.png "Hard Margin")

在样本空间中，超平面可通过如下的线性方程来描述（$\boldsymbol {w}$为空间法向量，决定超平面的方向；$b$为位移项，决定了超平面与原点之间的距离）：
$$
\begin{align}
\boldsymbol {w}^{T} \boldsymbol {x} + b = 0 \\
s.t. \quad\boldsymbol {w} = (w_1; w_2; ...; w_d)
\end{align}
$$

> Note: *s.t.*为*subject to*的缩写，限制条件之意。

![Hard Margin](/uploads/2018/hard-margin.png "Hard Margin")

空间中的任意一点$\boldsymbol {x}$到超平面的几何距离为：
$$
r = \frac {\left| \boldsymbol {w}^{T} \boldsymbol {x} + b \right|} {\left \| \boldsymbol {w} \right \|}
$$

假设超平面能将训练样本正确分类，即$(\boldsymbol {x}_i, y_i) \in D$，有：
$$
\begin{cases}
\boldsymbol {w}^{T} + b \geq +1, & y_i = +1 \\
\boldsymbol {w}^{T} + b \leq -1, & y_i = -1
\end{cases}
$$

可得间隔的表达式为：
$$
\gamma = \frac{2}{\left \| \boldsymbol {w} \right \|}
$$

## 硬间隔（Hard Margin）上的原始问题

硬间隔指的是：假设这些数据全部线性可分，即存在一个超平面能将不同类的样本完全划分开，所有样本都必须划分正确。

为了在最大化间隔下求得超平面，有：
$$
\begin{align}
& \max\limits_{\boldsymbol{w}, b} \frac {2} {\left \| \boldsymbol {w} \right \|} \\
& s.t. \  y_i ( \boldsymbol{w}^{T} \boldsymbol{x}_{i} + b) \geq 1, & i = 1, 2, ..., m
\end{align}
$$

仅需最大化$\left \| \boldsymbol {w} \right \| ^ {-1}$，这等价于最小化$\left \| \boldsymbol {w} \right \| ^{2}$，即：
$$
\begin{align}
& \min\limits_{\boldsymbol{w}, b}\frac{1}{2}\left \| \boldsymbol {w}^2 \right \| \\
& s.t. \  y_i ( \boldsymbol{w}^{T} \boldsymbol{x}_{i} + b) \geq 1, & i = 1, 2, ..., m
\end{align}
$$

> Note:
> - $\max\limits_{\boldsymbol{w}, b} \frac {2} {\left \| \boldsymbol {w} \right \|}$的含义为当$\frac {2} {\left \| \boldsymbol {w} \right \|}$取得最大值时，$\boldsymbol{w}$与$b$的取值。
> - $\left \| \boldsymbol {w} \right \| ^ {-1}$与$\left \| \boldsymbol {w} \right \| ^{2}$的等价与[正则化](TODO)有关。
> - $y_i ( \boldsymbol{w}^{T} \boldsymbol{x}_{i} + b) \geq 1$是一个trick，用于符号的转换。

这是一个*凸优化*[9][9]问题，更具体地说，这是一个*二次优化（ Quadratic Programming，QP）*[10][10]问题，即凸二次优化问题，属于*运筹学*[11][11]的范畴。它可以用任何现成的二次规划算法求解。

## 对偶问题（Dual Problem）
这个凸二次优化问题有着特殊的结构，可以通过*拉格朗日乘数（Lagrange Multiplier）*[12][12]将一个有$n$个变量与$k$个约束条件的最优化问题转换为一个解有$n + k$个变量的方程组的解的问题，得到其对偶问题。

根据$m$个约束条件，引入$m$个*拉格朗日乘子*，记为$\boldsymbol {\alpha} = (\alpha_1, \alpha_2, ..., \alpha_m)$，则该问题的拉格朗日函数可写为：
$$
\mathcal{L}(\boldsymbol{w}, b, \boldsymbol{\alpha}) = \frac{1}{2}\left \| \boldsymbol{w} \right \|^{2} + \sum_{i=1}^{m} \alpha_i (1 - y_i(\boldsymbol{w}^T\boldsymbol{x}_i + b))
$$

对$\mathcal{L}(\boldsymbol{w}, b, \boldsymbol{\alpha})$关于$\boldsymbol{w}$和$b$求极值，可得条件：
$$
\boldsymbol{w} = \sum_{i=1}^{m}\alpha_i y_i \boldsymbol{x}_i
$$

代回原公式，可得到*对偶形式（Dual Representation）*：
$$
\max_{\boldsymbol{\alpha}} = \sum_{i=1}^{m}\alpha_i - \frac{1}{2} \sum_{i=1}^{m}\sum_{j=1}^{m}\lambda_i \lambda_j \boldsymbol{x}_i^T \boldsymbol{x}_j \\
\begin{align}
& s.t. & \quad \sum_{i=1}^{m}\alpha_i y_i = 0, \\
&& \alpha_i \geq 0, \ i = 1, 2, 3, ..., m
\end{align}
$$

解出$\boldsymbol{\alpha}$后，可得到模型：
$$
\begin{split}
f(\boldsymbol{x}) &= \boldsymbol{w}^T \boldsymbol{x} + b \\
&= \sum_{i=1}^{m}\alpha_i y_i \boldsymbol{x}_i^T \boldsymbol{x} + b
\end{split}
$$

### Karush-Kuhn-Tucker（KTT)条件

# References

[1]: https://zh.wikipedia.org/wiki/支持向量机 "Wikipedia: 支持向量机"
[2]: https://charlesliuyx.github.io/2017/09/19/%E6%94%AF%E6%8C%81%E5%90%91%E9%87%8F%E6%9C%BASVM%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/ "【直观详解】支持向量机SVM"
[3]: http://www.blogjava.net/zhenandaci/category/31868.html "SVM入门系列"
[4]: http://www.blogjava.net/zhenandaci/archive/2009/02/13/254519.html "SVM入门系列：SVM入门（一）至（三）Refresh"
[5]: https://whuhan2013.github.io/blog/2017/02/13/vc-theroy-learn/ "机器学习基石之VC维理论"
[6]: https://my.oschina.net/hosee/blog/471475 "VC维再理解"
[7]: http://www.flickering.cn/machine_learning/2015/04/vc%E7%BB%B4%E7%9A%84%E6%9D%A5%E9%BE%99%E5%8E%BB%E8%84%89/ "VC维的来龙去脉"
[8]: https://zh.wikipedia.org/wiki/%E8%B6%85%E5%B9%B3%E9%9D%A2 "Wikipedia: 超平面"
[9]: https://zh.wikipedia.org/wiki/%E5%87%B8%E5%84%AA%E5%8C%96 "Wikipedia: 凸优化"
[10]: https://zh.wikipedia.org/wiki/%E4%BA%8C%E6%AC%A1%E8%A7%84%E5%88%92 "Wikipedia: 二次优化"
[11]: https://zh.wikipedia.org/wiki/%E9%81%8B%E7%B1%8C%E5%AD%B8 "Wikipedia: 运筹学"
[12]: https://zh.wikipedia.org/wiki/%E6%8B%89%E6%A0%BC%E6%9C%97%E6%97%A5%E4%B9%98%E6%95%B0 "Wikipedia: 拉格朗日乘数"