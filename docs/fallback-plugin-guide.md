# OpenCode Fallback 插件操作指南

## 概述

本项目提供 `opencode-runtime-fallback` 插件的安装、使用和删除操作指南。

## 插件信息

- **插件名称**: opencode-runtime-fallback
- **npm 包**: opencode-runtime-fallback
- **功能**: 为 opencode 提供模型 fallback 链支持，当主模型失败时自动切换到备选模型
- **配置位置**: `~/.config/opencode/opencode.json`

## 安装

### 前置条件

- 已安装 Node.js 和 npm
- opencode 已安装并可正常运行

### 安装步骤

1. **安装 npm 包**

```bash
npm install -g opencode-runtime-fallback
```

或者在项目目录安装：

```bash
npm install opencode-runtime-fallback
```

2. **配置插件**

在 `~/.config/opencode/opencode.json` 的 `plugin` 数组中添加插件名称：

```json
{
  "$schema": "https://opencode.ai/config.json",
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

### 配置说明

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `timeoutMs` | number | 60000 | 等待时间（毫秒），超时后触发 fallback |
| `chains` | string[][] | (必填) | Fallback 链数组，每个链是一个模型 ID 数组 |

### Chains 配置示例

```json
{
  "experimental": {
    "modelFallbackChain": {
      "timeoutMs": 60000,
      "chains": [
        ["模型A", "模型B", "模型C"],
        ["模型X", "模型Y"]
      ]
    }
  }
}
```

**工作原理**：
- 如果"模型A"失败，自动切换到"模型B"，再失败则切换到"模型C"
- 第二个链独立运作，用于其他模型的 fallback

## 使用

### 验证安装

重启 opencode 后，执行以下命令验证插件是否正常工作：

```bash
opencode --version
```

检查是否有 fallback 相关的日志或错误信息。

### 触发 Fallback

Fallback 会在以下情况自动触发：
- 模型返回错误（如 429 Rate Limit、500 Server Error）
- 模型响应超时（超过 `timeoutMs` 配置的时间）
- 模型不可用

### 查看日志

Fallback 切换时，opencode 会显示类似信息：

```
[Fallback] 模型A 失败，切换到 模型B
[Fallback] 模型B 失败，切换到 模型C
[Fallback] 所有模型均失败
```

## 配置调整

### 修改 Fallback 链

编辑 `~/.config/opencode/opencode.json` 中的 `experimental.modelFallbackChain.chains` 部分。

### 修改超时时间

编辑 `timeoutMs` 值（单位：毫秒）：

```json
{
  "timeoutMs": 30000  // 30 秒超时
}
```

## 删除

### 步骤

1. **从配置中移除插件**

在 `~/.config/opencode/opencode.json` 中，从 `plugin` 数组中移除 `"opencode-runtime-fallback"`：

```json
{
  "plugin": [],
  "experimental": {
    "modelFallbackChain": {
      // 可以保留配置，但插件移除后将不生效
    }
  }
}
```

2. **卸载 npm 包**

```bash
npm uninstall -g opencode-runtime-fallback
```

或者在项目目录：

```bash
npm uninstall opencode-runtime-fallback
```

3. **重启 opencode**

```bash
# 停止 opencode
# 重启 opencode
opencode
```

### 完全清除配置

如果需要完全清除 fallback 配置，编辑 `~/.config/opencode/opencode.json`，删除整个 `experimental.modelFallbackChain` 部分：

```json
{
  "plugin": [],
  "experimental": {}
}
```

## 与 oh-my-openagent 的关系

### 重要说明

如果以后安装 `oh-my-openagent (omO)` 插件：
- **omO 内置了自己的 runtime_fallback 功能**
- 需要先卸载 `opencode-runtime-fallback`，避免冲突
- omO 的 fallback 配置在 `oh-my-opencode.json` 中，格式不同

### 卸载顺序

如果同时使用 omO 和 fallback 插件：

1. 先从 `opencode.json` 中移除 `opencode-runtime-fallback`
2. 重启 opencode
3. 再安装 omO
4. 在 `oh-my-opencode.json` 中配置 omO 的 fallback

## 故障排除

### Fallback 不生效

1. 确认插件已正确添加到 `plugin` 数组
2. 确认 `experimental.modelFallbackChain.chains` 配置格式正确
3. 检查 opencode 重启后配置是否生效

### 所有模型都失败

- 检查网络连接
- 确认模型 API key 配置正确
- 检查各模型的限额和配额

### 插件冲突

如果遇到奇怪的错误：
1. 尝试禁用所有插件，只保留 `opencode-runtime-fallback`
2. 逐步启用其他插件，排查冲突

## 参考链接

- npm 包: https://www.npmjs.com/package/opencode-runtime-fallback
- opencode 官方文档: https://opencode.ai/docs/
