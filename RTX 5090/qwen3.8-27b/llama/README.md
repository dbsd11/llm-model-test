# Qwen3.8-27B llama.cpp 部署说明

## 模型信息

### 模型路径（本地）

- **Qwen3.8-27B-GGUF-unsloth**: `Qwen3.8-27B-GGUF-unsloth/Qwen3.8-27B-UD-Q4_K_M.gguf`
  - MMProj: `Qwen3.8-27B-GGUF-unsloth/mmproj-BF16.gguf`
  
- **qwen38-uncensored**: `qwen38-uncensored/orcarouter_Qwen3.8-27B-Uncensored-Q4_K_M.gguf`
  - MMProj: `qwen38-uncensored/mmproj-orcarouter_Qwen3.8-27B-Uncensored-bf16.gguf`

## 部署步骤

### 1. 启动服务

当前使用的启动命令（startup.bat 第 10 行）：

```bash
llama serve -m qwen38-uncensored\orcarouter_Qwen3.8-27B-Uncensored-Q4_K_M.gguf --host 0.0.0.0 --port 58080 --spec-type draft-mtp --spec-draft-p-min 0.6 --gpu-layers-draft all --spec-draft-n-max 3 --alias qwen3.8-27b -ngl all --cache-type-k q4_0 --cache-type-v q4_0 -b 2048 -ub 32 -fa on --cont-batching --jinja --image-min-tokens 1024 --metrics -c 50000 -np 1 -t 16 -tb 16
```

### 2. 验证服务

```bash
# 健康检查
curl http://localhost:58080/health

# 对话测试
curl http://localhost:58080/v1/chat/completions ^
  -H "Content-Type: application/json" ^
  -d "{\"model\": \"qwen3.8-27b\", \"messages\": [{\"role\": \"user\", \"content\": \"你好\"}], \"max_tokens\": 100}"
```

## 所有启动命令（startup.bat）

### 命令 1: Unsloth 模型 + reasoning-preserve + DRY 采样

```bash
llama serve -m Qwen3.8-27B-GGUF-unsloth\Qwen3.8-27B-UD-Q4_K_M.gguf ^
  --mmproj Qwen3.8-27B-GGUF-unsloth\mmproj-BF16.gguf ^
  --host 0.0.0.0 --port 58080 ^
  --spec-type draft-mtp --spec-draft-p-min 0.8 --spec-draft-n-max 3 ^
  --gpu-layers-draft all ^
  --alias qwen3.8-27b -ngl all ^
  --cache-type-k q4_0 --cache-type-v q4_0 ^
  -b 2048 -ub 32 -fa on --cont-batching --jinja ^
  --reasoning-preserve ^
  --image-min-tokens 1024 ^
  --dry-multiplier 0.8 --dry-base 1.75 --dry-allowed-length 2 ^
  --metrics ^
  -c 393216 -np 3
```

**特点**: 
- 使用 DRY 采样器控制重复
- 保留推理思维链
- 上下文 384K，3 并发

### 命令 2: Unsloth 模型 + 大上下文 512K

```bash
llama serve -m Qwen3.8-27B-GGUF-unsloth\Qwen3.8-27B-UD-Q4_K_M.gguf ^
  --mmproj Qwen3.8-27B-GGUF-unsloth\mmproj-BF16.gguf ^
  --host 0.0.0.0 --port 58080 ^
  --spec-type draft-mtp --spec-draft-p-min 0.6 --spec-draft-n-max 3 ^
  --gpu-layers-draft all ^
  --alias qwen3.8-27b -ngl all ^
  --cache-type-k q4_0 --cache-type-v q4_0 ^
  -b 2048 -ub 32 -fa on --cont-batching --jinja ^
  --image-min-tokens 1024 --metrics ^
  -c 524288 -np 4 -t 16
```

**特点**:
- 上下文 512K（524288）
- 4 并发，16 线程

### 命令 3: Unsloth 模型 + IQ4 缓存

```bash
llama serve -m Qwen3.8-27B-GGUF-unsloth\Qwen3.8-27B-UD-Q4_K_M.gguf ^
  --mmproj Qwen3.8-27B-GGUF-unsloth\mmproj-BF16.gguf ^
  --host 0.0.0.0 --port 58080 ^
  --spec-type draft-mtp --spec-draft-p-min 0.6 --spec-draft-n-max 3 ^
  --gpu-layers-draft all ^
  --alias qwen3.8-27b -ngl all ^
  --cache-type-k iq4_nl --cache-type-v iq4_nl ^
  -b 2048 -ub 32 -fa on --cont-batching --jinja ^
  --image-min-tokens 1024 --metrics ^
  -c 524288 -np 4 -t 16
```

