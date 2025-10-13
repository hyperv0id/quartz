---
aliases:
  - 证据下界
  - ELBO
---
## 联合分布 (Joint distribution)

$$
\text{联合分布 } p(x, z) \rightarrow \text{似然 } p(x)
$$

在生成模型中，我们会考虑观测数据 $x$ 和隐变量 $z$ 的联合分布 $p(x, z)$。基于似然的生成模型旨在学习一个能够最大化观测数据 $x$ 的似然函数 $p(x)$ 的模型。

## 似然 (Likelihood)

从联合分布 $p(x, z)$ 推导出观测数据的似然 $p(x)$ 有两种方法：

1.  对隐变量 $z$ 进行 **边缘化 (积分)**：
    $$
    p(x) = \int p(x, z)dz
    $$

2.  使用 **概率链式法则**:
    $$
    p(x) = \frac{p(x, z)}{p(z|x)}
    $$

## 挑战 (Challenges)

最大化似然函数 $p(x)$ 具有挑战性，因为它涉及到：

-   **对所有隐变量 $z$ 进行积分** (如方法1)，或者
-   需要得到 **真实的后验概率** $p(z|x)$ (如方法2)。

对于复杂的模型，这两种方法通常都是 **难以直接计算** 的。

## 最大对数似然的代理目标 (Proxy objective of maximum log-likelihood)

$$
\begin{align*}
\text{优化目标} \quad & \rightarrow \quad \text{代理目标} \\
\max \log p(x) \quad & \rightarrow \quad \max \text{ELBO}
\end{align*}
$$

利用前面的两个方程，我们推导出 **证据下界 (Evidence Lower Bound, ELBO)**，它是证据 (evidence) 的一个下界。证据被量化为观测数据的对数似然。对于优化隐变量模型而言，最大化 ELBO 可以作为最大化 **证据 (对数似然)** 的一个代理目标。

## ELBO 的方程 (Equation of ELBO)

$$
\mathbb{E}_{q_{\phi}(z|x)} \left[ \log \frac{p(x, z)}{q_{\phi}(z|x)} \right]
$$

-   $q_{\phi}(z|x)$ 是 **变分后验 (variational posterior)**，其参数 $\phi$ 需要通过优化来近似真实的后验分布 $p(z|x)$。
-   通过训练参数 $\phi$ 来最大化 ELBO (例如在[变分自编码器](Variational%20Autoencoders.md) VAE 中)，我们 **提升了这个下界 (increase the lower bound)**，并获得了能够对数据分布进行建模和采样的组件，从而实现了一个生成模型。

## 与证据 (Evidence) 的关系

$$
\log p(x) \geq \mathbb{E}_{q_{\phi}(z|x)} \left[ \log \frac{p(x, z)}{q_{\phi}(z|x)} \right]
$$

-   **证据 (Evidence)** 被量化为观测数据的对数似然 $\log p(x)$。
-   **ELBO (证据下界)** 则是这个证据的下界。

---
## ELBO 的推导
### 推导 1：基于琴生不等式

该推导利用对数函数的凹性以及琴生不等式。

$$
\begin{align*}
\log p(x) &= \log \int p(x, z) dz && \text{(应用公式 1: 边缘化)} \\
&= \log \int p(x, z) \frac{q_{\phi}(z|x)}{q_{\phi}(z|x)} dz && \text{(乘以 1，其中 } 1 = \frac{q_{\phi}(z|x)}{q_{\phi}(z|x)}) \\
&= \log \mathbb{E}_{q_{\phi}(z|x)} \left[ \frac{p(x, z)}{q_{\phi}(z|x)} \right] && \text{(期望的定义)} \\
&\geq \mathbb{E}_{q_{\phi}(z|x)} \left[ \log \frac{p(x, z)}{q_{\phi}(z|x)} \right] && \text{(应用琴生不等式)}
\end{align*}
$$

