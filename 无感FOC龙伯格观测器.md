# 无感FOC龙伯格观测器

无感FOC在没有编码器的情况下，可以使用观测器来观测电机反电动势得到电机的转速，下面介绍一种线性观测器--龙伯格观测器。

## 1、状态空间

在介绍龙伯格观测器之前，先来回顾一下现代控制理论中的状态空间方程。状态空间方程 **（State Space Model）** 是一种描述系统数学模型的方法。状态空间方程是现代控制理论的基础，它是以矩阵的形式描述系统状态变量、输入及输出之间的关系。相比较传统的描述单输入单输出系统 **SISO**的传递函数方法，它可以描述多输入多输出的系统 **（Multiple Input Multiple Output, MIMO）**。目前流行的一些算法，如模型预测控制、卡尔曼滤波器及最优化控制，都是在状态空间方程表达形式基础上发展而来。

以典型的弹簧阻尼系统为例，推导状态空间方程。

![20240123171012](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240123171012.png)

运动方程：

$$m \frac{dx^2_{(t)}}{dt^2}+b\frac{dx_{(t)}}{dt}+kx_{(t)}=f(t)$$

拉普拉斯变换后，得到：
$$(ms^2+bs+k)Y_{(s)}=U_{(s)}$$

在经典控制理论下得到传递函数：

$$G_{(s)}=\frac{Y_{(s)}}{U_{(s)}}=\frac{1}{ms^2+bs+k}$$

选择状态变量 $z_1(t)$ ,$z_2(t)$

$$z_1(t)=x_{(t)}，z_2(t)=\frac{dz_1(t)}{dt}=\frac{dx(t)}{dt}$$
$$m\frac{dz_2(t)}{dt}+bz_2(t)+kz_1(t)=f(t)\Rightarrow \frac{dz_2(t)}{dt}=\frac{1}{m} \left (f(t)-bz_2(t)-kz_1(t)\right )$$
$$\frac{d}{dt} \begin{bmatrix} z_1(t) \\ z_2(t) \end{bmatrix}=
\begin{bmatrix} 0 & 1 \\ -\frac{k}{m} & -\frac{b}{m} \end{bmatrix}
\begin{bmatrix} z_1(t) \\ z_2(t) \end{bmatrix}+\begin{bmatrix} 0 \\ \frac{1}{m} \end{bmatrix}u(t)$$

得到 $\Rightarrow$

$$y_{(t)}=\begin{bmatrix} 1 & 0 \end{bmatrix}\begin{bmatrix} z_1(t) \\ z_2(t) \end{bmatrix}+\begin{bmatrix} 0 \end{bmatrix}\begin{bmatrix} u_{(t)} \end{bmatrix}$$

将上述形式推广并得到状态空间方程的一般形式：

$$\frac{dz(t)}{dt}=Az(t)+Bu(t)$$

$$y(t)=Cz(t)+Du(t)$$

$z(t)$ 是状态变量，是一个n维向量， $z(t)=\left [ z_1(t),z_2(t),\cdot\cdot\cdot,z_n(t) \right ]$ ;

$y(t)$ 是状态变量，是一个m维向量， $y(t)=\left [ y_1(t),y_2(t),\cdot\cdot\cdot,y_n(t) \right ]$ ;

$x(t)$ 是状态变量，是一个p维向量， $x(t)=\left [ x_1(t),x_2(t),\cdot\cdot\cdot,x_n(t) \right ]$ ;

矩阵**A**是**n\*n**的矩阵，表示系统变量之间的关系，称为状态矩阵或者系统矩阵；矩阵**B**是**n\*p**矩阵，表示输入对状态变量的影响，称为输入矩阵或者控制矩阵；矩阵**C**是**m\*n**矩阵，表示系统的输出与系统状态变量的关系，称为输出矩阵；矩阵**D**是**m\*p**矩阵，表示系统的输入直接作用在系统输出的部分，称为直接传递矩阵。

