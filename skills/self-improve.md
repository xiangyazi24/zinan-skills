---
description: "Skill self-improvement loop. /self-improve <skill-name> = 对指定 skill 执行温习+总结+改进。三步：(1) 重新加载 skill 文档刷新约束，(2) 反思本 session 经验，(3) 提炼泛化教训写入 skill 的 ## Learned Tactics 区。可手动触发，也可由 cron 每 3 小时自动注入活跃 session。Trigger: /self-improve, '温习 skill / skill 进化 / 总结经验到 skill / skill self-improve'."
user-invocable: true
---

# /self-improve — Skill Self-Improvement Loop

让 skill 从实战中进化。用法：`/self-improve <skill-name>`（如 `/self-improve lean`）。

每次触发执行三步：温习 → 总结 → 改进。

## 触发方式

1. **手动**：`/self-improve lean`
2. **Cron 自动**：每 3 小时，cron 写 marker → PostCompact 时检测 → 提示 agent 执行
3. **口头**："温习一下 lean skill"、"skill 进化"、"总结经验到 skill 里"

## 三步流程

### Step 1: 温习（Refresh）

完整重新加载目标 skill 文档：

```
读 ~/repos/zinan-skills/skills/<skill-name>.md（完整读，不跳）
```

读完后内部检查：**核心约束是什么？这几个小时有没有偏离？** 有明显偏离就纠正并在 Telegram 简短报告。

### Step 2: 总结（Reflect）

回顾本 session 中与该 skill 相关的工作，提取候选经验。

**自问（内部思考，不输出）：**

1. 踩了什么坑？根因是什么？
2. 哪个做法特别有效，skill 里没写？
3. 爸爸纠正了我什么？（feedback 类，最高优先级）
4. 有没有反复出现的模式？

**三道筛——同时满足才算"真经验"：**

- **可泛化**：去掉文件名、变量名、项目名后，对未来所有 session 仍有用
- **非重复**：skill 文档（包括已有的 Learned Tactics）里没有实质等价的规则
- **经过验证**：本 session 确实验证了（不是猜测）

**泛化测试（写入前必须过这一关）：**
把写好的条目给一个不了解当前项目的人看——他能看懂吗？他能用到自己的项目里吗？
如果他需要知道"EDS 是什么"、"Ward 是什么"、"这个 repo 的 Probability/ 目录"才能理解这条规则，
那就是没泛化干净。**Why/How to apply 里可以用一个泛化的场景描述（"多项式恒等式验证"），
但绝不能出现当前项目的专有名词、具体函数名、具体文件结构。**

**硬性排除：**
- 具体工程细节（文件路径、build 错误、某个 sorry）→ 属于 UNDERSTANDING.md
- 只适用当前项目的知识 → 属于 UNDERSTANDING.md
- 条目里出现当前项目的专有名词（EDS, Ward, Somos, preΨ, ClockBudgets 等）→ 没泛化干净，重写
- 重复已有规则

### Step 3: 改进（Amend）

如果有通过筛选的经验（**没有就不写，宁缺毋滥**），写入 skill 文档末尾的隔离区。

#### 写入位置：skill 文档末尾的隔离 section

每个 skill 文档的最末尾有一个严格隔离的 section，**用明确的分隔线和标记与手写内容分开**：

```markdown
## Learned Tactics (Self-Improvement)

<!--
  此 section 由 /self-improve 自动维护，请勿手动编辑此区域内的内容。

  规则：
  - 此 section header 以上的所有内容是手写的 skill 正文，/self-improve 绝不修改。
  - 此 section header 以下的所有内容由 /self-improve 从实战经验中提炼写入。
  - 每条经验都通过了三道筛选：可泛化、非重复、经过验证。
  - 如需手动添加规则，请写在此 section 之上的正文区域。
  - 条目超过 30 条时 /self-improve 会自动压缩合并。

  识别方式：grep "## Learned Tactics (Self-Improvement)" 定位此 section。
-->

### [YYYY-MM-DD] 简短标题
一两句话描述规则本身。
**Why:** 触发这条经验的场景（泛化描述，不带具体工程细节）。
**How to apply:** 什么条件下应该想起这条规则。
```

**这是硬边界：**
- section header 以上 = 手写内容，/self-improve **绝不修改**
- section header 以下 = 自动维护区，只有 /self-improve 写入
- 识别靠 section header 文本 + HTML comment，不靠装饰线

#### 写入纪律

- 每次最多 1-3 条
- 新条目追加到 section 末尾
- 已有条目只在发现是错的时候才修正
- 超过 30 条（~500 行）→ 触发压缩：合并重叠、删除被 supersede 的旧条目
- **写完必须 git commit 落盘**（见下方并发控制）

### 完成后：汇报到 DM

**不管从哪个窗口触发，汇报统一发到 DM（chat_id 8672648562）**，方便爸爸在一个地方监控所有 skill 的进化。

用 `telegram_reply` 发送，格式：

有新增时：
```
[self-improve] lean (research窗口)
温习 ✓ | 新增 1 条：<标题>
当前 Learned Tactics 共 N 条
```

无新增时：
```
[self-improve] lean (dm窗口)
温习 ✓ | 无新增
```

锁冲突时：
```
[self-improve] lean (dm窗口)
温习 ✓ | 另一个 session 正在写入，本次跳过改进
```

**窗口名从 `tmux display-message -p -t "$TMUX_PANE" '#W'` 获取。**

## 多 Session 并发控制

**场景**：research 窗口和 dm 窗口同时在做 Lean 工作，cron 同时触发两个 session 的 `/self-improve lean`。

**机制：原子 mkdir 锁 + read-before-write**

