# Fitting

提取边缘之后，需要根据边缘点拟合出几何模型（直线、圆、曲线等），即使用数学模型描述边缘。

## 难点：

- 噪声 Noise
- 外点 Outliers
- 信息丢失 Missing data

## 常用方法

<table border="1">
    <tr>
        <th>场景</th>
        <th>方法</th>
    </tr>
    <tr>	 
        <td>已知点基本都在同一条线上</td>
        <td>最小二乘法 Least Squares</td>
    </tr>
    <tr>
        <td>存在较多外点 噪声点多</td>
        <td>Robust Fitting、RANSAC</td>
    </tr>
    <tr>
        <td>图像中存在多条线</td>
        <td>RANSAC、Hough Transform</td>
    </tr>
</table>


## Least Squares Line Fitting

### 核心思想

最小化：

$$
\sum (y_i - m*x_i -b )^2
$$

### 含义：

- 最小化 **y 方向残差（vertical residual）**
- 距离并不是点到直线的真实距离
- 目标是使残差平方和最小

### 缺点：

1. 无法表示竖直线
      - 因为采用 $y = mx + b$ 竖直线斜率趋于无穷大。

2. 对坐标旋转不具有不变性
      - 同一组点旋转后，拟合结果会改变。

## Total Least Squares（Orthogonal Least Squares）

### 核心思想

最小化：

$$
\sum_i{(\frac{ax_i+by_i+d}{\sqrt{a^2 + b^2}})^2}
$$

直线通常约束为：

$$
a^2 + b^2 = 1
$$

此时目标为：

$$
\sum{(ax_i + by_i +d)^2}
$$

### 特点：

- 使用点到直线的正交距离
- 可以处理任意方向的直线
- 对旋转具有不变性

### 缺点：

- 对外点非常敏感

## Robust estimators

### 核心思想

不同距离的点对总误差的贡献不同：

- 小残差 → 权重大
- 大残差 → 权重逐渐减小

### 常见M-estimators：

- Huber
- Tukey
- Geman-McClure
- Cauchy

例如 Geman-McClure 函数：

$$
\rho = \frac{u^2}{\sigma ^ 2 + u^2}
$$

- $u$ → 残差
- $\sigma$ → 控制外点影响范围

考虑不同距离的点对误差的贡献，需要选择合适的 $\sigma$

## RANSAC 随机采样一致性

### 流程

1. 随机选择最小样本集（直线需要 2 个点）
2. 根据样本建立模型
3. 计算其余点到模型的距离
4. 小于 Threshold 的点视为 Inliers
5. 统计支持该模型的 Inliers 数量
6. 重复上述过程
7. 选择拥有最多 Inliers 的模型，并可利用全部 Inliers 再次拟合

### 参数：

- Sample size（最小样本数）
- Distance threshold（距离阈值）
- Iteration number（迭代次数）

## Adaptive RANSAC

### 核心思想：

- 在迭代过程中动态估计外点率，从而更新所需迭代次数。

```
N = ∞
sample_count = 0

while sample_count < N:

    RANSAC()

    估计外点率 e

    根据 e 计算新的迭代次数 N*

    if N* < N:
        N = N*

    sample_count += 1
```

其中：

- 外点率：e
- 内点率：$w = 1 - e$

若每次随机取 $s$ 个点，希望成功概率达到 $p$，则所需迭代次数：

$$
N = \frac{log(1-p)}{log(1-w^s)}
$$

### Adaptive RANSAC 的本质就是：

> **随着发现更多内点，动态减小所需迭代次数，从而避免固定迭代次数造成的浪费。**


## Overfitting vs Underfitting

### Underfitting

模型过于简单：

- 无法描述真实数据
- 误差较大

### Overfitting

模型过于复杂：

- 连噪声也被拟合
- 泛化能力下降

目标：

> 在模型复杂度和拟合误差之间取得平衡。


## Parametric vs Non-parametric Fitting

### Parametric Fitting

- 模型形式已知，只需求参数。
- 例如：Line，Circle，Ellipse，Polynomial
- 特点：
      - 参数少
      - 计算效率高

### Non-parametric Fitting

- 模型形式未知，由数据决定形状。
- 例如：Spline，Kernel Regression
- 特点：
      - 更灵活
      - 计算复杂度较高