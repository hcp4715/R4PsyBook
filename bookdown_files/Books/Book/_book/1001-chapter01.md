# 格兰格因果性 {#causal}



## 介绍 {#causal-intro}

考虑两个时间序列之间的因果性。
这里的因果性指的是时间顺序上的关系，
如果$X_{t-1}, X_{t-2}, \dots$对$Y_t$有作用，
而$Y_{t-1}, Y_{t-2}, \dots$对$X_t$没有作用，
则称$\{X_t \}$是$\{ Y_t \}$的格兰格原因，
而$\{ Y_t \}$不是$\{ X_t \}$的格兰格原因。
如果$X_{t-1}, X_{t-2}, \dots$对$Y_t$有作用，
$Y_{t-1}, Y_{t-2}, \dots$对$X_t$也有作用，
则在没有进一步信息的情况下无法确定两个时间序列的因果性关系。

注意这种因果性与采样频率有关系，
在日数据或者月度数据中能发现的领先——滞后性质的因果关系，
到年度数据可能就以及混杂在以前变成同步的关系了。


## 格兰格因果性的定义 {#causal-def}

设$\{ \xi_t \}$为一个时间序列，
$\{ \boldsymbol{\eta}_t \}$为向量时间序列，
记
$$\begin{aligned}
\bar{\boldsymbol{\eta}}_t =& \{ \boldsymbol{\eta}_{t-1}, \boldsymbol{\eta}_{t-2}, \dots \} 
\end{aligned}$$

记
$\text{Pred}(\xi_t | \bar{\boldsymbol{\eta}}_t)$为基于
$\boldsymbol{\eta}_{t-1}, \boldsymbol{\eta}_{t-2}, \dots$
对$\xi_t$作的最小均方误差无偏预报，
其解为条件数学期望$E(\xi_t | \boldsymbol{\eta}_{t-1}, \boldsymbol{\eta}_{t-2}, \dots)$，
在一定条件下可以等于$\xi_t$在$\boldsymbol{\eta}_{t-1}, \boldsymbol{\eta}_{t-2}, \dots$张成的线性Hilbert空间的投影
（比如，$(\xi_t, \boldsymbol{\eta}_t)$为平稳正态多元时间序列），
即最优线性预测。
直观理解成基于过去的$\{\boldsymbol{\eta}_{t-1}, \boldsymbol{\eta}_{t-2}, \dots \}$的信息对当前的$\xi_t$作的最优预测。

令一步预测误差为
$$
  \varepsilon(\xi_t | \bar{\boldsymbol{\eta}}_t) 
  = \xi_t - \text{Pred}(\xi_t | \bar{\boldsymbol{\eta}}_t)
$$
令一步预测误差方差，或者均方误差，
为
$$
  \sigma^2(\xi_t | \bar{\boldsymbol{\eta}}_t)  
  = \text{Var}(\varepsilon_t(\xi_t | \bar{\boldsymbol{\eta}}_t) )
  = E \left[ \xi_t - \text{Pred}(\xi_t | \bar{\boldsymbol{\eta}}_t) \right]^2
$$


考虑两个时间序列$\{ X_t \}$和$\{ Y_t \}$，
$\{(X_t, Y_t) \}$宽平稳或严平稳。

