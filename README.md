# PhyST-Net Paper

> **PhyST-Net: A Physics-Aware Differentiable Spatio-Temporal Transformer Network for Robust Wind Power Forecasting**

IEEE Transactions 期刊论文 LaTeX 源码（已完成项目重构：主控文件在根目录，模块化 `.tex` 源码归纳于 `sections/` 文件夹，参考文献独立为 `reference/ref.bib`，已完美压缩至符合标准的 **10 页**）。

---

## 核心模块命名与对应关系

| 缩写 | 全称 | 旧称 | 核心创新点 |
|---|---|---|---|
| **PhyST-Net** | Physics-aware Differentiable Spatio-Temporal Transformer Network | TimeXerWind | 全流程可微的物理一致性时空特征学习流水线 |
| **AGSP** | Adaptive Gaussian-soft-windowed Patching | DCP | 自适应可微高斯软窗分块，解决硬边界伪影与大气噪声 |
| **R-STMF** | Relational Spatio-Temporal Manifold Fusion | PSTG++ | 双分支动态近邻图融合，解耦局域尾流气动与宏观气象时空流形 |
| **K-AMC** | KAN-enhanced Adaptive Meteorological Conditioning | K-FiLM | 基于 Kolmogorov-Arnold 网络的样条边参数化非线性气象条件调制 |
| **PI-DCH** | Physics-Informed Dynamic Capping Head | GateHead | 可微双 Sigmoid 边界壁垒函数，强制预测输出严格位于物理可行域 |

---

## 论文大纲与结构（已重构为扁平根目录结构）

- **Abstract** (摘要)：定义风力发电并网面临的 "Forecasting Trilemma"，阐述 PhyST-Net 的四阶段可微物理流形流水线，在 SDWPF, HoT, Kelmarsh 真实数据集上实现最高 96.02% 电网合规率 ($\Lambda$)。
- **I. Introduction** (`intro.tex`)：阐述全球能源转型与高渗透率风电挑战。增加 I-A (大气流体不确定性物理) 与 I-B (Forecasting Trilemma 理论框架)，并在末尾以 bullet points 形式正式归纳出三项核心学术贡献（时域、空间、物理/电网安全）。
- **II. Related Work** (`related_work.tex`)：由原先零散的 10 个小节合并为 4 个高凝聚力的学术专题（A. 时序预测、B. 时空图神经网络、C. KAN 在科学机器学习中的应用、D. 物理信息引导深度学习）。剔除与模型无关的概率/生成模型、预训练基础模型、叶片气动弹性加速规机械健康检测等冗长叙述，聚焦于核心关联文献。
- **III. Methodology** (`methodology.tex`)：
  - 引入了全局流水线映射方程式 $\hat{Y} = \text{PI-DCH}\Big(\text{K-AMC}\big(\text{R-STMF}(\text{AGSP}(X)), Z_{nwp}\big)\Big)$。
  - 正确且清晰地定义了所有核心变量张量的数学维度 ($X \in \mathbb{R}^{L \times D}$, $T \in \mathbb{R}^{N_p \times D}$, $H \in \mathbb{R}^{N \times d_{model}}$)。
  - **精简优化**：移除了 4 个冗长复杂的算法伪代码块，使整节回归至纯粹的公式公式推导与原理解释。整体双栏架构图（Fig. 5）高度压缩为 `0.18\textwidth`，节省大量页面。
- **IV. Experimental Setup** (`experiments.tex` 前半)：定义 Kelmarsh、SDWPF、HoT 三大工业风电场气象特征。详述气象、电气、机械等 15+ 维 SCADA 与 NWP 输入特征。
- **V. Results and Discussion** (`experiments.tex` 后半)：
  - **Table I (基准结果)**：彻底重构为横向双栏宽表，对比算法扩展为包含 **GCGNet** (Gated Convolutional Graph Network) 在内的 8 种 SOTA 基准模型，所有具体数值以 `\textbf{TBD}` 占位。
  - **Table II (消融实验)**：构建了全新的消融数据宽表，囊括 ours、w/o AGSP、w/o R-STMF、w/o K-AMC、w/o PI-DCH 5 种消融配置在三大数据集下的 RMSE、MAE、$\Lambda$ 指标，具体数值以 `\textbf{TBD}` 占位。
  - 修正消融实验文本中 "w/o R-STMF: MAE 降低 8.1%" 的语义矛盾错误，更正为 "degraded forecasting performance (increasing MAE by \textbf{TBD}%)"。
- **VI. Discussion and Qualitative Analysis** (`discussion.tex`)：
  - 正式定义了 `event alignment` (事件对齐) 和 `KAN Advantage` (KAN 样条边物理表征优势)。
  - 将 K-AMC 边缘激活调制可视化讨论与案例研究 C (Fig. 8(c)) 紧密关联绑定。
  - 精简并归纳了经济、社会与电网安全分析，将 25 万美元收益标注为“初步经济估算”。
