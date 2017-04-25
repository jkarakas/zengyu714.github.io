---
author: kimmy
layout: post
title:  作业 X射线投影成像
date:   2017-04-25 2017-04-25 22:25:51 +08:00
categories: Assignment
tags:
- Latex
- CT
---

* content
{:toc}


## 題目

1. 有一台X射线投影成像设备，对水模成像，水模为边长为20cm的正方体，在正中心有边长为5cm的正方体人体软组织等效材料，μ值请自行上网查找，然后计算该“软组织”与周围水模的原发对比度（10分）。<br>
    假设X射线野为30cm，并假设无其他抑制对比度的措施，求散射造成影响后，对比度变为多少？（10分）<br>
    所采用的DR系统，其H-D曲线为在探测范围内完美线性，斜率为5，求最终对比度（10分）。<br><br>

1. 已知一次胸片对人体胸部的当量剂量为0.05mSv，计算一名病人接受X射线胸部扫描时的吸收剂量和辐射能量（所需射线的组织加权系数请查阅教材或者资料，设人体胸部组织共重20kg；公式里的N值取为1）（10分）<br><br>

1. X射线能量为100keV，换算为焦耳为单位，则该X射线能量是多少？其频率又是多少？（10分）<br><br>

1. 求GE某种型号CT机器（自己上网搜）的120kVp的X射线束，等效能量可以也自己搜索（必须针对该型号而言），在普通水泥中的半价层（μ值可以Google查Nist网站得到）。（20分）<br><br>

1. 上网查找某厂家DSA设备，找到1个型号即可，然后分析它的图像质量的技术参数。（15分）<br><br>

1. 上网，搜索任意1台DR设备，按照书上所说X射线投影成像系统的质量评价指标，分析系统性能具体参数值。（15分）

## 解答 1

### 1. (1)

$$
\begin{equation}
\begin{aligned}
I_1 &= I_0 \cdot e^{-\mu_2 l_2}\\
I_2 &= I_0 \cdot e^{-[\mu_2 (l_2 - l_1) + \mu_1 l_1]}\\
\Rightarrow \\
C_r &= ln(\dfrac{I_2}{I_1}) \\
   &= \dfrac {\mu_2 (l_2 - l_1) + \mu_1 l_1}{\mu_2 l_2} \\
   &= 1 - \dfrac {l_1}{l_2}(1 - \dfrac{\mu_1}{\mu_2}) \\ \\
l_1 &=5\ \qquad \mu_1 = 0.78 \\
l_2 &=20 \qquad \mu_2 = 1   \\
C_r &= 1 - 0.25 \cdot (1 - 0.78) = 0.945
\end{aligned}
\end{equation}
$$

### 1. (2)

$$
\begin{equation}
\begin{aligned}
C'_r &= ln(\dfrac{I_2 + I_s}{I_1 + I_s})\\
    &= ln(\dfrac{I_2 + s I_a}{I_1 + s I_a})\\
    &= \dfrac {C_r}{1 + s} = SC_r
\end{aligned}
\end{equation}
$$

查图表得，当厚度为5，X射线野为30cm，S ≈ 0.51

$$C'_r = 0.51 \cdot C_r \approx 0.48 $$

### 1. (3)

$$\Delta D = \gamma \cdot C'_r = 5 \cdot 0.48 \approx 2.4 $$

### 讨论
+ 人体的软组织与水的有效原子序数相近，实验证明水和软组织对X线的吸收也非常一致，可用水作软组织的仿真模型。
+ 骨的有效原子序数是14，可用原子序数为13的铝替代作仿真模型。

## 解答 2

### 2. (1) 吸收剂量

$$
H = DQN \\
D = \dfrac {H}{QN} = \dfrac {0.05mSv}{1 \cdot 1} = 0.05mSv
$$

### 2. (2) 辐射能量

$$
D = \dfrac {dE}{dm} \\
E = D \cdot m = 20kg \cdot 0.05mSv = 0.1 J
$$

## 解答 3

### 3. (1) X射线能量

$$
E = 100 keV = 100 \cdot 1.6 \cdot 10^{-16}J = 1.6 \times 10^{-14}J
$$

### 3. (2) X射线频率

$$
E = h \nu \\
\nu = \dfrac {E}{h} = \dfrac {1.6 \times 10^{-14}J}{6.6260693×10^{-34} J·s} = 2.4147 \times 10^{19} Hz
$$

## 解答 4

### 4. Discovery XR656

<p align="center"><img src="http://www3.gehealthcare.com/~/media/images/product/product-categories/radiography/discovery%20xr656%20plus/gehc-discovery-xr656-plus_spotlight.jpg" width = "500" height = "400" alt="Revolution CT"></p>

+ Minimum inherent filtration is 2.7 mm aluminum equivalent @ 71 kVp at a system level (tube + collimator)

$$
HVL_{120kVp} = \dfrac {120 kVp \cdot 2.7 mm}{71 kVp} = 4.56mm \\
E = 40keV \\
$$

$$
\mu = 1.464 \\
HVL_{concrete} = \dfrac{ln2}{\mu}= 0.47cm
$$


## 解答 5

### 5. DSA 设备：Philips Allura Xper FD20

1. 旋转采集和3D血管重建技术

    + Rotation rate, deg/sec: Variable up to 25
    + Ct image reconstruction: 25 or 60 seconds (dataset size dependent)<br><br>
2. 实时图象融合，将CT图象、MR图象与血管图象融合<br><br>
3. 图象分辨率（空间 + 时间）

    + 512 x 512 60 fps
    + 1024 x 1024 15 and 30 fps
    + Other 2048 x 2048, 0.5 to 6 fps<br><br>
4. 图象存儲

    + 1024 x 1024, Images: 50000
    + Additional Storage: up to 100,000 images
    + Digital imaging system: Integrated

### 讨论

成像越精细，动态分辨率越低，即系统对运动部位血管的瞬间成像能力降低。<br>
但是在相对较低的空间分辨率下，FD20也可以达到（30~60帧/s）的高频帧，有利于观察心脏、大血管和动静脉异常交通位置。

## 解答 6

### 6. DR 设备：Discovery XR656

1. X射线源

    +  Maxiray 100 radiographic tube
    +  Typical Dynamic Range 0.6 uR – 7.8 mR @ RQA5<br><br>

1. 图象分辨率

    + Active Matrix：2022 x 2022 pixels
    + Image Depth：	 14 Bit
    + Pixel Pitch：  200 microns<br><br>

2. 检测系统的分辨率分析

    + Typical DQE：68% (@ 0lp and RQA5, per IEC 62220-1)<br>
        即成像系统中输出信号(信噪比平方)与输入信号(信噪比平方)之比
