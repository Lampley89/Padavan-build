graph TD
    %% 定义样式
    classDef input fill:#f9f,stroke:#333,stroke-width:2px,color:black;
    classDef upstream fill:#e1f5fe,stroke:#0277bd,stroke-width:2px,color:black;
    classDef midstream fill:#fff9c4,stroke:#fbc02d,stroke-width:2px,color:black;
    classDef downstream fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px,color:black;
    classDef visual fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:black;

    %% --- 输入阶段 ---
    Start((原始测序数据<br>FASTQ / BCL)):::input

    %% --- 上游分析 ---
    subgraph Upstream [上游分析: 数据预处理]
        direction TB
        Align[比对与定量<br>CellRanger / STARsolo]:::upstream
        Matrix[生成原始表达矩阵<br>Raw Count Matrix]:::upstream
    end

    %% --- 中游分析 ---
    subgraph Midstream [中游分析: 核心处理]
        direction TB
        QC[质量控制 QC<br>去除低质量细胞/高线粒体/红细胞]:::midstream
        Doublet[双细胞/多细胞去除<br>DoubletFinder / Scrublet]:::midstream
        Norm[数据标准化与归一化<br>LogNormalize / SCTransform]:::midstream
        HVG[高变基因选择<br>Feature Selection]:::midstream
        PCA[线性降维<br>PCA]:::midstream
        Batch[去除批次效应<br>Harmony / CCA / BBKNN]:::midstream
        Cluster[非线性降维与聚类<br>UMAP / t-SNE / Louvain / Leiden]:::midstream
        Annot[细胞类型注释<br>Manual / SingleR / CellTypist]:::midstream
    end

    %% --- 下游分析 ---
    subgraph Downstream [下游与高级分析: 生物学挖掘]
        direction TB
        DEG[差异基因分析<br>FindMarkers / Wilcoxon]:::downstream
        Traj[拟时序分析 / RNA速率<br>Monocle3 / Slingshot / scVelo]:::downstream
        CellChat[细胞通讯分析<br>CellChat / CellPhoneDB]:::downstream
        Enrich[功能富集分析<br>GO / KEGG / GSEA / GSVA]:::downstream
        Regulon[转录因子调控网络<br>SCENIC / pySCENIC]:::downstream
        CNV[拷贝数变异推断<br>inferCNV]:::downstream
    end

    %% --- 连接关系 ---
    Start --> Align
    Align --> Matrix
    Matrix --> QC
    QC --> Doublet
    Doublet --> Norm
    Norm --> HVG
    HVG --> PCA
    PCA --> Batch
    Batch --> Cluster
    Cluster --> Annot

    %% 注释后的分支
    Annot --> DEG
    Annot --> Traj
    Annot --> CellChat
    Annot --> Regulon
    Annot --> CNV
    DEG --> Enrich

    %% 可视化输出
    Visual[可视化展示<br>DimPlot / FeaturePlot / DotPlot / VlnPlot]:::visual
    Cluster -.-> Visual
    Annot -.-> Visual
    Traj -.-> Visual
