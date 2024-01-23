# 无感FOC非线性磁链观测器

非线性磁链观测器可以快速实现正反转切换；可以观测到极低的转速而不需要强拖启动。但是在零速时带载能力很弱，所以一般情况下还是使用IF开环+速度环闭环控制电机。

非线性磁链观测器的状态变量为磁链值，观测的磁链值收敛于电机实际磁链值，观测器收敛。非线性是由于观测器存在**sin**和**cos**项，所以是非线性观测器。

表贴式永磁同步电机**alpha-beta**轴电压方程如下所示：

$$\begin{bmatrix}u_{\alpha } \\
u_{\beta}\end{bmatrix}=\begin{bmatrix}R_s+L_s\frac{d}{dt}  & 0\\
0 & R_s+L_s\frac{d}{dt}\end{bmatrix}+\begin{bmatrix}w_e\psi_f \end{bmatrix}\begin{bmatrix} -sin\theta \\
cos\theta\end{bmatrix}$$

$$\frac{di_{\alpha}}{dt}=-\frac{-R_s}{L_s}i_{\alpha}+\frac{\omega_{e}\psi_f}{L_s}sin\theta-\frac{u_{\alpha}}{L_s}$$
$$\frac{di_{\beta}}{dt}=-\frac{-R_s}{L_s}i_{\beta}-\frac{\omega_{e}\psi_f}{L_s}cos\theta-\frac{u_{\beta}}{L_s}$$
将公式变换：

$$L\begin{bmatrix}
 \dot{i_{\alpha}}\\
 \dot{i_{\beta}}
\end{bmatrix}
=-R_s
\begin{bmatrix}
i_{\alpha} \\
i_{\beta}
\end{bmatrix}
+
\omega_{e}\psi_m
\begin{bmatrix}
sin\theta \\
-cos\theta
\end{bmatrix}
+
\begin{bmatrix}
u_{\alpha} \\
u_{\beta}
\end{bmatrix}$$


定义状态变量：

$$\begin{bmatrix}
\dot{x_1} \\
\dot{x_2}
\end{bmatrix}
=L\begin{bmatrix}
i_{\alpha}\\
i_{\beta}
\end{bmatrix}
-w_e\psi_m
\begin{bmatrix}
sin\theta \\
-cos\theta
\end{bmatrix}
=\begin{bmatrix}
y_1 \\
y_2
\end{bmatrix}$$


$$\begin{bmatrix}
y_1 \\
y_2
\end{bmatrix}
=-R_s
\begin{bmatrix}
i_{\alpha} \\
i_{\beta}
\end{bmatrix}
+
\begin{bmatrix}
u_{\alpha} \\
u_{\beta}
\end{bmatrix}$$

将上述方程积分：

$$\begin{bmatrix}
x_1 \\
x_2
\end{bmatrix}
=L
\begin{bmatrix}
i_{\alpha} \\
i_{\beta}
\end{bmatrix}
+\psi_{m}
\begin{bmatrix}
cos\theta \\
sin\theta
\end{bmatrix}$$

定义控件向量 $eta(x)$:

$$eta(x)=
\begin{bmatrix}
eta(x_1) \\
eta(x_2)
\end{bmatrix}
=\begin{bmatrix}
x_1 \\
x_2
\end{bmatrix}
-L
\begin{bmatrix}
i_{\alpha} \\
i_{\beta}
\end{bmatrix}
=\psi_m
\begin{bmatrix}
cos\theta \\
sin\theta
\end{bmatrix}$$

非线性磁链观测器模型：

$$\begin{bmatrix}
\dot{\hat{x}}_1 \\
\dot{\hat{x}}_2
\end{bmatrix}
=\begin{bmatrix}
y_1 \\
y_2
\end{bmatrix}
+\frac{\gamma}{2}
\begin{bmatrix}
eta(x_1) \\
eta(x_2)
\end{bmatrix}
(\psi^2_m-\Vert eta(x)\Vert^2)$$

收敛条件： $\Vert eta(x)\Vert^2=\psi^2_m$

模型离散化：

$$\begin{bmatrix}\hat{x}_{1k} \\
\hat{x}_{2k}\end{bmatrix}=T_s\begin{bmatrix}\begin{bmatrix}y_{1(k-1)} \\
y_{2(k-1)}\end{bmatrix}+\frac{\gamma}{2}\begin{bmatrix}eta(x_1)_{k-1} \\
eta(x_2)_{k-1}\end{bmatrix}(\psi^2_m-\Vert eta(x)\Vert^2)\end{bmatrix}+\begin{bmatrix}\hat{x}_{1(k-1)} \\
\hat{x}_{2(k-1)}\end{bmatrix}$$


$$\begin{bmatrix}
cos\hat{\theta} \\
sin\hat{\theta}
\end{bmatrix}
=\frac{1}{\psi_m}
\left (
\begin{bmatrix}
\hat{x}_{1k} \\
\hat{x}_{2k}
\end{bmatrix}
-L\begin{bmatrix}
i_{\alpha} \\
i_{\beta}
\end{bmatrix}
\right )$$


### 基于反电动势的PLL：

基于反电动势观测器系统观测的是反电动势电压，由于电压经过电感相位相对电流相位超前90°，故观测器得到的角度需要将90°补偿回来：

实际值： $\hat{\theta}^{'}=\hat{\theta}-\frac{\pi}{2}$ ， $\hat{\theta}$ 是观测值。


![20240123152646](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240123152646.png)


$$sin(\theta-\hat{\theta})=sin\theta cos\hat{\theta}-cos\theta sin\hat{\theta}=sin\theta cos(\hat{\theta}-\frac{\pi}{2})-cos\theta sin(\hat{\theta}-\frac{\pi}{2})=-sin\theta sin\hat{\theta}-cos\theta cos\hat{\theta}$$

### 基于磁链观测器的PLL：

![20240123154143](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240123154143.png)

$$sin(\theta-\hat{\theta})=sin\theta cos\hat{\theta}-cos\theta sin\hat{\theta}$$

## 参考文献

**[1] Sensorless Control of Surface-Mount Permanent-Magnet Synchronous Motors Based on a Nonlinear Observer.**
