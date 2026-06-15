# PhyST-Net 组会汇报：物理感知的可微时空预测网络 (附核心代码实现解析)

**目标受众**：课题组导师与同行
**汇报时长**：20分钟
**核心策略**：重理论、清逻辑、展代码、上干货。将 `Problem Definition` 的严谨推导与 `develop` 分支中的 PyTorch 落地代码无缝结合，展示从理论到落地的完整闭环。

---

## 📊 PPT 页面设计 (15页大纲)

### 第一部分：背景与动机 (Background & Motivation)

*   **Slide 1: 标题页**
    *   题目：PhyST-Net: Physics-aware Differentiable Spatio-Temporal Transformer Network
    *   汇报人 / 日期
*   **Slide 2: 汇报提纲**
    *   1. 预测的“三难困境” (The Trilemma)
    *   2. 严格的问题定义 (Mathematical Formulation)
    *   3. 核心方法论与代码落地 (Methodology & Code)
    *   4. 实验进展与下一步计划
*   **Slide 3: 研究背景**
    *   时空网络预测的核心挑战：节点非孤立、动态拓扑、以及**强依赖未来外生变量 (Future ExG)**。
    *   应用场景：风电场功率预测、城市交通流、智能电网。
*   **Slide 4: 现存挑战 —— 预测的“三难困境” (The Forecasting Trilemma)**
    *   **Capacity vs. Noise**: Transformer 硬切片(Hard Patching) 导致物理连续事件的边界伪影。
    *   **Complexity vs. Multi-scale**: 单一图卷积操作混淆了“全局宏观趋势”和“局部微观扰动”。
    *   **Consistency vs. Data-driven**: 纯数据驱动模型无视物理定律（如输出负发电量或超越物理容量），且事后硬截断 (Hard Clip) 破坏梯度。

### 第二部分：问题定义 (Problem Formulation)

*   **Slide 5: 数据表示与防泄漏假设**
    *   公式化图结构 $G=(V, E, A)$ 与 元数据 $M_G$ 的解耦。
    *   明确区分 **Historical ExG ($\mathbf{E}^h$)**（用于推测潜状态）和 **Future ExG ($\mathbf{E}^f$)**（作为未来演化的条件驱动）。
    *   **核心干货**：提出**严格防泄漏假设**，仅允许推理时客观可知的 NWP（数值天气预报）作为 $\mathbf{E}^f$，解决现实部署中的 Leakage 问题。
*   **Slide 6: 数学建模与通用目标**
    *   统一输入元组：$\mathcal{X}_t \triangleq \big(\mathbf{X}, \mathbf{E}^{h}, \mathbf{E}^{f}, A, M_G\big)$。
    *   模型解耦：预测函数 $\hat{\mathbf{Y}} = \mathcal{C}_{\Omega}\big(\mathcal{H}_{\theta}(\mathcal{X}_t)\big)$，将无约束主干 $\mathcal{H}_\theta$ 与物理约束算子 $\mathcal{C}_\Omega$ 分离。
    *   通用目标：以 MAE 作为基础优化目标，保障对异常值的鲁棒性。

### 第三部分：核心方法论与代码落地 (Methodology & Implementation)

*   **Slide 7: PhyST-Net 架构总览**
    *   展示由 AGSP $\rightarrow$ 时空编码 $\rightarrow$ R-STMF $\rightarrow$ K-AMC $\rightarrow$ PI-DCH 构成的四阶流水线。
*   **Slide 8: 创新一：AGSP (自适应高斯软窗口 Patching)**
    *   **理论**：替代硬切片，引入连续的高斯密度函数，充当自适应低通滤波器。
    *   **代码干货 (`_embed_dcp`)**：
        *   学习参数：中心点 `mu` 和 带宽 `log_sigma`。
        *   数值稳定性处理：`W = torch.exp(-dist_sq / (2 * (sigma**2) + 1e-4))`，并利用 `W / (W.sum + 1e-8)` 实现“单位分解 (Partition of Unity)”。
        *   软切片实现：将加权后的时间序列直接进行线性映射 `self.value_projection(x_weighted)`。
*   **Slide 9: 创新二：R-STMF (关系型时空流形融合)**
    *   **理论**：解耦系统性宏观影响与局部邻居影响。
    *   **代码干货 (`PSTG_TimeMixerFusion`)**：
        *   利用 1D Conv 将时序分解为 `seasonal` (宏观周期) 和 `trend` (局部趋势) 两大流形。
        *   **安全距离掩码与剪枝**：先施加物理距离掩码 `dist_mask` 将非邻居置为 `-1e9`，再执行 `torch.topk(attn, k_val, dim=-1)`，确保图路由的物理合理性和稀疏性。
        *   门控融合：使用可学习的 Sigmoid `fusion_gate` 将双分支重组。