## 2、矩阵特征根与极点
当矩阵B左乘变换矩阵A时，实际上对B的列空间进行了投影或拉伸等线性变换，该变换会影响矩阵B的列空间，方向、长度或者量同时改变，取决于A的性质。
当矩阵B右乘变换矩阵A时，实际上对B的行空间进行了投影或拉伸等线性变换，该变换会影响矩阵B的行空间，方向、长度或者量同时改变，取决于A的性质。

在线性代数中，对于给定的一个方阵A，它的特征向量v经过矩阵A线性变换的作用后，得到的新向量仍然与原来的v保持在同一条直线上，但其长度和方向也许会发生改变。即
$$Av=\lambda v$$
其中$\lambda$为标量，即特征向量的长度在矩阵A线性变换下缩放的比例，称为矩阵A的特征值。有两个例子如下图中所示，可知$v_b$是矩阵A的特征向量，$\lambda$是其特征值。

![20240123173835](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240123173835.png)

$$Av=\lambda v$$
$$Av- \lambda v=0$$
$$\left ( A-\lambda I\right )v=0$$

其中，I为单位矩阵，维度与A相同，如果式有非零解，则矩阵    $A-\lambda I$   的行列式必须为0，则 $\left | A-\lambda I \right |=0$ ，
将 

$$A=\begin{bmatrix} 1 & 1 \\ 4 & -2 \end{bmatrix} \Rightarrow A_1=\begin{bmatrix} 1 \\ 4 \end{bmatrix},A_2=\begin{bmatrix} 1 \\ -2 \end{bmatrix}$$

代入可得  $\lambda^2+\lambda-6=0$  ,称为矩阵A的特征方程，可以得到

矩阵A的两个特征值：  $\lambda_1=2,\lambda_2=-3$ ，求得特征向量

$$v_1=\begin{bmatrix} 1 \\ 1 \end{bmatrix},v_2=\begin{bmatrix} 0.5 \\ -2 \end{bmatrix}$$

得新的向量

$$Av_1=\begin{bmatrix} 2 \\ 2 \end{bmatrix},Av_2=\begin{bmatrix} -1.5 \\ 6 \end{bmatrix}$$

![20240123174759](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240123174759.png)

对于单输入输出系统来说，对状态方程进行拉普拉斯变换：

$$L\left [ \frac{dz(t)}{dt} \right ]=L\left [ Az(t)+Bu(t) \right ]$$

$$L\left[ y(t) \right ]=L\left [ Cz(t)+Du(t) \right]$$

考虑零初始状态，   $z_1(0)=z_2(0)=0$，   可将上式整理为：

$$sZ(s)=Az(s)+BU(s)$$

$$Y(s)=CZ(s)+DU(s)$$

整理后可得：

$$Z(s)=\left ( sI-A \right )^{-1}BU(s)$$

则

$$Y(s)=\left ( C \left ( sI-A \right)^{-1}B +D \right )U(s)$$

通常，状态空间方程的D矩阵都会取0，即D=0，同时    $\left ( sI-A \right)^{-1}=\frac{\left ( sI-A \right)^{*}}{\left | sI-A \right|}$，可得

$$G(s)=\frac{Y(s)}{U(s)}=\frac{C\left(sI-A\right)^*B}{\left | sI-A \right |}$$

令    $\left | sI-A \right |=0$，求得极点，也就是矩阵A特征值   $\lambda$。（将极点和特征值联系起来了）。

![20240123202840](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240123202840.png)

## 3、龙伯格观测器

龙伯格观测器也叫状态观测器，由龙伯格提出，解决了**线性系统**在满足**可观性条件**下的状态重构问题，给出了由线性系统输入输出构造状态观测器的一般理论。

**状态能观测性判据**：对于n维线性时不变系统而言，它的状态能观测的充分必要条件是能观测矩阵   $O=\left[C \quad CA \quad \cdot\cdot\cdot \quad CA^{n-1}\right]$的秩为 $n$，即 $RANK(O)=n$;

