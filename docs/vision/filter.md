# Filter
A filter is a weight matrix (kernel), and convolution is the operation that applies the filter to an image.

- Filter = Kernel
- Convolution = Operation → multiply the kernel weights with image pixels and sum them to obtain the output value.

Filters are used for:

- Image enhancement
- Smoothing
- Noise reduction
- Edge detection
- Feature extraction

## Properties of Filters

### Linearity

$$
filter(af_1 + bf_2) = afilter(f_1) + bfilter(f_2)
$$

### Shift Invariance

$$
filter(shift(f_1)) = shift(filter(f_1))
$$

## Boundary Handling

Convolution usually reduces the image size. Therefore, padding is commonly used to extend the image boundary so that the input and output sizes remain the same.

The purpose of padding is to **keep the input and output dimensions unchanged.**

Assume the original image is:

```
1 2 3
4 5 6
7 8 9
```

### Zero Padding

Fill the boundary with zeros. → Commonly used in CNNs.

```
0 0 0 0 0
0 1 2 3 0
0 4 5 6 0
0 7 8 9 0
0 0 0 0 0
```

### Replicate

Extend the outermost pixels outward.

```
1 1 2 3 3
1 1 2 3 3
4 4 5 6 6
7 7 8 9 9
7 7 8 9 9
```

### Reflect

Reflect the image across its boundaries. → One of the most commonly used modes in OpenCV.

```
5 4 5 6 5
2 1 2 3 2
5 4 5 6 5
8 7 8 9 8
5 4 5 6 5
```

### Circular

Treat the image as periodic, connecting the left edge to the right edge and the top edge to the bottom edge. → Commonly used in FFT-based processing.

```
9 7 8 9 7
3 1 2 3 1
6 4 5 6 4
9 7 8 9 7
3 1 2 3 1
```

## Common Kernels

### Identity
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

### Left Shift
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

### Right Shift
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

### Mean Filter
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

### Sharpening Filters

#### Sharpening A
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

Characteristics:

- Uses all eight neighboring pixels.
- Produces more isotropic sharpening.
- Has uniform responses in all directions.
- Better preserves high-frequency information.
- **More sensitive to noise !!!**

#### Sharpening B
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

Characteristics:

- Laplacian sharpening.
- Uses four-neighborhood pixels.
- Equivalent to: Original Image + Laplacian
- Moderate edge enhancement.
- Less sensitive to noise.
- Computationally simple.
- Commonly used in OpenCV.

#### Sharpening C
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

Characteristics:

- Strong sharpening effect.
- Enhances edges significantly.
- May produce halo artifacts.
- Amplifies noise.

#### Summary
<table border="1">
    <tr>
        <th>Kernel</th>
        <th>Neighborhood</th>
        <th>Sharpening Strength</th>
        <th>Noise Amplification</th>
        <th>Characteristics</th>
    </tr>
    <tr>	 
        <td>Sharpening A</td>
        <td>Four-neighborhood</td>
        <td>Moderate</td>
        <td>Low	Most</td>
        <td>commonly used</td>
    </tr>
    <tr>
        <td>Sharpening B</td>
        <td>Eight-neighborhood</td>
        <td>Smooth and natural</td>
        <td>Medium</td>
        <td>Unsharp Mask</td>
    </tr>
    <tr>
        <td>Sharpening C</td>
        <td>Eight-neighborhood</td>
        <td>Strong</td>
        <td>High</td>
        <td>May cause over-sharpening</td>
    </tr>
</table>

## Gaussian Filter

The center of a Gaussian kernel has larger weights, while the weights decrease toward the edges.

Normalizing the Gaussian kernel prevents the image intensity from being attenuated.

Effect of Standard Deviation
- Smaller $\sigma$
    - Larger center weight
    - Weaker smoothing
    - Less blur
- Larger $\sigma$
    - More evenly distributed weights
    - Stronger smoothing
    - More blur

Effect of Window Size
- Larger window size
    - More elements in normalization
    - Stronger smoothing
- Smaller window size
    - Fewer elements in normalization
    - Weaker smoothing

Rule of Thumb:

$$
w = 2 * 3 \sigma + 1
$$

Gaussian filters remove high-frequency information.

The convolution of two Gaussian functions is still a Gaussian.
Therefore:

- Multiple Gaussian convolutions are equivalent to a single larger Gaussian convolution.
- Large Gaussian kernels can be decomposed into several smaller Gaussian kernels to improve computational efficiency.
- A 2D Gaussian convolution can be separated into two 1D convolutions.

An `n x n` 2D convolution with complexity (`O(n^2)`) can be decomposed into: a horizontal convolution `1 x n` and a vertical convolution `n x 1` with complexity (`O(2n)`)

## Image Noise

- Salt-and-Pepper Noise (Impulse Noise)
    - Random black and white pixels.
    - Removed effectively by the Median Filter.
- Gaussian Noise
    - Random noise following a Gaussian distribution.
    - Commonly reduced using the Gaussian Filter.
- Uniform Noise
    - Noise values are uniformly distributed.
    - Can be reduced by:
        - Mean Filter
        - Gaussian Filter
- Poisson Noise
    - Signal-dependent noise.
    - Brighter regions contain stronger noise, while darker regions contain weaker noise.
    - Common methods:
        - Gaussian Filter
        - Non-Local Means (NLM)

---
**Non-Local Means Filter**

Instead of considering only neighboring pixels, the Non-Local Means filter searches for similar patches across the entire image and averages them according to their similarity.