*   **Slide 10: 创新三：K-AMC (基于 KAN 的外生变量调制)**
    *   **理论**：以高表达能力引入未来外生变量。
    *   **代码干货 (`KFiLM` 模块)**：
        *   将 NWP 变量通过 `AdaptiveAvgPool1d` 降采样对齐到 Patch 维度。
        *   引入 `KAN` (Kolmogorov-Arnold Network)，设置 `grid_size=5, spline_order=3`，预测生成 $\gamma$ 和 $\beta$ 参数。
        *   **稳定调制策略**：采用残差放缩 `A + 0.1 * (gamma * A + beta - A)` 进行 FiLM 调制，极大稳定了训练初期的梯度。
*   **Slide 11: 创新四：PI-DCH (物理信息动态封顶头)**
    *   **理论**：可微物理约束 $\mathcal{C}_\Omega$，避免 Hard Clip 导致的 Train-Inference Mismatch。
    *   **代码干货 (`_apply_dynamic_capping`)**：
        *   通过 `GateHead` 预测屏障门极性 `u = sigmoid(linear(H_F))`。
        *   **动态包络生成**：反归一化风速数据 `speed_phys`，构建双重 Sigmoid 掩码 `f_v`，将切入风速 (<3m/s) 和切出风速 (>24m/s) 区域的容量天花板动态压缩至 0。
        *   **可微约束计算**：软逼近 `dec_out = dec_out + u * (cap_dyn - dec_out)`，这使得超出物理极限的损失能完美流回主干网络进行反向传播。

### 第四部分：优化与总结 (Optimization & Summary)

*   **Slide 12: 计算复杂度优势**
    *   利用 AGSP 的 Token 压缩和 R-STMF 的 Top-K 掩码，将自注意力复杂度从 $O(N \cdot T^2 + T \cdot N^2)$ 成功解耦并降阶至 $O(N \cdot P^2 + P \cdot K \cdot N)$。
*   **Slide 13: 多目标物理罚项损失**
    *   展示优化公式：$\mathcal{L}_{mae} + \lambda(\mathcal{L}_{neg} + \mathcal{L}_{cap})$。
    *   说明代码中如何利用 ReLU (`[x]+`) 对软逼近预测值 $\hat{Y}_{soft}$ 进行原生的越界惩罚。
*   **Slide 14: 当前进度 (Progress)**
    *   理论闭环：`Problem Definition` 与 `Methodology` 写作与数学推导完毕。
    *   代码工程：`TimeXerWind` 工程下的 `develop` 分支已完全对齐上述四大核心模块。
*   **Slide 15: 下一步计划与 Q&A (Next Steps)**
    *   绘制高质量论文配图（主架构、微观模块图）。
    *   在真实风电场/交通数据集上运行完整的 Benchmark 和消融实验。
    *   Q&A 环节。

---

## 🎙️ 20分钟详细演讲逐字稿 (Speaker Script)

### [0:00-3:00] 开场与背景动机 (Slide 1-4)
> 各位老师、同学，大家好。今天我汇报的题目是 PhyST-Net：一个物理感知的可微时空预测网络。这是我们近期在时空图预测领域的一项理论与架构设计工作。
> 
> 在切入正题前，我们先看看时空预测的核心挑战。比如风电场预测或交通流量预测，这些节点不是孤立的。它们的未来轨迹受三个维度的影响：第一是自身历史，第二是空间图依赖，第三则是至关重要的“未来外部驱动力”，也就是天气预报或调度约束。
>
> 当我们尝试用现有的顶尖模型去解决这些实际问题时，遭遇了 **“预测三难困境” (The Forecasting Trilemma)**：
> 1. **容量与噪声的矛盾**：主流 Transformer 的“硬切片 (Hard Patching)”会强行把连续的物理事件切断，产生边界伪影；
> 2. **多尺度空间复杂性**：传统的图卷积把“全场统一的气象宏观影响”和“相邻节点间的局部物理交互”混为一谈；
> 3. **物理一致性缺失**：纯数据驱动模型由于不了解物理法则，经常预测出负发电量，或者超出设备装机极限的值。更糟糕的是，如果只是简单地在输出端强行把异常值设为 0 (Hard Clip)，会破坏梯度流，导致训练和推理产生严重的 Mismatch。