**开环观测器：**

$$\frac{d\hat{z}(t)}{dt}=A\hat{z}(t)+Bu(t)$$

$$\hat{y}(t)=C\hat{z}(t)+Du(t)$$

  $\hat{z}(t)$  代表   $z(t)$估计值；    $\hat{y}(t)$代表   $y(t)$估计值

![20240123203636](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240123203636.png)

**闭环观测器：**
$$\frac{d\hat{z}(t)}{dt}=A\hat{z}(t)+Bu(t)+L\left( y(t)-\hat{y}(t) \right)\Rightarrow\frac{d\hat{z}(t)}{dt}=\left( A-LC \right)\hat{z}(t)+\left( B-LD \right)u(t)+Ly(t)$$
$L$是观测矩阵，影响系统是否收敛及收敛速度。由以下框图可知，当估计输出 $\hat{y}(t)$和实际输出 $y(t)$误差为0时， $\frac{d\hat{z}(t)}{dt}$ 和 $\frac{dz\left(t\right)}{dt}$ 相等，即可准确的得到未知的状态变量 $z(t)$。

![20240123204246](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240123204246.png)

为了分析闭环观测器的收敛性及收敛速度，引入观测误差， $e=z(t)-\hat{z}(t)$，得到状态误差方程：
$$\frac{de}{dt}=\frac{d\left( z(t)-\hat{z}(t) \right)}{dt}=Az(t)+Bu(t)-(A-LC)\hat{z}(t)-(B-LD)u(t)-Ly(t)=(A-LC)(z(t)-\hat{z}(t))=(A-LC)e(t)$$

解上述一阶微分方程得： $e(t)=C_0e^{(A-LC)t},t\ge0$
上述表明，为了使观测状态 $\frac{dz(t)}{dt}$趋近于实际的 $z(t)$，也就是误差e趋近于0，则状态误差方程应收敛。
$$\frac{de(t)}{dt}=(A-LC)e(t)$$
状态矩阵 $(A-LC)$的特征值应具有负实部，且负实部的大小会影响状态逼近的速度，特征值负实部绝对值越大，逼近速度越快。

在旋转坐标系下，电机dq轴方程
$$u_d=Ri_d+L_d\frac{di_d}{dt}-w_eL_qi_q$$
$$u_q=Ri_q+L_q\frac{di_q}{dt}+w_e(L_di_d+\psi_f)$$
改写以上方程，将对角元素变成对称形式

$$\begin{bmatrix} u_d \\ u_q \end{bmatrix}=\begin{bmatrix} R+\frac{d}{dt}L_d & -w_eL_q \\ w_eL_q & R+\frac{d}{dt}L_d \end{bmatrix}\begin{bmatrix} i_d \\ i_q \end{bmatrix}+\begin{bmatrix} 0 \\ (L_d-L_q)(w_ei_d-\frac{di_q}{dt})+w_e\psi_f \end{bmatrix}$$

旋转坐标系下方程通过反PARK变换得到静止坐标系下电机方程

$$\begin{bmatrix} u_{\alpha} \\ u_{\beta} \end{bmatrix}=\begin{bmatrix} R+\frac{d}{dt}L_d & w_e(L_d-L_q) \\ -w_e(L_d-L_q) & R+\frac{d}{dt}L_d \end{bmatrix}\begin{bmatrix} i_{\alpha} \\ i_{\beta} \end{bmatrix}+\begin{bmatrix} (L_d-L_q)(w_ei_d-\frac{di_q}{dt})+w_e\psi_f \end{bmatrix}\begin{bmatrix} -sin\theta_e \\ cos\theta_e \end{bmatrix}$$

静止坐标系下，电机的等效模型如下所示

![20240123210340](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240123210340.png)

得扩展反电势（EMF）： 

