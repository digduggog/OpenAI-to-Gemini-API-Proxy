# Gemini API Proxy（可用于Gemini-Cli）
一个轻量级的 Google Gemini API 兼容代理服务器，允许您使用 Gemini API 格式调用 OpenAI 兼容的 LLM 服务。

## 🔥 Gemini CLI 快速设置

当前Gemini CLI不能简单的使用gemini之外的模型，所以基于此需求开发了一个python工具。

**使用方法：**
1. 修改 `config.json`，添加您的 API 提供商配置。
2. 安装依赖，运行 `python gemini_proxy_for_kimi.py`。
3. 在程序启动时，根据提示选择要使用的 API 提供商。
4. 设置环境变量：
   ```bash
   export GOOGLE_GEMINI_BASE_URL=http://localhost:8000/
   export GEMINI_API_KEY=sk-1234
   ```
5. 在 Gemini CLI 中使用 `/auth`，选择 "Use Gemini API Key"。

> **⚠️ 重要说明：当前版本仅在 Moonshot Kimi 上进行过完整测试和优化。其他 OpenAI 兼容服务需要您自行测试和调整。**

[English Documentation](README.md) | [英文文档](README.md)

## ✨ 特性

- 🔄 **完整 API 兼容性** - 支持所有 Gemini API 端点
- 🔀 **智能格式转换** - Gemini ↔ OpenAI 格式无缝转换  
- 🌊 **流式响应支持** - 完整的 Server-Sent Events 流式处理
- 🛠️ **函数调用支持** - 工具调用的双向转换
- 🗣️ **多轮对话** - 完整的对话历史处理
- 📊 **模型映射** - 灵活的模型名称映射配置
- 📝 **详细日志** - 可配置的访问日志和详细请求日志
- ⚙️ **配置文件** - JSON 配置文件统一管理
- 🔁 **自动重试** - 请求失败时自动重试

## 🚀 快速开始

### 1. 环境要求

- Python 3.8+
- 依赖包：`fastapi`、`uvicorn`、`openai`

### 2. 安装依赖

```bash
pip install -r requirements.txt
```

### 3. 配置文件

创建 `config.json` 文件，现在支持多个提供商：

```json
{
    "providers": [
        {
            "name": "Moonshot",
            "openai_api_key": "sk-1234",
            "openai_base_url": "https://api.moonshot.cn/v1",
            "daily_limit": -1,
            "model_mapping": {
                "gemini-2.5-pro": "kimi-k2-0711-preview",
                "gemini-2.5-flash": "moonshot-v1-auto"
            },
            "default_openai_model": "kimi-k2-0711-preview"
        },
        {
            "name": "OpenAI",
            "openai_api_key": "sk-5678",
            "openai_base_url": "https://api.openai.com/v1",
            "daily_limit": -1,
            "model_mapping": {
                "gemini-2.5-pro": "gpt-4o",
                "gemini-2.5-flash": "gpt-4o-mini"
            },
            "default_openai_model": "gpt-4o-mini"
        }
    ],
    "server": {
        "host": "0.0.0.0",
        "port": 8000,
        "log_level": "info"
    },
    "logging": {
        "enable_detailed_logs": false,
        "enable_access_logs": true,
        "log_directory": "logs"
    },
    "retry": {
        "max_retries": 999,
        "wait_fixed": 5
    }
}
```

### 4. 启动服务

启动服务时，系统会提示您选择一个提供商：

```bash
python gemini_proxy_for_kimi.py
```

服务将在 `http://0.0.0.0:8000` 启动。

## 📖 API 使用

### 基本请求

```bash
curl -X POST http://localhost:8000/v1beta/models/gemini-2.5-pro:generateContent \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{"text": "Hello, how are you?"}]
    }],
    "generationConfig": {
      "temperature": 0.7,
      "maxOutputTokens": 200
    }
  }'
```

### 流式请求

```bash
curl -X POST http://localhost:8000/v1beta/models/gemini-2.5-pro:streamGenerateContent \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{"text": "Write a short story"}]
    }]
  }'
```

