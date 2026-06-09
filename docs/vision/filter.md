# Filter
滤波器（Filter）是一个**权重矩阵（Kernel）**，卷积（Convolution）是利用滤波器对图像进行运算的方法。

- Filter = Kernel
- Convolution = 运算过程 → 将权重（Filter）与图片像素进行相乘叠加得到对应的值

滤波器通过卷积运算对图像进行**增强、平滑、去噪、边缘检测**和**特征提取**。

## Filter 特性

### 线性

$$
filter(af_1 + bf_2) = afilter(f_1) + bfilter(f_2)
$$

### 平移不变性

$$
filter(shift(f_1)) = shift(filter(f_1))
$$

## 边界情况

卷积后图像尺寸通常会减小，因此常通过 Padding（填充）扩展边界，使输入输出尺寸保持一致。

这样做的的目的是：**固定输入输出的尺寸**

假设原图是一个 3x3 图像：

```
1 2 3
4 5 6
7 8 9
```

### Zero Padding

用0进行填充 → CNN常用

```
0 0 0 0 0
0 1 2 3 0
0 4 5 6 0
0 7 8 9 0
0 0 0 0 0
```

### Replicate

把最外面一圈像素向外延伸

```
1 1 2 3 3
1 1 2 3 3
4 4 5 6 6
7 7 8 9 9
7 7 8 9 9
```

### Reflect

把图像沿边界反射 → OpenCV 默认最常用

```
5 4 5 6 5
2 1 2 3 2
5 4 5 6 5
8 7 8 9 8
5 4 5 6 5
```

### Circular

把图像看成首尾相连，左边接右边，上边接下边。→ FFT 常用

```
9 7 8 9 7
3 1 2 3 1
6 4 5 6 4
9 7 8 9 7
3 1 2 3 1
```

## 常见的滤波核

### 原图
<table border="1">
    <tr>
        <td>0</td>
        <td>0</td>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
        <td>1</td>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
        <td>0</td>
        <td>0</td>
    </tr>
</table>

### 左移
<table border="1">
    <tr>
        <td>0</td>
        <td>0</td>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
        <td>0</td>
        <td>1</td>
    </tr>
    <tr>
        <td>0</td>
        <td>0</td>
        <td>0</td>
    </tr>
</table>

### 右移
<table border="1">
    <tr>
        <td>0</td>
        <td>0</td>
        <td>0</td>
    </tr>
    <tr>
        <td>1</td>
        <td>0</td>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
        <td>0</td>
        <td>0</td>
    </tr>
</table>

### 均值
<table border="1">
    <tr>
        <td>1/9</td>
        <td>1/9</td>
        <td>1/9</td>
    </tr>
    <tr>
        <td>1/9</td>
        <td>1/9</td>
        <td>1/9</td>
    </tr>
    <tr>
        <td>1/9</td>
        <td>1/9</td>
        <td>1/9</td>
    </tr>
</table>

### 锐化

#### 锐化A
<table border="1">
    <tr>
        <td>-1/9</td>
        <td>-1/9</td>
        <td>-1/9</td>
    </tr>
    <tr>
        <td>-1/9</td>
        <td>17/9</td>
        <td>-1/9</td>
    </tr>
    <tr>
        <td>-1/9</td>
        <td>-1/9</td>
        <td>-1/9</td>
    </tr>
</table>

- 利用上下左右和四个对角一共8个方向
- 锐化更加均匀
- 各方向响应一致
- 更接近真实高频
- **对噪声更敏感！！！**

#### 锐化B
<table border="1">
    <tr>
        <td>0</td>
        <td>-1</td>
        <td>0</td>
    </tr>
    <tr>
        <td>-1</td>
        <td>5</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>0</td>
        <td>-1</td>
        <td>0</td>
    </tr>
</table>

- 利用上下左右四个方向 → 相当于 原图 + Laplacian
- 边缘增强较温和
- 不容易放大噪声
- 计算简单
- OpenCV中常用

#### 锐化C
<table border="1">
    <tr>
        <td>-1</td>
        <td>-1</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>-1</td>
        <td>9</td>
        <td>-1</td>
    </tr>
    <tr>
        <td>-1</td>
        <td>-1</td>
        <td>-1</td>
    </tr>
</table>

- 锐化非常强
- 边缘很明显
- 容易产生振铃（Halo）
- 噪声也会被放大

#### 总结
<table border="1">
    <tr>
        <th>核</th>
        <th>利用邻域</th>
        <th>锐化效果</th>
        <th>噪声放大</th>
        <th>特点</th>
    </tr>
    <tr>
        <td>锐化A</td>
        <td>四邻域</td>
        <td>中等</td>
        <td>较小</td>
        <td>最常用</td>
    </tr>
    <tr>
        <td>锐化B</td>
        <td>八邻域</td>
        <td>平滑自然</td>
        <td>中等</td>
        <td>Unsharp Mask</td>
    </tr>
    <tr>
        <td>锐化C</td>
        <td>八邻域</td>
        <td>很强</td>
        <td>较大</td>
        <td>容易过锐</td>
    </tr>
</table>

## 高斯滤波器

滤波核中心的权重大，边缘的权重小。

对高斯核进行归一化，可以避免衰减图像。

标准差的影响：

- $\sigma$越小，中心比重越大，平滑效果越小，模糊弱
- $\sigma$越大，中心比重越小，平滑效果越大，模糊强

窗宽影响：

- 窗宽越大，归一化时分母越多，平滑效果越大
- 窗宽越小，归一化时分母越少，平滑效果越小

经验：

$$
w = 2 * 3 \sigma + 1
$$

高斯核可以过滤高频信息，高斯卷积的结果是另一个高斯 

⇒ **连续多次高斯卷积，等价于一个大的高斯核卷积**

⇒ **高斯核可以分解，大的高斯核运行时间长，拆分成多个小的高斯核可以提高程序的运行效率**

⇒ **高斯核二维卷积可以拆分成两个一维卷积**

`n x n` 二维卷积（ `O(n^2)` ）可以拆分成为水平方向 `1 x n` 和 竖直方向 `n x 1` 的组合卷积（ `O(2n)` ）

## 图片噪声

- 椒盐噪声（Impulse Noise）
    - 黑点 + 白点 → 中值滤波器 Median
- 随机噪声（Gaussian Noise）
    - 噪声满足随机分布 → 高斯滤波器
- 均匀噪声（Uniform Noise）
    - 噪声服从均匀分布（每个值出现概率相同）→ 均值滤波器，高斯滤波器
- 泊松噪声（Poisson Noise）
    - 噪声强度和信号强度有关（亮处噪声大，暗处噪声小）→ 高斯滤波器，非局部均值滤波器
    - 非局部均值滤波器 → 不只看附近像素，而是去全图找长的像的块，然后平均