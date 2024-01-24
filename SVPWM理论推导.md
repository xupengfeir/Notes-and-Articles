# SVPWM理论推导

三相电机控制最本质的原理就是：在A,B,C三相提供120°相位差的正弦电压，电机就会稳定的转动起来，调节正弦电压的幅值和频率，就能调节电机的转速和扭矩，这就是我们最终想要的输出。电路中所提供的输入是稳压直流电源，电机控制环节在于通过6路PWM，控制6个MOS管的开断，来达到直流电源变正弦交流的目的。而这就是电机控制的核心：PWM。PMSM常用的PWM控制技术有SPWM和SVPWM等。

![20240124164023](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124164023.png)

## 1、SPWM
在推导SVPWM之前，先对SPWM有一个清晰的认识。

正弦脉宽调制（Sinusoidal PWM，SPWM）原理：使用高频三角波作为载波，把需要输出的三相对称正弦波作为调制波，直接将所需要的正弦波等效成一系列幅值相等占空比不等的矩形波。在某一时刻需要多大的电压，调节一下PWM的占空比等效出来。常见的SPWM调制方式有两种：（a）单极性调制发（b）双极性调制法。

![20240124164114](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124164114.png)

单双极性的区别在于：输出端电压极性，单极性在调制波为正时，输出方波电压为正，在调制波为负时，输出方波电压为负；而双极性无论调制波的正负，输出方波电压均有正有负。

双极性的优点是不用去判断调制波的正负，比较调制波与载波的幅值即可，缺点是开关管开关通断次数增加，开关损耗增加。常用的也是双极性调制，一般应用于调速系统中。

**SPWM：相电压 $\frac{U_{dc}}{2}$ ，线电压 $\frac{\sqrt{3}U_{dc}}{2}$。（具体推导见SVPWM节）**

直流母线电压利用率定义：输出线电压波形幅值/直流母线电压	。
SPWM母线电压利用率（具体推导见SVPWM节）：

$$\frac{\frac{\sqrt{3}V_{dc}}{2}}{V_{dc}}=0.867$$

## 2、SVPWM

SVPWM是由三相功率逆变器的六个功率开关元件组成的特定开关模式产生的脉宽调制波，能够使输出电流波形尽可能接近于理想的正弦波。SVPWM与SPWM不同，它是从三相输出电压的整体效果出发，着眼于如何使电机获得理想圆形磁链轨迹。SVPWM技术与SPWM技术与SPWM相比较，绕组电流波形的谐波成分小，使得电机转矩脉动降低，旋转磁场更逼近圆形，而且使直流母线电压的利用率有了很大提高，且更易于实现数字化。

### 2.1 SVPWM基本原理
SVPWM的理论基础是平均值等效原理，记载一个开关周内通过对基本电压矢量加以组合，使其平均值与给定的电压矢量相等。在某个电压时刻，电压矢量旋转到某个区域中，可由组成这个区域的两个相邻的非零矢量和零矢量在时间上的不同组合来得到。

设直流母线侧电压为Udc，逆变器输出的三相电压为UA,UB,UC，其分别加载空间上互差120°的三相平面静止坐标系上，可以定义三个电压空间矢量UA(t),UB(t),UC(t)，方向始终在各相的轴线上，而大小则随时间按正弦规律做变化，时间相位互差120°。假设Um为相电压峰值，f为电源频率。

$$U_{A}(t)=U_mcos(\theta)$$

$$U_{B}(t)=U_mcos(\theta-\frac{2\pi}{3})$$

$$U_{C}(t)=U_mcos(\theta+\frac{2\pi}{3})$$

其中 $\theta=2\pi ft$, 则三相电压矢量相加合成空间矢量 $U(t)$ 为：

$$U(t)=U_{A}(t)+U_{B}(t)e^{\frac{j2\pi}{3}}+U_C(t)e^{\frac{j4\pi}{3}}=\frac{3}{2}U_me^{j\theta}$$

可见 $U(t)$ 是一个旋转的空间矢量，幅值为相电压峰值的1.5倍，Um为相电压峰值。空间矢量U(t)在三相坐标轴(a,b,c)上的投影就是对称的三相正弦量。

**2.1.1 母线电压利用率**

在实际的SVPWM调制中，是俗称的马鞍形电压波，是指端点对地电压，记abc为端点，n为星接点，g为地点，实际上Uab是正弦的，Uan是正弦的；Uag在SVPWM中是马鞍形的，在SPWM中还是正弦的。

在SVPWM中，星接点n对地的电压Ung是波动的，并不是固定的，表现为三次谐波；在SPWM中星接点n对地的电压Ung是固定的，表现为直流分量。

