# CE 模型自动路由 - 需求文档

## Problem Frame

用户有多个模型（MiniMax 2.7、GLM-5.1、DeepSeek-V4-Pro、Claude Opus 4.7、GPT-5.4 等），但在提问时不知道哪个模型最适合。期望在执行 `/ce:*` 命令时，系统能根据任务类型和复杂度自动选择合适的模型。

**受影响：** 所有使用 compound-engineering skill 的用户交互

**核心诉求：**
1. 命令级路由 — 不同 `/ce:*` 命令使用不同基础模型
2. 复杂度分级 — 复杂任务在命令内升级到更强模型
3. 轻量实现 — 不依赖重型插件（如 oh-my-openagent），保留现有的 plan/build agent 配置

## Requirements

**模型映射（已确认）**

| 用途 | 偏好模型 | 实际 provider/model ID |
|------|---------|----------------------|
| 快速/轻量任务 | MiniMax 2.7 | `minimax-cn-coding-plan/MiniMax-M2.7` |
| 主力模型 | GLM-5.1 | `zhipuai-coding-plan/glm-5.1` |
| 深度推理 | DeepSeek-V4-Pro | `opencode-go/deepseek-v4-pro` |
| 高强度任务 | Claude Opus 4.7 | `yunwu-claude/claude-opus-4-7` |
| 代码生成 | GPT-5.4 | `yunwu-gpt/gpt-5.4` |

**模型分配策略（已确认）**

| 命令 | 基础模型 | 升级触发条件 | Fallback 链 |
|------|---------|-------------|-------------|
| ce:brainstorm | MiniMax 2.7 | 深度推理/反事实推演 → GLM-5.1 | MiniMax 2.7 → GLM-5.1 |
| ce:ideate | MiniMax 2.7 | 颠覆性/突破性方案 → GLM-5.1 | MiniMax 2.7 → GLM-5.1 |
| ce:plan | GLM-5.1 | 极简单任务 → MiniMax 2.7；安全/分布式架构 → DeepSeek-V4-Pro | GLM-5.1 → DeepSeek-V4-Pro → Claude Opus 4.7 |
| ce:work | GLM-5.1 | 简单模板 → MiniMax 2.7；高难度编码 → DeepSeek-V4-Pro | GLM-5.1 → DeepSeek-V4-Pro → Claude Opus 4.7 |
| ce:review | MiniMax 2.7 | 安全/性能相关 → DeepSeek-V4-Pro；发布前/关键基础设施 → Claude Opus 4.7 | MiniMax 2.7 → DeepSeek-V4-Pro → Claude Opus 4.7 |

**R1.** 在 `opencode.json` 中添加 MiniMax provider 配置

**R2.** 在 `opencode.json` 中添加 DeepSeek provider 配置（如果 DeepSeek API 未配置）

**R3.** 为每个 `/ce:*` 命令创建 command 文件，指定对应的 model

**R4.** 在 agent 配置中设置 fallback_models 链，实现 API 错误降级

**R5.** Skill 内置的 subagent 委派机制保持不变 — ce:plan 和 ce:work 已有的复杂度分级（Inline/Serial subagent/Parallel subagent）自动工作

## Success Criteria

- 用户执行 `/ce:brainstorm` 时，系统使用 MiniMax 2.7
- 用户执行 `/ce:plan` 时，系统使用 GLM-5.1
- 复杂任务（ce:work 判断为 Large）自动 spawn subagent
- API 错误时自动降级到 fallback 链中的下一个模型
- 不安装 oh-my-openagent 插件，保持轻量

## Scope Boundaries

- 不使用 oh-my-openagent 的 Category 系统做运行时动态升级
- 不修改 compound-engineering skill 文件（ce:plan、ce:work 等）
- 不实现 JS 路由脚本（opencode 不支持）

## Key Decisions

- **纯配置方案**：通过 `opencode.json` 的 agent 配置和 command 文件实现模型路由
- **Fallback 链**：作为 API 错误的降级机制，而非复杂度路由机制
- **命令文件路由**：为每个 ce:* 创建 .md command 文件，指定 model

## Dependencies / Assumptions

- MiniMax 2.7 已有配置：`minimax-cn-coding-plan/MiniMax-M2.7`
- GLM-5.1 已配置：`zhipuai-coding-plan/glm-5.1`
- DeepSeek-V4-Pro：`opencode-go/deepseek-v4-pro`（需确认 provider 配置）
- Claude Opus 4.7 已配置：`yunwu-claude/claude-opus-4-7`
- GPT-5.4 已配置：`yunwu-gpt/gpt-5.4`
