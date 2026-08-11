---
name: recognize-image
description: |
  使用 vision MCP 工具识图/分析图片。当用户让你看图、识图、识别图片内容，或在结合原型图 / UI 稿 / 截图 / 设计稿进行开发时使用。当前模型非多模态（如 GLM-5.2）但用户贴了图或需要"看"图时也触发。支持本地文件路径、图片 URL、剪贴板粘贴三种输入。
allowed-tools:
  - mcp__plugin_image-recognition_vision__recognize_image
argument-hint: "[图片路径或URL（或先用 Win+Shift+S 截图到剪贴板）] [你想让它做什么]"
---

# 识图技能（recognize-image）

本技能通过 `recognize_image` 工具调用一个 OpenAI 兼容的视觉模型，让当前（可能非多模态的）会话获得"看图"能力。

## 何时触发

- 用户提到：看图 / 识图 / 识别 / 这张图 / 原型图 / UI 稿 / 截图 / 设计稿 / mockup
- 用户说"我刚截图了"、"复制到剪贴板了"、"这张图里"
- 需要把图片内容转成代码 / 需求 / 文档，但当前模型无法直接读图
- 用户贴了本地图片路径或图片 URL

## 工具用法

调用 `recognize_image`，参数：

- `image`（必填）：三种取值
  1. **本地路径**：如 `D:/proto/login.png` 或 `./hero.png`
  2. **图片 URL**：如 `https://example.com/cover.jpg`
  3. **剪贴板**：传字面量 `clipboard`（请先让用户截图/复制图片到剪贴板）
- `prompt`（可选，默认"详细描述这张图片"）：你想让视觉模型回答/做什么的指令
- `model` / `detail` / `max_tokens`（可选）：覆盖默认配置

返回：视觉模型对该图的文本回答。

## 场景化 prompt 范本

根据任务在 prompt 里套用对应模板，见 references/：

- **UI 稿 / 截图 → 前端代码**：`references/ui-to-code.md`
- **原型图 → 需求清单**：`references/prototype-to-req.md`
- **通用图片描述**：`references/generic-describe.md`

## 行为准则

1. **先确认输入**：如果用户只说"看这张图"但没给路径/URL，要么向用户索取，要么提示用户截图后用 `clipboard`。
2. **拿到结果再做后续推理**：工具返回的是模型对图的描述/回答，你（主模型）再基于此推理、写代码、出方案。不要假装自己能直接看到图。
3. **面向前端开发**：结合当前 workspace 的代码上下文（框架、组件库、目录结构）给出可落地建议，而不是脱离上下文的通用代码。
4. **错误处理**：若工具报错提示缺 `IMAGE_RECOGNITION_MODEL` / `API_KEY` / `BASE_URL`，引导用户在 Claude Code 设置里配置环境变量后重试。
5. **大图提醒**：超大图 base64 后体积膨胀，工具会按 `IMAGE_RECOGNITION_MAX_FILE_MB`（默认 15MB）拦截，提示用户裁剪或压缩。
