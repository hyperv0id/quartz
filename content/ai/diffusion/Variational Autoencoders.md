---
aliases:
  - VAE
  - 变分自编码器
---
推导过程：
1. **流形假设**：高维数据`x`实际上分布在一个低维流形上，由少数潜在因素$z$生成。
2. 我们构建一个**生成模型**，通过从一个简单的先验分布$p(z)$中采样，再通过解码器$p_θ(x|z)$生成$x$
3. $log p(x)$ 涉及到对$z$的积分，**难以直接计算 (intractable)**。
4. 最大化 $\log p(x)$的**[证据下界](Evidence%20Lower%20Bound.md)(ELBO)**，代理计算：$\mathbb{E}_{q_{\phi}(z|x)} \left[ \log \frac{p(x, z)}{q_{\phi}(z|x)} \right]$
5. 编码器$q_φ(z|x)$为给定的$x$提供一个$z$的**概率分布**。$p(x,z)$ 未知，解码器就是$p_θ(x|z)$，它只负责从$z$生成$x$

## 定义

给AE编码的潜空间离散点换成高斯分布，使用变分推断训练模型，使其具备生成能力

*   **变分 (Variational)**: 在由参数 $\phi$ 参数化的一系列后验分布中，优化寻找最佳的 $q_{\phi}(z|x)$。
*   **自编码器 (Autoencoder)**: 它类似于传统的自编码器，其中输入数据在经过中间的瓶颈表示 $z$ 后，被训练来预测自身。

## 联合优化参数 $\phi$ 和 $\theta$ 的 ELBO

$$
\begin{aligned}
\mathbb{E}_{q_{\phi}(z|x)}\left[\log \frac{p(x, z)}{q_{\phi}(z|x)}\right] &= \mathbb{E}_{q_{\phi}(z|x)}\left[\log \frac{p_{\theta}(x|z)p(z)}{q_{\phi}(z|x)}\right] & \text{(概率链式法则 Chain Rule of Probability)} \\
&= \mathbb{E}_{q_{\phi}(z|x)}[\log p_{\theta}(x|z)] + \mathbb{E}_{q_{\phi}(z|x)}\left[\log\frac{p(z)}{q_{\phi}(z|x)}\right] & \text{(拆分期望 Split the Expectation)} \\
&= \underbrace{\mathbb{E}_{q_{\phi}(z|x)}[\log p_{\theta}(x|z)]}_{\text{reconstruction term}} - \underbrace{D_{\text{KL}}(q_{\phi}(z|x) \| p(z))}_{\text{prior matching term}} & \text{(KL散度定义 Definition of KL Divergence)}
\end{aligned}
$$

*   **重构项 (Reconstruction term)**: 它衡量了从变分分布中，通过**解码器 $\theta$** 进行重构的可能性，确保学习到的潜在变量能有效地重新生成原始数据。
*   **先验匹配项 (Prior matching term)**: 它衡量了学习到的**编码器 $\phi$** 的变分分布与关于潜在变量的先验信念的相似程度。

## VAEs 的高斯建模 (Gaussian modeling of VAEs)
$$
q_{\phi}(z|x) = \mathcal{N}(z; \mu_{\phi}(x), \sigma^2_{\phi}(x)\mathbf{I})
$$
$$
p(z) = \mathcal{N}(z; \mathbf{0}, \mathbf{I})
$$

VAE 的编码器 $q_{\phi}(z|x)$ 通常被建模为具有对角协方差矩阵的多元高斯分布，而先验 $p(z)$ 通常是标准多元高斯分布。

## ELBO 的蒙特卡洛估计 (Monte Carlo Estimate of ELBO)
$$
\arg\max_{\phi,\theta} \mathbb{E}_{q_{\phi}(z|x)}[\log p_{\theta}(x|z)] - D_{\text{KL}}(q_{\phi}(z|x) \| p(z)) \approx \arg\max_{\phi,\theta} \sum_{l=1}^{L} \log p_{\theta}(x|z^{(l)}) - D_{\text{KL}}(q_{\phi}(z|x) \| p(z))
$$

虽然 KL 散度可以解析计算，但重构项中的期望是使用蒙特卡洛估计来近似的。这种近似依赖于对每个观测数据点 $x$，从 $q_{\phi}(z|x)$ 中随机采样潜在变量 $\{z^{(l)}\}_{l=1}^{L}$；这个过程通常是**不可微**的，因此给优化带来了挑战。
## 重参数化技巧 (Reparameterization trick)
$$
x = \mu + \sigma\epsilon \quad \text{with} \quad \epsilon \sim \mathcal{N}(\epsilon; \mathbf{0}, \mathbf{I})
$$
通过重参数化技巧，可以从标准高斯分布中采样，然后通过均值 $\mu$ 进行平移，并通过方差 $\sigma^2$ 进行缩放，从而实现从任意高斯分布中采样。重参数化技巧将一个随机变量重写为一个噪声变量的确定性函数，从而能够对非随机项进行梯度下降。

## ELBO 中的重参数化 (Reparameterzation in ELBO)
$$
z = \mu_{\phi}(x) + \sigma_{\phi}(x) \odot \epsilon \quad \text{with} \quad \epsilon \sim \mathcal{N}(\epsilon; \mathbf{0}, \mathbf{I})
$$
在 VAE 中，每个 $z$ 都被计算为输入 $x$ 和辅助噪声变量 $\epsilon$ 的确定性函数。与从分布中采样 $z$ 的不可微性相比，在对 $z$ 进行重参数化后，可以计算关于 $\phi$ 的梯度来优化 $\mu_{\phi}$ 和 $\sigma_{\phi}$。

## 数据生成 (Data generation)

![image-20251006162703656](http://assets.hypervoid.top/img/2025/10/06/image-20251006162703656-4291.png)

在训练 VAE 之后，可以通过直接从潜在空间 $p(z)$ 采样，然后将其通过解码器来生成新数据。

当潜在变量 $z$ 的维度小于输入 $x$ 的维度时，变分自编码器很有意义，因为这可能会产生紧凑且有意义的表示。**当学习到具有语义意义的潜在空间后**，潜在向量在传递给解码器之前可以被修改，以精确控制生成的数据。

## 变体
- [Hierarchical Variational Autoencoders](Hierarchical%20Variational%20Autoencoders)
- [Markovian HVAE](Markovian%20HVAE)
- [Variational Diffusion Models](Variational%20Diffusion%20Models)