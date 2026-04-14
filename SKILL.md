---
name: designing-skills
description: Use when creating, refactoring, or evaluating any skill document — for Claude Code superpowers skills or OpenClaw agent skills.
---

# Designing Skills

## Layer 0 — 决策清单（扫一遍再动手）

| 问题 | → 答案决定什么 |
|------|---------------|
| 知识量 > 300 行？ | 是 → 移到外部文件，SKILL.md 只留接口 |
| 多平台有差异？ | 是 → `templates/platforms/` 分文件 |
| 需要搜索/过滤？ | 是 → CSV + 脚本，不要 flat list |
| 有分叉决策？ | 是 → 流程图；线性流程 → 编号列表 |
| 是否可复用？ | 否 → 不要建 skill，放 CLAUDE.md |

---

## Layer 1 — 标准流程

### 1. 分类

```
简单 skill    →  SKILL.md 单文件（< 200 词）
中等 skill    →  SKILL.md + 1-2 个支撑文件
复杂 skill    →  三层架构（见 Layer 2）
```

### 2. Frontmatter 规则

```yaml
---
name: verb-first-with-hyphens      # 动词开头，只用字母数字和连字符
description: Use when [触发条件]   # 只写 WHEN，不写 HOW（否则 Agent 跳过正文）
---
```

**description 反模式（会导致 Agent 不读正文）：**
```yaml
# ❌ 描述了流程
description: Use when saving recipes — calls XHS API, extracts structure, writes to Obsidian

# ✅ 只写触发条件
description: Use when the user shares a Xiaohongshu link and asks to save a recipe
```

### 3. SKILL.md 内部结构（渐进式披露）

```markdown
## Layer 0 — 快速参考（表格/清单，可在 10 秒内扫完）
## Layer 1 — 标准流程（编号步骤，覆盖 80% 场景）
## Layer 2 — 深度参考（链接到外部文件，复杂场景按需读取）
```

### 4. Token 预算

| 类型 | 上限 |
|------|------|
| 常驻 skill（每次对话都加载） | 200 词 |
| 按需 skill | 500 词 |
| 外部参考文件 | 不限（按需加载） |

### 5. 命名规范

| 场景 | 规范 | 示例 |
|------|------|------|
| 动作/流程 | 动名词 `-ing` | `designing-skills`, `writing-obsidian` |
| 工具/平台 | 名词 | `xhs-downloader`, `obsidian-write` |
| OpenClaw skill | 灵活（与上一致即可） | `recipe-xhs`, `bilibili` |

### 6. 测试要求

写完 skill 必须用 subagent 测试：  
给一个没有 skill 的 agent 同样的任务 → 观察失败 → skill 修正 → 验证通过。  
未经测试的 skill 不可投入使用。

---

## Layer 2 — 深度参考

见 [patterns.md](patterns.md)：完整模板、三层架构样例、反模式列表、BM25 检索实现参考。
