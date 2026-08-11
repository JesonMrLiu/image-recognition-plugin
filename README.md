# image-recognition — Claude Code 识图插件

让 Claude Code 里的**非多模态模型**（如 GLM-5.2）也能"看图"。

通过一个走 **OpenAI 兼容协议**的识图 MCP server，把任意 OpenAI 兼容的视觉模型（GLM-4V、Qwen-VL、GPT-4o 等）接入 Claude Code。配套一个 skill，在你看图、识图、结合原型图/UI 稿开发时自动触发。

## 仓库关系（两个独立 git 仓库）

| 仓库 | 作用 | 本仓库? |
|---|---|---|
| **image-recognition**（本仓库） | Claude Code 插件：skill + 插件清单，通过 `npx` 引用 MCP server | ✅ |
| **image-recognition-mcp** | 独立 MCP server（TypeScript npm 包 `claude-image-recognition-mcp`） | ❌ 另一个仓库 |

本仓库**不含任何 JS 代码**。插件清单里的 `mcpServers.vision` 用 `npx -y claude-image-recognition-mcp@^0.1.0` 拉取已发布的 server，因此 git 安装本插件后开箱即用，无需 `npm install` / 构建步骤。

> MCP server 仓库地址：<在此填 image-recognition-mcp 的 git 地址>
> 如需修改 server 行为，去那个仓库改源码、发新版、再升本仓库 `plugin.json` 里 `@^x.y.z` 的版本号。

## 目录结构

```
image-recognition/
├─ .claude-plugin/
│  ├─ plugin.json          # 插件清单（mcpServers.vision → npx claude-image-recognition-mcp）
│  └─ marketplace.json     # 本地 / git 分发清单
└─ skills/recognize-image/
   ├─ SKILL.md             # 触发识图的 skill（命名空间 mcp__plugin_image-recognition_vision__recognize_image）
   └─ references/          # 场景化 prompt 模板（UI→代码 / 原型→需求 / 通用描述）
```

## 配置环境变量

所有变量以 `IMAGE_RECOGNITION_` 为前缀，由本插件的 `plugin.json` 注入到 server。

| 变量 | 必填 | 默认 | 说明 |
|---|---|---|---|
| `IMAGE_RECOGNITION_API_KEY` | ✅ | — | 视觉模型的 API Key |
| `IMAGE_RECOGNITION_BASE_URL` | ✅ | — | OpenAI 兼容根，如 `https://open.bigmodel.cn/api/paas/v4` |
| `IMAGE_RECOGNITION_MODEL` | ✅ | — | 视觉模型 id，如 `glm-4v-plus` |
| `IMAGE_RECOGNITION_DETAIL` | ❌ | `high` | `low` / `high` / `auto` |
| `IMAGE_RECOGNITION_MAX_TOKENS` | ❌ | `2048` | 64–8192 |
| `IMAGE_RECOGNITION_TIMEOUT_MS` | ❌ | `60000` | 单请求超时 |
| `IMAGE_RECOGNITION_DOWNLOAD_URL` | ❌ | `0` | 置 `1` 时把 URL 图片下载转 base64 |
| `IMAGE_RECOGNITION_MAX_FILE_MB` | ❌ | `15` | 本地图片体积上限 |

> 常见 base_url：智谱 GLM `https://open.bigmodel.cn/api/paas/v4`；OpenAI `https://api.openai.com/v1`；阿里 DashScope 兼容模式 `https://dashscope.aliyuncs.com/compatible-mode/v1`。

## 安装

### 方式一：本地 marketplace

```
/plugin marketplace add <本仓库本地路径或 git 地址>
/plugin install image-recognition
```

### 方式二：项目级启用

在项目 `.claude/settings.local.json`：

```jsonc
{ "enabledMcpjsonServers": ["vision"] }
```

### 配置环境变量

在系统环境变量或 Claude Code env 注入里设置三个必填项（示例为智谱 GLM-4V）：

```bash
export IMAGE_RECOGNITION_API_KEY=你的key
export IMAGE_RECOGNITION_BASE_URL=https://open.bigmodel.cn/api/paas/v4
export IMAGE_RECOGNITION_MODEL=glm-4v-plus
```

## 使用

在 Claude Code 里直接说：

> "看下这张图 D:/proto/login.png，把登录表单转成 React 组件"

会自动触发 `recognize-image` skill → 调用 `recognize_image` 工具。`/mcp` 里可确认 `vision` server 已 connected。

插件形态下工具命名空间：`mcp__plugin_image-recognition_vision__recognize_image`。

## 不装插件、只用 MCP server

见 **image-recognition-mcp** 仓库的 README（独立 `npm` 包，可单独在 `.mcp.json` 配置）。

## 许可证

MIT
