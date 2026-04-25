English | [简体中文](README.md)

# 🚀 OpenCode CE Model Auto-Router

> 📌 **Easily route different `/ce-*` commands to the most suitable AI models!**

## 📖 About

This project provides a complete **OpenCode model auto-routing solution** that delivers:

- 🎯 **Command-level routing** — Different `/ce-*` commands use different base models
- ⚡ **Automatic fallback** — Seamlessly switch to backup models on API errors (Fallback Chain)
- 💡 **Lightweight** — No heavy plugins required, preserves existing compound-engineering setup

## 🎯 Model Assignment Strategy

| Command | Base Model | Purpose |
|---------|-----------|---------|
| `/ce-brainstorm` | Lightweight model | Brainstorming, requirement exploration |
| `/ce-ideate` | Lightweight model | Ideation, idea generation |
| `/ce-plan` | Main model | Technical planning, solution design |
| `/ce-work` | Main model | Code implementation, feature development |
| `/ce-review` | Lightweight model | Code review, quality assurance |

> 💡 **Design philosophy**: Lightweight tasks (brainstorming, ideation, review) use cost-effective models; core tasks (planning, implementation) use powerful models for quality.

**Author's actual configuration**:

| Command | Actual Model | Provider |
|---------|-------------|----------|
| `/ce-brainstorm` | MiniMax-M2.7 | `minimax-cn-coding-plan/MiniMax-M2.7` |
| `/ce-ideate` | MiniMax-M2.7 | `minimax-cn-coding-plan/MiniMax-M2.7` |
| `/ce-plan` | GLM-5.1 | `zhipuai-coding-plan/glm-5.1` |
| `/ce-work` | GLM-5.1 | `zhipuai-coding-plan/glm-5.1` |
| `/ce-review` | MiniMax-M2.7 | `minimax-cn-coding-plan/MiniMax-M2.7` |

> 🎯 Rationale: MiniMax-M2.7 is fast and cost-effective, ideal for generative tasks like brainstorming and code review; GLM-5.1 has strong reasoning capabilities, ideal for tasks requiring deep thinking like planning and implementation.

## 📁 Project Structure

```
ce-model-router/
├── README.md                    # 📚 Documentation (Chinese)
├── README_en.md                 # 📚 Documentation (English - you are here)
├── LICENSE                      # 📄 MIT License
├── commands/                    # 💬 OpenCode command files
│   ├── ce-brainstorm.md         # /ce-brainstorm command config
│   ├── ce-ideate.md             # /ce-ideate command config
│   ├── ce-plan.md               # /ce-plan command config
│   ├── ce-work.md               # /ce-work command config
│   └── ce-review.md             # /ce-review command config
├── config/                      # ⚙️ Configuration files
│   └── opencode.json            # OpenCode config with Fallback chains
└── docs/                        # 📋 Project documentation
    ├── requirements.md          # Requirements document
    ├── plan.md                  # Implementation plan
    └── fallback-plugin-guide.md # Fallback plugin guide
```

## 🔧 Quick Start

### 📋 Prerequisites