注意：macOS 没有 `flock`。用 `mkdir` 作为原子锁（POSIX 保证 mkdir 是原子的）。

```
写入流程（Step 3 内部）：

LOCKDIR="/tmp/self_improve_<skill>.lock"

1. 尝试获取锁：mkdir "$LOCKDIR" 2>/dev/null
   - 成功（exit 0）→ 拿到锁，继续
   - 失败 → 检查锁龄：如果锁目录 mtime > 30 分钟，判定为 stale（上个持有者被压缩中断），
     rm -rf 回收后重试一次
   - 仍然失败 → 另一个 session 正在写，只做温习，跳过写入

2. 拿到锁后，重新读 skill 文档的 Learned Tactics section（另一个 session 可能刚写过）
   - 检查候选经验是否已被另一个 session 加过（去重）
   - 过滤掉已存在的

3. 追加新条目

4. git add + git commit（在锁内完成）

5. 释放锁：rm -rf "$LOCKDIR"
```

**实际命令**：
```bash
LOCKDIR="/tmp/self_improve_lean.lock"

if mkdir "$LOCKDIR" 2>/dev/null; then
    # 拿到锁，写入 + commit
    cd ~/repos/zinan-skills
    # ... Edit the file ...
    git add skills/lean.md
    git commit -m "self-improve(lean): <title>"
    rm -rf "$LOCKDIR"
elif [ -d "$LOCKDIR" ]; then
    # 检查是否 stale（>30 min）
    lock_age=$(( $(date +%s) - $(stat -f %m "$LOCKDIR") ))
    if [ "$lock_age" -gt 1800 ]; then
        rm -rf "$LOCKDIR"
        # 重试一次
        if mkdir "$LOCKDIR" 2>/dev/null; then
            # 写入 + commit ...
            rm -rf "$LOCKDIR"
        fi
    else
        echo "Another session is writing, skip amend"
    fi
fi
```

**设计要点：**
- macOS 没有 flock，mkdir 是 POSIX 原子操作，跨平台可靠
- 锁粒度 per-skill（`/tmp/self_improve_lean.lock`），不同 skill 互不阻塞
- 30 分钟 stale 回收：防止 session 被压缩中断后锁永远卡住（首次实战已验证）
- 拿不到锁 = 温习照做，改进下次再来

## Cron 配置

**Cron 脚本**：`~/.openclaw/workspace/scripts/self-improve-cron.sh`
**Cron entry**：`0 */3 * * *`

Cron **直接 send-keys 注入** `/self-improve <skill>` 到活跃窗口（不再用被动 marker 机制——
marker 依赖 PostCompact 消费，session 不压缩就永远不触发，实战已验证此 bug）。

流程：
1. 检测 research / dm 窗口是否有活跃的 Claude Code session
2. 从窗口内容自动判断在跑哪个 skill（lean/xhs/poly/hunt）
3. 检查 2.5 小时内是否已注入过（`/tmp/self_improve_done_<win>` marker，防重复）
4. `tmux send-keys -t "zinan:<win>" "/self-improve <skill>" Enter`

**PostCompact hook 仍然保留**作为补充触发——万一 cron 判断不到 skill 但 session 确实在用，
压缩时会提示 agent 手动执行。

## 初始化

第一次对某个 skill 执行 `/self-improve` 时，如果文档末尾还没有隔离 section，自动添加：

```markdown

## Learned Tactics (Self-Improvement)

<!--
  此 section 由 /self-improve 自动维护，请勿手动编辑此区域内的内容。

  规则：
  - 此 section header 以上的所有内容是手写的 skill 正文，/self-improve 绝不修改。
  - 此 section header 以下的所有内容由 /self-improve 从实战经验中提炼写入。
  - 每条经验都通过了三道筛选：可泛化、非重复、经过验证。
  - 如需手动添加规则，请写在此 section 之上的正文区域。
  - 条目超过 30 条时 /self-improve 会自动压缩合并。

  识别方式：grep "## Learned Tactics (Self-Improvement)" 定位此 section。
-->

```

## 与现有系统的边界

| 系统 | 作用域 | 频率 | 写入目标 |
|------|--------|------|----------|
| 每日三省反思 | 紫楠整体人格 | 1次/天 | reflections/daily/, growth.md |
| lessons/ | 跨 skill 的通用教训 | 事件驱动 | zinan/lessons/NNNN-*.md |
| **self-improve** | **单个 skill** | **每 3 小时** | **skill 文档 § Learned Tactics** |
| UNDERSTANDING.md | 单个项目 | session 边界 | 项目根目录 |

**不重叠**：skill 专属的 → self-improve；跨 skill 的 → lessons/；项目特定的 → UNDERSTANDING.md

## 示例

**输入**：`/self-improve lean`

**执行**：
1. 读 `~/repos/zinan-skills/skills/lean.md`，刷新约束
2. 回顾：今天推 sorry 时忘记先看依赖图，浪费了时间
3. 过滤通过（可泛化 ✓ 非重复 ✓ 验证 ✓）
4. flock → re-read → 追加：

```markdown
### [2026-06-22] Resolve sorry dependency order before grinding
Before attacking a sorry, check whether it depends on OTHER sorries' results.
Grinding a downstream sorry while its upstream is still open wastes the proof
effort — the upstream resolution may change the type.
**Why:** Repeatedly proved things that became invalid when upstream sorry resolved differently.
**How to apply:** At session start, `grep sorry` + build dependency DAG. Attack leaves first.
```

5. `git commit -m "self-improve(lean): resolve sorry dependency order before grinding"`
6. Release lock
7. TG："温习了 /lean skill，新增 1 条：Resolve sorry dependency order before grinding。"

$ARGUMENTS is the target skill name (required). Example: `lean`, `xhs`, `poly`.
