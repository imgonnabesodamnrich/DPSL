# 论文笔记：深度渐进收缩学习（Deep Progressive Shrinkage Learning, DPSL）

>针对强噪声环境下机械复合故障诊断中存在的特征重叠问题
> 
>**Paper Title:** Mechanical compound fault diagnosis via suppressing intra-class dispersions: A deep progressive shrinkage perspective
>
>**Journal:** Measurement (Elsevier), Vol. 199, 2022
>
>**DOI:** [10.1016/j.measurement.2022.111433](https://doi.org/10.1016/j.measurement.2022.111433)

![Journal](https://img.shields.io/badge/Journal-Measurement-blue) ![DOI](https://img.shields.io/badge/DOI-10.1016%2Fj.measurement.2022.111433-green) ![Topic](https://img.shields.io/badge/Topic-Deep_Learning-orange) ![Topic](https://img.shields.io/badge/Topic-Fault_Diagnosis-yellow)


## 目录
* [1. 研究背景](#1-研究背景)
* [2. 问题阐述：类内离散与特征重叠](#2-问题阐述类内离散与特征重叠)
* [3. 方法论：深度渐进收缩学习 (DPSL)](#3-方法论深度渐进收缩学习-dpsl)
    * [3.1 特征级收缩：自适应软阈值去噪](#31-特征级收缩自适应软阈值去噪)
    * [3.2 决策级收缩：中心损失约束](#32-决策级收缩中心损失约束)
* [4. 实验验证](#4-实验验证)
* [5. 结论](#5-结论)

---

### 1. 研究背景

在旋转机械的故障诊断任务中，复合故障（Compound Faults）指两个或多个单一故障同时发生的情况（例如轴承内圈与外圈同时受损）。相比于单一故障，复合故障的振动信号特征更为复杂。传统的深度学习方法在处理单一故障时表现较好，但在强噪声干扰下识别复合故障时，准确率往往受到限制。

### 2. 问题阐述：类内离散与特征重叠

论文认为，复合故障诊断的主要难点在于特征空间中的**类间重叠（Inter-class overlap）**。造成这一现象的原因主要有两点：

1.  **复合故障的耦合性**：复合故障包含单一故障的特征成分，导致其在特征空间中的分布与相关的单一故障高度相似。
2.  **噪声引起的类内离散（Intra-class dispersion）**：工业现场的强背景噪声会使同一类故障样本的特征分布变得松散。高类内离散度会导致不同类别的特征边界模糊，进而产生重叠。

因此，为了提高诊断精度，需要同时实现“消除噪声干扰”与“抑制类内离散”。

<div align="center">
  <img width="90%" src="https://github.com/user-attachments/assets/d1ed046d-9ff9-49b8-bdc7-e893a54e22e1" />
  <p><em>图1 样本分布示意图 </em></p>
</div>

### 3. 方法论：深度渐进收缩学习 (DPSL)

针对上述问题，论文构建了 DPSL 框架，通过特征级和决策级两个层面的收缩策略来优化特征分布。

<div align="center">
  <img width="60%" src="https://github.com/user-attachments/assets/b8911a48-c812-49a8-8154-5e31e9f9c67f" />
  <p><em>图2 DPSL整体架构 </em></p>
</div>

#### 3.1 特征级收缩：自适应软阈值去噪

在特征提取阶段，该方法引入了**残差收缩构建单元 (Residual Shrinkage Building Unit, RSBU)**。

*   **机制**：利用软阈值（Soft Thresholding）函数作为非线性层。
*   **原理**：软阈值函数将绝对值低于阈值的特征置为零，这些特征通常对应噪声；绝对值高于阈值的特征则向零收缩。
*   **自适应性**：阈值并非人工固定，而是通过一个子网络利用注意力机制根据输入信号的特征自适应学习得到。这使得模型能够在逐层特征提取过程中，保留有用故障信息并剔除噪声相关特征。

<div align="center">
  <img width="60%" src="https://github.com/user-attachments/assets/bbb1e413-e76c-42a3-82ae-dbd7a5faee1c" />
  <p><em>图3 (a) RBU (b) RSBU </em></p>
</div>

#### 3.2 决策级收缩：中心损失约束

在分类决策阶段，为了解决类内特征分布松散的问题，该方法联合使用了**交叉熵损失（Cross-entropy Loss）**与**中心损失（Center Loss）**。

*   **交叉熵损失**：用于区分不同类别的特征（增大类间距离）。
*   **中心损失**：用于约束特征空间内部的分布。它通过惩罚特征与其对应类中心的距离，迫使同类样本在特征空间中更加紧凑（减小类内距离）。
*   **效果**：通过联合优化，模型输出的特征在空间分布上呈现出“类内紧致、类间分离”的特性，从而减少了复合故障与单一故障的重叠区域。

### 4. 实验验证

实验采用了传动系统诊断模拟器（DDS）数据集，包含健康、单一故障及复合故障数据，并添加了不同信噪比（SNR）的高斯白噪声进行测试。

**表1 行星齿轮箱的健康状态类别**

| 类别 | 描述 | 标签 | 信号长度 (数据点) | 样本数 |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 健康状态 (轴承和齿轮) | H | 1024 | 800 |
| 2 | 齿根裂纹 (齿轮) | F1 | 1024 | 800 |
| 3 | 表面点蚀 (齿轮) | F2 | 1024 | 800 |
| 4 | 断齿 (齿轮) | F3 | 1024 | 800 |
| 5 | 缺齿 (齿轮) | F4 | 1024 | 800 |
| 6 | 滚珠故障 (轴承) | F5 | 1024 | 800 |
| 7 | 内圈故障 (轴承) | F6 | 1024 | 800 |
| 8 | 外圈故障 (轴承) | F7 | 1024 | 800 |
| 9 | 复合故障 (轴承) | C8 | 1024 | 800 |

**主要实验结论：**

1.  **抗噪性能**：在 -3dB 至 9dB 的不同噪声环境下，DPSL 的平均诊断准确率均优于对比方法（CNN, DRN, DRSN）。在 -3dB 的强噪声环境下，传统 CNN 方法准确率显著下降，而 DPSL 仍保持了较高的识别率。
2.  **泛化能力**：混淆矩阵分析显示，DPSL 在区分复合故障（如内圈+外圈+滚珠复合故障）及其对应的单一故障方面表现更优，误判率更低。
3.  **可视化验证**：t-SNE 特征可视化结果表明，经过 DPSL 处理后的高层特征在二维空间中呈现出清晰的聚类效果，不同故障类别之间的边界明确，验证了该方法抑制类内离散的有效性。

### 5. 结论

该研究提出了一种结合软阈值去噪与中心损失约束的诊断框架。实验证明，该方法能有效降低噪声干扰并压缩类内特征分布，从而提升强噪声环境下机械复合故障的诊断精度。

---

### 文献来源

**Paper Title:** Mechanical compound fault diagnosis via suppressing intra-class dispersions: A deep progressive shrinkage perspective

**Journal:** Measurement (Elsevier), Vol. 199, 2022

**DOI:** [10.1016/j.measurement.2022.111433](https://doi.org/10.1016/j.measurement.2022.111433)