**SVPWM：  $U_{ag}=U_{an}+U_{ng}$ ，等效为：马鞍形波=正弦波+三次谐波；**

**SPWM：  $U_{ag}=U_{an}+U_{ng}$ ，等效为：正弦波=正弦波+直流分量**

且不管是SVPWM还是SPWM，某相的端点对地最高电压Uag、Ubg、Ucg都能到Vdc（直流母线电压）。下图（a）表示SPWM接地电压和线电压的关系，（b）表示SVPWM接地电压和线电压的关系。

![20240124184418](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124184418.png)

SPWM调制，相电压幅值为 $Udc/2$ ，线电压幅值是 $\sqrt{3}U_{dc}/2$ ，利用率为0.867；而SVPWM调制，根据基尔霍夫电流定律，相电压最高为 $\frac{2}{3}U_{dc}$ ，但如图（b）线电压幅值Udc和相电压却不满足 $\sqrt{3}$ 倍的关系，为什么呢？

首先看相电压和线电压之间的数学公式：

$$U_{an}=Asin(x),U_{bn}=Asin(x-120)$$

$$U_{ab}=U_{an}-U_{bn}=\sqrt{3}Asin(x+30)$$

因为SPWM相电压始终是正弦波，所以线电压和相电压始终保持 $\sqrt{3}$ 倍幅值的关系；但是SVPWM的线电压和相电压虽然是正弦波，但是并不是直接乘 $\sqrt{3}$ 的倍数关系，这又是为什么呢？

因为SVPWM调制时，它的星接点电压是浮动的，具体是 $U_{ag}=A_{3w}sin(3\theta)$ 的三次谐波， $A_{3w}$ 可由傅里叶级数计算得到。此时相电压是在星接点（中性点）电势（对地的三次谐波）波动的基础上的正弦波（基波），所以线电压和相电压不呈简单的 $\sqrt{3}$ 倍关系。

因此以上分析可知，SPWM母线电压利用率为0.867，SVPWM母线电压利用率为1，SVPWM母线电压利用率比SPWM母线电压利用率高(1-0.8667)/0.8667=15.4%。

**2.1.2 电压矢量合成**

前面提到，SVPWM线电压幅值是，由于星接点n对g的电势是时刻在波动的，电势波动范围[Udc/3,2Udc/3]，因此相电压Uan幅值最大值为2Udc/3。U4由相电压Uan，Ubn，Ucn矢量合成而来，U4计算公式：

$$U_4=\frac{2}{3}U_{dc}cos(\theta)+\frac{2}{3}U_{dc}cos(\theta-\frac{2}{3}\pi)e^{\frac{j2\pi}{3}}+\frac{2}{3}U_{dc}cos(\theta+\frac{2\pi}{3})e^\frac{j4\pi}{3}=U_{dc}e^{j\theta}$$

可见，合成电压矢量 $U_4$ 的模为  $U_{dc}$，而不是一些书籍里面写的  $\frac{2}{3}U_{dc}$。

使用**伏秒平衡原则**计算基本电压空间矢量的幅值(以下假设在第Ⅰ扇区)：

![20240124185418](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124185418.png)

![20240124185511](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124185511.png)

因此最大线性调制输出是正六边形的内接圆，也称为最大不失真边界，合成矢量电压输出是一个幅值为 $\frac{\sqrt{3}}{2}U_{dc}$ 旋转的正弦波信号。正六边形是   $T_4+T_6=T_s$ ，即零矢量作用时间 $T_0$ 时合成矢量电压输出最大边界。

当 $U_{out}$ 处于内切圆和正六边形之间时，此时为失真状态；当 $U_{out}$ 处于正六边形和外切圆之间时，即 $T_4+T_6\ge T_s$ ，此时处于过调制状态，严重失真，程序上应对此种情况进行处理。

![20240124185800](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124185800.png)

**补充：**
由前面合成电压矢量公式可知，相电压幅值为 $\frac{2}{3}U_{dc}$ , 可以合成矢量圆 $U_{dc}e^{j\theta}$ ,但最终实际得到的是一个边长为 $U_{dc}$ 的正六边形，可知相电压 $U_mcos(\theta)$ 中的幅值 $U_m$ 小于 $\frac{2}{3}U_{dc}$ (受中性点波动的影响)。又因为SVPWM目的是合成一个完整的电压圆，即如上图所示的内切圆，可计算出 $U_m=\frac{\sqrt{3}}{2}\frac{2}{3}U_{dc}$ , 所以 $\frac{\sqrt{3}}{3}U_{dc}$ 是等效之后的相电压幅值（去掉了中性点波动的影响），等效相电压 $U_{an}=\frac{\sqrt{3}}{3}U_{dc}cos(\theta)$, 此时线电压


