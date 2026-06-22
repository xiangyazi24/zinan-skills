---
description: "Skill self-improvement loop. /self-improve <skill-name> = 对指定 skill 执行温习+总结+改进。三步：(1) 重新加载 skill 文档刷新约束，(2) 反思本 session 经验，(3) 提炼泛化教训写入 skill 的 ## Learned Tactics 区。可手动触发，也可由 cron 每 3 小时自动注入活跃 session。Trigger: /self-improve, '温习 skill / skill 进化 / 总结经验到 skill / skill self-improve'."
user-invocable: true
---

# /self-improve — Skill Self-Improvement Loop

让 skill 从实战中进化。每次触发执行三步：温习 → 总结 → 改进。

## 触发方式

1. **手动**：`/self-improve lean`（或任何 skill 名）
2. **Cron 自动**：每 3 小时注入活跃的 research/dm session（见下方 Cron 配置）
3. **PostCompact hook**：context 压缩后自动提示重新加载当前 skill
4. **口头**："温习一下 lean skill"、"skill 进化"、"总结经验到 skill 里"

## 三步流程

### Step 1: 温习（Refresh）

重新加载目标 skill 文档，强制刷新约束记忆。

```
读 ~/repos/zinan-skills/skills/<skill-name>.md（完整读，不跳）
```

读完后在心里过一遍：**这个 skill 的核心约束是什么？我这几个小时有没有偏离？** 如果有明显偏离，在 Telegram 简短报告："温习完毕，发现我 drift 了 XXX，已纠正。"

### Step 2: 总结（Reflect）

回顾本 session 中与该 skill 相关的工作，提取候选经验。

**提问自己这些问题（内部思考，不输出）：**

1. 这次 session 里我踩了什么坑？根因是什么？
2. 哪个做法特别有效，以前 skill 里没写？
3. 爸爸纠正了我什么？（feedback 类，最高优先级）
4. 有没有反复出现的模式，值得编码成规则？

**过滤标准——只有同时满足这三条才算"真经验"：**

- **可泛化**：去掉具体的文件名、变量名、项目名后，这条经验对同一 skill 的未来所有 session 仍然有用
- **非重复**：skill 文档里还没有实质等价的规则
- **经过验证**：这次 session 里确实验证了这条经验（不是猜测或"感觉应该这样"）

**硬性排除：**
- 具体的工程细节（某个文件路径、某次 build 的错误信息、某个具体的 sorry）
- 只适用于当前项目的特定知识（放 UNDERSTANDING.md 而不是 skill）
- 重复已有规则（哪怕换了措辞）

### Step 3: 改进（Amend）

如果 Step 2 产生了通过过滤的候选经验（**没有就不写，宁缺毋滥**），追加到 skill 文档末尾的 `## Learned Tactics (Self-Improvement)` section。

**写入格式：**
```markdown
## Learned Tactics (Self-Improvement)

_This section is maintained by the self-improvement loop. Each entry was extracted
from real session experience and passed the generalizability filter._

### [YYYY-MM-DD] 简短标题
一两句话描述规则本身。
**Why:** 触发这条经验的场景（泛化描述，不带具体工程细节）。
**How to apply:** 什么时候/什么条件下应该想起这条规则。
```

**写入纪律：**
- 每次最多加 1-3 条。不是写日记，是提炼规则。
- 新条目追加到 section 末尾，不改动已有条目（除非发现已有条目是错的——那就修正它）。
- 如果 section 超过 30 条（~500 行），触发一次"压缩"：合并重叠的条目，删除被后来条目 supersede 的旧条目。
- 写完后 `cd ~/repos/zinan-skills && git add skills/<name>.md && git commit -m "self-improve(<name>): <一句话>"` 落盘。

### 完成后

在 Telegram 简短报告：
- "温习了 /lean skill，没有新经验要加。"（大多数情况）
- "温习了 /lean skill，新增 1 条经验：<标题>。"（有新增时）

