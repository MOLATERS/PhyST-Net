# PhyST-Net / TimeXerWind 20 分钟组会汇报 PPT 讲稿

## Slide 1. 标题

**页面内容**
- PhyST-Net / TimeXerWind
- 面向风电功率预测的外生变量感知时空图 Transformer
- 关键词：ExG-aware、spatio-temporal graph Transformer、physics-informed forecasting

**建议图表**
- 背景图：风场 + 功率曲线
- 角落放整体架构缩略图

**讲稿**
各位老师好，今天汇报的是 PhyST-Net，也就是工程实现中的 TimeXerWind。这个工作面向风电功率预测，核心目标不是单纯堆一个更大的时序模型，而是把历史功率、SCADA 气象、未来 NWP 或未来气象代理、风机空间关系，以及功率物理边界，拆成清晰的建模路径。今天我主要讲论文 intro 和 method 的主线，并用当前工程已经结算的阶段性结果支撑。

**时间**：约 0.5 分钟

---

## Slide 2. 研究背景

**页面内容**
- 风电预测直接影响调度、备用容量、并网安全
- 风电序列非平稳：爬坡、突变、季节性、湍流噪声并存
- 功率不是自驱动序列，强烈受未来气象驱动
- 多风机之间存在尾流、地形、天气锋面等空间耦合
- 预测值必须满足物理可行性：不能为负，不能超过动态容量边界

**建议图表**
- 风速、风向、功率随时间变化曲线
- 风机布局示意图

**讲稿**
风电功率预测的难点在于，它不是普通的单变量时间序列预测。功率变化由历史运行状态、未来气象条件和风机之间的空间影响共同决定。尤其在调度场景中，模型不仅要准，还要稳定、可解释，并且输出不能违反物理边界。比如负功率或者超过额定容量的预测，即使误差指标看起来不差，也会降低调度可信度。

**时间**：约 1.5 分钟

---

## Slide 3. 核心问题：Forecasting Trilemma

**页面内容**
- Model Capacity vs. High-Frequency Noise：Transformer 容量强，但容易吸收高频随机扰动
- Domain Consistency vs. Infeasible Output：纯数据驱动模型容易产生不可行预测
- Spatio-Temporal Complexity vs. Multi-scale Spatial Mechanisms：局部尾流与全局天气场不能混成一个空间算子

**建议图表**
- 三角图：Capacity / Consistency / Complexity
- 论文中的 multi-scale process 示意图

**讲稿**
论文 intro 把问题总结为 Forecasting Trilemma。第一，模型容量越强，越容易把湍流和局部扰动也学进去；第二，预测模型如果不显式处理物理约束，就可能输出不可行功率；第三，风场空间关系是多尺度的，近邻尾流、地形影响和天气锋面不是同一种关系。如果只用一个固定图或者一个全连接注意力，很容易把不同机制混在一起。

**时间**：约 1.5 分钟

---

## Slide 4. 现有方法不足

**页面内容**
- 固定 patch 会切断连续物理事件
- 普通 STGNN 往往把所有空间关系混合传播
- 历史外生变量与未来外生变量容易接口混乱
- 物理约束常被当作后处理，而不是模型内部路径
- 长预测窗口下，气象驱动与历史惯性的重要性会变化

**建议图表**
- 固定 patch vs soft-window patch 对比
- post-hoc clamp 与 in-head constraint 对比

**讲稿**
现有方法的核心问题不是某一个模块不够强，而是建模契约不清楚。比如固定 patch 在爬坡事件中会把连续过程切成几段；普通图神经网络会让所有风机关系通过同一个传播机制混合；外生变量如果只是简单拼接，很难区分历史观测和未来驱动。物理约束如果只在最后 clamp，一方面不能参与训练，另一方面也解释不了模型为什么会靠近边界。

**时间**：约 1.5 分钟

---

## Slide 5. 总体思路

**页面内容**
- PhyST-Net 定义 forecasting contract
- 历史功率路径：学习节点自身时间动态
- 历史 ExG 路径：提供历史运行环境
- 未来 ExG 路径：只在预测 horizon 条件调制中注入
- 图结构路径：建模风机间关系
- 物理约束路径：在输出头内生成可行预测

**建议图表**
- 输入契约流程图：Historical Power / Historical ExG / Future ExG / Graph / Capacity

**讲稿**
PhyST-Net 的整体思路是把输入和知识拆清楚。历史功率和历史气象负责形成上下文；未来气象只进入未来条件调制路径，避免和历史信息混用；风机位置和图结构进入空间关系模块；容量和物理边界进入预测头。这样模型不是把所有东西拼成一个大向量，而是让每类信息在最合适的位置发挥作用。

**时间**：约 1.5 分钟

---

## Slide 6. 系统架构总览

