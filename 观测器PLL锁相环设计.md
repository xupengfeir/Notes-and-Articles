# 观测器PLL锁相环设计

旋转坐标系下方程通过反PARK变换得到静止坐标系下电机方程

![20240125194918](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125194918.png)

在永磁同步电机中，   $L_d=L_q=L_s$

$$\begin{bmatrix}
    e_\alpha \\
    e_\beta
\end{bmatrix}=
\begin{bmatrix}
    -w_e\psi_fsin\theta_e \\
    w_e\psi_f\cos\theta_e
\end{bmatrix}$$

观测器的状态变量

![20240125195231](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125195231.png)

是否可以这样计算 $\theta$ 值  


![image](https://github.com/xupengfeir/Notes-and-Articles/assets/154572489/8ebcbf30-7a40-4bd2-9d62-f5e92cbd6c2c)


不可以。当  $\theta$ 处于 $\frac{\pi}{2}$ 附近时，正反切函数为无穷值，反正切函数不能很好地计算 $\theta$。（未经过闭环反馈，无法判断是否是准确值）。

**锁相环PLL：**

为了对基准信号与反馈信号进行频率比较，二者的相位必须相同且锁住，任何时间都不能改变，这样才能方便的比较频率，所以叫锁相。为了快速稳定输出系统，整个系统加入反馈成为闭环，所以叫环（loop）。

使用锁相环PLL计算转子角度 $\theta$。

![20240125195744](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125195744.png)

扩展反电动势e有以下关系式，可知e和电机转子角度有关系，可以使用负反馈计算转子角度。

![20240125195820](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125195820.png)

由极限定理可知， $\theta_e-\hat{\theta}_e \rightarrow0$ 时,

![20240125200137](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125200137.png)

![20240125200153](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125200153.png)

正交型PLL

![20240125200201](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125200201.png)


PLL低通滤波器参数 $K_p,k_i$ 求解：


![20240125200326](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240125200326.png)


$w_n$（带宽）越大，系统响应越快； $\xi$ 越大，最大超调量Mp就越小。
工程上  $\xi$ 通常选择1， $w_n$ 和电机最大转速相关。


特征方程： $s^2+2\xi w_ns+w^2_n=0$ ，求根得到：

$$\lambda_1=-\xi w_n+w_n\sqrt{\xi^2-1} $$
$$\lambda_1=-\xi w_n-w_n\sqrt{\xi^2-1} $$


当且仅当系统稳定时， $\lambda<0$ 。根据特征根的取值范围，选择合适的 $k_p,k_i$ 值。

注：

在ST电机库中，通常给定

$k_p=\frac{0.48w_{e[max]}}{T},k_i=\frac{0.0029w_{e[max]}}{T^2},w_{e[max]}$ 电机最大转速，单位rad/s。