> **数学背景知识**
>
> -   **期望定义 (Expectation definition):** $\mathbb{E}_{f(x)}(g(x)) = \int g(x)f(x)dx$
> -   **琴生不等式 (Jensen's Inequality):** 对于一个凸函数 $f$，$f(\mathbb{E}(X)) \leq \mathbb{E}(f(X))$。反之，对于一个凹函数（如对数函数 $\log$），则有 $f(\mathbb{E}(X)) \geq \mathbb{E}(f(X))$。
>
> ![image-20251006150256780](http://assets.hypervoid.top/img/2025/10/06/image-20251006150256780-3ef8.png)

### 2：基于 KL 散度

$$
\begin{align*}
\log p(x) &= \log \int p(x,z) dz \\
&= \int q_{\phi}(z|x) (\log p(x)) dz && \text{(乘以 1，其中 } 1 = \int q_{\phi}(z|x)dz) \\
&= \mathbb{E}_{q_{\phi}(z|x)} [\log p(x)] && \text{(期望的定义)} \\
&= \mathbb{E}_{q_{\phi}(z|x)} \left[\log \frac{p(x, z)}{p(z|x)}\right] && \text{(应用公式 2: 概率链式法则)} \\
&= \mathbb{E}_{q_{\phi}(z|x)} \left[\log \frac{p(x, z)}{q_{\phi}(z|x)} \frac{q_{\phi}(z|x)}{p(z|x)}\right] && \text{(乘以 1)} \\
&= \mathbb{E}_{q_{\phi}(z|x)} \left[\log \frac{p(x, z)}{q_{\phi}(z|x)}\right] + \mathbb{E}_{q_{\phi}(z|x)} \left[\log \frac{q_{\phi}(z|x)}{p(z|x)}\right] && \text{(拆分对数项)} \\
&= \text{ELBO} - D_{KL}(q_{\phi}(z|x) \parallel p(z|x)) && \text{(KL 散度的定义)} \\
\end{align*}
$$
从上式可以整理出最终关系：
$$
\log p(x) = \text{ELBO} + D_{KL}(q_{\phi}(z|x) \parallel p(z|x))
$$
因为 KL 散度总是大于等于 0 ($D_{KL} \geq 0$)，所以 $\log p(x) \geq \text{ELBO}$。

> **相关概念**
>
> -   **KL 散度 (KL Divergence):**
>     $$ D_{KL}(P \parallel Q) = \int p(x) \log \frac{p(x)}{q(x)} dx = \mathbb{E}_{p(x)}\left(\log \frac{p(x)}{q(x)}\right) \geq 0$$
>     它衡量两个分布的差异，分布越接近，KL 散度值越小（理想情况下为 0）。
> -   **模型架构图:**
>     -   **编码器 (Encoder) $q_{\phi}(z|x)$:** 将观测数据 $x$ 映射到隐空间，生成隐变量 $z$ 的分布。它是在 **近似 (approximate)** 无法直接计算的真实后验 $p(z|x)$。
>     -   **解码器 (Decoder) $p(x|z)$:** 从隐空间采样一个 $z$，重构出数据 $x$。
>     -   ![image-20251006150346654](http://assets.hypervoid.top/img/2025/10/06/image-20251006150346654-1801.png)

---

## 核心概念总结

-   **KL 散度 (KL Divergence):** 在推导1中，琴生不等式导致了 ELBO 和证据之间的差距项被“隐藏”了。推导2明确了这个差距就是 KL 散度。理解这个非负项是掌握 ELBO 与证据之间关系的关键，也是理解为什么优化 ELBO 是一个合理目标的原因。

-   **隐式目标 (Implicit Objective):** 引入隐变量 $z$ 是为了捕捉观测数据背后的深层结构。虽然我们的目标是让变分后验 $q_{\phi}(z|x)$ 尽可能匹配真实后验 $p(z|x)$（即最小化它们的 KL 散度），但由于真实后验 $p(z|x)$ 未知，因此无法直接进行优化。

-   **联合优化 (Joint Optimization):** 对于给定的数据，证据 $\log p(x)$ 相对于变分参数 $\phi$ 是一个常数。因此，根据关系式 $\log p(x) = \text{ELBO} + D_{KL}$，**最大化 ELBO** 就等价于 **最小化 KL 散度** $D_{KL}(q_{\phi}(z|x) \parallel p(z|x))$。通过优化 ELBO，我们可以间接地让近似的变分后验 $q_{\phi}(z|x)$ 更接近真实的后验 $p(z|x)$。