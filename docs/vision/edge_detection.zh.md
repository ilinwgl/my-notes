# 边缘检测

边就是信号突变的位置，也就是梯度幅值的局部最大值位置。

边缘检测的核心目标：

- 找到图像中**亮度/结构发生突变的位置**，通常对应梯度显著变化或信号不连续点。

## 边缘来源

- 几何边
- 表面法线变化
- 材质/颜色变化
- 光照变化

在计算机视觉中我们通过卷积来实现求导，也就是求梯度。

<div style="display:flex; gap:20px;">

<table border="1">
    <tr>
        <td>-1</td>
    </tr>
    <tr>
        <td>+1</td>
    </tr>
</table>

<table border="1">
    <tr>
        <td>-1</td>
        <td>+1</td>
    </tr>
</table>

</div>

## 一阶求导（Gradient-based）

**核心思想：**
> <span style="color:#d62728; font-weight:bold;">一阶导数 = 梯度</span> → <span style="color:#1f77b4; font-weight:bold;">边缘 = 梯度幅值大的位置</span>

### Prewitt

<div style="display:flex; gap:20px;">
    <table border="1">
        <tr>
            <td>-1</td>
            <td>0</td>
            <td>+1</td>
        </tr>
        <tr>
            <td>-1</td>
            <td>0</td>
            <td>+1</td>
        </tr>
        <tr>
            <td>-1</td>
            <td>0</td>
            <td>+1</td>
        </tr>
    </table>
    <table border="1">
        <tr>
            <td>-1</td>
            <td>-1</td>
            <td>-1</td>
        </tr>
        <tr>
            <td>0</td>
            <td>0</td>
            <td>0</td>
        </tr>
        <tr>
            <td>+1</td>
            <td>+1</td>
            <td>+1</td>
        </tr>
    </table>
</div>

- 3x3 固定卷积核
- 用均匀权重估计梯度
- 特点：
    - 简单
    - 对噪声敏感
    - 精度一般

### Sobel
<div style="display:flex; gap:20px;">
    <table border="1">
        <tr>
            <td>-1</td>
            <td>0</td>
            <td>+1</td>
        </tr>
        <tr>
            <td>-2</td>
            <td>0</td>
            <td>+2</td>
        </tr>
        <tr>
            <td>-1</td>
            <td>0</td>
            <td>+1</td>
        </tr>
    </table>
    <table border="1">
        <tr>
            <td>-1</td>
            <td>-2</td>
            <td>-1</td>
        </tr>
        <tr>
            <td>0</td>
            <td>0</td>
            <td>0</td>
        </tr>
        <tr>
            <td>+1</td>
            <td>+2</td>
            <td>+1</td>
        </tr>
    </table>
</div>

- 在Prewitt基础上加入中心加权
- 特点：
    - 中心权重更高
    - 抗噪能力更强
    - 可分离 → 一维平滑核与一维差分核组合构成 例如 $[-1, 2, -1]^T · [-1, 0, +1]$

### Scharr

<table border="1">
        <tr>
            <td>-3</td>
            <td>0</td>
            <td>+3</td>
        </tr>
        <tr>
            <td>-10</td>
            <td>0</td>
            <td>+10</td>
        </tr>
        <tr>
            <td>-3</td>
            <td>0</td>
            <td>+3</td>
        </tr>
    </table>

- 特点：
    - 更好的旋转对称性
    - 更高精度方向估计
    - 比Sobel更稳定

### Roberts

<div style="display:flex; gap:20px;">
    <table border="1">
        <tr>
            <td>0</td>
            <td>+1</td>
        </tr>
        <tr>
            <td>-1</td>
            <td>0</td>
        </tr>
    </table>
    <table border="1">
        <tr>
            <td>1</td>
            <td>0</td>
        </tr>
        <tr>
            <td>0</td>
            <td>-1</td>
        </tr>
    </table>
</div>

- Roberts 算子计算对角线方向的差分，用于检测45°方向边缘
- 特点：
    - 极其敏感
    - 对噪声很弱
    - 适合快速粗检测

### 高斯偏导核

- 图片有噪声时，我们需要先对图片使用高斯滤波器进行去噪，然后再对去噪结果进行求导，找到极值点。→ **带平滑的梯度算子**

- 这个过程中，高斯滤波器和求梯度的过程都是进行卷积，因此可以将这两个合并在一起，也就是对高斯核求导，即高斯偏导核。→ 一个**已经包含平滑效果的一阶导数滤波器**
    - 高斯核 → 过滤高频信息
    - 高斯偏导核 → 提取边缘信息

- 是Canny的梯度计算基础

