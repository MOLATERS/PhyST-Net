# PhyST-Net Paper

> **PhyST-Net: A Physics-Aware Differentiable Spatio-Temporal Transformer Network for Robust Wind Power Forecasting**

IEEE Transactions 期刊论文 LaTeX 源码。

## 模块命名

| 缩写 | 全称 | 核心贡献 |
|------|------|----------|
| PhyST-Net | Physics-aware Differentiable Spatio-Temporal Transformer Network | 全流程可微的物理一致性时空特征学习框架 |
| AGSP | Adaptive Gaussian-soft-windowed Patching | 高斯核基函数解决时域硬分块伪影，实现亚步长采样 |
| R-STMF | Relational Spatio-Temporal Manifold Fusion | 双分支 KNN 关系图解耦局域尾流与宏观气象时空流形 |
| K-AMC | KAN-enhanced Adaptive Meteorological Conditioning | 基于 K-A 理论对非平稳 NWP 信号非线性调制 |
| PI-DCH | Physics-Informed Dynamic Capping Head | 可微门控算子强制输出满足物理可行域限制 |

## 论文大纲（当前 12 页）

### Abstract
- 背景：风电并网间歇性挑战
- 问题定义："Forecasting Trilemma"（模型容量 vs 物理一致性 vs 时空复杂度）
- 方法：提出 PhyST-Net，四阶段物理信息流水线（AGSP → R-STMF → K-AMC → PI-DCH）
- 结果：三数据集验证，最高 96.02% 电网合规率

### I. Introduction (`intro.tex`)
1. 能源转型背景与风电渗透率提升
2. 风电并网技术挑战
3. **I-A 大气流体不确定性物理**：多尺度过程、风斜坡事件
4. **I-B Forecasting Trilemma 理论框架**：
   - I-B-1 模型容量 vs 大气噪声
   - I-B-2 物理一致性与电网安全
   - I-B-3 时空复杂度与缩放动态
- *图占位：Fig.1 能源转型趋势 / Fig.2 多尺度大气过程*

### II. Related Work (`related_work.tex`)
1. 经典统计方法（ARIMA, Kalman）
2. 机器学习过渡（SVR, RF, GBDT）
3. 深度学习革命：RNN → Transformer
4. Patching 范式（PatchTST, iTransformer）
5. 深度生成模型与概率预测（VAE, GAN, Diffusion）
6. 跨域迁移与预训练
7. 气动弹性耦合与机械状态建模
8. 空间建模与图神经网络
9. KAN 可解释性
10. Physics-Informed Deep Learning (PIDL)
- *图占位：Fig.3 WPF 技术时间线 / Fig.4 注意力图可视化*

### III. Methodology (`methodology.tex`)
1. **III-A AGSP**：自适应高斯软窗分块
   - 连续分块物理直觉
   - 数学公式（高斯权重、Partition of Unity、Token 聚合）
   - 自适应低通滤波机制
   - *Algorithm 1: AGSP 权重计算*
2. **III-B R-STMF**：多尺度时空推理
   - 气动解耦原则
   - Seasonal Branch（近场尾流，方向-距离核）
   - Trend Branch（远场天气锋，自注意力图）
   - 图融合逻辑（可学习门控）
   - *Algorithm 2: R-STMF 双分支图构建*
   - *图占位：Fig.5 整体架构图（跨双栏大图）*
3. **III-C K-AMC**：KAN 增强气象调制
   - MLP 在物理建模中的理论局限
   - KAN 范式：边上的样条函数
   - 调制公式（γ, β 生成 + 残差缩放）
   - *Algorithm 3: K-AMC 调制逻辑*
4. **III-D PI-DCH**：物理流形投影
   - 可微障碍函数与机械安全
   - 双 Sigmoid 平滑裁剪 + τ 退火
   - *Algorithm 4: PI-DCH 裁剪逻辑*

### IV. Experimental Setup (`experiments.tex`)
1. **IV-A 工业数据集物理画像**：
   - SDWPF（134 turbines，海岸季风）
   - HoT（21 turbines，苏格兰高地）
   - Kelmarsh（6 turbines，小型集群）
2. **IV-B 特征工程**：气象 / 电气 / 机械 15+ 特征
3. **IV-C 实现细节**：PyTorch 2.1, hidden=128, 3 layers, AdamW, RTX 5090
- *图占位：Fig.6 测试场地概览 / Fig.7 SCADA 传感器分布*

### V. Results and Discussion (`experiments.tex` 后半)
1. **V-A 基准性能与电网合规**（Table I：超短期 RMSE/MAE/Λ）
2. **V-B 超参数鲁棒性与计算开销**（K_near/K_far 敏感性, 12ms 推理）
3. **V-C 消融实验**（w/o AGSP +5.8% / w/o R-STMF +8.1% / w/o K-AMC +3.4% / w/o PI-DCH -4.5%）

### VI. Discussion (`discussion.tex`)
1. 时序连续性在分词化中的作用
2. 空间拓扑感知与解耦
3. K-AMC 样条物理在 NWP 集成中的优势
4. 跨气候区泛化能力
5. 社会影响与环境可持续性
6. 电网稳定性与物理安全
7. 高合规率预测的经济影响（$250K/年估算）

