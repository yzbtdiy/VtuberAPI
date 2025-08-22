# VTuber API - Rust 实现

基于 Rust 和 Rig AI 框架构建的高性能 VTuber 弹幕处理系统。

## 🚀 最新更新 (v0.2.0)

### 新增功能
- **多提供商嵌入向量系统**: 支持 `rust_hash`, `openai`, `dmeta-embedding-zh` 三种嵌入提供商
- **纯 WebSocket 实现**: 移除 axum 依赖，使用 tokio-tungstenite 提供更轻量的实现
- **灵活的 TTS 配置**: 支持 OpenAI TTS 和 IndexTTS 两种语音合成服务
- **智能向量存储**: Qdrant 集成支持多维度向量和提供商标记

详细更新内容请查看 [EMBEDDING_PROVIDERS_UPDATE.md](EMBEDDING_PROVIDERS_UPDATE.md)

## 项目概述

这是一个高性能的 VTuber 弹幕处理系统的 Rust 实现，使用最新的 Rig 0.18.2 框架进行 AI 操作。它提供实时 WebSocket 通信、基于签名的身份验证、3. **启动 IndexTTS 服务**：
   确保 IndexTTS 服务在指定的地址运行。

4. **测试 IndexTTS**：
   使用提供的测试脚本：
   ```bash
   python test_indextts.py
   ```

## 嵌入向量 (Embedding) 配置

VTuber API 支持多种嵌入向量提供商：

### 支持的嵌入提供商

#### 1. OpenAI Embeddings
使用 OpenAI 的 text-embedding-3-small 模型生成 1536 维嵌入向量。

#### 2. Dmeta Embedding ZH
本地部署的中文嵌入模型，生成 768 维嵌入向量，特别适合中文文本处理。

#### 3. Rust Hash (默认)
基于哈希的占位符方法，生成 1536 维确定性嵌入向量，用于开发和测试。

### 配置嵌入提供商

在 `config.json` 中设置 `embedding_provider`：

```json
{
  "server": {
    "embedding_provider": "openai"  // 可选: "openai", "dmeta-embedding-zh", "rust_hash"
  },
  "openai": {
    "embedding_model": "text-embedding-3-small"
  },
  "dmeta_embedding": {
    "url": "http://localhost:8000",
    "model": "dmeta-embedding-zh"
  }
}
```

### Dmeta Embedding ZH API

Dmeta Embedding ZH 使用标准的嵌入 API 格式：

```bash
curl -X POST 'http://localhost:8000/v1/embeddings' \
  -H 'Content-Type: application/json' \
  -d '{
    "input": "测试文本",
    "model": "dmeta-embedding-zh"
  }'
```

响应格式：
```json
{
  "object": "list",
  "data": [
    {
      "object": "embedding",
      "embedding": [0.015861066058278084, ...],
      "index": 0
    }
  ],
  "model": "dmeta-embedding-zh",
  "usage": {
    "prompt_tokens": 8,
    "total_tokens": 8
  }
}
```和文本转语音功能的智能弹幕处理。该系统设计用于与各种 AI API 端点配合工作，并支持不同 AI 提供商的灵活配置。

## 功能特性

- **纯 WebSocket 架构**：基于 tokio-tungstenite 的轻量级 WebSocket 服务器，无需 HTTP 框架
- **Rig AI 框架 0.18.2**：利用最新的 Rig 版本进行智能多代理处理
- **灵活的 AI API 支持**：可配置的基础 URL 支持多种 AI 端点
- **长期记忆系统**：基于 Qdrant 向量数据库的智能记忆和上下文检索
- **强制身份验证**：基于 HMAC-SHA256 签名的身份验证
- **意图分析**：自动分类弹幕内容
- **多模态响应**：文本、音频和图像生成
- **实时进度更新**：处理过程中的实时状态报告
- **高性能**：基于 Rust 2024 edition 构建，保证速度和可靠性
- **JSON 配置管理**：结构化的 config.json 配置文件，支持复杂配置
- **图像 URL 响应**：返回图像 URL 而非二进制数据，提高兼容性
- **最小化依赖**：移除了 axum、tower 等 HTTP 框架依赖，使用纯 WebSocket 实现

