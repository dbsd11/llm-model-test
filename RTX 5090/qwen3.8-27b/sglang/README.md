# Qwen3.8-27B SGLang 部署说明

## 模型信息

### 模型路径（本地）

- **orcarouter/Qwen3.8-27B-Uncensored-NVFP4**:
  - 路径： `./Qwen3.8-27B-Uncensored-NVFP4`
  - 挂载到容器: `/data/MODELS/Qwen3.8-27B`

## 镜像信息

- **SGLang 镜像**: `lmsysorg/sglang:latest`

## 性能指标

- **Token 生成速度**: 60 - 120 token/s

## 部署步骤

### 1. 启动服务

```bash
docker compose up -d
```

### 2. 验证服务

```bash
# 查看日志
docker compose logs -f

# 健康检查
curl http://localhost:58080/health

# 对话测试
curl http://localhost:58080/v1/chat/completions ^
  -H "Content-Type: application/json" ^
  -d "{\"model\": \"qwen3.8-27b\", \"messages\": [{\"role\": \"user\", \"content\": \"你好\"}], \"max_tokens\": 100}"
```

### 3. 停止服务

```bash
docker compose down
```

## Docker Compose 配置

详见 [docker-compose.yml](./docker-compose.yml)

```yaml
services:
  Qwen3.8-27B-SGLang:
    image: lmsysorg/sglang:latest
    container_name: Qwen3.8-27B-SGLang-NVFP4
    ports:
      - "58080:8000"
    entrypoint: sglang serve
    command: [
      "--model-path", "/data/MODELS/Qwen3.8-27B",
      "--served-model-name", "qwen3.8-27b",
      "--host", "0.0.0.0",
      "--port", "8000",
      "--quantization", "modelopt_mixed",
      "--mem-fraction-static", "0.93",
      "--kv-cache-dtype", "fp8_e4m3",
      "--attention-backend", "flashinfer",
      "--cuda-graph-max-bs-decode", "1",
      "--context-length", "65536",
      "--max-running-requests", "4",
      "--speculative-algorithm", "EAGLE",
      "--speculative-num-steps", "3",
      "--speculative-eagle-topk", "1",
      "--speculative-num-draft-tokens", "4",
      "--enable-linear-replayssm-spec",
      "--reasoning-parser", "qwen3",
      "--tool-call-parser", "qwen3_coder",
      "--mamba-radix-cache-strategy", "extra_buffer_lazy",
      "--mamba-ssm-dtype", "bfloat16"
    ]
    volumes:
      - ./Qwen3.8-27B-Uncensored-NVFP4:/data/MODELS/Qwen3.8-27B
    shm_size: 32g
```

## 关键参数说明

### SGLang 参数

- `--quantization modelopt_mixed`: 使用 ModelOpt 混合量化
- `--mem-fraction-static 0.93`: 静态显存分配 93%
- `--context-length 65536`: 上下文长度 64K
- `--max-running-requests 4`: 最大运行请求数
- `--speculative-algorithm EAGLE`: 使用 EAGLE 投机算法
- `--speculative-num-steps 3`: 投机步数
- `--speculative-eagle-topk 1`: EAGLE top-k 值
- `--speculative-num-draft-tokens 4`: draft token 数量
- `--attention-backend flashinfer`: FlashInfer 后端
- `--mamba-radix-cache-strategy extra_buffer_lazy`: Mamba 缓存策略
- `--mamba-ssm-dtype bfloat16`: Mamba SSM 数据类型
- `--cuda-graph-max-bs-decode 1`: CUDA Graph 最大 decode batch size

### 通用配置

- `shm_size 32g`: 共享内存大小
- `ports 58080:8000`: 端口映射
- `device_ids ["0"]`: 使用 GPU 0

## 特点

- 使用 SGLang 框架（非 vLLM）
- 量化方式: modelopt_mixed
- 显存控制: 静态分配 93%
- 投机算法: EAGLE
- 最大并发: 4