### VII. Case Studies (`case_studies.tex`)
1. Case A：HoT 风斜坡（PI-DCH 防越界）
2. Case B：SDWPF 大规模集群同步（R-STMF 传播延迟识别）
3. Case C：Kelmarsh NWP 偏差校正（K-AMC 样条局部修正）
4. Case D：低风速持续与负功率避免（PI-DCH 零输出映射）

### VIII. Conclusion (`main.tex`)
- 总结：PhyST-Net 解决 Forecasting Trilemma
- 工业/政策影响 / 社会环境影响 / 局限与未来方向

---

## 待斟酌与补充事项

### 关键内容缺失（影响投稿）

| 优先级 | 问题 | 说明 |
|--------|------|------|
| **P0** | **基准对比严重不足** | Table I 仅对比 1 个 Baseline (Transformer)。IEEE Trans 通常要求 6-10 个基准，包括经典(DLinear)、Transformer 系列(PatchTST, iTransformer, Informer)、STGNN 系列(MTGNN, AGCRN)等 |
| **P0** | **消融实验缺数据表** | 当前仅有文字描述百分比变化，需要完整的三数据集×四消融条件的 RMSE/MAE/Λ 数据表 |
| **P0** | **所有 7 张图均为占位符** | 使用 `\rule{}{}` 黑块，需替换为实际 `\includegraphics` |
| **P0** | **Case Studies 无图** | 四个案例研究均无可视化图示，审稿人会认为缺乏证据支撑 |
| **P1** | **统计显著性检验缺失** | 仅报告单一数值，未提供置信区间或多次运行的标准差 |
| **P1** | **仅超短期预测** | 只有 96-step (24h) 超短期，缺少短中期预测结果（72h, 240h）|

### 需要斟酌的表述

| 位置 | 问题 | 建议 |
|------|------|------|
| Abstract | "unprecedented grid compliance rates" | "unprecedented" 绝对化用词，审稿易质疑，改为 "state-of-the-art" 或给出文献对比 |
| main.tex:45 | 经济影响 $250,000 估算无依据 | 需补充计算方法或标注为初步估算，否则删除 |
| main.tex:42-55 | Conclusion 中工业/社会/环境影响节 | 内容空洞无数据支撑，建议大幅精简或移入 Discussion |
| intro.tex | IBR 缩写仅出现一次 | 在 Intro 定义后全文未再使用，考虑删除或替换 |
| related_work.tex | Section 标题 "Detailed Evolution of Forecasting Methodologies" | "Detailed" 不符合 IEEE 风格，改为 "Related Work" 或 "Literature Review" |
| experiments.tex | "w/o R-STMF: reduced MAE by 8.1%" | 语义矛盾：消融移除组件应该性能下降（增加误差），"reduced MAE" 含义不清 |
| discussion.tex | K-AMC 样条"Visualization of the K-AMC edge functions" | 声称有可视化但文中无图，要么加图要么删此句 |
| discussion.tex | "event alignment" "KAN Advantage" | 这些概念未正式定义就使用，需先给出定义 |
| 全文 | "Forecasting Trilemma" 反复出现 | 过度使用引号标注，首次定义后后续不再加引号 |

### 格式与规范补充

| 问题 | 说明 |
|------|------|
| 无基金致谢 | 提交终稿前需确认基金信息并添加 |
| 无通讯作者 | 终稿需标注，投稿版不标（当前已正确移除） |
| `\markboth` | 当前为 `{\it Paper}`，需根据阶段调整 |
| Overleaf 项目命名 | 按 X.Li_PSTN_paper-01a_2605 格式 |

### 图片清单（7 个占位 + 4 个案例图）

| 编号 | 内容 | 状态 |
|------|------|------|
| Fig.1 | Global Energy Transition Trends | 占位 |
| Fig.2 | Multi-Scale Atmospheric Processes | 占位 |
| Fig.3 | Technological Timeline of WPF | 占位 |
| Fig.4 | Attention Maps Visualization | 占位 |
| Fig.5 | Overall Architecture of PhyST-Net | 占位（核心图） |
| Fig.6 | Industrial Test Sites Overview | 占位 |
| Fig.7 | SCADA Sensor Suit Distribution | 占位 |
| Fig.A | Case A: Wind Ramp at HoT | 缺失 |
| Fig.B | Case B: Cluster Synchronization | 缺失 |
| Fig.C | Case C: NWP Bias Correction | 缺失 |
| Fig.D | Case D: Negative Power Avoidance | 缺失 |

## 项目结构

```
source/
  main.tex            # 主文件（含内联 thebibliography）
  nomenclature.tex    # 符号表
  intro.tex           # 第 I 节
  related_work.tex    # 第 II 节
  methodology.tex     # 第 III 节
  experiments.tex     # 第 IV-V 节
  discussion.tex      # 第 VI 节
  case_studies.tex    # 第 VII 节
IEEEtran.cls          # IEEE Transactions 模板类文件
figures/              # 图片目录（待填充）
output/               # 编译输出 PDF
```

## 编译

本地 TeX Live，两遍 pdflatex 即可（无需 bibtex）：

```bash
cd source
pdflatex main.tex
pdflatex main.tex
```

## LaTeX 规范

- ASCII only，不含中文字符
- 引号用 `` ``text'' ``，不用直引号
- Em-dash `---`，en-dash `--`，连字符 `-`
- 禁止手打编号，一律 `\label{}` / `\ref{}`
- 摘要中缩写正文中须重新定义
- 公式后 `where` 不缩进
- 证明用 `\begin{IEEEproof}...\end{IEEEproof}`
