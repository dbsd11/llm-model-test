# Qwen3.8-27B nInfer 部署说明

## 模型信息

### 模型路径（本地）

- **v2 版本**: Ostfralla/Qwen3.8-27B-NVFP4-NInfer
  - 路径 `./Qwen3.8-27B-NVFP4-NInfer-v2`
  - 模型文件: `qwen3_8_27b_nvfp4.ninfer`
  - 挂载到容器: `/data/MODELS/Qwen3.8-27B`

- **v3 版本**: neroued/Qwen3.8-27B-nvfp4-NInfer
  - 路径：`./Qwen3.8-27B-nvfp4-NInfer-v3`
  - 模型文件: `qwen3_8_27b_nvfp4.ninfer`
  - 挂载到容器: `/data/MODELS/Qwen3.8-27B`

## 镜像信息

- **v2 镜像**: `ninfer-v2:local`
  - 构建来源: https://github.com/koloved/ninfer (master 分支)

- **v3 镜像**: `ninfer-v3:local`
  - 构建来源: https://github.com/Neroued/ninfer

## 部署步骤

### 方式一：使用 v2 版本

#### 1. 启动服务

```bash
docker compose -f docker-compose-infer-v2.yaml up -d
```

#### 2. 验证服务

```bash
# 查看日志
docker compose -f docker-compose-infer-v2.yaml logs -f

# 健康检查
curl http://localhost:58080/health

# 对话测试
curl http://localhost:58080/v1/chat/completions ^
  -H "Content-Type: application/json" ^
  -d "{\"model\": \"qwen3.8-27b\", \"messages\": [{\"role\": \"user\", \"content\": \"你好\"}], \"max_tokens\": 100}"
```

#### 3. 停止服务

```bash
docker compose -f docker-compose-infer-v2.yaml down
```

### 方式二：使用 v3 版本

#### 1. 启动服务

```bash
docker compose -f docker-compose-infer-v3.yaml up -d
```

#### 2. 验证服务

```bash
# 查看日志
docker compose -f docker-compose-infer-v3.yaml logs -f

# 健康检查
curl http://localhost:58080/health
```

#### 3. 停止服务

```bash
docker compose -f docker-compose-infer-v3.yaml down
```

## Docker Compose 配置

### v2 版本配置

详见 [docker-compose-infer-v2.yml](./docker-compose-infer-v2.yml)

### v3 版本配置

详见 [docker-compose-infer-v3.yml](./docker-compose-infer-v3.yml)

## 关键参数

- `ninfer-serve`: nInfer 服务启动命令
- `--kv-dtype fp8`: KV 缓存使用 FP8 数据类型
- `--max-context 131072`: 最大上下文长度 128K
- `--max-concurrency 4`: 最大并发数 4
- `--prefill-chunk 2048`: 预填充块大小
- `--host-state-slots 16`: 主机状态槽位数
- `--host-kv-mib 0`: 主机 KV 缓存大小（0表示不使用主机缓存）
- `--spec mtp`: 推测解码使用 MTP
- `--draft-tokens 4`: 推测解码 draft token 数量
- `--lm-head-draft`: 启用 LM head draft
- `--preserve-thinking`: 保留思考过程
- `--vision`: 启用视觉支持
- `shm_size 32g`: 共享内存大小

## 性能指标

- **Token 生成速度**: 150 - 450 token/s