### [3:00-7:00] 严谨的问题定义 (Slide 5-6)
> 为了打破这个困境，我们在论文的 **Problem Definition** 阶段进行了严格的重构。
>
> 我们首先在图的定义上，将空间拓扑 $A$ 和节点元数据 $M_G$ 彻底解耦。接着，我们对“外生变量 (ExG)”进行了非常严厉的区分：
> 我们指出，模型在训练时看到的数据，不代表在实际部署中能拿到。因此，我们加入了**严格的防泄漏假设**。只有客观存在的天气预报 (NWP) 才能作为未来条件驱动 $\mathbf{E}^f$，而绝不使用任何“偷看未来”的变量。
> 
> 最终，我们的预测函数被解构为 $\hat{\mathbf{Y}} = \mathcal{C}_{\Omega}\big(\mathcal{H}_{\theta}(\mathcal{X}_t)\big)$。也就是说，模型是一个无约束的时空主干 $\mathcal{H}_{\theta}$，外面包着一个物理约束算子 $\mathcal{C}_{\Omega}$。基础优化目标采用了抗噪性更强的通用 MAE 损失。

### [7:00-15:00] 核心方法论与代码落地解析 (Slide 7-11)
> 接下来是干货部分，也就是我们如何通过代码落地这四大创新模块的。
>
> **第一，AGSP (自适应高斯软窗口 Patching)**：
> 为了解决硬切片的伪影问题，我们在 `EnEmbedding` 类的 `_embed_dcp` 函数中实现了 Differentiable Continuous Patching。我们用可学习的高斯密度函数替代了硬窗口。在代码中，我们学习中心点 $\mu$ 和带宽 $\sigma$。为了保证数值稳定，我们使用了 `+ 1e-4` 避免除零，并通过归一化实现了单位分解 (Partition of Unity)。这让模型在剧烈波动时能自动密集采样，在平缓期能放大感受野。
>
> **第二，R-STMF (关系型时空流形融合)**：
> 对应我们的 `PSTG_TimeMixerFusion` 类。我们通过 1D 卷积将序列解耦为 `seasonal` (宏观趋势) 和 `trend` (局部变化)。在计算注意力图时，我们引入了强物理先验。在执行 Top-$K$ 稀疏邻居提取前，我们先施加了 `dist_mask`，强制把非空间邻居的 Attention 设为负无穷 `-1e9`。这确保了信息只沿着真实的物理网络传播，极大提高了计算效率和鲁棒性。
>
> **第三，K-AMC (基于 KAN 的外生变量调制)**：
> 这是用来注入未来 NWP 数据的。对应的代码类是 `KFiLM`。传统的做法是直接把特征拼接。我们创新性地引入了 KAN 网络。我们将未来的 NWP 特征对齐到 Patch 维度后，利用 B-Spline 样条函数预测出 $\gamma$ 和 $\beta$ 参数。更关键的是，在代码实现中，我们采用了 `A + 0.1 * (gamma * A + beta - A)` 的残差放缩设计，这有效避免了训练初期的梯度爆炸。
>
> **第四，PI-DCH (物理信息动态封顶头)**：
> 它是物理约束算子 $\mathcal{C}_{\Omega}$ 的实例化。在 `_apply_dynamic_capping` 逻辑中，我们先从输入中提取真实风速并反归一化。基于风机物理特性，我们构造了双重 Sigmoid 掩码：当风速低于切入风速 (3m/s) 或高于切出风速 (24m/s) 时，动态天花板 $C_{dyn}$ 自动塌缩为 0。
> 模型首先会计算一个可微的屏障门 $u$，使得预测值“软着陆”向动态天花板靠近 (`dec_out + u * (cap_dyn - dec_out)`)。通过这种“软逼近”，超限惩罚的梯度能够无缝回传，完美解决了 Hard Clip 的梯度断裂问题。

### [15:00-20:00] 优化、进度与下一步 (Slide 12-15)
> 在模型复杂度方面，借助 AGSP 的 Token 压缩和 R-STMF 的稀疏掩码，我们将传统注意力机制的 $O(T^2)$ 瓶颈解耦降阶到了 $O(P^2)$。
>
> 在优化目标上，我们将 $\mathcal{L}_{mae}$ 与 $\mathcal{L}_{penalty}$ 结合。通过计算超出物理极值的 ReLU 惩罚项，强迫模型内化物理约束。
>
> **汇报目前的进度**：目前这篇论文的引言、问题定义以及核心的方法论部分，也就是刚才讲的数学推导与代码对齐，已经全部完成，论文核心骨架已经稳固。并且，对应的模型架构在我们的 `wind-power-forecast` 的 `develop` 分支中已经有了详实的 PyTorch 实现支撑。
>
> **我们的下一步计划是**：
> 1. 把理论模块转化为精美的架构图和微观模块配图。
> 2. 正式在 5090 服务器上推进物理数据集上的 Benchmark 对比与消融实验。
>
> 以上就是我本周的重点工作进展，干货较多，感谢大家的聆听，欢迎各位老师和同学批评指正。