## 系统架构

### 核心组件

1. **WebSocket 服务器** (`src/api/websocket_server.rs`)：基于 tokio-tungstenite 的纯 WebSocket 服务器
2. **身份验证服务** (`src/auth/mod.rs`)：HMAC-SHA256 签名验证
3. **AI 代理** (`src/agents/mod.rs`)：基于 Rig 的意图分析和响应生成
4. **工具集** (`src/tools/mod.rs`)：图像生成和 TTS 集成
5. **工作流引擎** (`src/workflows/mod.rs`)：编排完整的处理流水线
6. **客户端管理器** (`src/services/mod.rs`)：管理 WebSocket 连接和身份验证
7. **长期记忆系统** (`src/memory/mod.rs`)：基于 Qdrant 的向量存储和语义搜索

### AI 处理流水线

1. **意图分析**：使用 Rig 代理分类弹幕意图
2. **响应生成**：针对不同内容类型的专门代理
3. **图像生成**：AI 驱动的绘画功能用于艺术请求
4. **音频合成**：文本转语音功能用于语音响应
5. **进度报告**：整个处理过程的实时更新

## 快速开始

### 系统要求

- Rust 1.70+ 和 Cargo (推荐使用 2024 edition)
- 兼容 OpenAI API 格式的 AI API 端点
- AI 服务的 API 密钥
- （可选）Qdrant 向量数据库用于长期记忆功能
- （可选）用于增强 RAG 功能的向量数据库

### 安装步骤

```bash
# 克隆仓库
git clone <repository-url>
cd VtuberAPI

# 构建项目
cargo build

# 复制配置文件
cp config.example.json config.json
# 编辑 config.json 文件配置您的 API 设置
```

### 配置说明

编辑 `config.json` 文件进行配置：

```json
{
  "server": {
    "host": "localhost",
    "port": 8000,
    "tts_provider": "openai",
    "embedding_provider": "openai"
  },
  "auth": {
    "secret_key": "your_secret_key_here_change_in_production",
    "valid_api_keys": [
      "your-api-key-1",
      "your-api-key-2"
    ],
    "timestamp_tolerance": 300
  },
  "openai": {
    "api_key": "your_openai_api_key_here",
    "base_url": "https://api.openai.com/v1",
    "model": "gpt-3.5-turbo",
    "embedding_model": "text-embedding-3-small",
    "tts_model": "tts-1",
    "tts_voice": "alloy"
  },
  "indextts": {
    "url": "http://localhost:11996",
    "model": "tts-1",
    "voice": "jay_klee"
  },
  "dmeta_embedding": {
    "url": "http://localhost:8000",
    "model": "dmeta-embedding-zh"
  },
  "processing": {
    "max_danmaku_length": 100,
    "response_timeout": 30,
    "max_execution_time": 120
  },
  "long_term_memory": {
    "enabled": true,
    "qdrant": {
      "url": "http://localhost:6334",
      "collection_name": "vtuber_memory",
      "vector_size": 1536,
      "distance": "Cosine"
    },
    "context": {
      "max_context_length": 10,
      "similarity_threshold": 0.7,
      "memory_retention_days": 30
    }
  },
  "logging": {
    "rust_log": "info",
    "otel_sdk_disabled": true,
    "crewai_telemetry_disabled": true
  }
}
```

**注意**：此系统使用结构化的 JSON 配置文件而非环境变量，以实现更可靠和灵活的配置管理。

## TTS (文本转语音) 配置

VTuber API 支持两种 TTS 服务：

### OpenAI TTS
默认使用 OpenAI 的文本转语音服务，支持多种语音模型。

### IndexTTS
IndexTTS 是一个本地部署的 TTS 服务，支持更多语音选择和更快的响应速度。

#### 配置 IndexTTS

