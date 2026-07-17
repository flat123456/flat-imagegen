# flat-imagegen

适用于 **非官方模型网关** 的 Codex 生图 skill。  
默认通过 OpenAI 兼容的 `Images API`（`POST /v1/images/generations`）把文生图 / 参考图编辑接到 `https://gateway.aimsg.uk/v1`；可选兼容 `Responses + image_generation`。

A short English blurb:

> A Codex image-generation skill for unofficial / gateway-hosted OpenAI-compatible models. It turns text prompts and reference images into local PNG/JPEG/WebP files through `POST /v1/images/generations` by default, with optional Responses compatibility mode.

适合这些场景：

- 你在用 Codex / 本地 agent，但图片能力不走官方 OpenAI 直连
- 你的模型服务是第三方 / 自建 / 网关转发的 OpenAI 兼容接口
- 你想要一个可脚本化、可落盘、可重复调用的本地生图 skill

## 简介

`flat-imagegen` 是一个最小可运行的 skill 包：

| 文件 | 作用 |
| --- | --- |
| `SKILL.md` | skill 入口：何时触发、默认参数、工作流 |
| `scripts/flat_image_gen.mjs` | 真正发请求、解析 SSE、保存图片的脚本 |
| `references/api.md` | API 请求形态与环境变量说明 |
| `auth.json` | 本地密钥（不提交） |
| `.gitignore` | 忽略密钥和生成结果 |

默认行为：

- Base URL：`https://gateway.aimsg.uk/v1`
- 默认 API 模式：`images`（/images/generations）
- 图片模型：`gpt-image-2`
- 兼容模式 Responses 模型：`gpt-5.5`
- 输出目录：`data/generated-images`
- 鉴权：`auth.json` 里的 `FLAT_API_KEY`，或环境变量 `FLAT_API_KEY`

## 在 Codex 中安装

### 方式 A：直接放进 skills 目录（最稳）

1. 克隆或下载本仓库：

```powershell
git clone https://github.com/flat123456/flat-imagegen.git
```

2. 把整个目录放到 Codex 可识别的 skills 根目录，例如：

```text
D:\Codex\skills\flat-imagegen\
  SKILL.md
  scripts\flat_image_gen.mjs
  references\api.md
  README.md
```

3. 在 skill 根目录创建 `auth.json`：

```json
{
  "FLAT_API_KEY": "your_gateway_token"
}
```

4. 重启 Codex 会话，或让当前会话重新加载 skills。
5. 直接说：

```text
用 flat-imagegen 生成一张 1024x1024 的产品海报
```

Codex 识别到 skill 后，会按 `SKILL.md` 调用本地脚本。

### 方式 B：从 GitHub 安装到本地 skills

如果你使用 Codex / Hermes 的 skill installer，可安装整个仓库目录，而不是只拷贝 `SKILL.md`：

```text
https://github.com/flat123456/flat-imagegen
```

安装后同样需要配置 `auth.json` 或 `FLAT_API_KEY`。

### 安装后检查

```powershell
node --check D:\Codex\skills\flat-imagegen\scripts\flat_image_gen.mjs
```

确认：

- 目录名与 skill 名一致：`flat-imagegen`
- `SKILL.md` 存在
- `scripts/flat_image_gen.mjs` 存在
- `auth.json` 已填 key，且未被提交

## 安装（通用）

把整个目录放进 Codex / Hermes 的 skills 目录，例如：

```text
<skills-root>/flat-imagegen/
  SKILL.md
  scripts/flat_image_gen.mjs
  references/api.md
```

然后创建本地密钥：

```json
{
  "FLAT_API_KEY": "your_gateway_token"
}
```

也可以：

```powershell
$env:FLAT_API_KEY = "your_gateway_token"
```

验证脚本语法：

```powershell
node --check scripts/flat_image_gen.mjs
```

## 快速使用

### 文生图

```powershell
node scripts/flat_image_gen.mjs `
  --prompt "A friendly robot mascot" `
  --size 1024x1024 `
  --output-format png
