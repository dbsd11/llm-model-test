# LLM 模型部署文档

## 目录结构

```

RTX 5090/                              # 显卡类型
└── qwen3.8-27b/                       # 模型
    ├── llama/                         # llama.cpp 部署
    │   └── README.md                  # 部署说明 + 命令行配置
    ├── vllm/                          # vLLM 部署
    │   ├── README.md
    │   ├── docker-compose.yml
    │   └── docker-compose-vllm-uncensored.yml
    ├── sglang/                        # SGLang 部署
    │   ├── README.md
    │   └── docker-compose.yml
    └── ninfer/                        # nInfer 部署
        ├── README.md
        ├── docker-compose-infer-v2.yml
        └── docker-compose-infer-v3.yml
```

## 快速开始

### llama.cpp
- 模型: Qwen3.8-27B-GGUF-unsloth (Q4_K_M) / qwen38-uncensored
- 端口: 58080
- 方式: 命令行启动
- 文档: [llama.cpp 部署说明](./RTX%205090/qwen3.8-27b/llama/README.md)

### vLLM
- 模型: Qwen3.8-27B-NVFP4 (compressed-tensors)
- 端口: 58080
- 方式: Docker Compose
- 文档: [vLLM 部署说明](./RTX%205090/qwen3.8-27b/vllm/README.md)

### SGLang
- 模型: Qwen3.8-27B-Uncensored-NVFP4 (modelopt_mixed)
- 端口: 58080
- 方式: Docker Compose
- 文档: [SGLang 部署说明](./RTX%205090/qwen3.8-27b/sglang/README.md)

### nInfer
- 模型: Qwen3.8-27B-NVFP4-NInfer (v2/v3)
- 端口: 58080
- 方式: Docker Compose
- 文档: [nInfer 部署说明](./RTX%205090/qwen3.8-27b/ninfer/README.md)

## 通用信息

### 显卡
- RTX 5090

### 服务端口
- 统一使用: 58080

### 模型路径
所有模型均为本地路径，相对于各自配置目录