$$U_{ab}=U_{an}-U_{bn}=\frac{\sqrt{3}}{3}U_{dc}cos(\theta)-\frac{\sqrt{3}}{3}U_{dc}cos(\theta-\frac{2\pi}{3})=U_{dc}cos(\theta+\frac{\pi}{6})$$


运算结果解释了为什么SVPWM调制线电压是 $U_{dc}$ , 直流母线利用率为1。

总结：最大不失真电压幅值 $\frac{\sqrt{3}}{2}U_{dc}$ ，合成矢量电压  $|U_4|=U_{dc}$  ，线电压 $U_{ab}=U_{dc}cos(\theta+\frac{\pi}{6})$ ，等效相电 压 $U_{an}=\frac{\sqrt{3}}{3}U_{dc}cos(\theta)$ ，实际相电压 $U_{an}=\frac{2}{3}U_{dc}cos(\theta)$  或   $U_{an}=\frac{1}{3}U_{dc}cos(\theta)$  （未考虑方向）。

**2.1.3 开关函数定义**

定义开关函数Sx(x=a,b,c)为：

$$S_x=\begin{cases}
1 \quad\quad1上桥臂导通\\
0 \quad\quad0下桥臂导通
\end{cases}$$ 

![20240124191604](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124191604.png)

(Sa,Sb,Sc)的全部可能组合有八个，包括6个非零矢量U1（001）、U2（010）、U3（011）、U4（100）、U5（101）、U6（110）和两个零矢量U0（000）、U7（111）。

![20240124191626](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124191626.png)

### 2.2 SVPWM法则推导

SVPWM算法的理论基础是平均值等效原理，记载一个开关周期Ts内通过对基本电压矢量组合，使其平均值与给定电压矢量相等。伏秒平衡原则公式如下。

$$T_sU_{out}=T_4U_4+T_6U_6+T_0U_0$$
$$T_0+T_4+T_6=T_s$$

在两相参考坐标系（α，β）中，令 $U_{out}$ 和α轴之间的夹角θ。

alpha轴：  $|U_{out}|cos\theta=\frac{T_4}{T_s}|U_4|+\frac{T_6}{T_s}|U_6|cos\frac{\pi}{3}$

beta轴：   $|U_{out}|sin\theta=\frac{T_6}{T_s}|U_6|sin\frac{\pi}{3}$

因为 $|U_4|=|U_6|=\frac{2}{3}U_{dc}$ （后面涉及到静止坐标系上的运算统一使用  $\frac{2}{3}U_{dc}$），所以得到各矢量的状态保持时间为：

$$\begin{cases}
T_4=mT_ssin(\frac{\pi}{3}-\theta) \\
T_6=mT_ssin(\theta)
\end{cases}$$

式中m为SVPWM调制系数，  $m=\frac{\sqrt{3}|U_{out}|}{U_{dc}}$

零电压矢量分配的时间：  $T_7=T_0=(T_s-T_4-T_6)/2$ (七段式)或者 $T_7=(T_s-T_4-T_6)$ (五段式)。

**补充：合成矢量 $|U_4|=U_{dc}$ 还是 $|U_4|=\frac{2}{3}U_{dc}$ ，具体看使用的坐标系。前面分析相电压和线电压的关系时，得出 $|U_4|=U_{dc}$ 是在三相坐标系下结论；而这里在alpha-beta坐标系下求调制系数m，则  $|U_4|=\frac{2}{3}U_{dc}$ （3s-2s等幅值变换）。**

**假设假设合成电压矢量幅值是一个和任何值无关的常数值，令 $|U_4|=|U_6|=U_v$ ，那么  $m=\frac{2\sqrt{3}|U_{out}|}{3U_v}$ 。 $|U_{out}|$ 最大只能取 $U_v$ 或者最大不失真圆 $\frac{\sqrt{3}}{2}U_v$ ，因此不管U(1~6)等于多少，  $\frac{|U_{out}|}{U_v}$ 始终是一个数值，调制系数m大小和 $U_v$ 无关。即 $U_v=\frac{2}{3}U_{dc}$ 还是  $U_v=U_{dc}$ ，T4或T6在  $|U_4|=|U_6|=\frac{2}{3}U_{dc}$  和  $|U_4|=|U_6|=U_{dc}$  时值时一致的，而时间T4，T6决定了PWM的输出，所以最终反映在PWM占空比输出上，合成电压矢量等于 $\frac{2}{3}U_{dc}$ 和 $U_{dc}$ 是一样的。**

