# Fitting

After edge extraction, we need to fit geometric models (lines, circles, curves, etc.) to the edge points, i.e., use mathematical models to describe edges.

## Difficulties:

- Noise
- Outliers
- Missing data

## Common Methods

<table border="1">
    <tr>
        <th>Scenario</th>
        <th>Method</th>
    </tr>
    <tr>	 
        <td>Most points lie approximately on a single line</td>
        <td>Least Squares</td>
    </tr>
    <tr>
        <td>Many outliers / noisy points</td>
        <td>Robust Fitting、RANSAC</td>
    </tr>
    <tr>
        <td>Multiple lines exist in the image</td>
        <td>RANSAC、Hough Transform</td>
    </tr>
</table>


## Least Squares Line Fitting

### Core Idea

Minimize:

$$
\sum (y_i - m*x_i -b )^2
$$

### Limitations:

- Cannot represent vertical lines
- Not rotation invariant

## Total Least Squares (Orthogonal Least Squares)

### Core Idea

Minimize:

$$
\sum_i{(\frac{ax_i+by_i+d}{\sqrt{a^2 + b^2}})^2}
$$

Constraint:

$$
a^2 + b^2 = 1
$$

Equivalent form:

$$
\sum{(ax_i + by_i +d)^2}
$$

### Properties:

- Uses orthogonal distance
- Rotation invariant
- Works for arbitrary orientations

### Limitation:

- Sensitive to outliers

## Robust Estimators

### Core Idea

- Small residual → high weight
- Large residual → low weight

### Common M-estimators:

- Huber
- Tukey
- Geman-McClure
- Cauchy

Example Geman-McClure：

$$
\rho = \frac{u^2}{\sigma ^ 2 + u^2}
$$

- $u$ → residual
- $\sigma$ → controls outlier influence


## RANSAC (Random Sample Consensus)

### Procedure

- Randomly sample minimal set (2 points for line)
- Fit model
- Compute distances to all points
- Points below threshold are inliers
- Count inliers
- Repeat
- Choose best model

### Parameters:

- Sample size
- Threshold
- Iterations

## Adaptive RANSAC

### Core Idea
> Dynamically estimate outlier ratio and update iterations

```
N = ∞
sample_count = 0

while sample_count < N:

    RANSAC()

    estimate outlier ratio e

    compute N*

    if N* < N:
        N = N*

    sample_count += 1
```
- outlier ratio: $e$
- inlier ratio: $w = 1 - e$
 
If each iteration randomly selects $s$ points and we want the success probability to reach $p$, then the required number of iterations is:

$$
N = \frac{log(1-p)}{log(1-w^s)}
$$

### Key Idea:
> As more inliers are found, the required number of iterations decreases, avoiding unnecessary computation from a fixed iteration count.

## Overfitting vs Underfitting

### Underfitting
Model is too simple:
- cannot represent the data well
- Large error

### Overfitting
Model is too complex:
- Fits noise as well
- Poor generalization

Goal:
> Balance model complexity and fitting error

## Parametric vs Non-parametric Fitting

### Parametric Fitting

- Known model form
- Examples: Line, Circle, Ellipse, Polynomial
- Features:
    - Few parameters
    - Efficient

### Non-parametric Fitting

- Unknown model form
- Examples: Spline, Kernel Regression
- Features:
    - Flexible
    - Expensive