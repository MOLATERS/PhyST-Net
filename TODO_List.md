# 风电预测论文 PhyST-Net 后续实验与数据填充 TODO 清单

本清单详细梳理了您在 GPU 实验跑完之后，需要手动填充至 LaTeX 源码中的所有关键数据、消融对照表指标以及案例可视化图片文件。

---

## 1. 填充 Table I（基准对比实验）数据
- **源码位置**：[experiments.tex](file:///C:/Users/Molat/OneDrive/桌面/PhyST-Net_Paper/sections/experiments.tex#L70)
- **待办事项**：
  - 将 8 个对比基准模型（GBDT、DLinear、Baseline Transformer、Informer、Autoformer、FEDformer、PatchTST、iTransformer）以及时空图基准 **GCGNet** 和我们的 **PhyST-Net (Ours)** 在三大数据集（**Kelmarsh**、**SDWPF**、**HoT**）下的超短期预测实测指标填入表格：
    - `RMSE` 误差指标
    - `MAE` 误差指标
    - `Lambda ($\Lambda$)` 电网合规率百分比（例如：96.02\%）
  - 替换所有的 `\textbf{TBD}` 占位符。

## 2. 填充 Table II（消融实验对照）数据
- **源码位置**：[experiments.tex](file:///C:/Users/Molat/OneDrive/桌面/PhyST-Net_Paper/sections/experiments.tex#L115)
- **待办事项**：
  - 将 5 种不同的消融控制配置在三大数据集下的 RMSE、MAE、合规率填入消融对照表，定量评估四大创新的增量学术价值：
    - `PhyST-Net (Ours)`（全功能模型）
    - `w/o AGSP`（移除自适应高斯软窗，改用硬分块，对比时域分词连续性作用）
    - `w/o R-STMF`（移除多尺度时空流形融合，改用静态距离图，对比近场尾流与天气锋解耦）
    - `w/o K-AMC`（移除 KAN 样条调制，改用标准 MLP，对比非线性条件注入优势）
    - `w/o PI-DCH`（移除可微 Sigmoid 裁剪，改用未约束线性输出，对比物理可行域刚性拦截）
  - 替换所有的 `\textbf{TBD}` 占位符。

## 3. 替换正文中的数值与比例占位符
- **源码位置**：[experiments.tex](file:///C:/Users/Molat/OneDrive/桌面/PhyST-Net_Paper/sections/experiments.tex) 和 [discussion.tex](file:///C:/Users/Molat/OneDrive/桌面/PhyST-Net_Paper/sections/discussion.tex)
- **待办事项**：
  - 在正文中全局搜索 `TBD` 关键字，根据实测结果更新文本描述占位符。
  - 特别注意替换以下定量描述：
    - [experiments.tex](file:///C:/Users/Molat/OneDrive/桌面/PhyST-Net_Paper/sections/experiments.tex#L54)：我们在三数据集上实现的 state-of-the-art 合规率具体百分比。
    - [experiments.tex](file:///C:/Users/Molat/OneDrive/桌面/PhyST-Net_Paper/sections/experiments.tex#L88)：K-AMC 样条边计算相比传统 MLP 增加的参数和耗时比例 `\textbf{TBD}\%`，以及 PhyST-Net 单步推理的平均延迟 `\textbf{TBD}` 毫秒。
    - [experiments.tex](file:///C:/Users/Molat/OneDrive/桌面/PhyST-Net_Paper/sections/experiments.tex#L94-L98)：消融配置移除时，三大指标下降的具体实测比例（如：RMSE 增加 `\textbf{TBD}\%`）。

## 4. 绘制极端天气案例分析图并注入源码
- **图像存储目录**：`figures/`（建议保存为无损 `.pdf` 或高清晰度 `.png` 格式，如 `case_a.pdf` 等）
- **实验设计与作图要求**（请参考 `implementation_plan.md` 中的详细实验设计方案）：
  - **Case A（HoT 风斜坡刚性截断）**：绘制时间 vs 预测功率。展示 Backbone Transformer 在大风斜坡下的机械过载预测（超出 1.0 容量）vs PhyST-Net PI-DCH 刚性锁死在 1.0 的光滑对比曲线。
  - **Case B（SDWPF 超大集群气动延时捕捉）**：提取并绘制 R-STMF 空间注意力流动热力图，展示天气前锋在上游和下游风机之间行进时呈现的相位延迟推移。
  - **Case C（Kelmarsh 地形阴影下 NWP 条件纠偏）**：绘制实测 NWP 风速 vs 纠偏权重曲线。展示 K-AMC 样条函数在地形遮蔽区间内自动 learn 出来的 piecewise non-linear 扣减修正过程。
  - **Case D（SDWPF 低风速 persist 负功率规避）**：绘制时间 vs 预测功率。展示传统 MSE Backbone 在切入风速下产生的无物理逻辑负数波动曲线（低至 -2%）vs PI-DCH 完美归零的水平平直直线对比。
- **源码替换**：
  - 打开 [case_studies.tex](file:///C:/Users/Molat/OneDrive/桌面/PhyST-Net_Paper/sections/case_studies.tex#L7-L10)。
  - 将 4 联子图环境中的黑框占位代码 `\rule{0.23\textwidth}{0.18\textwidth}` 依次替换为实际引图代码（**注意：重构后图片文件夹与源文件同在根目录下，因此引用路径无需 `../` 前缀**）：
    - `\includegraphics[width=0.23\textwidth]{figures/case_a.pdf}`
    - `\includegraphics[width=0.23\textwidth]{figures/case_b.pdf}`
    - `\includegraphics[width=0.23\textwidth]{figures/case_c.pdf}`
    - `\includegraphics[width=0.23\textwidth]{figures/case_d.pdf}`

## 5. 编译与终稿审查
- **待办事项**：
  - 注入数据与图表后，在本地项目根目录下执行标准的编译循环：
    ```bash
    pdflatex -interaction=nonstopmode main.tex
    bibtex main
    pdflatex -interaction=nonstopmode main.tex
    pdflatex -interaction=nonstopmode main.tex
    ```
  - 确认所有交叉引用和图片均已解析，控制编译 PDF 总页数恰好卡在 **10页** 限制内，没有行溢出（Overfull hbox）或文本折叠警告。