**页面内容**
- AGSP：连续可微 patching
- Transformer backbone：学习 patch-level 时间依赖
- R-STMF：关系型时空流形融合
- K-AMC：未来气象条件调制
- PI-DCH：物理约束动态封顶输出

**建议图表**
- 使用 `/home/lx/wind-power-forecast/docs/figures/timexerwind_tkde_framework.svg`
- 或论文 `sections/architecture_figure`

**讲稿**
整体流水线是：先用 AGSP 把历史序列转成连续 token，再用 Transformer 学时间依赖；随后 R-STMF 对不同尺度的空间关系进行稀疏图融合；然后 K-AMC 用未来 ExG 生成 gamma 和 beta，对 latent state 做条件调制；最后 PI-DCH 在输出头里进行动态容量约束。这里四个模块分别对应时间连续性、空间复杂性、未来驱动和物理可行性。

**时间**：约 2 分钟

---

## Slide 7. 模块一：AGSP / DCP

**页面内容**
- Adaptive Gaussian-soft-windowed Patching
- 用可学习 Gaussian soft-window 替代固定 patch
- learnable center `mu`，learnable bandwidth `sigma`
- `mu` 均匀初始化，`sigma clamp` 防止窗口塌缩
- 作用：保留连续事件，过滤高频局部噪声

**建议图表**
- 多个 Gaussian window 覆盖时间轴
- 固定切块与软窗口 token 的对比

**讲稿**
AGSP 对应代码旧名 DCP。传统 patching 是硬切分，但风电中的爬坡和突变不会刚好对齐 patch 边界。AGSP 用一组可学习的 Gaussian soft-window 对历史序列加权聚合。工程实现里，窗口中心 `mu` 采用均匀初始化，`log_sigma` 可学习，并通过 clamp 避免窗口过窄或塌缩。这样模型可以自动决定哪些时间段需要更细的感受野，哪些稳定区间可以更平滑地聚合。

**时间**：约 1.5 分钟

---

## Slide 8. 模块二：R-STMF / PSTG++

**页面内容**
- Relational Spatio-Temporal Manifold Fusion
- 距离候选 mask + Top-K sparse attention
- seasonal / trend 或局部 / 低频双分支融合
- 避免全图 dense attention 带来的噪声传播
- 提升空间关系可解释性和可扩展性

**建议图表**
- 风机节点图 + Top-K 边
- 双分支空间融合结构图

**讲稿**
R-STMF 对应工程旧名 PSTG++。它解决的是空间关系混合的问题。风机之间既有近邻尾流，也有更大尺度的天气共同驱动。R-STMF 先把 temporal state 分解成不同分支，再在每个分支上做 relational attention，并用候选 mask 和 Top-K 控制信息传播范围。这样做的好处是，模型不会让所有节点都互相传播噪声，而是学习更稀疏、更有结构的空间依赖。

**时间**：约 1.5 分钟

---

## Slide 9. 模块三：K-AMC / KFiLM

**页面内容**
- KAN-enhanced Adaptive Meteorological Conditioning
- 未来 ExG 不在输入端简单拼接，而是在 latent state 上调制
- KAN spline 生成 `gamma / beta`
- adaptive pooling 将 horizon-wise ExG 对齐到 patch-wise latent
- 作用：让未来气象驱动预测窗口内的状态演化

**建议图表**
- Future ExG -> KAN -> gamma/beta -> latent modulation
- 风速、风向对功率预测的调制示意

**讲稿**
K-AMC 对应旧名 KFiLM。风电预测中，未来气象是非常关键的信息，但它应该以预测 horizon 的条件形式进入模型，而不是和历史变量混在一起。K-AMC 先把 horizon-wise 的未来 ExG 通过 adaptive pooling 对齐到 patch-wise latent，再用 KAN spline 生成 `gamma` 和 `beta`，对图增强后的状态做 scale-shift 调制。直观上，它让模型在不同未来风况下使用不同的表示方式。

**时间**：约 1.5 分钟

---

## Slide 10. 模块四：PI-DCH

**页面内容**
- Physics-Informed Dynamic Capping Head
- 先生成 unconstrained prediction
- 再用 gate soft steering 到动态容量包络
- 推理阶段执行严格 clamp
- 解决负功率、超容量等不可行输出
- 约束进入训练路径，而不是纯后处理

**建议图表**
- 原始预测曲线、动态容量上界、约束后预测曲线
- GateHead 工作流程图

**讲稿**
PI-DCH 对应旧名 GateHead 或 Dynamic Capping。它不是简单在最后把结果截断，而是在预测头内部引入动态容量包络。模型先产生未约束预测，再通过 gate 把预测平滑拉向可行区间，训练时这个 soft steering 可以参与反向传播；推理时再执行严格 clamp，保证输出在 0 到动态容量上界之间。这个模块的意义是把物理可行性变成模型结构的一部分。

**时间**：约 1.5 分钟

---