$$\begin{bmatrix} e_{\alpha} \\ e_{\beta} \end{bmatrix}=\begin{bmatrix} (L_d-L_q)(w_ei_d-\frac{di_q}{dt})+w_e\psi_f \end{bmatrix}\begin{bmatrix} -sin\theta_e \\ cos\theta_e \end{bmatrix}$$

将以上静止坐标系下电机电压方程改写成电流的状态方程

$$\frac{d}{dt}\begin{bmatrix} i_{\alpha} \\ i_{\beta} \end{bmatrix}=\frac{1}{L_d}\begin{bmatrix} -R & -(L_d-L_q)w_e \\ (L_d-L_q)w_e & -R\end{bmatrix}\begin{bmatrix} i_{\alpha} \\ i_{\beta} \end{bmatrix}+\frac{1}{L_d}\begin{bmatrix} u_{\alpha} \\ u_{\beta} \end{bmatrix}-\frac{1}{L_d}\begin{bmatrix} e_{\alpha} \\ e_{\beta} \end{bmatrix}$$

对于表贴式三相PMSM，重写静止坐标系下的电流方程
$$\frac{d}{dt}i_{s}=Ai_s+Bu_s+K_eE_s$$

 $\Rightarrow K_e=\begin{bmatrix} -\frac{1}{L_s} & 0 \\ 0 & -\frac{1}{L_s} \end{bmatrix}$  为反电动势系数
$$\Rightarrow E_s=\begin{bmatrix} e_{\alpha} \\ e_{\beta} \end{bmatrix}=\begin{bmatrix} -\psi_fw_esin\theta_e \\ \psi_fw_ecos\theta_e \end{bmatrix},\dot{E_s}=\frac{d}{dt}\begin{bmatrix} -\psi_fw_esin\theta_e \\ \psi_fw_ecos\theta_e \end{bmatrix}=w_e\begin{bmatrix} -e_{\beta} \\ e_{\alpha} \end{bmatrix}$$

$$\Rightarrow \frac{d}{dt}\begin{bmatrix} i_{\alpha} \\ i_{\beta} \end{bmatrix}=\begin{bmatrix} -\frac{R}{L_s} & 0 \\ 0 & -\frac{R}{L_s} \end{bmatrix}\begin{bmatrix} i_{\alpha} \\ i_{\beta} \end{bmatrix}+\begin{bmatrix} \frac{1}{L_s} & 0 \\ 0 & \frac{1}{L_s} \end{bmatrix}\begin{bmatrix} u_{\alpha} \\ u_{\beta} \end{bmatrix}+\begin{bmatrix} -\frac{1}{L_s}&0 \\ 0&-\frac{1}{L_s} \end{bmatrix}\begin{bmatrix} e_{\alpha} \\ e_{\beta} \end{bmatrix}$$

![20240123212422](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240123212422.png)

$u_{\alpha},u_{\beta},i_{\alpha},i_{\beta}$ 是已知量，因此可计算出反电动势 $e_{\alpha},e_{\beta}$，反电动势含有电机的转速信息。
根据上述框图，建立状态观测方程：

$$u(t)=[u_{\alpha}\quad u_{\beta}]^T$$
$$z(t)=[i_{\alpha}\quad i_{\beta} \quad e_{\alpha}\quad e_{\beta}]^T$$
$$\frac{dz(t)}{dt}=\begin{bmatrix} \frac{d}{dt}i_{\alpha}\quad \frac{d}{dt}i_{\beta}\quad \frac{d}{dt}e_{\alpha}\quad \frac{d}{dt}e_{\beta} \end{bmatrix}^T$$
$$y(t)=[i_{\alpha}\quad i_{\beta}]^T$$
得到
 $$\frac{dz(t)}{dt}=Az(t)+Bu(t) \\ y(t)=Cz(t)$$
