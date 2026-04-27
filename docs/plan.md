# CE 模型自动路由配置 - 实施计划

## Overview

在 opencode 中配置模型自动路由，使不同的 `/ce:*` 命令使用不同的基础模型，并通过 fallback 链实现 API 错误时的自动降级。

## Problem Frame

用户有多个模型（MiniMax 2.7、GLM-5.1、DeepSeek-V4-Pro、Claude Opus 4.7、GPT-5.4 等），需要在执行 `/ce:*` 命令时自动选择合适的模型。

**技术可行性确认：**
- Command 文件 `model` 字段：✅ 可用（官方文档确认）
- `fallback_models`：❌ 原生不支持，需安装 `opencode-runtime-fallback` 插件
- MiniMax 2.7：✅ 已通过 `/connect` 连接
- GLM-5.1：✅ `zhipuai-coding-plan/glm-5.1` 可用（opencode 内置 provider）
- DeepSeek-V4-Pro：✅ 已通过 `/connect` 连接

## Requirements Trace

- R1. 安装 `opencode-runtime-fallback` 插件
- R2. 在 command 文件中为每个 `/ce:*` 命令指定对应的 model
- R3. 配置 fallback 链，实现 API 错误降级

## Scope Boundaries

- 不修改 compound-engineering skill 文件
- 不使用 oh-my-openagent 的 Category 系统

## Key Technical Decisions

- **Command 文件路由**：为每个 `/ce:*` 创建 `.md` 命令文件，通过 frontmatter 的 `model` 字段指定模型
- **Fallback 链**：通过 `opencode-runtime-fallback` 插件实现，配置在 `opencode.json` 的 `experimental.modelFallbackChain` 中
- **Agent 配置**：不修改，保持现有的 compound-engineering agent 配置

## Implementation Units

### Unit 1: 安装 opencode-runtime-fallback 插件

**Goal:** 安装并配置 fallback 插件

**Files:**
- Modify: `~/.config/opencode/opencode.json`

**Approach:**
```bash
npm install opencode-runtime-fallback
```

在 `opencode.json` 中添加：
```json
{
  "plugin": ["opencode-runtime-fallback"],
  "experimental": {
    "modelFallbackChain": {
      "timeoutMs": 60000,
      "chains": [
        ["minimax-cn-coding-plan/MiniMax-M2.7", "zhipuai-coding-plan/glm-5.1"],
        ["zhipuai-coding-plan/glm-5.1", "opencode-go/deepseek-v4-pro", "yunwu-claude/claude-opus-4-7"],
        ["yunwu-claude/claude-opus-4-7", "yunwu-gpt/gpt-5.4"]
      ]
    }
  }
}
```

**Verification:**
- 插件安装成功
- 配置写入成功

### Unit 2: 为 ce:brainstorm 创建 command 文件

**Goal:** 创建 `/ce:brainstorm` 命令文件，指定 MiniMax 2.7 为基础模型

**Files:**
- Create: `~/.config/opencode/commands/ce-brainstorm.md`

### Unit 3: 为 ce:ideate 创建 command 文件

**Goal:** 创建 `/ce:ideate` 命令文件

**Files:**
- Create: `~/.config/opencode/commands/ce-ideate.md`

### Unit 4: 为 ce:plan 创建 command 文件

**Goal:** 创建 `/ce:plan` 命令文件

**Files:**
- Create: `~/.config/opencode/commands/ce-plan.md`

### Unit 5: 为 ce:work 创建 command 文件

**Goal:** 创建 `/ce:work` 命令文件

**Files:**
- Create: `~/.config/opencode/commands/ce-work.md`

### Unit 6: 为 ce:review 创建 command 文件

**Goal:** 创建 `/ce:review` 命令文件

**Files:**
- Create: `~/.config/opencode/commands/ce-review.md`

### Unit 7: 验证配置生效

**Goal:** 验证模型路由配置生效

**Approach:**
- 执行 `/ce:brainstorm` 命令，确认使用 MiniMax 2.7
- 执行 `/ce:plan` 命令，确认使用 GLM-5.1
- 检查 fallback 链配置是否正确加载

**Verification:**
- 命令执行成功
- 模型选择符合预期

## Risks & Dependencies

| Risk | Mitigation |
|------|------------|
| Fallback 插件与 omO 冲突 | 如果以后安装 omO，需先卸载此插件（omO 内置 fallback） |
| 模型 ID 格式不正确 | 确认所有模型 ID 的正确格式 |