* 如果
$$
\sigma^2(Y_t | \bar Y_t, \bar X_t) < \sigma^2(Y_t | \bar Y_t)
$$
则称$\{ X_t \}$是$\{ Y_t \}$的**格兰格原因**，
记作$X_t \Rightarrow Y_t$。
这不排除$\{ Y_t \}$也可以是$\{ X_t \}$的格兰格原因。
* 如果$X_t \Rightarrow Y_t$，而且$Y_t \Rightarrow X_t$，
则称互相有**反馈**关系，
记作$X_t \Leftrightarrow Y_t$。
* 如果
$$
\sigma^2(Y_t | \bar Y_t, X_t, \bar X_t) < \sigma^2(Y_t | \bar Y_t, \bar X_t)
$$
即除了过去的信息，
增加同时刻的$X_t$信息后对$Y_t$预测有改进，
则称$\{X_t \}$对$\{Y_t \}$有瞬时因果性。
这时$\{Y_t \}$对$\{X_t \}$也有瞬时因果性。
* 如果$X_t \Rightarrow Y_t$，
则存在最小的正整数$m$，
使得
$$
\sigma^2(Y_t | \bar Y_t, X_{t-m}, X_{t-m-1}, \dots) 
< \sigma^2(Y_t | \bar Y_t, X_{t-m-1}, X_{t-m-2}, \dots) 
$$
称$m$为**因果性滞后值**(causality lag)。
如果$m>1$，
这意味着在已有$Y_{t-1}, Y_{t-2}, \dots$和$X_{t-m}, X_{t-m-1}, \dots$的条件下，
增加$X_{t-1}$, \dots, $X_{t-m+1}$不能改进对$Y_t$的预测。