其中，
$$A=\begin{bmatrix} -\frac{R}{L_s} & 0 & -\frac{1}{L_s}&0 \\ 0 & -\frac{R}{L_s} & 0 & -\frac{1}{L_s} \\ 0 & 0 & 0 & -w_e \\ 0 & 0 & w_e & 0 \end{bmatrix},B=\begin{bmatrix} \frac{1}{L_s} & 0 \\ 0 & \frac{1}{L_s} \\ 0 & 0 \\ 0 & 0 \end{bmatrix},C=\begin{bmatrix} 1 & 0& 0& 0 \\ 0 & 1 &0 &0 \end{bmatrix}$$
判断系统是否稳定，可以从状态矩阵A的特征值入手,根据闭环状态观测器框图引入状态观测器，得
$$\frac{d\hat{z}(t)}{dt}=A\hat{z}(t)+Bu(t)+L(y(t)-\hat{y}(t))=A\hat{z}(t)+Bu(t)+LC(z(t)-\hat{z}(t))$$
$$L=\begin{bmatrix} L_1 & 0 & L_2 & 0 \\ 0 & L_1 & 0 & L_2 \end{bmatrix}^T$$
$\Rightarrow$ 真实系统
$$\begin{bmatrix} \frac{d}{dt}i_{\alpha} \\ \frac{d}{dt}i_{\beta}\\ \frac{d}{dt}e_{\alpha}\\ \frac{d}{dt}e_{\beta} \end{bmatrix}=\begin{bmatrix} -\frac{R}{L_s} & 0 & -\frac{1}{L_s}&0 \\ 0 & -\frac{R}{L_s} & 0 & -\frac{1}{L_s} \\ 0 & 0 & 0 & -w_e \\ 0 & 0 & w_e & 0 \end{bmatrix}\begin{bmatrix}i_{\alpha}\\ i_{\beta} \\ e_{\alpha}\\ e_{\beta}\end{bmatrix}+\begin{bmatrix} \frac{1}{L_s} & 0 \\ 0 & \frac{1}{L_s} \\ 0 & 0 \\ 0 & 0 \end{bmatrix}\begin{bmatrix} u_{\alpha} \\ u_{\beta} \end{bmatrix},\begin{bmatrix} i_{\alpha} \\ i_{\beta} \end{bmatrix}=\begin{bmatrix} 1 & 0& 0& 0 \\ 0 & 1 &0 &0 \end{bmatrix}\begin{bmatrix}i_{\alpha}\\ i_{\beta} \\ e_{\alpha}\\ e_{\beta}\end{bmatrix}$$
$\Rightarrow$ 估计系统（全阶观测器）
$$\begin{bmatrix} \frac{d}{dt}\hat{i}_{\alpha} \\ \frac{d}{dt}\hat{i}_{\beta}\\ \frac{d}{dt}\hat{e}_{\alpha}\\ \frac{d}{dt}\hat{e}_{\beta} \end{bmatrix}=\begin{bmatrix} -\frac{R}{L_s} & 0 & -\frac{1}{L_s}&0 \\ 0 & -\frac{R}{L_s} & 0 & -\frac{1}{L_s} \\ 0 & 0 & 0 & -w_e \\ 0 & 0 & w_e & 0 \end{bmatrix}\begin{bmatrix}\hat{i}_{\alpha}\\ \hat{i}_{\beta} \\ \hat{e}_{\alpha}\\ \hat{e}_{\beta}\end{bmatrix}+\begin{bmatrix} \frac{1}{L_s} & 0 \\ 0 & \frac{1}{L_s} \\ 0 & 0 \\ 0 & 0 \end{bmatrix}\begin{bmatrix} u_{\alpha} \\ u_{\beta} \end{bmatrix}+\begin{bmatrix} L_1 & 0 \\ 0 & L_1 \\ L_2 & 0 \\ 0 & L_2 \end{bmatrix}\begin{bmatrix} 1 & 0& 0& 0 \\ 0 & 1 &0 &0 \end{bmatrix}\begin{bmatrix}i_{\alpha}-\hat{i_{\alpha}} \\ i_{\beta}-\hat{i_{\beta}} \\ e_{\alpha}-\hat{e_{\alpha}}\\ e_{\beta}-\hat{e}_{\beta}\end{bmatrix}$$
由前面可知，有状态误差方程
$$\frac{de(t)}{dt}=(A-LC)e(t)$$
得到状态误差矩阵特征方程
$$A-LC=\begin{bmatrix} -\frac{R_s}{L_s}-l_1 & 0 & -\frac{1}{L_s} & 0 \\ 0 & -\frac{R_s}{L_s}-l_1 & 0 & -\frac{1}L_{s} \\ -l_2 & 0 & 0 & -w_e \\ 0 & -l_2 & w_e & 0 \end{bmatrix}$$
$$\lambda I-(A-LC)=\begin{bmatrix} \lambda+\frac{R_s}{L_s}+l_1 & 0 & \frac{1}{L_s} & 0 \\ 0 & \lambda+\frac{R_s}{L_s}+l_1 & 0 & \frac{1}L_{s} \\ l_2 & 0 & \lambda & w_e \\ 0 & l_2 & -w_e & \lambda \end{bmatrix}$$
令 $|\lambda I-(A-LC)|=0$,当且仅当 $\lambda<0$时，系统收敛稳定。