## Cron 配置

每 3 小时自动触发。cron 脚本检测哪个 tmux 窗口在跑哪个 skill（通过最近的 conversation 内容判断），然后注入触发消息。

**Cron 脚本**：`~/.openclaw/workspace/scripts/self-improve-cron.sh`

```bash
#!/bin/bash
# 每 3 小时检测活跃 session 并注入 self-improvement 触发
# 由 crontab 调用

SKILLS=("lean" "xhs" "poly" "hunt")
WINDOWS=("research" "dm")

for win in "${WINDOWS[@]}"; do
    # 检测窗口是否活跃（最近 10 分钟有输出）
    if ! tmux has-session -t "zinan" 2>/dev/null; then continue; fi
    
    # 注入温习提示（不用 send-keys 直接发命令，而是通过 Telegram 发消息触发）
    # 这样不会打断正在编辑的命令
    # 具体实现：写一个 marker 文件，PostCompact hook 或下次 user prompt 时检测到
    touch "/tmp/self_improve_due_${win}"
done
```

**替代触发方式（更轻量）：**
- **PostCompact hook 增强**：context 压缩时检查 `/tmp/self_improve_due_*`，如果存在就在 hook 输出中提醒"该温习 skill 了"
- **UserPromptSubmit hook**：检测 session 时长，超过 3 小时自动建议温习

## 与现有系统的关系

| 系统 | 作用域 | 频率 | 写入目标 |
|------|--------|------|----------|
| 每日三省反思 | 紫楠整体人格 | 1次/天 | reflections/daily/, growth.md |
| lessons/ | 跨 skill 的通用教训 | 事件驱动 | zinan/lessons/NNNN-*.md |
| **self-improve** | **单个 skill** | **每 3 小时** | **skill 文档 § Learned Tactics** |
| UNDERSTANDING.md | 单个项目 | session 边界 | 项目根目录 |

**不重叠原则：**
- 只适用于某个 skill 的 → self-improve 写到 skill doc
- 跨 skill 通用的 → lessons/ 系统
- 项目特定的 → UNDERSTANDING.md
- 关于紫楠人格的 → growth.md / reflections

## 初始化一个 skill 的 Self-Improvement section

对于还没有 `## Learned Tactics` section 的 skill，第一次 `/self-improve <name>` 时自动在文档末尾添加：

```markdown

## Learned Tactics (Self-Improvement)

_This section is maintained by the self-improvement loop. Each entry was extracted
from real session experience and passed the generalizability filter._

```

## 示例

**输入**：`/self-improve lean`

**执行过程**：
1. 读 `~/repos/zinan-skills/skills/lean.md`（255 行），刷新所有约束
2. 回顾本 session：今天在 HolonomicCRN 项目推了 3 个 sorry，发现一个模式——每次忘记先 `grep sorry` 看全局状态就开始推某个 sorry，经常推到一半发现依赖另一个 sorry 的结果
3. 过滤：这条可泛化吗？是的——"推 sorry 前先看依赖图"适用于所有 Lean 项目。重复吗？skill 里没有。验证了吗？今天确实因此浪费了一个小时。
4. 写入：

```markdown
### [2026-06-22] Resolve sorry dependency order before grinding
Before attacking a sorry, check whether it depends on OTHER sorries' results
(`grep sorry` + read the sorry's type to see what it assumes). Grinding a
downstream sorry while its upstream is still open wastes the proof effort —
the upstream resolution may change the type.
**Why:** Repeatedly wasted ~1h proving things that became invalid when an
upstream sorry was resolved differently.
**How to apply:** At the start of any sorry-grinding session, build the
dependency DAG first. Attack leaves (no sorry-dependencies) before interior
nodes.
```

5. Commit: `self-improve(lean): resolve sorry dependency order before grinding`
6. TG 报告："温习了 /lean skill，新增 1 条经验：Resolve sorry dependency order before grinding。"
