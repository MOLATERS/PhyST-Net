# PhyST-Net Paper

> **PhyST-Net: A Physics-Aware Differentiable Spatio-Temporal Transformer Network for Constrained Network Forecasting**

IEEE Transactions 期刊论文 LaTeX 源码与配套组会汇报资料。该项目已完成高度模块化重构与理论公式的严密对齐。

---

## 核心模块与物理意义对应关系

| 缩写 | 全称 | 核心理论与架构创新点 |
|---|---|---|
| **PhyST-Net** | Physics-aware Differentiable Spatio-Temporal Transformer Network | 全流程可微的物理一致性时空特征学习流水线 |
| **AGSP** | Adaptive Gaussian-soft-windowed Patching | **连续 Token 化**：自适应可微高斯软窗分块，充当动态低通滤波器，解决硬边界伪影与高频大气噪声 |
| **Global Token** | Global Information Sink (含 Cross-Attention) | **环境感知**：不仅作为序列级 Memory Bottleneck 提取宏观趋势，更通过专门针对历史外生变量（SCADA/TimeMarks）的 **Cross-Attention**，充当跨模态信息汇聚点 |
| **R-STMF** | Relational Spatio-Temporal Manifold Fusion | **多尺度空间解耦**：利用双分支解耦局域微观波动与宏观气象时空流形，并引入 Top-$K$ 安全距离掩码进行稀疏图路由 |
| **K-AMC** | KAN-enhanced Adaptive Meteorological Conditioning | **外生驱动注入**：基于 Kolmogorov-Arnold 网络的样条边参数化非线性条件调制，采用稳定的残差 FiLM 放缩策略 |
| **PI-DCH** | Physics-Informed Dynamic Capping Head | **物理安全边界**：可微双 Sigmoid 动态封顶头，利用屏障门机制 (barrier gate) 将预测软逼近物理可行域，解决 Hard Clip 导致的梯度断裂 |

---

## 项目文件结构

重构后的项目结构高度模块化且井然有序，主控文件位于根目录，模块化章节、参考文献、图表以及配套文档分别存放在独立的文件夹中：

```
.
├── main.tex                 # LaTeX 主控制文件
├── IEEEtran.cls             # IEEE Transactions 模板类文件
├── README.md                # 本项目说明文档
├── GEMINI.md                # 智能体系统级架构规范与任务上下文
├── AGENTS.md                # 多智能体协作协议配置
├── output/
│   └── PhyST-Net_Latest_Draft.pdf  # 最新编译生成的论文 PDF
├── docs/                    # 📚 核心配套说明文档与演示物料
│   ├── PhyST-Net_PPT_Script.md            # 20分钟组会汇报 PPT 结构大纲与逐字稿 (含核心干货)
│   ├── PhyST-Net_Architecture_CheatSheet.md # 架构设计、局部数据流解析与核心公式速查表 (Mermaid图表)
│   ├── TODO_List.md                       # 历史与未来待办事项清单
│   └── front_matter_revision_plan.md      # 前置修订计划存档
├── sections/                # 📝 模块化论文源码
│   ├── intro.tex            # 第 I 节：引言 (The Forecasting Trilemma)
│   ├── problem_definition.tex # 第 II 节：问题定义 (防泄漏假设与 MAE 通用目标对齐)
│   ├── methodology.tex      # 第 III 节：方法论 (四级核心模块的数学解构，已补充 Global Token Cross-Attention)
│   ├── related_work.tex     # 第 IV 节：相关工作
│   ├── experiments.tex      # 第 V 节：实验设置与结果
│   ├── discussion.tex       # 第 VI 节：定性讨论
│   ├── case_studies.tex     # 第 VII 节：案例研究
│   └── architecture_figure.tex # 架构矢量图配置
├── reference/               # 🔖 参考文献数据库
│   └── reference.tex        # 独立化分离的参考文献列表
├── reference_pdfs/          # 📄 核心参考论文原始 PDF 存档
└── figures/                 # 🖼️ 绘图 PDF/PNG 存放目录
```

---

## 编译方法

本项目支持跨平台编译。请确保系统已安装完整的 TeX Live 环境。推荐在本地使用以下标准编译序列解析内部的交叉引用：

```bash
pdflatex -interaction=nonstopmode main.tex
pdflatex -interaction=nonstopmode main.tex
pdflatex -interaction=nonstopmode main.tex
```

*(如果本地缺乏 LaTeX 环境，依据 `GEMINI.md` 的设定，可利用自动化脚本在远端 3090 服务器 [100.64.169.118] 完成静默编译并取回 PDF。)*

---

## 下一步待办事项 (Next Steps)

1. **图表绘制**：依据 `docs/PhyST-Net_Architecture_CheatSheet.md` 中梳理的数据流与模块结构，绘制精美的全幅矢量主架构图（替换占位符）及局部微观模块图。
2. **实验推进**：在 GPU 服务器上（如 5090）基于真实风电场/交通数据集完成 Benchmark 对比与消融实验。
3. **数据填充**：实验完成后，将 `sections/experiments.tex` 表格中的 `\textbf{TBD}` 占位符替换为真实的 RMSE、MAE 和 $\Lambda$（电网合规率）指标。
4. **可视化填充**：生成四组验证假设的案例研究曲线图（Case A-D），放入 `figures/` 文件夹中并解除代码注释。