## 前向欧拉法离散：
$\frac{dx}{dt}=\frac{x(k+1)-x(k)}{T}$ ,T为采样周期， $\frac{dx}{dt}$为上一时刻的微分。

![20240123222024](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240123222024.png)

对观测器离散：
$$\frac{d\hat{z}(t)}{dt}=A\hat{z}(t)+Bu(t)+L(y(t)-\hat{y}(t)) \Rightarrow \hat{z}(k+1)=(AT+I)\hat{z}(k)+BTu(k)+LT(y(k)-\hat{y}(k))$$

![20240123222400](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240123222400.png)

得到：

![20240123222447](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240123222447.png)

去耦（先设置  $w_e=0$，再求解 L1,L2），简化观测器模型：

方程组一：

$$\hat{i}_{\alpha(k+1)}=(1-\frac{R_s}{L_s}T-L_1T)\hat{i}_{\alpha(k)}-\frac{1}{L_s}T\hat{e}_{\alpha(k)}+\frac{1}{L_s}Tu_{\alpha(k)}+L_1Ti_{\alpha(k)}$$

$$\hat{e}_{\alpha(k+1)}=\hat{e}_{\alpha(k)}+L_2T(i_{\alpha(k)}-\hat{i}_{\alpha(k)})$$

方程组二：

$$\hat{i}_{\beta(k+1)}=(1-\frac{R_s}{L_s}T-L_1T)\hat{i}_{\beta(k)}-\frac{1}{L_s}T\hat{e}_{\beta(k)}+\frac{1}{L_s}Tu_{\beta(k)}+L_1Ti_{\beta(k)}$$

$$\hat{e}_{\beta(k+1)}=\hat{e}_{\beta(k)}+L_2T(i_{\beta(k)}-\hat{i}_{\beta(k)})$$

得到观测矩阵  $A_0$:

$$A_0=\begin{bmatrix} 1-\frac{R_s}{L_s}-L_1T & -\frac{1}{L_s}T \\ L_2T & 1 \end{bmatrix}$$

$$|\lambda I-A_0|=\begin{vmatrix} \lambda -1+\frac{R_s}{L_s}+L_1T & \frac{1}{L_s}T \\ -L_2T & \lambda-1 \end{vmatrix}=(\lambda -1+\frac{R_s}{L_s}+L_1T)(\lambda -1)+\frac{L_2}{L_s}T^2=0$$

求出  $L_1,L_2$ ,当且仅当特征值   $\lambda<0$ 时，系统收敛稳定。