**特点**:
- 使用 IQ4_NL 量化缓存（更省显存）
- 上下文 512K

### 命令 4: Uncensored 模型 + MMProj + 大 batch

```bash
llama serve -m qwen38-uncensored\orcarouter_Qwen3.8-27B-Uncensored-Q4_K_M.gguf ^
  --mmproj qwen38-uncensored\mmproj-orcarouter_Qwen3.8-27B-Uncensored-bf16.gguf ^
  --host 0.0.0.0 --port 58080 ^
  --spec-type draft-mtp --spec-draft-p-min 0.6 --spec-draft-n-max 3 ^
  --gpu-layers-draft all ^
  --alias qwen3.8-27b -ngl all ^
  --cache-type-k q4_0 --cache-type-v q4_0 ^
  -b 2048 -ub 64 -fa on --cont-batching --jinja ^
  --image-min-tokens 1024 --metrics ^
  -c 524288 -np 4 -t 16
```

**特点**:
- 使用 uncensored 无审查模型
- 包含多模态投影文件（MMProj）
- 批次大小 64（-ub 64）

### 命令 5: Uncensored 模型（当前使用）✅

```bash
llama serve -m qwen38-uncensored\orcarouter_Qwen3.8-27B-Uncensored-Q4_K_M.gguf ^
  --host 0.0.0.0 --port 58080 ^
  --spec-type draft-mtp --spec-draft-p-min 0.6 --spec-draft-n-max 3 ^
  --gpu-layers-draft all ^
  --alias qwen3.8-27b -ngl all ^
  --cache-type-k q4_0 --cache-type-v q4_0 ^
  -b 2048 -ub 32 -fa on --cont-batching --jinja ^
  --image-min-tokens 1024 --metrics ^
  -c 50000 -np 1 -t 16 -tb 16
```

**特点**:
- 当前使用的配置
- 上下文 50K
- 1 并发，16 线程，16 线程批次
- 不使用 MMProj（纯文本）

## 关键参数说明

### 基础参数
- `-m`: 模型文件路径
- `--mmproj`: 多模态投影文件（视觉支持）
- `--host 0.0.0.0`: 监听所有接口
- `--port 58080`: 服务端口
- `--alias qwen3.8-27b`: 模型别名

### GPU 相关
- `-ngl all`: 所有层加载到 GPU
- `--gpu-layers-draft all`: 推测解码层也加载到 GPU
- `-fa on`: 开启 Flash Attention

### 推测解码
- `--spec-type draft-mtp`: 使用 MTP 推测解码
- `--spec-draft-p-min 0.6`: 推测最小概率阈值
- `--spec-draft-n-max 3`: 最大推测 token 数

### 缓存
- `--cache-type-k q4_0`: K 缓存量化
- `--cache-type-v q4_0`: V 缓存量化
- `--cache-type-k iq4_nl`: 使用 IQ4_NL 量化（命令 3）

### 批处理
- `-b 2048`: 批次大小
- `-ub 32` 或 `-ub 64`: micro-batch 大小
- `--cont-batching`: 连续批处理

### 上下文
- `-c 50000`: 上下文大小 50K（当前）
- `-c 393216`: 上下文 384K（命令 1）
- `-c 524288`: 上下文 512K（命令 2、3、4）

### 并发和线程
- `-np 1`: 并行请求数（当前）
- `-np 3` 或 `-np 4`: 3-4 并发
- `-t 16`: 线程数
- `-tb 16`: 线程批次

### 其他功能
- `--jinja`: 使用 Jinja 模板
- `--image-min-tokens 1024`: 图像最小 token 数
- `--metrics`: 启用指标
- `--reasoning-preserve`: 保留推理思维链（命令 1）
- `--dry-multiplier 0.8`: DRY 采样乘数（命令 1）
- `--dry-base 1.75`: DRY 采样基数（命令 1）
- `--dry-allowed-length 2`: DRY 允许长度（命令 1）

## 性能指标

- **Token 生成速度**: 60 - 100 token/s