### Token 计数

```bash
curl -X POST http://localhost:8000/v1beta/models/gemini-2.5-pro:countTokens \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{"text": "Count tokens for this text"}]
    }]
  }'
```

### 函数调用

```bash
curl -X POST http://localhost:8000/v1beta/models/gemini-2.5-pro:generateContent \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{"text": "What is the weather like in Beijing?"}]
    }],
    "tools": [{
      "functionDeclarations": [{
        "name": "get_weather",
        "description": "Get weather information for a city",
        "parameters": {
          "type": "object",
          "properties": {
            "city": {"type": "string", "description": "City name"},
            "date": {"type": "string", "description": "Date in YYYY-MM-DD format"}
          },
          "required": ["city"]
        }
      }]
    }]
  }'
```

## 🔧 配置说明

### 完整配置文件 (config.json)

```json
{
    "providers": [
        {
            "name": "Moonshot",
            "openai_api_key": "sk-1234",
            "openai_base_url": "https://api.moonshot.cn/v1",
            "daily_limit": -1,
            "model_mapping": {
                "gemini-2.5-pro": "kimi-k2-0711-preview",
                "gemini-2.5-flash": "moonshot-v1-auto"
            },
            "default_openai_model": "kimi-k2-0711-preview"
        }
    ],
    "server": {
        "host": "0.0.0.0",
        "port": 8000,
        "log_level": "info"
    },
    "logging": {
        "enable_detailed_logs": false,
        "enable_access_logs": true,
        "log_directory": "logs"
    },
    "retry": {
        "max_retries": 999,
        "wait_fixed": 5
    }
}
```

### 配置项说明

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| `providers` | API 提供商配置列表。 | `[]` |
| `providers[].name` | 提供商名称，用于启动时选择。 | `未命名提供商` |
| `providers[].openai_api_key` | 提供商的 OpenAI API 密钥。 | 必需 |
| `providers[].openai_base_url` | 提供商的 OpenAI API 基础 URL。 | `https://api.openai.com/v1` |
| `providers[].daily_limit` | 提供商的每日请求限制。-1 表示无限制。 | `-1` |
| `providers[].model_mapping` | 提供商的 Gemini 模型到 OpenAI 模型的映射。 | `{}` |
| `providers[].default_openai_model` | 提供商的默认 OpenAI 模型。 | `gpt-3.5-turbo` |
| `server.host` | 监听地址 | `0.0.0.0` |
| `server.port` | 监听端口 | `8000` |
| `server.log_level` | 日志级别 | `info` |
| `logging.enable_detailed_logs` | 启用详细请求日志 | `false` |
| `logging.enable_access_logs` | 启用访问日志 | `true` |
| `logging.log_directory` | 日志目录 | `logs` |
| `retry.max_retries` | 失败时最大重试次数 | `3` |
| `retry.wait_fixed` | 每次重试之间的固定等待时间（秒） | `2` |

### 提供商故障切换逻辑

服务内置了基于各提供商 `daily_limit` 的自动故障切换机制，以确保高可用性：

1.  **首选提供商**：服务将始终优先使用您在启动时选择的提供商。
2.  **无限额度切换**：如果首选提供商达到其每日限额，服务将自动寻找并切换到第一个可用的无限额度（`"daily_limit": -1`）的提供商。
3.  **有限额度切换**：如果没有可用的无限额度提供商，服务将接着寻找并切换到第一个仍有剩余请求额度的提供商。
4.  **服务不可用**：如果所有配置的提供商都已达到其每日限额，API 将返回 `503 Service Unavailable` 错误。

## 📊 支持的 LLM 服务

### ✅ 经过测试的服务

#### Kimi (月之暗面) - 推荐
```json
{
  "openai_api_key": "sk-xxx",
  "openai_base_url": "https://api.moonshot.cn/v1",
  "model_mapping": {
    "gemini-2.5-pro": "kimi-k2-0711-preview",
    "gemini-2.5-flash": "kimi-k2-0711-preview"
  }
}
```