1. **启用 IndexTTS**：
   在 `config.json` 中设置：
   ```json
   {
     "server": {
       "tts_provider": "indextts"
     },
     "indextts": {
       "url": "http://localhost:11996",
       "model": "tts-1",
       "voice": "jay_klee"
     }
   }
   ```

2. **切换回 OpenAI TTS**：
   ```json
   {
     "server": {
       "tts_provider": "openai"
     }
   }
   ```

2. **启动 IndexTTS 服务**：
   确保 IndexTTS 服务在指定的地址运行。

3. **测试 IndexTTS**：
   使用提供的测试脚本：
   ```bash
   python test_indextts.py
   ```

#### API 参考

IndexTTS 使用与 OpenAI 兼容的 API 格式：

```bash
curl -X POST 'http://localhost:11996/audio/speech' \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "tts-1",
    "input": "你好",
    "voice": "jay_klee"
  }'
```

响应体为音频数据 (MP3 格式)。

## 长期记忆功能

VTuber API 支持基于 Qdrant 向量数据库的智能长期记忆系统，能够记住用户的互动历史并提供上下文相关的响应。

### 功能特性

- **语义搜索**：基于内容相似性检索相关的历史互动
- **用户专属记忆**：为每个用户维护独立的记忆空间
- **意图感知**：根据意图类型组织和检索记忆
- **可配置阈值**：支持自定义相似性阈值和上下文长度
- **自动清理**：可配置的记忆保留期限

### 配置说明

在 `config.json` 中配置长期记忆：

```json
{
  "long_term_memory": {
    "enabled": true,                    // 启用/禁用长期记忆
    "qdrant": {
      "url": "http://localhost:6334",  // Qdrant 服务器地址
      "collection_name": "vtuber_memory", // 集合名称
      "vector_size": 1536,             // 向量维度（与嵌入模型匹配）
      "distance": "Cosine"             // 距离算法：Cosine/Dot/Euclid/Manhattan
    },
    "context": {
      "max_context_length": 10,        // 最大上下文条目数
      "similarity_threshold": 0.7,     // 相似性阈值（0.0-1.0）
      "memory_retention_days": 30      // 记忆保留天数
    }
  }
}
```

### Qdrant 设置

1. **安装 Qdrant**：
   
   **Windows**：
   ```bash
   # 下载最新版本的 Qdrant Windows 二进制文件
   # 访问：https://github.com/qdrant/qdrant/releases
   # 下载 qdrant-x86_64-pc-windows-msvc.zip
   
   # 解压并运行
   qdrant.exe
   ```
   
   **Linux/macOS**：
   ```bash
   # 下载二进制文件
   wget https://github.com/qdrant/qdrant/releases/latest/download/qdrant-x86_64-unknown-linux-musl.tar.gz
   tar -xzf qdrant-x86_64-unknown-linux-musl.tar.gz
   
   # 运行 Qdrant
   ./qdrant
   ```

2. **验证连接**：
   ```bash
   curl http://localhost:6333/
   ```

3. **查看集合**：
   ```bash
   curl http://localhost:6333/collections
   ```

### 工作原理

1. **存储互动**：每次用户互动都会被转换为向量并存储在 Qdrant 中
2. **上下文检索**：处理新请求时，系统会搜索相关的历史互动
3. **智能响应**：AI 代理会基于检索到的上下文生成更个性化的响应
4. **记忆管理**：系统会自动清理过期的记忆以保持性能

### 运行程序

```bash
# 开发模式
cargo run

# 生产构建和运行
cargo build --release
./target/release/VtuberAPI

# Windows PowerShell
.\target\release\VtuberAPI.exe
```

## API 参考

### WebSocket 端点

- `ws://localhost:8000` - 主要 WebSocket 端点（直接连接，无需路径）

**注意**：此版本移除了所有 HTTP API 端点（`/health`、`/stats` 等），专注于纯 WebSocket 通信。如需监控和管理功能，可通过 WebSocket 消息实现。

### 消息类型

