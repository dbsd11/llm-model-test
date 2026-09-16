# Qwen3.8-27B vLLM 部署说明

## 模型信息

### 模型路径（本地）

- **unsloth/Qwen3.8-27B-NVFP4**: 
  - 路径：`./Qwen3.8-27B-NVFP4`
  - 挂载到容器: `/data/MODELS/Qwen3.8-27B`

- **orcarouter/Qwen3.8-27B-Uncensored-NVFP4**:
  - 路径： `./Qwen3.8-27B-Uncensored-NVFP4`
  - 挂载到容器: `/data/MODELS/Qwen3.8-27B`

## 镜像信息

- **vLLM 镜像**: `vllm/vllm-openai:latest`

## 部署步骤

### 方式一：vLLM + NVFP4 量化模型

#### 1. 启动服务

```bash
docker compose -f docker-compose.yml up -d
```

#### 2. 验证服务

```bash
# 查看日志
docker compose -f docker-compose.yml logs -f

# 健康检查
curl http://localhost:58080/health

# 对话测试
curl http://localhost:58080/v1/chat/completions ^
  -H "Content-Type: application/json" ^
  -d "{\"model\": \"qwen3.8-27b\", \"messages\": [{\"role\": \"user\", \"content\": \"你好\"}], \"max_tokens\": 100}"
```

#### 3. 停止服务

```bash
docker compose -f docker-compose.yml down
```

### 方式二：vLLM + Uncensored 模型

#### 1. 启动服务

```bash
docker compose -f docker-compose-vllm-uncensored.yml up -d
```

#### 2. 验证服务

```bash
# 查看日志
docker compose -f docker-compose-vllm-uncensored.yml logs -f

# 健康检查
curl http://localhost:58080/health
```

#### 3. 停止服务

```bash
docker compose -f docker-compose-vllm-uncensored.yml down
```

## Docker Compose 配置

### 配置 1: vLLM + NVFP4 量化模型

详见 [docker-compose.yml](./docker-compose.yml)

```yaml
services:
  Qwen3.8-27B:
    container_name: Qwen3.8-27B-NVFP4
    image: vllm/vllm-openai:latest
    ports:
      - "58080:8000"
    command: [
      "/data/MODELS/Qwen3.8-27B",
      "--quantization", "compressed-tensors",
      "--kv-cache-dtype", "fp8_e4m3",
      "--kv-cache-memory-bytes", "5905580032",
      "--gpu-memory-utilization", "0.94",
      "--max-model-len", "65536",
      "--max-num-batched-tokens", "8192",
      "--speculative-config", "{\"method\":\"mtp\",\"num_speculative_tokens\":3}",
      "--attention-config.backend", "FLASHINFER",
      "--attention-config.flash_attn_version", "2",
      "--enable-prefix-caching",
      "--enable-chunked-prefill",
      "--reasoning-parser", "qwen3",
      "--enable-auto-tool-choice",
      "--tool-call-parser", "qwen3_coder",
      "--async-scheduling",
      "--no-enforce-eager",
      "--compilation-config.mode", "3",
      "--compilation-config.cudagraph_mode", "PIECEWISE",
      "--served-model-name", "qwen3.8-27b",
      "--host", "0.0.0.0",
      "--port", "8000",
      "--mamba-cache-dtype", "bfloat16"
    ]
    volumes:
      - ./Qwen3.8-27B-NVFP4:/data/MODELS/Qwen3.8-27B
    shm_size: 32g
```

**特点**:
- 使用 compressed-tensors 量化（NVFP4）
- KV 缓存 FP8 量化
- KV 缓存内存限制 ~5.5GB

### 配置 2: vLLM + Uncensored 模型

详见 [docker-compose-vllm-uncensored.yml](./docker-compose-vllm-uncensored.yml)

## 关键参数说明

### vLLM 参数

- `--quantization compressed-tensors`: 使用压缩张量量化（NVFP4）
- `--kv-cache-dtype fp8_e4m3`: KV 缓存使用 FP8
- `--kv-cache-memory-bytes 5905580032`: KV 缓存内存限制（~5.5GB）
- `--gpu-memory-utilization 0.94`: GPU 显存利用率 94%
- `--max-model-len 65536`: 最大模型长度 64K
- `--max-num-batched-tokens 8192`: 最大批处理 token 数
- `--speculative-config`: 推测解码配置（MTP，3个token）
- `--attention-config.backend FLASHINFER`: 使用 FlashInfer 后端
- `--enable-prefix-caching`: 启用前缀缓存
- `--enable-chunked-prefill`: 启用分块预填充
- `--reasoning-parser qwen3`: Qwen3 推理解析器
- `--tool-call-parser qwen3_coder`: 工具调用解析器
- `--compilation-config.mode 3`: 编译优化级别
- `--async-scheduling`: 异步调度
- `--no-enforce-eager`: 不强制 eager 模式

### 通用配置

- `shm_size 32g`: 共享内存大小
- `ports 58080:8000`: 端口映射
- `device_ids ["0"]`: 使用 GPU 0

## 命令行启动参考（vLLM）

```bash
vllm serve Qwen3.8-27B-NVFP4 ^
  --served-model-name qwen3.8-27b ^
  --quantization compressed-tensors ^
  --host 0.0.0.0 ^
  --port 58080 ^
  --max-model-len 65536 ^
  --max-num-batched-tokens 4096 ^
  --kv-cache-dtype fp8_e4m3 ^
  --kv-cache-memory-bytes 5905580032 ^
  --gpu-memory-utilization 0.94 ^
  --attention-config.backend FLASHINFER ^
  --attention-config.flash_attn_version 2 ^
  --enable-auto-tool-choice ^
  --tool-call-parser qwen3_xml ^
  --reasoning-parser qwen3 ^
  --speculative-config '{"method":"mtp","num_speculative_tokens":3}' ^
  --no-enforce-eager ^
  --async-scheduling ^
  --compilation-config.mode 3 ^
  --compilation-config.cudagraph_mode PIECEWISE ^
  --enable-prefix-caching ^
  --enable-chunked-prefill ^
  --trust-remote-code ^
  --dtype bfloat16
```

## 性能指标

- **Token 生成速度**: 60 - 180 token/s