### ⚠️ 需要自行测试的服务

> **注意：以下服务理论上兼容，但需要您自行测试和调整。可以打开配置文件中的enable_detailed_logs，会输出请求和输出的详细信息，然后针对性适配。

#### OpenAI
```json
{
  "openai_api_key": "sk-xxx",
  "openai_base_url": "https://api.openai.com/v1",
  "model_mapping": {
    "gemini-2.5-pro": "gpt-4o",
    "gemini-2.5-flash": "gpt-4o-mini"
  }
}
```

#### Azure OpenAI
```json
{
  "openai_api_key": "your-azure-key",
  "openai_base_url": "https://your-resource.openai.azure.com/openai/deployments/your-deployment",
  "model_mapping": {
    "gemini-2.5-pro": "gpt-4",
    "gemini-2.5-flash": "gpt-35-turbo"
  }
}
```

#### DeepSeek
```json
{
  "openai_api_key": "sk-xxx", 
  "openai_base_url": "https://api.deepseek.com/v1",
  "model_mapping": {
    "gemini-2.5-pro": "deepseek-chat",
    "gemini-2.5-flash": "deepseek-chat"
  }
}
```

#### 智谱 AI
```json
{
  "openai_api_key": "your-zhipu-key",
  "openai_base_url": "https://open.bigmodel.cn/api/paas/v4",
  "model_mapping": {
    "gemini-2.5-pro": "glm-4",
    "gemini-2.5-flash": "glm-4-flash"
  }
}
```

#### Ollama (本地部署)
```json
{
  "openai_api_key": "ollama",
  "openai_base_url": "http://localhost:11434/v1",
  "model_mapping": {
    "gemini-2.5-pro": "llama3:8b",
    "gemini-2.5-flash": "llama3:8b"
  }
}
```

### 🛠️ 自定义其他服务

如果您需要适配其他 OpenAI 兼容的服务：

1. **Clone 项目**：`git clone `
2. **修改配置**：更新 `config.json` 中的 API 端点和模型映射
3. **测试功能**：重点测试流式响应、函数调用、多轮对话，可以打开配置文件中的enable_detailed_logs，会输出请求和输出的详细信息，然后针对性适配。
4. **优化代码**：根据目标服务的特性调整转换逻辑


## 📝 日志系统

### 访问日志
简洁的访问日志，显示每个请求的基本信息：
```
🚀 POST /v1beta/models/gemini-2.5-pro:generateContent - 200 - Model: gemini-2.5-pro - ID: abc12345 - 2.341s
🚀 POST /v1beta/models/gemini-2.5-pro:streamGenerateContent - 200 - Model: gemini-2.5-pro(stream) - ID: def67890 - 5.123s
```

### 详细日志
完整的请求/响应转换过程（可选开启）：
- `1_GEMINI_REQUEST` - 原始 Gemini 请求
- `2_OPENAI_REQUEST` - 转换后的 OpenAI 请求  
- `3_OPENAI_RESPONSE` - OpenAI 原始响应
- `4_GEMINI_RESPONSE` - 最终 Gemini 响应

## 🚀 生产部署

### 使用 Gunicorn (推荐)

1. 安装 Gunicorn：
```bash
pip install gunicorn
```

2. 创建启动脚本 `start.sh`：
```bash
#!/bin/bash
gunicorn gemini_proxy_for_kimi:app \
  --worker-class uvicorn.workers.UvicornWorker \
  --workers 4 \
  --bind 0.0.0.0:8000 \
  --access-logfile - \
  --error-logfile - \
  --log-level info
```

3. 运行：
```bash
chmod +x start.sh
./start.sh
```

### 使用 systemd 服务

创建 `/etc/systemd/system/gemini-proxy.service`：