#### 身份验证
```json
{
  "type": "auth",
  "auth_data": {
    "type": "signature",
    "user_id": "user123",
    "api_key": "your-api-key",
    "timestamp": "2024-01-01T00:00:00Z",
    "nonce": "random-string",
    "signature": "hmac-sha256-signature"
  }
}
```

#### 弹幕处理
```json
{
  "type": "danmaku",
  "content": "画一只可爱的小猫咪",
  "user_id": "user123",
  "timestamp": "2024-01-01T00:00:00Z"
}
```

#### 进度更新
```json
{
  "type": "progress",
  "stage": "image_generation_start",
  "message": "🎨 正在为您创作图片，请稍等片刻...",
  "image_prompt": "可爱的小猫咪"
}
```

#### 带图像 URL 的响应
```json
{
  "type": "danmaku_response",
  "content": "已为您创作了一幅可爱的小猫咪图片！",
  "image_url": "https://your-api.com/generated-image-url",
  "intent": "image_generation",
  "user_id": "user123",
  "timestamp": "2024-01-01T00:00:00Z"
}
```

## 开发指南

### 项目结构

```
src/
├── main.rs              # 应用程序入口点
├── config/              # 配置管理
├── models/              # 数据结构和类型
├── auth/                # 身份验证服务
├── agents/              # Rig AI 代理
├── tools/               # 外部服务集成
├── workflows/           # 处理编排
├── services/            # 客户端和连接管理
├── memory/              # 长期记忆和向量存储
└── api/                 # WebSocket 服务器和端点
```

### 核心设计模式

1. **Rig 0.18.1 集成**：使用最新的 Rig 框架进行代理和完成模式
2. **灵活的 AI API 支持**：为不同 AI 提供商提供可配置的基础 URL
3. **JSON 配置管理**：结构化的 config.json 文件，支持复杂配置
4. **异步/等待**：使用 Tokio 运行时的完全异步
5. **类型安全**：利用 Rust 的类型系统保证可靠性
6. **错误处理**：使用 `anyhow` 进行全面的错误传播
7. **模块化架构**：清晰的关注点分离
8. **基于 URL 的媒体**：返回 URL 而非二进制数据以提高性能

### 添加新功能

1. **新意图类型**：扩展 `IntentType` 枚举并添加相应的代理
2. **附加工具**：在 `src/tools/` 中实现新工具
3. **增强身份验证**：扩展 `AuthService` 以支持新的认证方法
4. **向量存储**：为 RAG 功能添加 Rig 向量存储集成

## 测试

```bash
# 运行所有测试
cargo test

# 带输出运行
cargo test -- --nocapture

# 检查代码质量
cargo check
cargo clippy

# 格式化代码
cargo fmt
```

### WebSocket 测试

使用包含的 `websocket_test.html` 文件测试 WebSocket 连接：

1. 在 Web 浏览器中打开 `websocket_test.html`
2. 配置您的身份验证凭据
3. 测试身份验证和弹幕处理

## 性能特点

- **内存高效**：Rust 的零成本抽象
- **并发处理**：Tokio 异步运行时
- **连接池**：高效的 HTTP 客户端重用
- **流式响应**：实时数据处理

## 安全性

- **强制身份验证**：不允许匿名访问
- **HMAC 签名**：加密安全的身份验证
- **时间戳验证**：防止重放攻击
- **输入验证**：全面的请求验证

## 监控

服务器提供内置监控端点：

- `/health` 健康检查
- `/stats` 连接统计
- 使用 `tracing` 的结构化日志

## 生产部署

### 环境设置

1. 为生产环境设置强 `secret_key`
2. 为您的授权客户端配置 `valid_api_keys`
3. 设置您的 AI API `base_url` 和凭据
4. 设置适当的 `rust_log` 级别（例如，`"info"`）
5. 考虑为生产部署设置反向代理
6. 确保您的 AI API 支持所需的端点

## 核心依赖

