# 锁相环PLL工作原理

## 1、基本原理
锁相环 **（Phase-Locked Loop, PLL）** 是一个能够比较输出与输入 **相位差** 的反馈系统，利用外部输入的参考信号控制环路内部振荡信号的**频率和相位**，使振荡信号同步至参考信号。

简单的PLL由鉴相器、低通滤波器和压控振荡器闭环组成。通过反复的鉴相和调整，最终VCO的输出信号频率和输入信号频率一致，这时PLL电路进入锁定状态。

![20240124135716](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124135716.png)

### 应用：

（1）频率合成器

![20240124135933](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124135933.png)

 $f_{out}=\frac{N}{M}f_{ref}$ ,  $\frac{N}{M}$ 就是分频或者倍频系数。

类比于电流镜，镜像，保证了输出和输入信号一致或者同步放大或缩小。

（2）时钟恢复与数据重定时

当数据在传输过程中受到噪声干扰，造成数据存在误码情况，可以使用PLL对数据进行重定时。数据经过PLL后得到恢复后的时钟CLK，在CLK的基础上使用采样电路对数据重新采样，得到重定时后的数据DATA。

![20240124140433](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124140433.png)


### 鉴相器（Phase Detector，PD）：
检测信号的相位差，使用异或门电路将相位差转换成电压信号，线性关系如下图所示。转换后的电压信号 $V_{PD}$ 是矩形波，具有高频成分，需要低通滤波器将高频成分滤除。

![20240124140543](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124140543.png)

![20240124140602](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124140602.png)

![20240124140622](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124140622.png)

![20240124140650](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124140650.png)

### PLL对相位的阶跃响应：
![20240124140737](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124140737.png)

PLL把相位锁定之后，输入信号和输出信号相位同步。当某一时刻外部干扰Vin信号，使其相位不同步，出现相位差，产生了相位的阶跃，如 $\phi_{in}$ 所示。相位差经过鉴相器得到 $V_{PD}$， $V_{PD}$ 经过低通滤波器得到 $V_{cont}$， $V_{cont}$ 处于上升的状态，上升的 $V_{cont}$ 输入VCO后，驱使VCO振荡频率加快，使滞后的输出信号逐渐追上输入信号。


## 2、压控振荡器（Voltage-Controlled Oscillator，VCO）

### 起振条件
振荡器会产生一个周期性电压信号输出，没有输入信号，但是可以持续地输出周期性振荡的电压信号，通常用于电子系统中产生时钟信号。
![20240124141218](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124141218.png)

闭环传递函数： $\frac{V_{out}}{V_{in}}(s)=\frac{H(s)}{1+H(s)}$ ，令 $s=jw_0$ ，当 $H(jw_0)=-1$ 时，特征根处在虚轴上，系统处于临界稳定的状态，系统输出表现为振荡的性质。系统输出振荡就是振荡器需要的。

![20240124141414](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124141414.png)

在如上图所示的负反馈系统，当  $\angle H(jw_0)=180°$

$$V_x=V_0+|H(jw_0)|V_0+|H(jw_0)|^2V_0+|H(jw_0)|^3V_0+\cdot\cdot\cdot$$

如果  $|H(jw_0)|<1\Rightarrow V_x=\frac{V_0}{1-|H(jw_0)|}<\infty$

如果 $|H(jw_0)|>1\Rightarrow V_x$ 不收敛；此时振荡器才存在无限振荡。

因此 $|H(jw_0)|\ge1,\angle H(jw_0)=180°$ ，该反馈系统才可能会振荡。

巴克豪森准则：电子振荡器系统信号由输入到输出再反馈到输入的相位差为360°，且增益大于1，为振荡器振荡的必要条件。（ $\angle H(jw_0)$ +负反馈=360°）

给出一个两级放大器的例子，如下所示。

![20240124142241](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124142241.png)

两级放大器，正反馈，直流相移0°（直流正反馈，直接锁定），最大交流相移180°。不满足起振条件。

给出一个三级放大器的例子，如下所示。

![20240124142317](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124142317.png)

三级放大器，负反馈，直流相移180°，最大交流相移270°。总相移450°，满足起振条件之一的相移大于360°。

电路传递函数

$$H(s)=-\frac{A^3_0}{(1+\frac{s}{w_0})^3}$$

3个直流 $-A_0$ 贡献180°，3个极点还需要各贡献60°。

**起振相位条件**

$$tan^{-1}\frac{w_{0sc}}{w_0}=60°\Rightarrow w_{0sc}=\sqrt3 w_0$$

**起振幅值条件**

$$\frac{A^3_0}{\left [ \sqrt{1+\left ( \frac{w_{0sc}}{w_0} \right)^2} \right ]^3}=1\Rightarrow A_0=2$$

![20240124142937](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124142937.png)

传递函数：

$$G(s)=\frac{V_{out}(s)}{V_{in}(s)}=\frac{-\frac{A^3_0}{(1+\frac{s}{w_0})^3}}{1+\frac{A^3_0}{(1+\frac{s}{w_0})^3}}=\frac{-A_0^3}{(1+\frac{s}{w_0})^3+A^3_0}$$

分式分母：

$$(1+\frac{s}{w_0})^3+A^3_0=\left ( 1+\frac{s}{w_0}+A_0 \right )\left [ \left ( 1+\frac{s}{w_0} \right)^2-\left ( 1+\frac{s}{w_0} \right)A_0+A^2_0\right ]=0$$

闭环系统极点：

$$s_0=(-A_0-1)w_0$$

$$s_{2,3}=\left [ \frac{A_0(1\pm j\sqrt3)}{2}-1 \right ]w_0$$

将这3个特征根画在复平面上

![20240124143006](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124143006.png)

可知

 $A_0=2,s_{2,3}=\left [ \frac{A_0(1\pm j\sqrt3)}{2}-1 \right ]w_0$ ，时，系统存在等幅值振荡；（线性）

 $A_0>2,s_{2,3}=\left [ \frac{A_0(1\pm j\sqrt3)}{2}-1 \right ]w_0$ ，时，系统存在增幅振荡（电源电压为定值，该振荡会饱和，进入非线性状态，会引起频率的改变）。

只取输出信号没有饱和的情况。
$$V_{out}(t)=a\exp\left( \frac{A_0-2}{2}w_0t \right)cos\left( \frac{A_0\sqrt{3}}{2}w_0t \right)$$

理想情况下，VCO输入和输出呈线性关系

![20240124144009](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124144009.png)

 $w_{out}=w_0+K_{VCO}V_{out}$ ，其中 $K_{VCO}$ 为  $VCO$  的增益，其单位为：  $rad/(s.v)$ 

对于一个交流信号，气象为变化的速度即频率，因此

$$w=\frac{d\phi}{dt}$$

所以VCO输出信号的相位为

$$\phi_{out}=\int w_{out}dt+\phi_0=w_0t+K_{VCO}\int V_{cont}dt+\phi_0$$

在VCO中，我们最关心盈出相位


$$\phi_{ex}=K_{VCO}\int V_{cont}dt \Rightarrow \phi_{ex}(s)=\frac{K_{VCO}}{s}V_{cont}(s)$$


因此，可以将VCO理解成输入为控制电压，输出为盈出相位的系统，因此该系统是一个理想的积分器，其传递函数为

$$H_{VCO}(s)=\frac{\phi_{ex}}{V_{cont}}(s)=\frac{K_{VCO}}{s}$$




