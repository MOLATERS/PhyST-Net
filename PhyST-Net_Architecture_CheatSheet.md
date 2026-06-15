# PhyST-Net 架构设计与公式速查表 (Architecture Cheat Sheet)

## 1. PhyST-Net 整体架构与数据流 (Overall Architecture)

整体架构遵循 `Encoder-Decoder` 范式，但物理约束贯穿始终。

```mermaid
graph TD
    %% Inputs
    subgraph Inputs ["Input Tuple X_t"]
        X["Historical Target (X)<br/>Shape: [B, T, N, Dx]"]
        Eh["Historical ExG (E^h)<br/>Shape: [B, T, Dh]"]
        Ef["Future ExG (E^f)<br/>Shape: [B, H, Df]"]
        A["Graph Topology (A)"]
        MG["Metadata (M_G)"]
    end

    %% Phase I
    X & Eh --> AGSP
    MG -.-> AGSP
    subgraph Phase1 ["Phase I: Continuous Tokenization"]
        AGSP["AGSP<br/>(Adaptive Gaussian-soft-windowed Patching)"]
    end
    AGSP -- "Continuous Tokens Z<br/>[B, N, P, d]" --> STEnc

    %% Phase II
    subgraph Phase2 ["Phase II: Temporal Encoding"]
        STEnc["STEnc (Multi-Head Self-Attention)<br/>+ Global Token (z_global)"]
    end
    STEnc -- "H_T = [R || U]<br/>[B, N, P+1, d]" --> RSTMF

    %% Phase III
    A -.-> RSTMF
    subgraph Phase3 ["Phase III: Graph Adaptation"]
        RSTMF["R-STMF<br/>(Relational Spatio-Temporal Manifold Fusion)"]
    end
    RSTMF -- "Graph-Enhanced State H_G<br/>[B, N, P+1, d]" --> KAMC

    %% Phase IV
    Ef --> KAMC
    subgraph Phase4 ["Phase IV: Exogenous Modulation"]
        KAMC["K-AMC<br/>(KAN-enhanced Adaptive Modulation)"]
    end
    KAMC -- "Conditioned State H_F<br/>[B, N, P+1, d]" --> PIDCH

    %% Phase V
    subgraph Phase5 ["Phase V: Constraint Decoding"]
        PIDCH["PI-DCH<br/>(Physics-Informed Dynamic Capping Head)"]
    end
    
    %% Output
    PIDCH -- "Feasible Forecast Y_hat<br/>[B, H, N, Dy]" --> Output(["Output Y_hat"])

    classDef input fill:#e1f5fe,stroke:#1565c0,stroke-width:2px;
    classDef phase fill:#f3e5f5,stroke:#0288d1,stroke-width:2px;
    class X,Eh,Ef,A,MG input;
    class AGSP,STEnc,RSTMF,KAMC,PIDCH phase;
```

## 2. 局部核心模块拆解与公式

### (1) AGSP (自适应高斯软窗口 Patching)
解决传统 Hard Patching 带来的物理事件边界割裂（边界伪影）问题。

```mermaid
graph LR
    Input["Time Series (X, E^h)"] --> CalcDist["Calculate Distance: (t - μ)²"]
    CalcDist --> Gaussian["Apply Gaussian Weight:<br/>W_i = exp(-dist² / 2σ²)"]
    Gaussian --> Norm["Partition of Unity:<br/>W_i / Σ W_j"]
    Norm --> WeightedSum["Weighted Aggregation"]
    WeightedSum --> Proj["Linear Projection (ψ)"]
    Proj --> AddEmb["+ Node Emb (e_n)<br/>+ Patch Emb (p_i)"]
    AddEmb --> Z["Continuous Token Z"]
    
    classDef block fill:#fff3e0,stroke:#6a1b9a,stroke-width:2px;
    class CalcDist,Gaussian,Norm,WeightedSum,Proj,AddEmb block;
```
*   **核心公式**：
    $$W_i(\tau)=\exp\left(-\frac{(\tau-\mu_i)^2}{2\sigma_i^2+\epsilon}\right)$$
*   **数据流**：$[B, N, T, D] \xrightarrow{\text{Soft Windowing}} [B, N, P, D_{model}]$

### (2) R-STMF (关系型时空流形融合)
解耦宏观气象趋势与局部物理拓扑，避免图卷积的信息混淆。

