---
title: EKF详解
date: 2019-05-23 11:40:14
categories: SLAM
tags:
- SLAM
- EKF
top: 100
mathjax: true
---

####1. 高斯函数

\begin{equation} 
p(x) = \det(2\pi\Sigma)^{- \frac {1}{2} }\exp{ \{ -\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu) \}}
\label{eq:Gaussion}
\end{equation}

>所有的高斯技术都共享了基本思想，即置信度用多元正态分布来表示。$x$的密度用两个参数来表示，均值$\mu$和协方差$\Sigma$，均值$\mu$是一个向量，它与状态$x$的维数相同。协方差是对称半正定的二次型。其维数等于状态$x$的维数的二次方。高斯滤波中的参数均值和方差称为矩参数，这是因为均值和方差是概率分布的一阶矩和二阶矩；正态分布的其他矩都是零。

####2. 线性高斯系统
KF是由Swerling（1950）和Kalman（1960）作为线性高斯系统中的预测和滤波技术而发明的，是用矩来定义的。
KF用矩参数来表示置信度：在时刻$t$，置信度用均值$\mu_t$和方差$\Sigma_t$表示、如果除了贝叶斯滤波的马尔科夫假设以外，还具有如下的三个特性，则后验就是高斯的。

-------------------------------
* 状态转移概率$p(x_t | u_t, x_{t-1})$必须是带有随机高斯噪声的参数的线性函数，可有下式表示:
\begin{equation}
x_t = A_tx_{t-1} + B_tu_t + \varepsilon_t
\label{eq:motion}
\end{equation}
式中，$x_t$和$x_{t-1}$都是状态向量，它们都是$n$维列向量；$u_t$为时刻$t$的控制向量。式(2)中，$A_t$为$n \times n$的矩阵，$B_t$为$n \times m$的矩阵，$n$为状态向量$x_t$的维数，$m$为控制向量$u_t$的维数。式(2)中的随机变量\varepsilon_t是一个高斯随机向量，表示由状态转移引入的不确定性。其维数与状态向量维数相同，均值为0，方差用$R_t$表示。式(2)中的状态转移概率称为线性高斯，反映了它与带有附加高斯噪声的自变量呈线性关系。
式(2)定义了状态转移概率$p(x_t | u_t, x_{t-1})$。这个概率可由公式(2)带入到多元正态分布的定义式(1)来得到。后验状态的均值由$A_tx_{t-1} + B_tu_t$给定，方差由$R_t$给定：
\begin{equation}
p(x_t | u_t, x_{t-1}) = \det(2\pi R_t)^{- \frac {1}{2} }\exp{ \{ -\frac{1}{2}(x_t-A_tx_{t-1} - B_tu_t)^TR_t^{-1}(x_t-A_tx_{t-1} - B_tu_t) \}}
\label{eq:status}
\end{equation}

* 观测概率$p(z_t | x_t)$也与带有高斯噪声的自变量呈线性关系：
\begin{equation}
z_t = C_tx_t + \delta _t
\label{eq:project}
\end{equation}
式中，$C_t$为$k \times n$的矩阵，$k$为观测向量$z_t$的维数；向量$\delta _t$为观测噪声。$\delta _t$服从均值为0、方差为$Q_t$的多变量高斯分布。因此观测概率由下面的多元正态分布给定：
\begin{equation}
p(z_t | x_t) = \det(2\pi Q_t)^{- \frac {1}{2} }\exp{ \{ -\frac{1}{2}(z_t - C_tx_t)^TQ_t^{-1}(z_t-C_tx_t) \}}
\label{eq:measure}
\end{equation}

* 最后，初始置信度必须${\rm bel}(x_0)$必须是正态分布的。这里用$\mu_0$表示初始置信度的均值，用$\Sigma_0$表示协方差：
\begin{equation}
{\rm bel}(x_0) = p (x_0)= \det(2\pi \Sigma_0)^{- \frac {1}{2} }\exp{ \{ -\frac{1}{2}(x_0 - \mu_0)^T\Sigma_0^{-1}(x_0-\mu_0) \}}
\label{eq:initial}
\end{equation}

这三个假设足以保证后验${\rm bel}(x_t)$在任何时刻$t$总符合高斯分布。

####3. KF算法(Kalman fliter algorithm)
KF算法如图所示，KF表示均值为$\mu_t$、方差为$\Sigma_t$的状态量在时刻$t$的置信度{\rm bel}(x_t)。KF的输入是$t-1$时刻的置信度，其均值和方差分别用$\mu_{t-1}$和$\Sigma_{t-1}$表示。为了更新这些参数，KF需要控制向量$u_t$和测量向量$z_t$。输出的是时刻$t$的置信度，均值为$\mu_t$，方差为$\Sigma_t$。


>Algorithm Kalman_filter($\mu_{t-1}$,$\Sigma_{t-1}$,$u_t$,$z_t$):





####3.3 线性高斯系统










