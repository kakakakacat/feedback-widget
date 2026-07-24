# feedback-widget

一个零依赖、可复用的**浮动留言组件**：右下角一个 ✉️ 按钮，点开是留言弹窗，支持
文字留言 + 图片/文件（粘贴、拖拽、点击上传），直接发到你的邮箱。

- **纯前端**，一行 `<script>` 引入，适配任意静态站点
- 文字留言通过 [Web3Forms](https://web3forms.com) 发到你邮箱（免费）
- 附件走你自己的 **Cloudflare R2**（免费 10GB），邮件里附下载链接
- 空间不足时按上传时间**自动淘汰最老文件**
- 深色玻璃拟态弹窗，强调色可配置

## 快速开始（只发文字）

在页面底部加一行：

```html
<script
  src="/feedback-widget.js"
  data-access-key="你的_WEB3FORMS_ACCESS_KEY"
  data-site="我的网站"
  data-accent="#0071e3"></script>
```

去 [web3forms.com](https://web3forms.com) 用你的收件邮箱免费获取 access key。
**Web3Forms 免费版只支持文字**；要传文件，按下面配置 R2。

## 启用文件上传（Cloudflare Pages + R2）

1. 在 Cloudflare 控制台开通 **R2**（免费额度 10GB），创建一个 bucket，例如 `feedback-uploads`。
2. 把本仓库的 `functions/api/feedback/` 目录整个拷进你的 Pages 项目。
3. 在项目 `wrangler.toml` 里绑定该 bucket：

   ```toml
   [[r2_buckets]]
   binding = "FEEDBACK_BUCKET"
   bucket_name = "feedback-uploads"
   ```

4. 给 script 标签加上 `data-upload-url`，指向上传接口：

   ```html
   <script
     src="/feedback-widget.js"
     data-access-key="你的_WEB3FORMS_ACCESS_KEY"
     data-site="我的网站"
     data-accent="#0071e3"
     data-upload-url="/api/feedback/upload"></script>
   ```

配置 `data-upload-url` 后，弹窗才会出现文件上传区。上传的文件存进 R2，留言邮件里
带每个文件的下载链接（`/api/feedback/file/<随机id>`，链接不可猜测）。

## 配置项（script 的 data 属性）

| 属性 | 必填 | 说明 |
|---|---|---|
| `data-access-key` | 是 | Web3Forms 的 access key（前端公开使用，可随时后台重置） |
| `data-site` | 否 | 站点名，用于邮件标题/正文，默认取页面 `<title>` |
| `data-accent` | 否 | 强调色，默认 `#ff4c61` |
| `data-upload-url` | 否 | 附件上传接口地址；不填则隐藏文件功能、只发文字 |
| `data-max-file-mb` | 否 | 单文件大小上限（MB），默认 `100` |
| `data-turnstile-key` | 否 | Cloudflare Turnstile 站点密钥；配置后上传附件前需过人机验证 |

## 防刷 / 安全

上传接口是公开的，为避免被恶意刷爆 R2，内置了多层防护：

- **容量封顶 + 淘汰**：总量超过 `FEEDBACK_CAP_BYTES`（默认约 9.5GB）时，按上传时间删最老文件，永不超免费 10GB。
- **单请求限制**（环境变量可调）：`FEEDBACK_MAX_FILES`（默认 6）、`FEEDBACK_MAX_FILE_BYTES`（默认 25MB）、`FEEDBACK_MAX_REQUEST_BYTES`（默认 50MB）。
- **Cloudflare Turnstile（推荐）**：在控制台建一个 Turnstile widget，把 **Site Key** 填到 `data-turnstile-key`，把 **Secret Key** 设为 Pages 项目的加密环境变量 `TURNSTILE_SECRET`。配置后，上传接口会校验人机验证 token，机器人被拦在门外。仅拦附件上传（写 R2 的接口），纯文字留言不打扰。

## R2 容量与淘汰

- 上传接口 `functions/api/feedback/upload.js` 有容量上限 `FEEDBACK_CAP_BYTES`
  （环境变量，默认约 9.5GB，留在免费 10GB 以内）。
- 当新文件放不下时，按对象的上传时间**从最老的开始删**，直到腾出空间。
- 取文件接口 `functions/api/feedback/file/[key].js` 按原文件名流式返回。

## 说明

- 组件纯浏览器端运行；Web3Forms 免费版禁止服务器端调用，所以邮件必须由浏览器发出。
- access key 按设计就是放在前端的公开值，靠 Web3Forms 的垃圾过滤防滥用。

## 许可

MIT
