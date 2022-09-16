---
title: Statistical Ineuqalities 모음집
use_math: true
comments: true
---

# 1. Cauch-Schwarts Inequality
$$\left( \sum_{k=1}^n a_k b_k \right)^2 \leq \left( \sum_{k=1}^n a_k^2 \right) \left( \sum_{k=1}^n b_k^2 \right)$$

In case of random variable,   
$$\left| \mathbb{E}\left[\left|X\right|\right]\left|Y\right| \right| \leq \sqrt{\mathbb{E}\left [ X^{2} \right] \mathbb{E}\left [X^{2} \right]}$$

# 2. Markov Inequality
$$\Pr\left [ \left| X\right| \geq a \right ] \leq \frac{\mathbb{E}\left [ \left|X \right| \right ]}{a}$$

## 2-2. Chebyshev's Inequality
$$\Pr\left [ \left| X - \mathbb{E}\left ( \left|X \right| \right )\right| \geq a \right ] \leq \frac{\mathrm{Var}\left ( X  \right )}{a^{2}}$$

# 3. Jensen's Inequality
If $\varphi $ is a convex function,
$$\varphi \left( \mathbb{E}\left[ X\right]\right) \leq \mathbb{E} \left[\varphi\left( X\right) \right]$$
Vice versa in case of concave function.