- ✅ [OpenCode](https://opencode.ai/) installed
- ✅ [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) installed (CE skill system)
- ✅ Node.js and npm installed
- ✅ At least one AI model API Key

> ⚠️ **Requirement**: This project depends on skills provided by compound-engineering-plugin (`ce:brainstorm`, `ce:plan`, `ce:work`, `ce:review`, etc.). Please install and configure it first following the [compound-engineering-plugin docs](https://github.com/EveryInc/compound-engineering-plugin).

### Step 1: Connect Your Models

OpenCode supports **75+ LLM providers**. Use the `/connect` command in OpenCode to connect your provider:

```
/connect
```

Follow the prompts to enter your API Key. Common providers include:

| Provider | Example Models | Get API Key |
|----------|---------------|-------------|
| 🟢 OpenAI | GPT-5.4, GPT-5.2 | [platform.openai.com](https://platform.openai.com/) |
| 🟣 Anthropic | Claude Opus 4.7, Sonnet 4.6 | [console.anthropic.com](https://console.anthropic.com/) |
| 🔵 Zhipu AI | GLM-5.1, GLM-4.7 | [open.bigmodel.cn](https://open.bigmodel.cn/) |
| 🟡 MiniMax | MiniMax-M2.7 | [platform.minimaxi.com](https://platform.minimaxi.com/) |
| 🔴 DeepSeek | DeepSeek-V4-Pro | [platform.deepseek.com](https://platform.deepseek.com/) |
| 🟠 Google | Gemini 3 Pro | [aistudio.google.com](https://aistudio.google.com/) |
| ⚪ OpenRouter | Various models | [openrouter.ai](https://openrouter.ai/) |
| 🏠 Local Models | Llama, Qwen, etc. | Run via Ollama, LM Studio |

> 💡 **Tip**: You don't need all providers — 1-2 are enough to start. The recommended minimum is one lightweight model + one main model.

#### 🔍 View Available Models

After connecting a provider, view all available models in OpenCode:

```
/models
```

Model IDs follow the format `provider_id/model_id`, for example:
- `openai/gpt-5.4`
- `anthropic/claude-opus-4-7`
- `zhipuai-coding-plan/glm-5.1`
- `minimax-cn-coding-plan/MiniMax-M2.7`

> 📝 **Note down the model IDs you want to use** — you'll need them in the next steps.

#### ⚙️ Custom Providers (Optional)

To use a non-built-in provider or custom model, configure it in `opencode.json`:

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

### Step 2: Install Fallback Plugin

```bash
npm install -g opencode-runtime-fallback
```

### Step 3: Configure Fallback Chains

Add the following to `~/.config/opencode/opencode.json`:

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

**Example (Zhipu + OpenAI)**:

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

**Example (Local + Cloud)**:

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

> 💡 **Rule of thumb**: Each chain starts with the cheapest/fastest model and escalates to stronger/more expensive ones.

### Step 4: Customize Command Model Assignments

Open the `.md` files in the `commands/` directory and change the `model` field to your model ID:

```markdown
---
description: Brainstorm - Explore requirements and approaches
argument-hint: "[feature idea or problem to explore]"
allowed-tools: Read, Write, Grep, Glob, Bash
skill: ce:brainstorm
model: YOUR_LIGHTWEIGHT_MODEL    ← Change to your lightweight model
---
```

**Recommended model types per command**:

| Command File | Recommended Type | Examples |
|-------------|-----------------|----------|
| `ce-brainstorm.md` | 🏷️ Lightweight / Cheap | `minimax-cn-coding-plan/MiniMax-M2.7`, `ollama/qwen3-coder` |
| `ce-ideate.md` | 🏷️ Lightweight / Cheap | `zhipuai-coding-plan/glm-4.5-air`, `openai/gpt-5.4-nano` |
| `ce-plan.md` | 🏋️ Main / High Quality | `zhipuai-coding-plan/glm-5.1`, `openai/gpt-5.4` |
| `ce-work.md` | 🏋️ Main / High Quality | `openai/gpt-5.4`, `anthropic/claude-sonnet-4-6` |
| `ce-review.md` | 🏷️ Lightweight / Cheap | `minimax-cn-coding-plan/MiniMax-M2.7`, `openai/gpt-5.4-mini` |

### Step 5: Deploy Command Files

Copy all `.md` files from the `commands/` directory to `~/.config/opencode/commands/`:

**macOS / Linux**:

```bash
cp commands/*.md ~/.config/opencode/commands/
```

**Windows (PowerShell)**:

```powershell
Copy-Item commands\*.md $env:USERPROFILE\.config\opencode\commands\
```

### Step 6: Restart OpenCode

Restart OpenCode for all changes to take effect.

## 🔄 Fallback Chains Explained

Fallback chains define the degradation order when a model fails:

```
Model A (primary) → Model B (fallback 1) → Model C (fallback 2)
```

**Trigger conditions**:
- ⏱️ Model response timeout (default 60 seconds)
- 🚫 API errors (429 Rate Limit, 500 Server Error, etc.)
- ❌ Model unavailable

**Configuration principles**:
- 📊 Each chain starts with a cheap model and escalates to stronger ones
- 🔗 The same model can appear in multiple chains
- ⚡ Shorter chains mean faster fallback

## 🎨 Customization

### Change the Default Model

Edit the `model` field in the corresponding `.md` file:

```markdown
---
model: openai/gpt-5.4    # Change to your preferred model
---
```

### Add New CE Commands

Create a new `.md` file in `~/.config/opencode/commands/`:

```markdown
---
description: Command description
argument-hint: "[argument hint]"
allowed-tools: Read, Write, Grep, Glob, Bash
skill: ce:corresponding-skill-name
model: YOUR_MODEL_ID
---

# /command-name — Command Title

Command execution instructions.
```

### Change Timeout

Edit the `timeoutMs` value in `opencode.json`:

```json
{
  "experimental": {
    "modelFallbackChain": {
      "timeoutMs": 30000  // 30 second timeout
    }
  }
}
```

## 🆘 Troubleshooting

### ❓ Commands not showing up?

1. ✅ Confirm `.md` files are in `~/.config/opencode/commands/`
2. ✅ Confirm filenames use lowercase letters and hyphens
3. ✅ Restart OpenCode

### ❓ Fallback not working?

1. ✅ Confirm `opencode-runtime-fallback` is installed: `npm list -g opencode-runtime-fallback`
2. ✅ Confirm `opencode.json` `plugin` array includes `"opencode-runtime-fallback"`
3. ✅ Confirm model IDs in `chains` are correct (verify with `/models`)
4. ✅ Restart OpenCode

### ❓ Model unavailable?

1. ✅ Use `/models` to view available models
2. ✅ Use `/connect` to configure a model provider
3. ✅ Check that your API Key is valid and has sufficient quota

### ⚠️ Compatibility with oh-my-openagent

`oh-my-openagent (omO)` has its own built-in runtime_fallback. If you install omO, you must uninstall `opencode-runtime-fallback` first to avoid conflicts. See [Fallback Plugin Guide](docs/fallback-plugin-guide.md) for details.

## 📊 Technical Details

- **Config format**: OpenCode official command file format (Markdown + YAML frontmatter)
- **Plugin mechanism**: Based on `opencode-runtime-fallback` plugin
- **Routing method**: Model specified via `model` field in command files
- **Fallback mechanism**: Automatic switching based on timeout and API errors
- **Model discovery**: Connect providers via `/connect`, view models via `/models`

## 🤝 Contributing

Issues and Pull Requests are welcome!

## 📜 License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

## 🙏 Acknowledgements

- [OpenCode](https://opencode.ai/) — AI coding assistant
- [opencode-runtime-fallback](https://www.npmjs.com/package/opencode-runtime-fallback) — Fallback plugin
- [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) — CE skill system
- All open-source model providers

---

> 💡 **Tip**: This configuration is fully customizable — adjust model assignments based on your own models and budget.
