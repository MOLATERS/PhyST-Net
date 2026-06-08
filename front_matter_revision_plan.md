# 前端章节打磨执行计划 (Front Matter Revision Plan)

**执行状态**: 已完成 (Completed)
**目标任务**: 根据顶会/顶刊 (TKDE, NeurIPS, ICLR 等) 写作规范，打磨摘要、引言、相关工作、问题定义与方法论。
**核心基调**: 通用的 ExG-aware spatio-temporal graph forecasting / domain-constrained network forecasting 框架。剥离特定领域（如风电）的强绑定，将其升华为实例化任务。

---

## 1. 摘要 (Abstract)
**执行逻辑**:
- **第一句 (背景与价值)**: 介绍时空预测在复杂网络（交通、电网等）中的核心作用与价值。
- **第二句 (核心局限)**: 指出现有方法在动态空间依赖、外部变量融合及物理约束处理上的不足（如直接拼接导致的数据泄露与梯度消失）。
- **第三句 (提出方法)**: 隆重推出本文的通用框架 PhyST-Net。
- **第四至 X 句 (关键模块)**: 依次概括 AGSP（自适应软窗口）、R-STMF（多分支关系流形融合）、K-AMC（外生变量调制）和 PI-DCH（物理信息动态封顶头）的核心机制。
- **最后一句 (实验总结)**: 总结模型在多真实数据集（后续以风电为实例化）上，在 MAE, RMSE, MAPE 等指标上对主流 Baseline 的全面超越。
*操作: 修改了 `main.tex` 中的 abstract 环境。*

## 2. 引言 (Introduction)
**执行逻辑**:
- **第一部分 (研究背景)**: 阐述时空预测任务在宏观趋势与微观噪声交织下的挑战。
- **第二部分 (现有方法局限)**: 提出极具理论深度的 **"预测三难题" (Forecasting Trilemma)**：模型容量与高频噪声的矛盾、领域物理一致性的缺失、以及多尺度动态空间的复杂度。
- **第三部分 (本文解决方案)**: 提出 PhyST-Net，说明它是如何系统性逐一击破上述 Trilemma 的。
- **第四部分 (本文贡献)**: 清晰列出四点核心贡献（通用公式化、连续时间 Token 化、模块化图适配器、前瞻性物理约束解码）。
*操作: 已在 `sections/intro.tex` 中深度打磨完成。*

## 3. 相关工作 (Related Work)
**执行逻辑**:
- 摒弃“流水账”，按方法类别与挑战设立小标题（时序 Transformer、时空图预测、外生变量感知、可解释条件建模、可扩展性、物理约束感知）。
- 引入了 2024-2026 最新 SOTA（TimeXer, GCGNet, TimeFilter, TimeMixer++, ExoST 等）。
- 重点指出传统方法在处理 ExG 变量时的“时间不平衡效应 (temporal imbalance effects)”与接口混淆，凸显本工作的接口级创新。
*操作: 已在 `sections/related_work.tex` 中深度打磨完成。*

## 4. 问题定义 (Problem Definition)
**执行逻辑**:
- 使用高级“符号压缩”策略，定义联合输入元组 $\mathcal{X}_t \triangleq (\mathbf{X}, \mathbf{E}^h, \mathbf{E}^f, A, M_G)$。
- 给出极其精简、严密的预测函数公式 $\hat{\mathbf{Y}} = \mathcal{C}_{\Omega}\big(\mathcal{H}_{\theta}(\mathcal{X}_t)\big)$ 与包含惩罚项的优化目标损失函数。
*操作: 已在 `sections/problem_definition.tex` 中深度打磨完成。*

## 5. 方法论 (Methodology)
**执行逻辑**:
- **整体框架**: 提供输入、输出与模块串联关系，并附带 Algorithm 1 详细伪代码。
- **AGSP (时间表示)**: 对齐“复杂度平衡序列建模”，引入连续软窗口公式。
- **R-STMF (时空交互)**: 借鉴 TimeMixer++ 与 TimeFilter，引入 Global Token 和 Top-K 动态路由进行多分支（宏观/微观）解耦。
- **K-AMC (泛化/外生交互)**: 引入 B-spline KAN 机制对特征进行自适应调制。
- **PI-DCH (鲁棒/约束增强)**: 提出通过软截断解决硬截断引起的梯度消失 (Gradient Vanishing) 问题，完美内化物理边界。
*操作: 已在 `sections/methodology.tex` 中深度打磨完成。*