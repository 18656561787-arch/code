```mermaid
graph LR
    Input(["输入 bev_feat<br/>(bs, 256, 85, 50)"]) 

    subgraph Block1 [第一阶段: 高维增强]
        B1_1["DWConvBlock<br/>(dim: 768, k: 3)"]
        B1_2["DownsampleBlock<br/>(s: 1, out: 512)"]
    end

    subgraph Block2 [第二阶段: 特征压缩]
        B2_1["DWConvBlock<br/>(dim: 512, k: 3)"]
        B2_2["DownsampleBlock<br/>(s: 1, out: 256)"]
    end

    subgraph Block3 [第三阶段: 空间下采样]
        B3_1["DWConvBlock<br/>(dim: 256, k: 3)"]
        B3_2["DownsampleBlock<br/>(s: 2, out: 256)"]
    end

    subgraph Block4 [第四阶段: 最终平滑]
        B4_1["DWConvBlock<br/>(dim: 256, k: 3)"]
    end

    Input --> Block1
    Block1 --> Block2
    Block2 --> Block3
    Block3 --> Block4
    Block4 --> Output(["输出 conv_feat<br/>(bs, 256, 43, 25)"])

    style Block1 fill:#f0f7ff,stroke:#005cc5
    style Block2 fill:#fff7e6,stroke:#d46b08
    style Block3 fill:#f6ffed,stroke:#389e0d
    style Block4 fill:#f9f0ff,stroke:#722ed1
