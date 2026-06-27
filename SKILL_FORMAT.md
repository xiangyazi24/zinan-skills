# Skill 文件写法规范

所有 `skills/*.md` 必须满足这个格式，否则 digest 自动生成（`gen-skill-digest.py`）
出来的摘要质量会很差，refresh 温习就会失效。

## 规则

### 1. 标题即摘要

`##` 标题必须能独立传达这个 section 在讲什么。看标题就知道规则，不用读正文。

好：`## Banned anti-patterns (fingerprint self-check)`
差：`## Rules` / `## Notes` / `## 其他`

### 2. 首句即结论

每个 `##` section 的第一段的第一句话必须是这个 section 的核心结论或规则。
Digest 只提取标题 + 首句，如果首句是背景铺垫，digest 就没有用。

好：`Once the handshake completes:` → 直接告诉你什么条件下做什么
差：`This section discusses...` / `In order to understand...`

### 3. 编号列表保持 `N. **粗体标题**` 格式

Digest 会提取 `1. **关键词**` 形式的编号项。如果你的列表不是这个格式，
Digest 会漏掉。

好：`1. **Estimating time.** "multi-week" / ...`
差：`1. Don't estimate time` / `- estimating time is bad`

### 4. Learned Tactics section 格式固定

由 `/self-improve` 自动维护，格式见 `self-improve.md`。不要手动编辑。

### 5. 不要在正文里写长段落

长段落在 digest 里只保留第一句，后面全丢。如果一个规则需要多句话解释，
用 `**粗体**` 标记关键约束——digest 会提取所有 bold 短语。

### 6. 用 `##` 不用 `#`

`#` 级标题只用于文件标题（`# /automode — autonomous-execution mode`）。
正文分节全部用 `##` 或 `###`。Digest 只扫 `##` 和 `###`。

## 检查方法

```bash
# 扫描一个 skill 文件，看哪些 section 标题不够 descriptive
python3 scripts/gen-skill-digest.py lean
# 然后读 lean.digest.md，看每个 section 的摘要是否能独立理解
# 如果某个 section 的摘要是废话（"This section..."），回去改 full 文件
```

## 定期审计

self-improve full 模式（每 7 小时）时，顺便检查 skill 文件是否符合这个规范。
如果发现不符合的 section，直接修复标题和首句，然后重新生成 digest。
