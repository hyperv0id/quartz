---
aliases:
  - MHVAE
---



![image-20251007121832229](http://assets.hypervoid.top/img/2025/10/07/image-20251007121832229-c0e1.png)

## 1. 基本概念

在具有 $T$ 个层级的一般 HVAE (Hierarchical VAE) 中，每个隐变量 $z_t$ 的生成都依赖于**所有更上层**的隐变量 $\{z_{t+1}, ..., z_T\}$。

然而，在 **马尔可夫 HVAE (MHVAE)** 中，模型引入了马尔可夫假设，简化了依赖关系：每个隐变量 $z_t$ 只依赖于其**紧邻的上一层**隐变量 $z_{t+1}$。

## 2. 模型结构

MHVAE 的结构可以看作一个双向的马尔可夫链：

*   **生成过程 (Decoder / $p_\theta$)**: 这是一个自顶向下的过程。
    1.  从最高层隐变量的先验分布 $p(z_T)$ 中采样得到 $z_T$。
    2.  然后逐层向下生成，每一层 $z_{t-1}$ 的生成只依赖于其上一层 $z_t$，即 $p_\theta(z_{t-1}|z_t)$。
    3.  最后，根据最底层隐变量 $z_1$ 生成观测数据 $x$，即 $p_\theta(x|z_1)$。
    4.  整个过程可以表示为：$z_T \rightarrow z_{T-1} \rightarrow \dots \rightarrow z_1 \rightarrow x$。

*   **推断过程 (Encoder / $q_\phi$)**: 这是一个自底向上的过程。
    1.  从观测数据 $x$ 出发，推断最底层的隐变量 $z_1$，即 $q_\phi(z_1|x)$。
    2.  然后逐层向上推断，每一层 $z_t$ 的推断只依赖于其下一层 $z_{t-1}$，即 $q_\phi(z_t|z_{t-1})$。
    3.  整个过程可以表示为：$x \rightarrow z_1 \rightarrow z_2 \rightarrow \dots \rightarrow z_T$。

## 3. 核心公式

### 3.1 联合分布与近似后验

根据上述马尔可夫假设，我们可以定义模型的联合分布和近似后验分布。

*   **联合分布 (Joint Distribution - 生成模型)**:
    $$
    p(x, z_{1:T}) = p(z_T) p_\theta(x|z_1) \prod_{t=2}^{T} p_\theta(z_{t-1}|z_t)
    $$

*   **近似后验 (Approximate Posterior - 推断模型)**:
    $$
    q_\phi(z_{1:T}|x) = q_\phi(z_1|x) \prod_{t=2}^{T} q_\phi(z_t|z_{t-1})
    $$

### 3.2 [ELBO](Evidence%20Lower%20Bound.md) (证据下界) 推导

与标准 VAE 类似，我们通过最大化证据下界 (Evidence Lower Bound, ELBO) 来训练模型。

1.  **目标**: 对数边际似然 $\log p(x)$。
    $$
    \log p(x) = \log \int p(x, z_{1:T}) dz_{1:T}
    $$
2.  **引入近似后验 $q_\phi$**:
    $$
    = \log \int p(x, z_{1:T}) \frac{q_\phi(z_{1:T}|x)}{q_\phi(z_{1:T}|x)} dz_{1:T}
    $$
3.  **改写为期望形式**:
    $$
    = \log \mathbb{E}_{q_\phi(z_{1:T}|x)} \left[ \frac{p(x, z_{1:T})}{q_\phi(z_{1:T}|x)} \right]
    $$
4.  **应用琴生不等式 (Jensen's Inequality)**:
    $$
    \ge \mathbb{E}_{q_\phi(z_{1:T}|x)} \left[ \log \frac{p(x, z_{1:T})}{q_\phi(z_{1:T}|x)} \right]
    $$
    这个不等式的右侧就是 ELBO。

### 3.3 将分布代入 ELBO

最后，我们将 3.1 节中定义的联合分布和近似后验的具体形式代入 ELBO 表达式中，得到最终需要优化的目标函数。

$$
\text{ELBO} = \mathbb{E}_{q_\phi(z_{1:T}|x)} \left[ \log \frac{p(x, z_{1:T})}{q_\phi(z_{1:T}|x)} \right]
$$

将 $p(x, z_{1:T})$ 和 $q_\phi(z_{1:T}|x)$ 代入：

$$
= \mathbb{E}_{q_\phi(z_{1:T}|x)} \left[ \log \frac{p(z_T)p_\theta(x|z_1)\prod_{t=2}^{T} p_\theta(z_{t-1}|z_t)}{q_\phi(z_1|x)\prod_{t=2}^{T} q_\phi(z_t|z_{t-1})} \right]
$$

通过对数运算法则展开后，这个表达式可以被分解为 **重构项** 和一系列 **KL散度项**，分别对应于生成数据 $x$ 的能力和各层级上后验分布与先验分布的匹配程度。模型的训练目标就是最大化这个 ELBO。