```ini
[Unit]
Description=Gemini API Proxy
After=network.target

[Service]
Type=exec
User=your-user
Group=your-group
WorkingDirectory=/path/to/your/app
ExecStart=/path/to/your/venv/bin/gunicorn gemini_proxy_for_kimi:app \
  --worker-class uvicorn.workers.UvicornWorker \
  --workers 4 \
  --bind 0.0.0.0:8000
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

启动服务：
```bash
sudo systemctl daemon-reload
sudo systemctl enable gemini-proxy
sudo systemctl start gemini-proxy
```

### Nginx 反向代理

创建 Nginx 配置：

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # 支持流式响应
        proxy_buffering off;
        proxy_cache off;
        
        # 增加超时时间
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

### 性能优化

1. **Worker 数量**：根据 CPU 核心数设置，建议 `2 * CPU_CORES + 1`

2. **内存优化**：
```bash
# 限制内存使用
gunicorn --max-requests 1000 --max-requests-jitter 100 ...
```

3. **连接池**：在配置中增加连接池设置

4. **缓存**：可以添加 Redis 缓存层来缓存响应

## 🔍 监控和维护

### 健康检查

```bash
curl http://localhost:8000/health
```

返回：
```json
{"status": "healthy", "service": "gemini-proxy"}
```

### 日志轮转

使用 logrotate 管理日志文件：

```bash
# /etc/logrotate.d/gemini-proxy
/path/to/your/app/logs/*.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    create 644 your-user your-group
    postrotate
        systemctl reload gemini-proxy
    endscript
}
```

### 监控指标

访问日志包含以下信息：
- 请求方法和路径
- 响应状态码
- 使用的模型
- 请求 ID
- 响应时间

## 🛠️ 开发和调试

### 开发模式

```bash
# 启用详细日志
# 在 config.json 中设置
{
  "logging": {
    "enable_detailed_logs": true,
    "enable_access_logs": true
  }
}

# 启动开发服务器
python gemini_proxy_for_kimi.py
```

### 调试技巧

1. **查看详细日志**：启用 `enable_detailed_logs` 查看完整转换过程
2. **模型映射测试**：使用不同的 Gemini 模型名测试映射
3. **流式响应调试**：观察 SSE 数据流
4. **函数调用调试**：检查工具调用的格式转换

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

### 开发环境设置

```bash
git clone 
cd gemini-proxy
pip install -r requirements.txt
```

### 代码规范

- 遵循 PEP 8 代码风格
- 添加适当的类型注解
- 编写清晰的文档字符串
- 确保向后兼容性

## 📄 许可证

本项目采用 MIT 许可证。详见 [LICENSE](LICENSE) 文件。

## 🆘 常见问题

### Q: 支持哪些 Gemini API 端点？
A: 支持 `generateContent`、`streamGenerateContent`、`countTokens` 和 `health` 端点。

### Q: 目前支持哪些 LLM 服务？
A: **当前仅在 Moonshot Kimi 上进行过完整测试**。其他 OpenAI 兼容服务理论上可用，但需要您自行测试和调整。

### Q: 如何适配其他 LLM 服务？
A: 1) Clone 项目代码 2) 修改 `config.json` 配置 3) 重点测试流式响应、函数调用等功能 4) 根据需要调整代码逻辑 

### Q: 为什么只支持 Kimi？
A: 因为不同 LLM 服务在 API 细节、响应格式、错误处理等方面存在差异，需要针对性测试和优化。目前主要精力集中在 Kimi 的完整适配上。

### Q: 流式响应不工作怎么办？
A: 检查客户端是否正确处理 `text/event-stream` 格式，确保网络环境支持 SSE。

### Q: 如何处理大量并发请求？
A: 增加 Gunicorn worker 数量，使用负载均衡器，考虑添加缓存层。

## 🔗 相关链接

- [Google Gemini API 文档](https://ai.google.dev/gemini-api/docs/text-generation?hl)
- [Moonshot Kimi API 文档](https://platform.moonshot.cn/docs/introduction)
---

**⭐ 如果这个项目对您有帮助，请给我们一个 Star！**