::: {.example #causal-exaxylag1}
设$\{ \varepsilon_t, \eta_t \}$是相互独立的零均值白噪声列，
$\text{Var}(\varepsilon_t)=1$,
$\text{Var}(\eta_t)=1$,
考虑
$$\begin{aligned}
Y_t =& X_{t-1} + \varepsilon_t \\
X_t =& \eta_t + 0.5 \eta_{t-1}
\end{aligned}$$

:::

用$L(\cdot|\cdot)$表示最优线性预测，则
$$\begin{aligned}
& L(Y_t | \bar Y_t, \bar X_t) \\
=& L(X_{t-1} | X_{t-1}, \dots, Y_{t-1}, \dots)
+ L(\varepsilon_t | \bar Y_t, \bar X_t) \\
=& X_{t-1} + 0 \\
=& X_{t-1} \\
\sigma(Y_t | \bar Y_t, \bar X_t) =&
\text{Var}(\varepsilon_t) = 1
\end{aligned}$$
而
$$
Y_t = \eta_{t-1} + 0.5\eta_{t-2} + \varepsilon_t
$$
有
$$\begin{aligned}
\gamma_Y(0) = 2.25,
\gamma_Y(1) = 0.5,
\gamma_Y(k) = 0, k \geq 2
\end{aligned}$$
所以$\{Y_t \}$是一个MA(1)序列，
设其方程为
$$
Y_t = \zeta_t + b \zeta_{t-1}, 
\zeta_t \sim \text{WN}(0, \sigma_\zeta^2)
$$
可以解出
$$\begin{aligned}
\rho_Y(1) =& \frac{\gamma_Y(1)}{\gamma_Y(0)} = \frac{2}{9} \\
b =& \frac{1 - \sqrt{1 - 4 \rho_Y^2(1)}}{2 \rho_Y(1)}
\approx 0.2344 \\
\sigma_\zeta^2 =& \frac{\gamma_Y(1)}{b} \approx 2.1328
\end{aligned}$$
于是
$$\begin{aligned}
\sigma(Y_t | \bar Y_t)
=& \sigma_\zeta^2 = 2.1328
\end{aligned}$$
所以
$$\begin{aligned}
\sigma(Y_t | \bar Y_t, \bar X_t) = 1
< 2.1328 = \sigma(Y_t | \bar Y_t)
\end{aligned}$$
即$X_t$是$Y_t$的格兰格原因。

反之，
$X_t$是MA(1)序列，
有
$$
\eta_t = \frac{1}{1 + 0.5 B} X_t
= \sum_{j=0}^\infty (-0.5)^j X_{t-j}
$$
其中$B$是推移算子（滞后算子）。
于是
$$\begin{aligned}
L(X_t | \bar X_t)
=& L(\eta_t | \bar X_t)
+ 0.5 L(\eta_{t-1} | \bar X_t) \\
=& 0.5 \sum_{j=0}^\infty (-0.5)^j X_{t-1-j} \\
=& - \sum_{j=1}^\infty (-0.5)^j X_{t-j} \\
\sigma(X_t | \bar X_t)
=& \text{Var}(X_t - L(X_t | \bar X_t)) \\
=& \text{Var}(\eta_t) = 1
\end{aligned}$$
而
$$\begin{aligned}
L(X_t | \bar X_t, \bar Y_t) 
=& L(\eta_t | \bar X_t, \bar Y_t)
+ 0.5 L(\eta_{t-1} | \bar X_t, \bar Y_t) \\
=& 0 +
0.5 L(\sum_{j=0}^\infty (-0.5)^j X_{t-1-j} | \bar X_t, \bar Y_t) \\
=& -\sum_{j=1}^\infty (-0.5)^j X_{t-j} \\
=& L(X_t | \bar X_t)
\end{aligned}$$
所以$Y_t$不是$X_t$的格兰格原因。

考虑瞬时因果性。
$$\begin{aligned}
L(Y_t | \bar X_t, \bar Y_t, X_t)
=& X_{t-1} + 0 (\text{注意}\varepsilon_t\text{与}\{X_s, \forall s\}\text{不相关} \\
=& L(Y_t | \bar X_t, \bar Y_t)
\end{aligned}$$
所以$X_t$不是$Y_t$的瞬时格兰格原因。

○○○○○


::: {.example #causal-exaxylag2}
在例\@ref(exm:causal-exaxylag1)中，如果模型改成
$$\begin{aligned}
Y_t =& X_{t} + \varepsilon_t \\
X_t =& \eta_t + 0.5 \eta_{t-1}
\end{aligned}$$
有怎样的结果？

:::

这时
$$
Y_t = \varepsilon_t + \eta_t + 0.5 \eta_{t-1}
$$
仍有
$$\begin{aligned}
\gamma_Y(0) = 2.25,
\gamma_Y(1) = 0.5,
\gamma_Y(k) = 0, k \geq 2
\end{aligned}$$
所以$Y_t$还服从MA(1)模型
$$
Y_t = \zeta_t + b \zeta_{t-1},
b \approx 0.2344,
\sigma^2_\zeta \approx 2.1328
$$

$$\begin{aligned}
L(Y_t | \bar Y_t, \bar X_t)
=& L(X_t | \bar Y_t, \bar X_t) + 0 \\
=& L(\eta_t | \bar Y_t, \bar X_t)
+ 0.5 L(\eta_{t-1} | \bar Y_t, \bar X_t) \\
=& 0 + 0.5 L(\sum_{j=0}^\infty (-0.5)^j X_{t-1-j} | \bar Y_t, \bar X_t) \\
=& - \sum_{j=1}^\infty (-0.5)^j X_{t-j} \\
=& X_t - \eta_t \\
\sigma(Y_t | \bar Y_t, \bar X_t) 
=& \text{Var}(\varepsilon_t + \eta_t) = 2
\end{aligned}$$
而
$$
\sigma(Y_t | \bar Y_t)
= \sigma^2_\zeta \approx 2.1328
> \sigma(Y_t | \bar Y_t, \bar X_t) = 2
$$
所以$X_t$是$Y_t$的格兰格原因。

反之，
$$\begin{aligned}
L(X_t | \bar X_t, \bar Y_t)
=& - \sum_{j=1}^\infty (-0.5)^j X_{t-j} \\
=& L(X_t | \bar X_t)
\end{aligned}$$
所以$Y_t$不是$X_t$的格兰格原因。

考虑瞬时因果性。
$$\begin{aligned}
L(Y_t | \bar X_t, \bar Y_t, X_t)
=& X_{t} + 0 (\text{注意}\varepsilon_t\text{与}\{X_s, \forall s\}\text{不相关} \\
=& X_t \\
\sigma(Y_t | \bar X_t, \bar Y_t, X_t)
=& \text{Var}(\varepsilon) \\
=& 1 < 2 = \sigma(Y_t | \bar X_t, \bar Y_t)
\end{aligned}$$
所以$X_t$是$Y_t$的瞬时格兰格原因。







$$\begin{aligned}
[aaa]
\end{aligned}$$
