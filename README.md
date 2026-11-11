# OpenCode 模型列表（静态页面）

一个零依赖的单文件静态页面，实时请求 `https://opencode.ai/inference/v1/models`，
以表格展示 opencode 推理服务的全部模型，并提供多维过滤、排序、分页与详情查看。

## 文件

```
index.html   # 全部内容（HTML + CSS + JS 单文件，无构建、无依赖）
```

## 运行

接口已开启 CORS（`Access-Control-Allow-Origin: *`），因此**直接双击打开 `index.html` 即可使用**。

若浏览器策略拦截 `file://` 下的请求，用任意静态服务器即可：

```bash
python3 -m http.server 8080
# 打开 http://localhost:8080/index.html
```

或用 Node：

```bash
npx serve .
```

## 功能

### 数据
- 打开页面自动拉取模型列表，失败时给出可读的错误提示与重试按钮
- 顶部统计卡片：模型总数、提供商数量、免费模型数、开放权重数、最大上下文窗口、最后更新时间
- 「刷新」按钮重新拉取；「复制 JSON」导出全部原始数据

### 过滤（可组合）
| 维度 | 说明 |
| --- | --- |
| 关键字搜索 | 匹配名称 / ID / 描述 / 系列 / 规范 ID / 提供商（按 `/` 快速聚焦） |
| 提供商 | 由 `canonical_id` 前缀与 `family` 推导，共 18 家 |
| 模型系列 | 如 `claude-opus`、`gpt-codex` |
| 输入模态 | 图片 / 音频 / 视频 / 文档(PDF) / 仅文本 |
| API 格式 | `chat-completions`、`responses`、`messages`(Anthropic)、`generate-content`(Google) |
| 价格 | 免费（$0）/ 付费 |
| 上下文窗口 | ≥128K / ≥200K / ≥400K / ≥1M |
| 能力 | 工具调用、推理、结构化输出、可调推理强度、开放权重 |

已选条件会以标签形式展示，点击标签上的 ✕ 可单独移除；「重置筛选」一键清空。

### 排序与分页
- 点击表头 `模型 / 提供商 / 上下文窗口 / 最大输出 / 价格 / 发布时间` 切换升/降序
- 每页 25 / 50 / 100 条或全部显示，底部分页器支持省略号跳转

### 详情弹窗
点击任意行或「详情」按钮打开，含 5 个标签页：

- **概览** — 基本信息卡片（模型 ID、规范 ID、提供商、上下文、最大输入/输出、知识截止、发布/更新时间），并可一键复制模型 ID / 本模型 JSON / base_url
- **能力与模态** — 输入与输出模态完整列表、能力开关（工具、推理、推理强度档位、结构化输出、temperature/top_p/top_k 等）、Prompt Cache 配置
- **价格** — 输入 / 缓存读 / 缓存写 / 输出单价，阶梯定价（overrides）、系数调整（multipliers）及完整 `cost` 结构
- **API 接入** — 每种协议的 `base_url`、AI SDK 包名、`tool_choice` 选项、strict 模式、推理字段等
- **原始 JSON** — 接口返回的完整对象

### 交互细节
- 主题切换：浅色 / 深色 / 跟随系统，记忆在 `localStorage`
- `Esc` 关闭弹窗，`/` 聚焦搜索框
- 请求失败、复制成功均有 toast 提示
- 表格窄屏下横向滚动，弹窗自适应移动端

## 字段说明

| 字段 | 用途 |
| --- | --- |
| `id` | opencode 侧的模型 ID，用于调用 |
| `canonical_id` | 上游规范 ID（如 `anthropic/claude-opus-4-6`），从中提取提供商 |
| `family` | 模型系列 |
| `limit.context` / `limit.input` / `limit.output` | 上下文窗口与最大输入/输出 tokens |
| `cost.text.input/output.price` | 每 1M tokens 价格，`overrides` 为阶梯价，`multipliers` 为系数 |
| `capabilities` | 工具调用、推理、结构化输出、参数开关、缓存等 |
| `modalities.input` | 支持的输入 MIME 类型（文本/图片/音频/视频/PDF） |
| `api.*` | 各协议接入信息（`chat-completions` / `responses` / `messages` / `generate-content`） |

## 部署到 GitHub Pages

站点是零依赖的纯静态页面，`index.html` 位于仓库根目录，可直接由 GitHub Pages 托管。

### 1. 推送代码

```bash
git init
git add .
git commit -m "feat: opencode 模型列表静态页面"
git branch -M main
git remote add origin git@github.com:<你的用户名>/<仓库名>.git
git push -u origin main
```

### 2. 开启 Pages

**方式 A：GitHub Actions（推荐，自动部署）**

仓库已内置 `.github/workflows/deploy.yml`。进入
`Settings → Pages → Build and deployment → Source` 选择 **GitHub Actions**，
保存后即可在 `Actions` 面板看到 `Deploy to GitHub Pages` 任务自动运行。
之后每次 push 到 `main` 都会自动重新发布。

**方式 B：从分支部署（不需要 Actions）**

在 `Settings → Pages → Build and deployment → Source` 选择 **Deploy from a branch**，
分支选 `main`、目录选 `/ (root)`，保存即可。

### 3. 访问地址

```
https://<你的用户名>.github.io/<仓库名>/
```

> 首次发布需要 1～2 分钟构建，期间访问会显示 "site is being deployed"。
> 若 404，请确认仓库是 **public**（免费账户私有仓库不支持 Pages），
> 或等待 Pages 设置中的 DNS/环境检查完成。

### 说明

- 页面通过浏览器直接请求 `https://opencode.ai/inference/v1/models`（接口已开启 CORS），
  数据是实时的，GitHub Pages 只负责托管 HTML 本身。
- `.nojekyll` 用于在分支部署时跳过 Jekyll 处理，避免隐藏文件或下划线目录被忽略。
- 主题等状态存于浏览器 `localStorage`，与部署方式无关。