## 二阶求导（Second derivative）
**核心思想：**
> <span style="color:#d62728; font-weight:bold;">边缘 = 二阶导数的零交叉点（zero-crossing）</span>

### Laplacian

- 对图片直接求二阶导，寻找零交叉点
    - 不分方向（isotropic）
    - 二阶导算子
- 特点：
    - 对噪声极其敏感
    - 边缘 + 噪声混合

**二阶导 = 强化高频信号 + 噪声 = 高频信号 → Laplacian = “放大噪声机器”**

### LoG

- 标准流程：
    1. Gaussian smoothing（去噪）
    2. Laplacian（求二阶导）→ 找结构变化，强化边缘
    3. zero-crossing（找边缘）
- 先平滑再找变化
- 尺度控制
    - 小 $\sigma$ → 细节边缘
    - 大 $\sigma$ → 只保留大结构
- 特点：
    - 比 Laplacian 稳定
    - 经典理论模型

## 近似/差分

### DoG

**核心思想：**
> **类似于LoG，对图片使用两种高斯模糊：**
> <span style="color:#d62728; font-weight:bold;">$DoG = G(\sigma_1) − G(\sigma_2), where \sigma_2 > \sigma_1$</span>

- Gaussian是模糊器：
    - 小 $\sigma$ → 轻微模糊
    - 大 $\sigma$ → 强烈模糊
- 轻微模糊减强烈模糊，在平坦区域差值几乎为0，在边缘区域差异很大，响应强烈，也就可以提取边缘信息。
- 特点：
    - 计算更快
    - 多尺度分析
    - SIFT基础思想之一

## 工程常用

### Canny

- 检测流程
    1. **高斯平滑（Gaussian Smoothing）**
          - 对图像进行高斯滤波，抑制噪声，减少后续梯度计算的误差
    2. **计算梯度（Gradient Computation）**
          - 使用导数算子（如 Sobel 或高斯偏导核）计算图像的梯度幅值与方向
    3. **非极大值抑制（Non-Maximum Suppression, NMS）**
          - 沿梯度方向进行局部极大值检测，仅保留可能的边缘像素，使边缘变细化（thin edges）
    4. **双阈值检测（Double Thresholding）**
          - 根据高、低阈值将像素分为：
              - 强边缘（strong edges）
              - 弱边缘（weak edges）
              - 非边缘（suppressed）
    5. **滞后连接（Hysteresis）**
          - 保留与强边缘连通的弱边缘，去除孤立噪声点

#### **非最大化抑制**

- 沿梯度方向（quantized direction，如0°,45°,90°,135°）比较前后两个像素，如果梯度值为最大则保留。

#### **双阈值滞后连接（Hysteresis Thresholding）**

- **高阈值（high threshold）**：检测到的像素被判定为**强边缘（strong edges）**，一定保留
- **低阈值（low threshold）**：检测到的像素被判定为**弱边缘（weak edges）**，可能是边缘，也可能是噪声

- 最终结果通过连通性进行筛选：
    - **只有与强边缘相连的弱边缘才会被保留**
    - 与强边缘不连通的弱边缘会被抑制（去除）

👉 本质是利用**边缘的空间连续性（connectivity）**，在保留真实边缘的同时抑制噪声。

#### 总结
> <span style="color:#d62728; font-weight:bold;">Canny = Gaussian smoothing → Gradient → NMS → Double threshold → Hysteresis</span>

- 高检测率（good detection）
- 精确定位（good localization）
- 单一响应（single response）

## 非梯度方法

### 形态学（Morphology）

**核心思想:**
> **基于形态学操作：**
> <span style="color:#d62728; font-weight:bold;"> $Edge = Dilate(I) - Erode(I)$ </span>

- Dilate让亮的区域变大，物体变粗
- Erode让亮的区域变小，物体变细
- Dilate - Erode ⇒ 边界被扩张和收缩之间的差异 → 轮廓区域
- 特点：
    - 简单
    - 适合 binary / 工业图像
    - 不依赖梯度

### Phase Congruency

**核心思想:**
> <span style="color:#d62728; font-weight:bold;">边缘 = 多尺度傅里叶中“相位一致”的位置 </span>

- 图像可以分解为：
    - 不同频率（frequency）
    - 不同相位（phase）

- 看信号结构是否对齐
    - 边缘点 → 所有频率成分（低频 + 高频）在这个点“同时对齐” → 能量集中
    - 非边缘点 → 各个频率不同步，相位乱 → 不稳定、分散

- 特点：
    - 不依赖亮度
    - 对光照极鲁棒
    - 接近人类感知

![Edge_Detection](/docs/images/edge_detection_demo.png)