```toml
# 核心框架
rig-core = "0.18.2"          # 最新的 AI 代理框架

# 异步运行时
tokio = "1.34.0"             # 具有完整功能的异步运行时

# WebSocket
tokio-tungstenite = "0.27"   # 纯 WebSocket 实现
futures-util = "0.3"         # 异步流处理

# 序列化和错误处理
serde = "1.0"                # 序列化框架
serde_json = "1.0"           # JSON 支持
anyhow = "1.0.75"            # 错误处理
thiserror = "2.0"            # 错误派生宏

# 配置管理
config = "0.15.14"           # 结构化配置文件支持

# 向量存储和长期记忆
rig-qdrant = "0.1.23"       # Qdrant 向量存储集成
qdrant-client = "1.15.0"    # Qdrant 客户端

# 加密和实用程序
hmac = "0.12"                # HMAC 身份验证
sha2 = "0.10"                # SHA256 哈希
base64 = "0.22"              # Base64 编码
uuid = "1.0"                 # UUID 生成
chrono = "0.4"               # 日期和时间处理
```

**架构优化**：
- **移除的依赖**：axum、tower、tower-http 等 HTTP 框架依赖
- **保留的依赖**：tokio-tungstenite 用于纯 WebSocket 通信
- **更小的二进制文件**：减少了约 30% 的编译体积
- **更快的启动时间**：移除 HTTP 中间件和路由开销

## 故障排除

### 常见问题

1. **身份验证失败**：检查签名生成和时间戳验证
2. **AI API 连接问题**：验证 config.json 中的 base_url 和 api_key 配置
3. **WebSocket 连接问题**：检查防火墙和网络设置
4. **构建错误**：确保 Rust 版本 1.70+ 并运行 `cargo clean`
5. **配置文件错误**：检查 config.json 格式和必需字段
6. **Qdrant 连接失败**：
   - 确保 Qdrant 服务器正在运行（下载并启动 Qdrant 二进制文件）
   - 检查 config.json 中的 Qdrant URL 配置
   - 验证 Qdrant 版本兼容性
7. **长期记忆功能不工作**：
   - 检查 `long_term_memory.enabled` 是否设置为 `true`
   - 验证向量维度与嵌入模型匹配（默认 1536 for text-embedding-3-small）
   - 确认相似性阈值设置合理（0.0-1.0）

### 日志记录

通过 config.json 设置日志级别：
```json
{
  "logging": {
    "rust_log": "debug"
  }
}
```

或临时设置环境变量：
```bash
$env:RUST_LOG="debug"; cargo run           # PowerShell
RUST_LOG=debug cargo run                   # Bash/Zsh
RUST_LOG=VtuberAPI=debug cargo run        # 模块特定日志
```

## 许可证

MIT 许可证 - 详见 LICENSE 文件。

## 更新日志

### 版本 0.2.0 (2025-08-21)
- **架构重构**：移除 axum、tower 等 HTTP 框架依赖，使用纯 tokio-tungstenite 实现
- **WebSocket 优化**：直接在根路径提供 WebSocket 服务（`ws://localhost:8000`）
- **依赖精简**：减少约 30% 的编译体积和依赖复杂度
- **性能提升**：移除 HTTP 中间件开销，提升 WebSocket 连接性能
- **Rig 框架升级**：使用 Rig 0.18.2 的最新功能
- **测试页面更新**

### 版本 0.1.0 (2025-08-20)
- **Rig 框架升级**：使用 Rig 0.18.1 的最新功能
- **Rust 2024 Edition**：使用最新的 Rust 2024 edition
- **JSON 配置管理**：支持嵌套配置和复杂数据类型
- **长期记忆功能**：基于 Qdrant 向量数据库的智能记忆系统
- **改进的错误处理**：更好的配置验证和错误报告
- **灵活的日志配置**：通过配置文件管理日志级别和选项
- **WebSocket 通信**：实时双向通信支持
- **多模态响应**：文本、图像、音频的完整支持
- **HMAC-SHA256 身份验证**：带时间戳验证的安全认证
- **向量存储支持**：完整的 Qdrant 集成用于语义搜索
