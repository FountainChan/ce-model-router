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
| `/ce-brainstorm` | MiniMax 2.7 | 头脑风暴、需求探索 |
| `/ce-ideate` | MiniMax 2.7 | 创意发散、想法生成 |
| `/ce-plan` | GLM-5.1 | 技术规划、方案设计 |
| `/ce-work` | GLM-5.1 | 代码实现、功能开发 |
| `/ce-review` | MiniMax 2.7 | 代码审查、质量把控 |

## 📁 项目结构

```
ce-model-router/
├── README.md                    # 📚 项目说明文档
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

### 1️⃣ 安装 Fallback 插件

```bash
npm install -g opencode-runtime-fallback
```

### 2️⃣ 配置 OpenCode

将 `config/opencode.json` 中的配置合并到你的 `~/.config/opencode/opencode.json`：

```json
{
  "plugin": ["opencode-runtime-fallback"],
  "experimental": {
    "modelFallbackChain": {
      "timeoutMs": 60000,
      "chains": [
        ["minimax-cn-coding-plan/MiniMax-M2.7", "zhipuai-coding-plan/glm-5.1"],
        ["zhipuai-coding-plan/glm-5.1", "deepseek/deepseek-v4-pro", "yunwu-claude/claude-opus-4-7"],
        ["yunwu-claude/claude-opus-4-7", "yunwu-gpt/gpt-5.4"]
      ]
    }
  }
}
```

### 3️⃣ 部署命令文件

将 `commands/` 目录下的所有 `.md` 文件复制到 `~/.config/opencode/commands/`：

```bash
cp commands/*.md ~/.config/opencode/commands/
```

### 4️⃣ 重启 OpenCode

```bash
# 重启 OpenCode 使配置生效
```

## 🔄 Fallback 链说明

| 链序号 | 主模型 | 备选模型 1 | 备选模型 2 |
|--------|--------|-----------|-----------|
| Chain 1 | MiniMax 2.7 | GLM-5.1 | — |
| Chain 2 | GLM-5.1 | DeepSeek-V4-Pro | Claude Opus 4.7 |
| Chain 3 | Claude Opus 4.7 | GPT-5.4 | — |

**触发条件**：
- ⏱️ 模型响应超时（默认 60 秒）
- 🚫 API 错误（429 Rate Limit、500 Server Error 等）
- ❌ 模型不可用

## 🆘 故障排除

### Fallback 不生效？

1. ✅ 确认插件已添加到 `plugin` 数组
2. ✅ 确认 `experimental.modelFallbackChain.chains` 格式正确
3. ✅ 重启 OpenCode 使配置生效

### 与 oh-my-openagent 的兼容性？

⚠️ **重要提示**：`oh-my-openagent (omO)` 内置了自己的 runtime_fallback 功能。

- 如果同时安装 omO，需先卸载 `opencode-runtime-fallback`
- omO 的 fallback 配置在 `oh-my-opencode.json` 中

详见 [Fallback 插件操作指南](docs/fallback-plugin-guide.md)。

## 📊 技术细节

- **配置格式**：OpenCode 官方 command 文件格式（Markdown + YAML frontmatter）
- **插件机制**：基于 `opencode-runtime-fallback` 插件
- **路由方式**：通过 command 文件的 `model` 字段指定模型
- **降级机制**：基于 timeout 和 API error 的自动切换

## 📜 开源许可证

本项目采用 MIT 开源许可证，详见 [LICENSE](LICENSE) 文件。

## 🙏 致谢

- [OpenCode](https://opencode.ai/) — AI 编程助手
- [opencode-runtime-fallback](https://www.npmjs.com/package/opencode-runtime-fallback) — Fallback 插件
- 所有开源模型的提供者

---

> 💡 **提示**：本配置已针对成本控制和模型特性进行优化，MiniMax 2.7 用于轻量任务，GLM-5.1 作为主力模型。
