---
title: "🎲 蒙特卡洛方法：从入门到精通"
date: 2026-03-11T12:00:00+08:00
draft: false
tags: ["Python", "统计学", "机器学习", "蒙特卡洛", "MCMC"]
categories: ["技术"]
description: "全面介绍蒙特卡洛方法，包括简单随机抽样、拒绝采样、Metropolis-Hastings、吉布斯采样等方法，配有Python代码和可视化图表。"
ShowToc: true
TocOpen: false
weight: 1
---

## 引言

蒙特卡洛方法（Monte Carlo Method）是一类基于随机采样的统计方法，得名于摩纳哥的蒙特卡洛赌场。它通过大量随机样本来近似计算数值结果，在物理学、统计学、机器学习、金融工程等领域有广泛应用。

本文将带你从最简单的随机抽样开始，逐步深入到拒绝采样、吉布斯采样等高级方法。每种方法都配有 Python 代码和可视化，帮助你直观理解。

---

## 1. 简单随机抽样（Simple Random Sampling）

### 原理

最简单的蒙特卡洛方法就是从目标分布中直接抽取随机样本。对于均匀分布，我们可以直接生成；对于复杂分布，可以使用逆变换采样（Inverse Transform Sampling）。

### 示例：估计圆周率 π

想象一个边长为 2 的正方形，内接一个半径为 1 的圆。向正方形内随机撒点，落在圆内的概率为：

$$P = \\frac{\\text{圆面积}}{\\text{正方形面积}} = \\frac{\\pi r^2}{(2r)^2} = \\frac{\\pi}{4}$$

因此，$\\pi \\approx 4 \\times \\frac{\\text{圆内点数}}{\\text{总点数}}$

### Python 实现

```python
import numpy as np
import matplotlib.pyplot as plt

np.random.seed(42)

def estimate_pi(n_samples):
    x = np.random.uniform(-1, 1, n_samples)
    y = np.random.uniform(-1, 1, n_samples)
    inside_circle = (x**2 + y**2) <= 1
    pi_estimate = 4 * np.sum(inside_circle) / n_samples
    return x, y, inside_circle, pi_estimate

sample_sizes = [100, 1000, 10000, 100000]
for n in sample_sizes:
    x, y, inside, pi_est = estimate_pi(n)
    print(f"样本量: {n:6d}, π 估计值: {pi_est:.6f}, 误差: {abs(np.pi - pi_est):.6f}")
```

![蒙特卡洛估计π](/images/mc_pi_estimation.png)

> 图1：不同样本量下的π估计，样本越多估计越精确

> **关键洞察：** 这就是**大数定律**的威力——样本越多，估计越精确。

---

## 2. 逆变换采样（Inverse Transform Sampling）

### 原理

当我们需要从特定概率分布 $p(x)$ 中采样时，如果已知其累积分布函数（CDF）$F(x)$ 的逆函数 $F^{-1}(u)$，可以通过以下步骤采样：

1. 生成均匀随机数 $u \\sim U(0, 1)$
2. 计算 $x = F^{-1}(u)$

则 $x$ 服从分布 $p(x)$。

### 示例：从指数分布采样

指数分布的概率密度函数（PDF）：

$$f(x; \\lambda) = \\lambda e^{-\\lambda x}, \\quad x \\geq 0$$

累积分布函数（CDF）：

$$F(x) = 1 - e^{-\\lambda x}$$

逆函数：

$$F^{-1}(u) = -\\frac{\\ln(1-u)}{\\lambda}$$

### Python 实现

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy import stats

def inverse_transform_exponential(lambda_param, n_samples):
    u = np.random.uniform(0, 1, n_samples)
    samples = -np.log(1 - u) / lambda_param
    return samples