## Slide 11. 数据与实验设置

**页面内容**
- 当前项目结算数据集：EFJ、SDWPF、HoT、Kelmarsh
- 预测尺度统一解释为：ultra-short、short、medium
- 对应物理时间尺度约为：4h、72h、240h
- 历史窗口：4 天；label：1 天
- scaler 只在训练集拟合
- 输入 DWT 降噪，目标不降噪
- 未来气象在无真实 NWP 时作为 future-driver proxy

**建议图表**
- 数据集规模表
- 时间窗口切分图：lookback / label / prediction horizon

**讲稿**
实验设置上，当前工程已经对 EFJ、SDWPF、HoT 和 Kelmarsh 做了阶段性结算。三个 horizon 统一解释为 ultra-short、short 和 medium，对应不同调度尺度。训练时 normalization 只使用训练集统计，避免时间泄漏。输入侧使用 DWT 降噪来降低 SCADA 高频扰动，但目标功率不降噪，以保证评估仍然面对真实预测任务。需要强调的是，论文实验章节目前还是 placeholder，所以这里汇报的是工程当前结算结果，不把它说成最终投稿表格。

**时间**：约 1.5 分钟

---

## Slide 12. 阶段性实验结果

**页面内容**
- EFJ：92.74 / 88.97 / 87.57
- SDWPF：94.88 / 90.92 / 87.92
- HoT：92.04 / 87.51 / 84.85
- Kelmarsh：96.02 / 93.55 / 90.99
- 数值顺序：ultra-short / short / medium，指标为 Lambda %

**建议图表**
- `/home/lx/wind-power-forecast/docs/figures/academic_lambda_comparison.png`
- `/home/lx/wind-power-forecast/docs/figures/academic_r2_comparison.png`

**讲稿**
从当前结果看，PhyST-Net 在四个数据集、三个预测尺度上都取得了比较强的 Lambda 表现。Kelmarsh ultra-short 达到 96.02%，SDWPF ultra-short 达到 94.88%，EFJ ultra-short 达到 92.74%，HoT ultra-short 达到 92.04%。更重要的是，性能不是只集中在一个小数据集，而是在大规模 SDWPF、中等规模 HoT、小规模 Kelmarsh 以及 EFJ 上都有稳定结果。随着 horizon 拉长，Lambda 会下降，这是符合风电预测规律的，但 medium horizon 仍保持在 84.85% 到 90.99% 区间。

**时间**：约 2 分钟

---

## Slide 13. 消融与实现可信度

**页面内容**
- NoDCP 消融已完成部分 ultra-short 对照
- EFJ：Full 92.74 vs NoDCP 91.94
- Kelmarsh：Full 96.02 vs NoDCP 95.59
- SDWPF：Full 94.88 vs NoDCP 95.09
- 结论：AGSP/DCP 整体有价值，但存在数据集和超参依赖
- 不声称所有模块在所有场景必胜

**建议图表**
- Full vs NoDCP 条形图
- 一页小表：Dataset / Full / NoDCP / Difference

**讲稿**
消融部分目前最完整的是 NoDCP，也就是去掉 AGSP/DCP 的对照。EFJ 和 Kelmarsh 上 Full 模型更好，说明连续 soft-window patching 对这些场景有效。但 SDWPF ultra-short 上 NoDCP 是 95.09%，略高于 Full 的 94.88%。这个结果不能回避，也不能硬解释成模块全场景必胜。更合理的说法是：AGSP 的收益和数据集频率、风机规模、patch 数、超参设置有关。后续需要补齐 R-STMF、K-AMC、PI-DCH 的完整消融，形成最终论文证据链。

**时间**：约 1.5 分钟

---

## Slide 14. 总结与下一步

**页面内容**
- 贡献一：连续可微 tokenization，缓解固定 patch 边界问题
- 贡献二：多尺度稀疏图融合，降低空间噪声传播
- 贡献三：未来 ExG 条件调制 + 物理可行输出头
- 下一步：
  - 补齐论文主实验表
  - 完整模块消融
  - R-STMF / K-AMC 可视化
  - 复杂度与推理效率表
  - 典型爬坡案例分析

**建议图表**
- 三个贡献图标：Temporal / Graph / Physics
- Roadmap 表

**讲稿**
总结一下，PhyST-Net 的核心不是单个模块，而是一个面向风电预测的结构化建模契约：用 AGSP 解决连续时间 tokenization，用 R-STMF 解决多尺度空间传播，用 K-AMC 引入未来气象驱动，用 PI-DCH 保证输出物理可行。当前阶段性结果已经比较有说服力，尤其是多数据集 Lambda 表现较强。下一步工作重点是把实验章节从 placeholder 变成完整论文证据，包括主表、完整消融、可视化、复杂度和案例分析。

**时间**：约 1 分钟

---

**总时长**：约 20 分钟。