**如果认为U(1~6)是等于 $\frac{2}{3}U_{dc}$ ，那我们认为 $U_{out}$ 最大只能到 $\frac{2}{3}U_{dc}$ (六边形)，或者 $\frac{2}{3}U_{dc}*\frac{\sqrt{3}}{2}$ (内切圆)。**

**如果认为U(1~6)是等于 $U_{dc}$ ，那我们认为 $U_{out}$ 最大只能到 $U_{dc}$ (六边形)，或者 $U_{dc}*\frac{\sqrt{3}}{2}$ (内切圆)。**

**不管U(1~6)等于谁，MCU对PWM操作的占空比的规律是一致的。**


**2.2.1 7段式SVPWM（软件模式）**

在SVPWM方案中，零矢量的选择是最具灵活性的，适当选择零矢量，可最大限度减少开关次数。七段式SVPWM的基本矢量作用顺序的分配原则选定为：在每次开关状态转换时，只改变其中一项的开关状态，并且对零矢量在时间上进行平均分配，意识产生的PWM对称，从而有效降低PWM的谐波分量。**波形对称，谐波含量少，每个开关周期有6次开关切换**。

![20240124194326](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124194326.png)
![20240124194333](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124194333.png)


**2.2.2 5段式SVPWM（硬件模式）**

该方法采用每相开关器件在每个扇区状态维持不变的序列安排下，使得每个开关周期只有4次开关切换，但是会增大电流的谐波分量。

![20240124194428](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124194428.png)
![20240124194436](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124194436.png)
![20240124194449](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124194449.png)


### 2.3 SVPWM算法的实现

要实现SVPWM信号的实时调制，首先需要知道合成的参考电压矢量 $U_{out}$ 所在的区间位置，然后利用所在扇区的相邻两电压矢量和适当的零适量来合成参考电压矢量。

**2.3.1 合成矢量 $U_{out}$ 的扇区判断**

用 $u_{\alpha}$ 和 $u_{\beta}$ 表示合成矢量在α和β轴上的分量，定义 $U_1$ ， $U_2$ ， $U_3$ 三个变量，令

$$U_1=u_{\beta}$$

$$U_2=\frac{\sqrt{3}}{2}u_\alpha-\frac{1}{2}u_{\beta}$$

$$U_3=-\frac{\sqrt{3}}{2}u_\alpha-\frac{1}{2}u_\beta$$


再定义3个变量A,B,C，通过分析可以得出：

若 $U_1>0$ ,则A=1， 否则A=0;

若 $U_2>0$ ,则B=1， 否则B=0;

若 $U_3>0$ ,则C=1， 否则C=0;

令 $N=4C+2B+A$, 则 $U_{out}$所在扇区：

![20240124195309](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124195309.png)


**2.3.2 基本矢量作用时间**

$$u_{\alpha}=\frac{T_4}{T_s}|U_4|+\frac{T_6}{T_s}|U_6|cos\frac{\pi}{3}$$

$$u_{\beta}=\frac{T_6}{T_s}|U_6|sin\frac{\pi}{3}$$

$$\Rightarrow T_4=\frac{\sqrt{3}T_s}{U_{dc}}(\frac{\sqrt{3}}{2}u_\alpha-\frac{1}{2}u_{\beta})$$

$$\Rightarrow T_6=\frac{\sqrt{3}T_s}{U_{dc}}u_{\beta}$$

同理，得出其他各擅取各适量的作用时间。令

$$X=\frac{\sqrt{3}T_su_1}{U_{dc}},Y=-\frac{\sqrt{3}T_su_3}{U_{dc}},z=-\frac{\sqrt{3}T_su_2}{U_{dc}}$$

得到各扇区作用时间

![20240124200101](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124200101.png)

**若 $T_4+T_6>T_s$ ，需要进行过调制处理**

$$T_4=\frac{T_4}{T_4+T_6}T_s,T_6=\frac{T_6}{T_4+T_6}T_s$$

**2.3.3 扇区矢量切换点的确定**

首先定义时间偏移量Ta，Tb，Tc。使用和PWM波频率一样的三角载波信号辅助确定各个扇区矢量的时间切换点。

![20240124200312](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124200312.png)

$$T_a=(T_s-T_4-T_6)/4,T_b=T_a+T_4/2,T_c=T_b+T_6/2$$

则三相电压开关时间 $T_{cm1}$ ， $T_{cm2}$ ， $T_{cm3}$ 与各扇区关系：

![20240124200607](https://cdn.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240124200607.png)