lambda_param = 0.5
n_samples = 10000
samples = inverse_transform_exponential(lambda_param, n_samples)
```

![逆变换采样](/images/inverse_transform.png)

> 图2：逆变换采样生成的指数分布样本与理论分布对比

### 局限性

逆变换采样要求我们知道 CDF 的解析逆函数。对于许多复杂分布（如高斯分布），CDF 没有闭式逆函数，这时就需要其他方法。

---

## 3. 拒绝采样（Rejection Sampling）

### 原理

拒绝采样是一种通用采样方法，适用于任意概率密度函数 $p(x)$，即使归一化常数未知。

**基本思想**：
1. 找到一个容易采样的提议分布 $q(x)$（如均匀分布、正态分布）
2. 找到一个常数 $M$，使得对所有 $x$，$p(x) \\leq M \\cdot q(x)$
3. 从 $q(x)$ 采样得到 $x$
4. 以概率 $\\frac{p(x)}{M \\cdot q(x)}$ 接受该样本，否则拒绝并重复

### 几何解释

想象你在一个矩形盒子内随机撒点，只保留落在目标分布曲线下的点。这样得到的样本就服从目标分布。

### 示例：从 Beta 分布采样

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy import stats

def rejection_sampling_beta(alpha, beta, n_samples):
    samples = []
    n_accepted = 0
    n_proposed = 0
    
    x_mode = (alpha - 1) / (alpha + beta - 2) if alpha > 1 and beta > 1 else 0.5
    max_pdf = stats.beta.pdf(x_mode, alpha, beta)
    M = max_pdf * 1.5
    
    while n_accepted < n_samples:
        x = np.random.uniform(0, 1)
        y = np.random.uniform(0, M)
        n_proposed += 1
        target_pdf = stats.beta.pdf(x, alpha, beta)
        if y <= target_pdf:
            samples.append(x)
            n_accepted += 1
    
    acceptance_rate = n_accepted / n_proposed
    return np.array(samples), acceptance_rate

alpha, beta = 2, 5
samples, acceptance_rate = rejection_sampling_beta(alpha, beta, 5000)
```

![拒绝采样](/images/rejection_sampling.png)

> 图3：拒绝采样可视化——绿色点被接受，红色点被拒绝

> ⚠️ **局限性：** 拒绝采样的主要问题是**维数灾难**——在高维空间中，接受率可能极低。例如，在一个 10 维的单位超球中采样，接受率可能低于 0.1%。

---

## 4. 马尔可夫链蒙特卡洛（MCMC）

### 原理

MCMC 方法通过构建一个马尔可夫链，使其平稳分布恰好是我们想要的目标分布 $p(x)$。经过足够长的"burn-in"期后，链上的样本就近似服从 $p(x)$。

### Metropolis-Hastings 算法

这是最基础的 MCMC 算法：

