# xhs skill — DIGEST (auto-generated, do not edit)
# Full version: ~/repos/zinan-skills/skills/xhs.md


## 前置条件
MCP server 必须在跑：`http://localhost:18060/health` 返回 healthy。

## 触发词
`/xhs`, "搜小红书", "看看小红书", "小红书有什么新的", "xhs 搜一下"

## 子命令

### `/xhs`（无参数）— 全 topic 扫描
按预设 topic 列表搜索，每个 topic 取热门帖子，整理后发 Telegram。
  Key: 每个 topic 搜完后立刻发一条 Telegram，不要攒到最后一起发。

### `/xhs <关键词>` — 自定义搜索
用给定关键词搜索，返回结果。

### `/xhs detail <noteId>` — 帖子详情
拉单篇帖子的全文内容和评论。noteId 从搜索结果中获取。

### `/xhs topics` — 查看/编辑预设 topic
显示当前 topic 列表。爸爸可以说"加一个 topic: xxx"或"删掉 xxx topic"来调整。

### `/xhs start` / `/xhs stop` — 启动/停止 MCP server
启动或停止 xiaohongshu-mcp 后台进程。

## API 调用方式
搜索和详情都通过 curl 调用 HTTP API（不依赖 MCP 协议，不需要重启 session）：

## 输出格式
每条帖子输出：

## 帖子详情输出格式
标题

## 注意事项
- 搜索之间间隔 1-2 秒，不要太快

## 文件结构
~/repos/xiaohongshu/
