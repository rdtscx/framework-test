# 框架测试

兼容 OpenAI 与 Anthropic 接口的模拟服务，用于客户端联调、流式渲染和错误处理测试。请求不会调用真实模型，返回内容是 `ascii/` 目录下随机一份字符画。

## 使用方式

```bash
git clone https://github.com/rdtscx/framework-test.git
cd framework-test
npm install
FAKE_API_KEY=sk-your-key node server.js
```

服务默认监听 `http://localhost:3000`。把客户端的 Base URL 指向该地址，Key 填 `FAKE_API_KEY`：

```bash
curl http://localhost:3000/v1/chat/completions \
  -H "Authorization: Bearer sk-your-key" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-6.1-sol","messages":[{"role":"user","content":"你好"}]}'
```

部署到线上：

- Cloudflare Workers：`npx wrangler secret put FAKE_API_KEY`，然后 `npm run deploy:cf`
- Vercel：导入本仓库直接部署，`vercel.json` 已配置好 rewrite

## 相关网站

- 仓库：<https://github.com/rdtscx/framework-test>
- OpenAI API 文档：<https://platform.openai.com/docs>
- Anthropic API 文档：<https://docs.anthropic.com>
- Cloudflare Workers 文档：<https://developers.cloudflare.com/workers/>
- Vercel 文档：<https://vercel.com/docs>

## 接口

| 路径 | 说明 |
|---|---|
| `GET /v1/models` | 模型名列表 |
| `POST /v1/chat/completions`、`/v1/completions` | OpenAI |
| `POST /v1/responses` | OpenAI Responses API |
| `POST /v1/messages`、`/v1/messages/count_tokens` | Anthropic |
| `GET /health` | 健康检查 |
| `GET /` | 说明页 |

路径带或不带 `/api` 前缀都可以。鉴权支持 `Authorization: Bearer` 与 `x-api-key`。

## 行为

- 回答内容来自 `ascii/` 下随机一份字符画，默认用 markdown 代码块包裹。
- 问候词（`hello` / `你好` / `测试` / `hi`）返回英文问候。
- 提示中出现 “use … tool” 时返回一次 `tool_call`；客户端回传工具结果后继续回应。
- 支持 `stop` / `max_tokens` 截断，流式分片大小与间隔在 `config.js` 调整。

## 配置

全部在 `config.js`：

- `apiKey`：留空，使用环境变量 `FAKE_API_KEY`
- `models`：`/v1/models` 返回的展示列表，不限制请求里的模型名
- `requireAuth`：是否校验 Key，默认开启
- `streamChunkChars` / `streamDelayMs`：流式分片与延迟
- `debug`：请求日志

## 许可证

GPL-3.0，见 [LICENSE](LICENSE)。