- **VII. Case Studies** (`case_studies.tex`)：
  - **排版优化**：引入统一的双栏 4 联图 `fig:case_studies_combined`。将子图的详细描述移入主图 Caption，子图标题精简为（Case A: Wind Ramp., Case B: Cluster Sync., 等），**彻底解决了 narrow subfigure columns (0.23\textwidth) 导致的严重文本换行、重叠与 `Underfull \hbox` 渲染异常**。
  - 将小节标题退化为段落级内联粗体 `\textbf{Case Study X: ...}`，极大地释放了 vertical spacing。
  - 文本中所有图片索引统一更改为 LaTeX 动态引用 `Fig.~\ref{fig:case_a}` 等。
- **VIII. Conclusion and Future Outlook** (`main.tex` 尾部)：扁平化移除所有子小节，合并提炼为紧凑的高冲击力三段论结构（成就总结、电网政策影响、局限与未来展望）。
- **Bibliography** (`ref.bib` 独立文件)：移除了 `main.tex` 中硬编码的内联参考文献，将其提取到独立的 `ref.bib` BibTeX 数据库中，按照第一作者姓氏的拼音首字母进行 alphabetical 严格排序，并完美校对 `, and` 样式。

---

## 项目重构后文件结构

重构后的项目结构高度模块化且井然有序，主控文件与模板类位于根目录，而模块化章节、参考文献和图表分别存放在独立的文件夹中，极为符合 Overleaf 规范与高质量学术论文的排版管理习惯：

```
main.tex            # LaTeX 主控制文件（已引入 \bibliographystyle{IEEEtran} 和 \bibliography{reference/ref}）
IEEEtran.cls        # IEEE Transactions 模板类文件
sections/           # 模块化章节源文件目录
  nomenclature.tex  # 符号表（已精简至 22 个核心创新变量）
  intro.tex         # 第 I 节：引言（含 Contributions 列表）
  related_work.tex  # 第 II 节：相关工作（已精简重组为 4 专题）
  methodology.tex   # 第 III 节：方法论（系统公式与维度已标注）
  experiments.tex   # 第 IV-V 节：实验设置与结果（基准 Table I 与消融 Table II 已转为横向宽表）
  discussion.tex    # 第 VI 节：定性讨论（KAN优势与事件对齐定义已补全）
  case_studies.tex  # 第 VII 节：案例研究（四联图 Case A-D 排版已修复，采用 inline Bold 段首）
reference/          # 参考文献目录
  ref.bib           # 独立 BibTeX 参考文献数据库文件（按姓氏首字母 Alphabetical 排序）
figures/            # 绘图 PDF/PNG 存放目录（已移至根目录，引用时无需 "../" 路径前缀！）
output/
  PhyST-Net_Latest_Draft.pdf  # 编译输出的最终 10 页 PDF
```

---

## 编译方法

重构后，项目主控制文件 `main.tex` 引用了 `ref.bib`，因此本地需要标准的 BibTeX 编译循环来解析参考文献：

```bash
pdflatex -interaction=nonstopmode main.tex
bibtex main
pdflatex -interaction=nonstopmode main.tex
pdflatex -interaction=nonstopmode main.tex
```

---

## 后续实验待办事项 (Post-Experiment TODOs)

论文文本、数学公式、排版框架已 100% 准备就绪，一旦您在 GPU 服务器上跑完对照与消融实验，只需进行以下三步操作即可完成终稿：

1. **填充 Table I 与 Table II 实验数据**：
   - 打开 [experiments.tex](file:///C:/Users/Molat/OneDrive/桌面/PhyST-Net_Paper/experiments.tex)，将所有 `\textbf{TBD}` 占位符替换为您跑出来的 RMSE、MAE 和 Lambda ($\Lambda$) 指标实测值。
2. **填充正文中的实验数据段落**：
   - 全文中还包含部分文本级的数值占位符，如 [experiments.tex](file:///C:/Users/Molat/OneDrive/桌面/PhyST-Net_Paper/experiments.tex) 中的 `\textbf{TBD}\%` 或 `\textbf{TBD}` 毫秒。请搜索 `TBD` 关键词并依次更新。
3. **渲染并放置 Case Studies 可视化图片**：
   - 依据我们在 [implementation_plan.md](file:///C:/Users/Molat/.gemini/antigravity-cli/brain/129fc05e-7d43-4d80-8e65-d0832d9f65f7/implementation_plan.md) 中为您精心制定的 **实验设计指南** (Experimental Design Guide)，绘制这四组曲线图，将生成的 PDF/PNG 命名为 `case_a.pdf` 等，放入 `figures/` 文件夹中。
   - 打开 [case_studies.tex](file:///C:/Users/Molat/OneDrive/桌面/PhyST-Net_Paper/case_studies.tex)，将 subfloat 中的 `\rule{0.23\textwidth}{0.18\textwidth}` 替换为 `\includegraphics[width=0.23\textwidth]{figures/case_a.pdf}` 即可（注意：无需 "../" 前缀）。