```

### 参考图编辑

```powershell
node scripts/flat_image_gen.mjs `
  --prompt "Keep the subject, replace the background with a bright studio setup" `
  --image .\reference.png `
  --size 1024x1536
```

### 透明背景

```powershell
node scripts/flat_image_gen.mjs `
  --prompt "A red sticker icon on a transparent background" `
  --image-model gpt-image-1.5 `
  --background transparent `
  --output-format png
```

### 指定输出目录

```powershell
node scripts/flat_image_gen.mjs `
  --prompt "poster concept" `
  --output-dir .\out\posters
```

## 为什么适合“非官方模型”

官方 Codex 内置生图路径通常绑定自家 provider。  
这个 skill 把生图能力拆成：

1. 本地 skill 触发
2. 脚本调用你配置的 OpenAI 兼容网关
3. 网关再转发到可用的图片模型

因此你可以：

- 换 base URL
- 换 responses / image model id
- 继续用 Codex 的 skill 工作流
- 不把业务绑死在单一官方账号路径上

常见自定义方式：

```powershell
$env:FLAT_IMAGEGEN_BASE_URL = "https://gateway.aimsg.uk/v1"
$env:FLAT_IMAGEGEN_RESPONSES_MODEL = "gpt-5.5"
$env:FLAT_IMAGEGEN_IMAGE_MODEL = "gpt-image-2"
```

或 CLI：

```powershell
node scripts/flat_image_gen.mjs `
  --base-url "https://gateway.aimsg.uk/v1" `
  --responses-model "gpt-5.5" `
  --image-model "gpt-image-2" `
  --prompt "..."
```

## 请求形态

默认脚本调用：

```http
POST /v1/images/generations
Authorization: Bearer <FLAT_API_KEY>
```

核心 payload 风格：

- `model`: 通常 `gpt-image-2`
- `prompt`
- `size`
- `n`
- `response_format: "b64_json"`
- 可选图生图：`image` / `extra_body.image`

兼容模式才使用：

```http
POST /v1/responses
```

返回的图片结果会解码并写到本地。

## 环境变量

| 变量 | 含义 |
| --- | --- |
| `FLAT_API_KEY` | API key |
| `FLAT_IMAGEGEN_BASE_URL` | Responses base URL |
| `FLAT_IMAGEGEN_RESPONSES_MODEL` | 外层 responses 模型 |
| `FLAT_IMAGEGEN_IMAGE_MODEL` | 图片工具模型 |
| `FLAT_IMAGEGEN_IMAGE_PROMPT` | 默认 prompt |
| `FLAT_IMAGEGEN_IMAGE_SIZE` | 尺寸，如 `1024x1024` |
| `FLAT_IMAGEGEN_IMAGE_OUTPUT_FORMAT` | `png` / `jpeg` / `webp` |
| `FLAT_IMAGEGEN_IMAGE_BACKGROUND` | `transparent` / `opaque` / `auto` |
| `FLAT_IMAGEGEN_IMAGE_QUALITY` | 质量参数 |
| `FLAT_IMAGEGEN_IMAGE_COUNT` | 次数 |
| `FLAT_IMAGEGEN_TIMEOUT_MS` | 超时毫秒 |
| `FLAT_IMAGEGEN_OUTPUT_DIR` | 输出目录 |

## 安全注意

- 不要提交 `auth.json`
- 不要把 token 写进 skill 文档或脚本
- 参考图会 Base64 上传到第三方网关；私密素材请先确认可外传
- 生图结果默认落在本地 `data/generated-images/`

## 仓库状态

当前仓库已按 Flat / gateway 路径整理：

- skill 名：`flat-imagegen`
- 执行脚本：`scripts/flat_image_gen.mjs`
- 默认网关：`https://gateway.aimsg.uk/v1`

如果你只是想在 Codex 里用“非官方模型也能稳定生图”的 skill，放进 skills 目录、配好 `FLAT_API_KEY` 即可。