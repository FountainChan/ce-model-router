[English](README_en.md) | 简体中文

# 🚀 OpenCode CE 模型自动路由配置

> 📌 **轻松实现多模型智能路由，让不同的 `/ce-*` 命令自动使用最合适的 AI 模型！**

## 📖 项目简介

本项目提供了一套完整的 **OpenCode 模型自动路由方案**，实现了：

- 🎯 **命令级路由** — 不同 `/ce-*` 命令使用不同基础模型
- ⚡ **自动降级** — API 错误时自动切换到备选模型（Fallback Chain）
- 💡 **轻量实现** — 无需重型插件，保留现有 compound-engineering 配置

## 🎯 模型分配策略

| 命令 | 基础模型 | 用途说明 |
|------|---------|---------|
| `/ce-brainstorm` | 轻量模型 | 头脑风暴、需求探索 |
| `/ce-ideate` | 轻量模型 | 创意发散、想法生成 |
| `/ce-plan` | 主力模型 | 技术规划、方案设计 |
| `/ce-work` | 主力模型 | 代码实现、功能开发 |
| `/ce-review` | 轻量模型 | 代码审查、质量把控 |

> 💡 **设计理念**：轻量任务（头脑风暴、创意发散、代码审查）使用轻量模型以节省成本；核心任务（规划、实现）使用主力模型以确保质量。

## 📁 项目结构

```
ce-model-router/
├── README.md                    # 📚 项目说明文档（你正在阅读）
├── LICENSE                      # 📄 MIT 开源许可证
├── commands/                    # 💬 OpenCode 命令文件
│   ├── ce-brainstorm.md         # /ce-brainstorm 命令配置
│   ├── ce-ideate.md             # /ce-ideate 命令配置
│   ├── ce-plan.md               # /ce-plan 命令配置
│   ├── ce-work.md               # /ce-work 命令配置
│   └── ce-review.md             # /ce-review 命令配置
├── config/                      # ⚙️ 配置文件
│   └── opencode.json            # OpenCode 主配置文件（含 Fallback 链）
└── docs/                        # 📋 项目文档
    ├── requirements.md          # 需求文档
    ├── plan.md                  # 实施计划
    └── fallback-plugin-guide.md # Fallback 插件操作指南
```

## 🔧 快速开始

### 📋 前置条件

