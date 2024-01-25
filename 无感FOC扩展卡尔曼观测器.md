# 无感FOC扩展卡尔曼观测器

状态空间方程引入：

$$\hat{\dot{X}}_t=A\hat{X}_t+Bu_t$$

$$Z_t=HX_t$$

当T为1时离散化后：

$$\hat{X}_k=A\hat{X}_{k-1}+Bu_{k-1}$$
$$Z_k=HX_k$$

考虑系统噪声：

$$\hat{X}_k=A\hat{X}_{k-1}+Bu_{k-1}+W_k$$
$$Z_k=HX_k+V_k$$

其中 $W_k$ 为过程噪音， $V_k$ 为测量噪音；


卡尔曼观测器的作用就是通过递归的思想，找到最准确的  $W_k$ 和 $V_k$ ，使得观测器计算的结果无限接近于实际系统。卡尔曼滤波器是对一系列随机信号的估算方法，所以与其说它属于滤波器，不如称它为最优控制。


卡尔曼观测器模型引入：

表贴式永磁同步电机公式如下

![20240125154325](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125154325.png)


由于电机控制环路的采样时间非常短，我们可以近似的认为每个时刻的转速不变，进而简化卡尔曼观测器模型。

状态空间方程：


![20240125154511](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125154511.png)

卡尔曼观测器方程：

![20240125154703](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125154703.png)


 $W_k$ ， $V_k$ 满足正态分布：

$P(W_k)$ ~ $N(0,Q)$

$P(V_k)$ ~ $N(0,R)$

 $W_k$ 各项独立， $V_k$ 各项独立

$$Q=\begin{bmatrix}
\sigma^2_{w1} & 0 & 0 & 0 \\
0 & \sigma^2_{w1} & 0 & 0 \\
0 & 0 & \sigma^2_{w1} & 0 \\
0 & 0 & 0 & \sigma^2_{w1} 
\end{bmatrix},R=
\begin{bmatrix}
\sigma^2_{v1} & 0\\
0 & \sigma^2_{v2}
\end{bmatrix}$$

卡尔曼核心公式1：先验估计值  $\hat{X^-_k}$

先验估计值  $\hat{X^-_k}$ ：

![20240125155423](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125155423.png)


![20240125155509](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125155509.png)

![20240125155604](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125155604.png)


![20240125155656](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125155656.png)


求 $tr(P_K)$ 的极值，令

$$\frac{dtr(P_k)}{dK_k}=0, \Rightarrow -P^-_kH^T+K_k(HP^-_KH^T+R)=0$$

卡尔曼增益计算公式：

$$K_k=\frac{P^-_kH^T}{(HP^-_kH^T+R)}$$


![20240125160100](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125160100.png)
![20240125160119](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125160119.png)
![20240125160144](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125160144.png)


**扩展卡尔曼观测器**

由于卡尔曼观测器（滤波器）算法都是需要线性、离散的系统模型，但是在实际应用中所倡建的系统大多是非线性的，包括无刷和永磁同步电机系统。此时就需要扩展卡尔曼观测器 **（Extended Kalman Filter,EKF）** 的理论应用于实际。在使用EKF时，首先需要对非线性系统的模型方程进行线性化和离散化处理。

（1）线性化
1）一元系统线性化：

泰勒展开：  $f(x)=f(x_0)+\frac{\dot{f}(x_0)}{1!}(x-x_0),x-x_0 \rightarrow0$

2) 多元系统线性化：

![20240125160617](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125160617.png)

扩展卡尔曼观测器建模：

![20240125160655](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125160655.png)


线性化：

![20240125161127](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125161127.png)


前向欧拉法离散：

![20240125161052](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125161052.png)



![20240125161307](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125161307.png)


其中，Q矩阵为过程噪声的协方差矩阵，R矩阵为测量噪声的协方差矩阵，对系统的收敛性有着决定性的影响，选取的不适当可能会导致收敛过慢、抖动过大甚至完全发散。

参考矩阵：

![20240125161234](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125161234.png)


转速  $W_e$ 方差要偏大，且 $\theta$ 作为 $w_e$ 的积分，  $\theta$ 的方差要比 $W_e$小。