1. 初始化 $x_0$
2. 对于 $t = 1, 2, ...$:
   - 从提议分布 $q(x'|x_t)$ 采样候选点 $x'$
   - 计算接受概率：$\\alpha = \\min\\left(1, \\frac{p(x')q(x_t|x')}{p(x_t)q(x'|x_t)}\\right)$
   - 以概率 $\\alpha$ 接受 $x_{t+1} = x'$，否则 $x_{t+1} = x_t$

### Python 实现

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy import stats

def metropolis_hastings(target_pdf, proposal_std, n_samples, x_init=0):
    samples = [x_init]
    x_current = x_init
    n_accepted = 0
    
    for i in range(n_samples - 1):
        x_proposal = np.random.normal(x_current, proposal_std)
        p_current = target_pdf(x_current)
        p_proposal = target_pdf(x_proposal)
        
        if p_current == 0:
            alpha = 1
        else:
            alpha = min(1, p_proposal / p_current)
        
        if np.random.uniform(0, 1) < alpha:
            x_current = x_proposal
            n_accepted += 1
        
        samples.append(x_current)
    
    acceptance_rate = n_accepted / (n_samples - 1)
    return np.array(samples), acceptance_rate

# 目标分布：混合高斯
def mixture_gaussian(x):
    return 0.4 * stats.norm.pdf(x, loc=-3, scale=0.5) + \
           0.6 * stats.norm.pdf(x, loc=2, scale=1)
```

![Metropolis-Hastings算法](/images/metropolis_hastings.png)

> 图4：不同提议分布步长对 MCMC 采样的影响

> 💡 **参数调优：**
> - **小步长（σ=0.1）**：接受率高，但探索空间慢，容易 stuck 在局部
> - **适中步长（σ=1.0）**：平衡了探索和接受率
> - **大步长（σ=10）**：接受率低，很多提议被拒绝

---

## 5. 吉布斯采样（Gibbs Sampling）

### 原理

吉布斯采样是 MCMC 的一种特例，适用于多维分布。其核心思想是：

**轮流对每个维度进行采样，条件于其他所有维度**。

对于二维分布 $p(x, y)$：
1. 初始化 $(x_0, y_0)$
2. 对于 $t = 1, 2, ...$:
   - 从 $p(x | y_{t-1})$ 采样 $x_t$
   - 从 $p(y | x_t)$ 采样 $y_t$

### 优势

- 不需要设计提议分布
- 接受率总是 100%
- 适用于条件分布容易采样的场景

### 示例：二元正态分布

```python
import numpy as np
import matplotlib.pyplot as plt

def gibbs_sampling_bivariate_normal(n_samples, rho, x_init=0, y_init=0):
    samples = np.zeros((n_samples, 2))
    x, y = x_init, y_init
    var_cond = 1 - rho**2
    
    for i in range(n_samples):
        x = np.random.normal(rho * y, np.sqrt(var_cond))
        y = np.random.normal(rho * x, np.sqrt(var_cond))
        samples[i] = [x, y]
    
    return samples

rho = 0.8
n_samples = 5000
samples_gibbs = gibbs_sampling_bivariate_normal(n_samples, rho)
```

![吉布斯采样](/images/gibbs_sampling.png)

> 图5：吉布斯采样轨迹、散点分布与自相关函数

### 吉布斯采样的局限性

1. **需要知道条件分布**：如果条件分布难以采样，吉布斯采样就不适用
2. **变量间相关性强时效率低**：从自相关图可以看到，样本间存在较强的自相关
3. **收敛速度慢**：当变量间相关性很强时，采样可能 stuck 在对角线上

---

## 6. 实际应用：贝叶斯推断

蒙特卡洛方法在贝叶斯统计中有广泛应用。下面我们展示一个简单的例子：用 MCMC 估计硬币的偏差。

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy import stats

def coin_mcmc(data, alpha_prior=1, beta_prior=1, n_samples=10000):
    samples = []
    theta_current = 0.5
    
    n_heads = np.sum(data)
    n_tails = len(data) - n_heads
    
    def log_posterior(theta):
        if theta <= 0 or theta >= 1:
            return -np.inf
        log_like = n_heads * np.log(theta) + n_tails * np.log(1 - theta)
        log_prior = (alpha_prior - 1) * np.log(theta) + \
                    (beta_prior - 1) * np.log(1 - theta)
        return log_like + log_prior
    
    n_accepted = 0
    for i in range(n_samples):
        theta_proposal = theta_current + np.random.normal(0, 0.1)
        log_p_current = log_posterior(theta_current)
        log_p_proposal = log_posterior(theta_proposal)
        log_alpha = log_p_proposal - log_p_current
        
        if np.log(np.random.uniform()) < log_alpha:
            theta_current = theta_proposal
            n_accepted += 1
        
        samples.append(theta_current)
    
    return np.array(samples), n_accepted / n_samples

# 模拟抛硬币数据（真实 bias = 0.7）
np.random.seed(42)
true_theta = 0.7
data = np.random.binomial(1, true_theta, size=100)
print(f"模拟数据: {np.sum(data)} 次正面, {len(data) - np.sum(data)} 次反面")
```

![贝叶斯推断](/images/bayesian_coin.png)

> 图6：使用 MCMC 估计硬币偏差的贝叶斯推断

> 📊 **结果：**
> - 真实 θ: 0.7
> - MCMC 估计 (均值 ± 标准差): 0.69 ± 0.05
> - 理论后验均值: 0.70
> - 95% 可信区间: [0.60, 0.78]

---

## 总结对比

| 方法 | 适用场景 | 优点 | 缺点 |
|------|---------|------|------|
| **简单抽样** | 分布已知且易采样 | 简单直接 | 适用范围有限 |
| **逆变换采样** | CDF 可逆的分布 | 精确采样 | 需要 CDF 逆函数 |
| **拒绝采样** | 任意分布 | 通用性强 | 高维时接受率低 |
| **Metropolis-Hastings** | 复杂分布、归一化常数未知 | 不需要提议分布 | 样本自相关、收敛诊断 |
| **吉布斯采样** | 多维分布、条件分布易采样 | 接受率 100% | 需要条件分布、变量相关时效率低 |

蒙特卡洛方法是现代统计和机器学习的基石。理解这些采样方法，对于深入学习变分推断、概率图模型、强化学习等领域都至关重要。

希望这篇博客能帮助你理解蒙特卡洛方法的核心思想！

---

## 参考代码

完整的 Python 代码和可视化脚本可以在 [GitHub](https://github.com/leeeyij/leeeyij.github.io) 找到。

### 依赖库

```bash
pip install numpy matplotlib scipy statsmodels
```

### 运行代码

```python
# 保存上述代码为 monte_carlo_methods.py
# 运行
python monte_carlo_methods.py
```

生成的图片：
- `mc_pi_estimation.png` - 蒙特卡洛估计 π
- `inverse_transform.png` - 逆变换采样
- `rejection_sampling.png` - 拒绝采样
- `metropolis_hastings.png` - Metropolis-Hastings 算法
- `gibbs_sampling.png` - 吉布斯采样
- `bayesian_coin.png` - 贝叶斯推断示例