- ✅ 已安装 [OpenCode](https://opencode.ai/)
- ✅ 已安装 [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin)（CE skill 体系）
- ✅ 已安装 Node.js 和 npm
- ✅ 至少有一个可用的 AI 模型 API Key

> ⚠️ **前提**：本项目依赖 compound-engineering-plugin 提供的 `ce:brainstorm`、`ce:plan`、`ce:work`、`ce:review` 等 skill。请先按照 [compound-engineering-plugin 文档](https://github.com/EveryInc/compound-engineering-plugin) 安装配置。

### Step 1：连接模型

OpenCode 支持 **75+ LLM 提供商**。在 OpenCode 中使用 `/connect` 命令连接你的模型提供商：

```
/connect
```

按照提示输入 API Key 即可。常用的提供商包括：

| 提供商 | 模型示例 | 获取 API Key |
|--------|---------|-------------|
| 🟢 OpenAI | GPT-5.4、GPT-5.2 | [platform.openai.com](https://platform.openai.com/) |
| 🟣 Anthropic | Claude Opus 4.7、Sonnet 4.6 | [console.anthropic.com](https://console.anthropic.com/) |
| 🔵 智谱 AI | GLM-5.1、GLM-4.7 | [open.bigmodel.cn](https://open.bigmodel.cn/) |
| 🟡 MiniMax | MiniMax-M2.7 | [platform.minimaxi.com](https://platform.minimaxi.com/) |
| 🔴 DeepSeek | DeepSeek-V4-Pro | [platform.deepseek.com](https://platform.deepseek.com/) |
| 🟠 Google | Gemini 3 Pro | [aistudio.google.com](https://aistudio.google.com/) |
| ⚪ OpenRouter | 多种模型 | [openrouter.ai](https://openrouter.ai/) |
| 🏠 本地模型 | Llama、Qwen 等 | 通过 Ollama、LM Studio 运行 |

> 💡 **提示**：你不必使用上述所有提供商，选择 1-2 个即可开始。推荐的最低配置是一个轻量模型 + 一个主力模型。

#### 🔍 查看可用模型

连接提供商后，在 OpenCode 中输入以下命令查看所有可用模型：

```
/models
```

模型 ID 的格式为 `provider_id/model_id`，例如：
- `openai/gpt-5.4`
- `anthropic/claude-opus-4-7`
- `zhipuai-coding-plan/glm-5.1`
- `minimax-cn-coding-plan/MiniMax-M2.7`

> 📝 **记下你要使用的模型 ID**，后面的配置会用到。

#### ⚙️ 自定义提供商（可选）

如果需要使用非内置提供商或自定义模型，在 `opencode.json` 中配置：

```json
{
  "provider": {
    "my-provider": {
      "options": {
        "apiKey": "YOUR_API_KEY",
        "baseURL": "https://api.example.com/v1/"
      },
      "models": {
        "my-model": {
          "name": "My Custom Model",
          "limit": {
            "context": 128000,
            "output": 16384
          }
        }
      }
    }
  }
}
```

### Step 2：安装 Fallback 插件

```bash
npm install -g opencode-runtime-fallback
```

### Step 3：配置 Fallback 链

在 `~/.config/opencode/opencode.json` 中添加以下配置：

```json
{
  "plugin": ["opencode-runtime-fallback"],
  "experimental": {
    "modelFallbackChain": {
      "timeoutMs": 60000,
      "chains": [
        ["YOUR_LIGHTWEIGHT_MODEL", "YOUR_MAIN_MODEL"],
        ["YOUR_MAIN_MODEL", "YOUR_FALLBACK_MODEL_1", "YOUR_FALLBACK_MODEL_2"]
      ]
    }
  }
}
```

**配置示例（使用智谱 + OpenAI）**：

```json
{
  "plugin": ["opencode-runtime-fallback"],
  "experimental": {
    "modelFallbackChain": {
      "timeoutMs": 60000,
      "chains": [
        ["zhipuai-coding-plan/glm-4.5-air", "openai/gpt-5.4"],
        ["openai/gpt-5.4", "anthropic/claude-sonnet-4-6"]
      ]
    }
  }
}
```

**配置示例（使用本地 + 云端）**：

```json
{
  "plugin": ["opencode-runtime-fallback"],
  "experimental": {
    "modelFallbackChain": {
      "timeoutMs": 60000,
      "chains": [
        ["ollama/qwen3-coder", "openai/gpt-5.4"],
        ["openai/gpt-5.4", "anthropic/claude-sonnet-4-6"]
      ]
    }
  }
}
```

> 💡 **原则**：每条链从便宜/快速模型开始，逐步升级到更强/更贵的模型。

### Step 4：自定义命令文件的模型

打开 `commands/` 目录下的 `.md` 文件，修改 `model` 字段为你的模型 ID：

```markdown
---
description: 头脑风暴 - 探索需求和方法
argument-hint: "[feature idea or problem to explore]"
allowed-tools: Read, Write, Grep, Glob, Bash
skill: ce:brainstorm
model: YOUR_LIGHTWEIGHT_MODEL    ← 改成你的轻量模型
---
```

**各命令推荐的模型类型**：

| 命令文件 | 推荐模型类型 | 示例 |
|----------|------------|------|
| `ce-brainstorm.md` | 🏷️ 轻量/便宜 | `minimax-cn-coding-plan/MiniMax-M2.7`、`ollama/qwen3-coder` |
| `ce-ideate.md` | 🏷️ 轻量/便宜 | `zhipuai-coding-plan/glm-4.5-air`、`openai/gpt-5.4-nano` |
| `ce-plan.md` | 🏋️ 主力/高质量 | `zhipuai-coding-plan/glm-5.1`、`openai/gpt-5.4` |
| `ce-work.md` | 🏋️ 主力/高质量 | `openai/gpt-5.4`、`anthropic/claude-sonnet-4-6` |
| `ce-review.md` | 🏷️ 轻量/便宜 | `minimax-cn-coding-plan/MiniMax-M2.7`、`openai/gpt-5.4-mini` |

### Step 5：部署命令文件

将 `commands/` 目录下的所有 `.md` 文件复制到 `~/.config/opencode/commands/`：

**macOS / Linux**：

```bash
cp commands/*.md ~/.config/opencode/commands/
```

**Windows (PowerShell)**：

```powershell
Copy-Item commands\*.md $env:USERPROFILE\.config\opencode\commands\
```

### Step 6：重启 OpenCode

重启 OpenCode 使所有配置生效。

## 🔄 Fallback 链说明

Fallback 链定义了模型失败时的降级顺序：

```
模型 A（主） → 模型 B（备选1） → 模型 C（备选2）
```

**触发条件**：
- ⏱️ 模型响应超时（默认 60 秒）
- 🚫 API 错误（429 Rate Limit、500 Server Error 等）
- ❌ 模型不可用

**配置原则**：
- 📊 每条链从便宜模型开始，逐步升级到更强模型
- 🔗 不同模型可以出现在多条链中
- ⚡ 链越短，降级越快

## 🎨 自定义配置

### 修改默认模型

如果你想修改某个命令使用的模型，编辑对应的 `.md` 文件中的 `model` 字段：

```markdown
---
model: openai/gpt-5.4    # 改成你想要的模型
---
```

### 添加新的 ce 命令

在 `~/.config/opencode/commands/` 目录下创建新的 `.md` 文件：

```markdown
---
description: 命令描述
argument-hint: "[参数提示]"
allowed-tools: Read, Write, Grep, Glob, Bash
skill: ce:对应skill名称
model: YOUR_MODEL_ID
---

# /命令名 — 命令标题

命令的执行说明。
```

### 修改超时时间

编辑 `opencode.json` 中的 `timeoutMs` 值：

```json
{
  "experimental": {
    "modelFallbackChain": {
      "timeoutMs": 30000  // 30 秒超时
    }
  }
}
```

## 🆘 故障排除

### ❓ 看不到命令？

1. ✅ 确认 `.md` 文件已复制到 `~/.config/opencode/commands/` 目录
2. ✅ 确认文件名格式正确（小写字母 + 连字符）
3. ✅ 重启 OpenCode

### ❓ Fallback 不生效？

1. ✅ 确认 `opencode-runtime-fallback` 已安装：`npm list -g opencode-runtime-fallback`
2. ✅ 确认 `opencode.json` 的 `plugin` 数组包含 `"opencode-runtime-fallback"`
3. ✅ 确认 `chains` 中的模型 ID 格式正确（使用 `/models` 命令验证）
4. ✅ 重启 OpenCode

### ❓ 模型不可用？

1. ✅ 使用 `/models` 命令查看可用模型列表
2. ✅ 使用 `/connect` 命令配置模型提供商
3. ✅ 检查 API Key 是否有效、配额是否充足

### ⚠️ 与 oh-my-openagent 的兼容性

`oh-my-openagent (omO)` 内置了自己的 runtime_fallback 功能。如果同时安装 omO，需先卸载 `opencode-runtime-fallback`，避免冲突。详见 [Fallback 插件操作指南](docs/fallback-plugin-guide.md)。

## 📊 技术细节

- **配置格式**：OpenCode 官方 command 文件格式（Markdown + YAML frontmatter）
- **插件机制**：基于 `opencode-runtime-fallback` 插件
- **路由方式**：通过 command 文件的 `model` 字段指定模型
- **降级机制**：基于 timeout 和 API error 的自动切换
- **模型发现**：通过 `/connect` 连接提供商，通过 `/models` 查看可用模型

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

## 📜 开源许可证

本项目采用 MIT 开源许可证，详见 [LICENSE](LICENSE) 文件。

## 🙏 致谢

- [OpenCode](https://opencode.ai/) — AI 编程助手
- [opencode-runtime-fallback](https://www.npmjs.com/package/opencode-runtime-fallback) — Fallback 插件
- [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) — CE skill 体系
- 所有开源模型的提供者

---

> 💡 **提示**：本配置支持灵活自定义——你可以根据自己的模型和预算，自由调整每个命令使用的模型。
