---
description: "小红书信息搜集。/xhs = 按预设 topic（AI/Lean/数学/教职）搜索小红书热门内容，整理带链接的摘要发 Telegram。/xhs <关键词> = 自定义搜索。/xhs detail <noteId> = 拉单篇帖子全文+评论。依赖 xiaohongshu-mcp server (localhost:18060)。Trigger: /xhs, '搜小红书 / 看看小红书 / 小红书有什么新的 / xhs 搜一下'."
user-invocable: true
---

# /xhs — 小红书信息搜集

通过 xiaohongshu-mcp HTTP API 搜索小红书内容，整理成带链接的摘要。

## 前置条件

MCP server 必须在跑：`http://localhost:18060/health` 返回 healthy。
如果没跑，启动：`~/repos/xiaohongshu/start-mcp.sh`
首次需要扫码登录：`~/repos/xiaohongshu/bin/xiaohongshu-login-darwin-arm64`

## 触发词

`/xhs`, "搜小红书", "看看小红书", "小红书有什么新的", "xhs 搜一下"

## 子命令

### `/xhs`（无参数）— 全 topic 扫描

按预设 topic 列表搜索，每个 topic 取热门帖子，整理后发 Telegram。

预设 topic：

| 类别 | 搜索关键词 |
|------|-----------|
| AI/Lean | `AI数学 Lean 形式化`, `数学AI 定理证明` |
| 数学 | `纯数学 研究`, `数学家` |
| 教职 | `教职 助理教授 tenure track` |

**每个 topic 搜完后立刻发一条 Telegram，不要攒到最后一起发。**

### `/xhs <关键词>` — 自定义搜索

用给定关键词搜索，返回结果。

### `/xhs detail <noteId>` — 帖子详情

拉单篇帖子的全文内容和评论。noteId 从搜索结果中获取。

### `/xhs topics` — 查看/编辑预设 topic

显示当前 topic 列表。爸爸可以说"加一个 topic: xxx"或"删掉 xxx topic"来调整。
Topic 列表存在 `~/repos/xiaohongshu/topics.json`。

### `/xhs start` / `/xhs stop` — 启动/停止 MCP server

启动或停止 xiaohongshu-mcp 后台进程。

## API 调用方式

搜索和详情都通过 curl 调用 HTTP API（不依赖 MCP 协议，不需要重启 session）：

```bash
# 搜索
curl -s "http://localhost:18060/api/v1/feeds/search?keyword=<URL编码关键词>"

# 帖子详情（需要从搜索结果中拿 feed_id 和 xsec_token）
curl -s -X POST "http://localhost:18060/api/v1/feeds/detail" \
  -H "Content-Type: application/json" \
  -d '{"feed_id":"<id>","xsec_token":"<token>"}'

# 登录状态检查
curl -s "http://localhost:18060/api/v1/login/status"

# 健康检查
curl -s "http://localhost:18060/health"
```

## 输出格式

每条帖子输出：
```
标题 (N赞 M藏)
https://www.xiaohongshu.com/explore/<noteId>
一句话摘要（从 desc 提取核心信息）
```

**必须带链接。** 链接格式固定：`https://www.xiaohongshu.com/explore/<noteId>`
手机上点击会跳转小红书 App 打开。

按 topic 分组，每组一条 Telegram 消息。

## 帖子详情输出格式

```
标题
作者: xxx | N赞 M藏 K评
链接: https://www.xiaohongshu.com/explore/<noteId>

全文内容...

热评:
  用户A: 评论内容
  用户B: 评论内容
```

## 注意事项

- 搜索之间间隔 1-2 秒，不要太快
- 详情 API 需要浏览器渲染，较慢（每篇 5-10 秒），批量拉详情时要耐心
- xsec_token 和 feed_id 是配对的，必须用同一次搜索返回的 token
- 如果详情返回 `GET_FEED_DETAIL_FAILED`，重新搜索拿新 token 再试
- cookie 过期时需要重新扫码登录（`~/repos/xiaohongshu/bin/xiaohongshu-login-darwin-arm64`）
- 去重：同一批搜索中 id 含 `-` 的是广告，跳过；同 id 跨 topic 出现只保留一次

## 文件结构

```
~/repos/xiaohongshu/
├── bin/                          # 预编译二进制 + cookies
├── start-mcp.sh                  # 启动脚本
├── topics.json                   # 预设 topic 列表
├── xiaohongshu-mcp/              # MCP server 源码
├── MediaCrawler/                 # Python 批量爬虫（TODO）
├── XHS-Downloader/               # 媒体下载器（TODO）
└── TODO.md
```
