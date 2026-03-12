```mermaid
graph TD
    Start([输入 bev_feat: bs, 85, 50, 256]) --> Permute[维度置换: bs, 256, 85, 50]
    
    subgraph CNN_Encoder [特征提取层]
        Permute --> Conv[bev_encoder: 经过 4 组 DWConv + Downsample]
        Conv --> ConvOut[卷积特征: bs, 256, 43, 25]
    end

    ConvOut --> Split{并行计算}

    subgraph Branch_Learnable [支路 1: 可学习位置编码]
        Split --> L1[取出 Embedding 权重]
        L1 --> L2[repeat 为 Batch 维度]
        L2 --> LearnPos[learnable_pos: bs, 1075, 256]
    end

    subgraph Branch_Coord [支路 2: 真实坐标位置编码]
        Split --> C1[bev_pos_hw 插值对齐到 43x25]
        C1 --> C2[pos2posemb2d: 正余弦变换]
        C2 --> C3[coord_pos_encoder: MLP 映射]
        C3 --> CoordPos[coord_pos: bs, 1075, 256]
    end

    LearnPos --> Fusion[Softmax 权重融合]
    CoordPos --> Fusion
    Fusion --> FusedPos[fused_pos_emb: bs, 1075, 256]

    ConvOut --> Reshape[Flatten 展平: bs, 1075, 256]
    
    Reshape --> Add[特征 + 位置编码]
    FusedPos --> Add
    
    Add --> Final[OutputAdapter + LayerNorm]
    Final --> End([输出: fused_feat, fused_pos, mask])
