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

**硬性排除：**
- 具体工程细节（文件路径、build 错误、某个 sorry）→ 属于 UNDERSTANDING.md
- 只适用当前项目的知识 → 属于 UNDERSTANDING.md
- 重复已有规则

### Step 3: 改进（Amend）

如果有通过筛选的经验（**没有就不写，宁缺毋滥**），写入 skill 文档末尾的隔离区。

#### 写入位置：skill 文档末尾的隔离 section

每个 skill 文档的最末尾有一个严格隔离的 section，**用明确的分隔线和标记与手写内容分开**：

```markdown
# ═══════════════════════════════════════════════════════════════
# LEARNED TACTICS (Self-Improvement) — 以下内容由 /self-improve 自动维护
# 手写内容请写在此分隔线之上。此区域内的条目来自实战经验提炼。
# ═══════════════════════════════════════════════════════════════

### [YYYY-MM-DD] 简短标题
一两句话描述规则本身。
**Why:** 触发这条经验的场景（泛化描述，不带具体工程细节）。
**How to apply:** 什么条件下应该想起这条规则。
```

**这个分隔线是硬边界：**
- 分隔线以上 = 手写内容，/self-improve **绝不修改**
- 分隔线以下 = 自动维护区，只有 /self-improve 写入
- 手动想加规则？写在分隔线以上的正常 section 里

#### 写入纪律

- 每次最多 1-3 条
- 新条目追加到 section 末尾
- 已有条目只在发现是错的时候才修正
- 超过 30 条（~500 行）→ 触发压缩：合并重叠、删除被 supersede 的旧条目
- **写完必须 git commit 落盘**（见下方并发控制）

### 完成后

Telegram 简短报告：
- "温习了 /lean skill，没有新经验要加。"
- "温习了 /lean skill，新增 1 条：<标题>。"

## 多 Session 并发控制

**场景**：research 窗口和 dm 窗口同时在做 Lean 工作，cron 同时触发两个 session 的 `/self-improve lean`。

**机制：flock 文件锁 + read-before-write**

```
写入流程（Step 3 内部）：
1. 尝试获取锁：flock -n /tmp/self_improve_<skill>.lock
   - 拿到锁 → 继续
   - 拿不到 → 说明另一个 session 正在写，本次只做 Step 1+2（温习+总结），跳过写入
     报告："温习了 /lean skill，另一个 session 正在写入，本次跳过改进。"

2. 拿到锁后，重新读一遍 skill 文档的 Learned Tactics section（另一个 session 可能刚写过）
   - 检查候选经验是否已被另一个 session 加过（去重）
   - 过滤掉已存在的

3. 追加新条目

4. git add + git commit（在锁内完成）

5. 释放锁
```

**实际的 flock 命令**：
```bash
exec 9>/tmp/self_improve_lean.lock
if ! flock -n 9; then
    echo "Another session is writing, skip amend"
    exec 9>&-
    # 只做温习，跳过写入
else
    # 写入 + commit
    cd ~/repos/zinan-skills
    # ... Edit the file ...
    git add skills/lean.md
    git commit -m "self-improve(lean): <title>"
    flock -u 9
    exec 9>&-
fi
```

**为什么 flock 而不是更复杂的方案：**
- 所有 session 在同一台 Mac mini 上，flock 足够
- 锁粒度是 per-skill（`/tmp/self_improve_lean.lock`），不同 skill 互不阻塞
- 拿不到锁 = 温习照做，改进下次再来（3 小时后又有机会）

## Cron 配置

**Cron 脚本**：`~/.openclaw/workspace/scripts/self-improve-cron.sh`
**Cron entry**：`0 */3 * * *`

cron 只写 marker 文件（`/tmp/self_improve_due_<window>`），不直接 send-keys。
PostCompact hook 检测到 marker 后在 hook 输出里提示 agent 执行 `/self-improve`。

marker 有 6 小时过期保护——如果一个 marker 超过 6 小时没被消费，下次 cron 刷新它（避免堆积）。

## 初始化

第一次对某个 skill 执行 `/self-improve` 时，如果文档末尾还没有隔离 section，自动添加：

```markdown

# ═══════════════════════════════════════════════════════════════
# LEARNED TACTICS (Self-Improvement) — 以下内容由 /self-improve 自动维护
# 手写内容请写在此分隔线之上。此区域内的条目来自实战经验提炼。
# ═══════════════════════════════════════════════════════════════

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