```mermaid
graph TD
    HT["H_T (Temporal Encoded State)"] --> Split{"Decompose"}
    Split --> R["Macro-Trend (R)<br/>[Global Token]"]
    Split --> U["Local-Variation (U)<br/>[Patch Tokens]"]
    
    A["Raw Adjacency A"] --> MaskR["Top-K Mask (K_s)"]
    A --> MaskU["Top-K Mask (K_t)"]
    
    R & MaskR --> GraphS["Macro Graph Conv"]
    U & MaskU --> GraphT["Micro Graph Conv"]
    
    GraphS --> Fusion(("Gate Fusion (Sigmoid)"))
    GraphT --> Fusion
    Fusion --> HG["Graph-Enhanced State H_G"]
    
    classDef block fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    class Split,MaskR,MaskU,GraphS,GraphT,Fusion block;
```
*   **核心公式** (Top-K 稀疏掩码)：
    $$A_c=\operatorname{Softmax}\left(\operatorname{Mask}(S_c, K_c)\right)$$
*   **物理意义**：强制网络只能在其真实的物理“局部邻居”内传递高频扰动信号，极大降低复杂度 $O(P \cdot K \cdot N)$。

### (3) K-AMC (基于 KAN 的外生变量调制)
将客观可知的未来 NWP 数据，以极高表达力注入模型。

```mermaid
graph LR
    Ef["Future ExG (E^f)"] --> Pool["Align (AvgPool1d)<br/>H -> P"]
    Pool --> KAN["KAN Block<br/>(B-Spline Edge Functions)"]
    KAN --> Split{"Chunk"}
    Split --> Gamma["Scale (γ)"]
    Split --> Beta["Shift (β)"]
    
    HG["H_G"] --> FiLM(("FiLM Modulation"))
    Gamma --> FiLM
    Beta --> FiLM
    FiLM --> Residual["Residual Scaling<br/>+ 0.1 * (Mod - H_G)"]
    Residual --> HF["Conditioned State H_F"]
    
    classDef block fill:#ffebee,stroke:#c62828,stroke-width:2px;
    class Pool,KAN,Split,FiLM,Residual block;
```
*   **核心公式** (KAN 样条函数与残差 FiLM)：
    $$\phi(x) = w_b \operatorname{SiLU}(x) + \sum_{i=1}^{k} c_i B_i(x)$$
    $$H_F=H_G+\lambda\left(\gamma\odot H_G+\beta-H_G\right)$$

### (4) PI-DCH (物理信息动态封顶头)
构建安全边界，解决由于强制 Hard Clip 带来的反向传播梯度断裂问题。

```mermaid
graph TD
    HF["H_F"] --> Linear["Linear Decoder"]
    HF --> GateNet["Gate Network g(·)"]
    Linear --> Ytilde["Unconstrained Pred (Y_tilde)"]
    GateNet --> Sigmoid["Sigmoid (u)"]
    
    NWP["Wind Speed (from E^f)"] --> Cdyn["Dynamic Capacity (C_dyn)<br/>Double-Sigmoid Mask"]
    
    Ytilde & Sigmoid & Cdyn --> SoftCap["Soft Cap:<br/>Y_soft = Y_tilde + u * (C_dyn - Y_tilde)"]
    SoftCap --> Loss["Compute Loss:<br/>L_mae + λ*L_penalty"]
    SoftCap --> HardClip["Hard Clip (Inference):<br/>clip(Y_soft, 0, C_dyn)"]
    HardClip --> Yhat["Final Y_hat"]
    
    classDef block fill:#e0f7fa,stroke:#00695c,stroke-width:2px;
    class Linear,GateNet,Sigmoid,Cdyn,SoftCap,HardClip,Loss block;
```
*   **核心公式** (软逼近与可微罚项)：
    $$\hat{Y}_{soft}=\tilde{Y}+u\odot(C_{dyn}-\tilde{Y})$$
    $$\mathcal{L}_{cap} = \frac{1}{|\mathcal{D}|HN} \sum [\hat{Y}_{soft} - C_{dyn}]^+$$
*   **数据流最终收束**：预测输出维度为 $[B, H, N, D_y]$，且严格遵守物理域知识 $\Omega$ 定义的上下界。