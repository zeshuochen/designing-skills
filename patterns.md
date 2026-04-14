# Skill 设计深度参考

## 三层架构（复杂 skill 标准目录）

```
skill-name/
├── SKILL.md                  # Layer 0-1：快速参考 + 标准流程
├── patterns.md               # Layer 2：深度模板和参考（本文件角色）
├── data/
│   └── *.csv                 # 结构化知识库（可用 BM25 检索）
├── scripts/
│   ├── core.py               # BM25 搜索引擎核心
│   └── search.py             # CLI 接口
└── templates/
    ├── base/
    │   ├── skill-content.md  # 通用工作流
    │   └── quick-reference.md
    └── platforms/
        ├── claude.json       # Claude Code 差异化
        └── cursor.json       # Cursor 差异化
```

**何时升级到三层架构：**
- 知识条目 > 50 条（styles、colors、guidelines…）
- 需要语义检索而非关键词匹配
- 多 AI 平台需要差异化行为

---

## SKILL.md 完整模板

```markdown
---
name: skill-name
description: Use when [具体触发场景 + 症状，不超过 500 字符]
---

# Skill 名称

## Layer 0 — 快速参考
[10 秒可扫完的表格或清单]

---

## Layer 1 — 标准流程
[编号步骤，覆盖 80% 使用场景]

### 1. 步骤一
### 2. 步骤二

---

## Layer 2 — 深度参考
见 [reference.md](reference.md)：[一句话说明里面有什么]
```

---

## 渐进式披露设计原则

```
用户/Agent 需求强度
      │
  低  ├── Layer 0：扫一眼就够（决策表、清单）
      │
  中  ├── Layer 1：读正文（标准流程、常见模式）
      │
  高  └── Layer 2：按需深入（外部文件、数据、脚本）
```

**核心原则：每层独立可用。** Agent 只读 Layer 0 也能做出正确决策；不需要从头读到尾。

---

## 反模式对照表

| 反模式 | 问题 | 修正 |
|--------|------|------|
| description 里写流程摘要 | Agent 按摘要行动，跳过正文 | 只写触发条件 |
| 所有知识 flat list 在 SKILL.md | token 爆炸，难以检索 | 移到 CSV + 脚本 |
| 流程图用于线性步骤 | 增加认知负担，无决策价值 | 改用编号列表 |
| 多语言重复示例 | 维护成本高，质量摊薄 | 一个最佳示例即可 |
| 在 SKILL.md 里 @force-load 外部文件 | 每次对话全量加载 | 用相对路径软引用 |
| 为一次性任务建 skill | skill 泛滥，难以维护 | 放 CLAUDE.md 或直接做 |
| 未测试就部署 | 不知道 Agent 会不会用错 | 必须 subagent 红绿测试 |

---

## BM25 检索实现参考（CSV 知识库场景）

```python
# 最小可用实现
import csv, math, re
from collections import Counter

def tokenize(text):
    return re.findall(r'\w+', text.lower())

class BM25:
    def __init__(self, k1=1.5, b=0.75):
        self.k1, self.b = k1, b

    def fit(self, corpus):
        self.corpus = corpus
        self.N = len(corpus)
        self.avgdl = sum(len(d) for d in corpus) / self.N
        self.df = Counter(t for doc in corpus for t in set(doc))
        self.idf = {t: math.log((self.N - f + 0.5) / (f + 0.5) + 1)
                    for t, f in self.df.items()}

    def score(self, query_tokens, doc):
        tf = Counter(doc)
        dl = len(doc)
        return sum(
            self.idf.get(t, 0) * tf[t] * (self.k1 + 1)
            / (tf[t] + self.k1 * (1 - self.b + self.b * dl / self.avgdl))
            for t in query_tokens
        )

def search_csv(csv_path, query, top_k=5, truncate=300):
    rows = list(csv.DictReader(open(csv_path, encoding='utf-8')))
    texts = [tokenize(' '.join(r.values())) for r in rows]
    bm25 = BM25()
    bm25.fit(texts)
    q = tokenize(query)
    scored = sorted(enumerate(rows), key=lambda x: bm25.score(q, texts[x[0]]), reverse=True)
    return [str(r)[:truncate] for _, r in scored[:top_k]]
```

---

## Symlink 单一真相（多工作区共享）

```bash
# 从各 workspace 指向 main workspace 的 skill
ln -s ~/.openclaw/workspace/skills/obsidian-write \
      ~/.openclaw/workspace-eris/skills/obsidian-write
ln -s ~/.openclaw/workspace/skills/obsidian-write \
      ~/.openclaw/workspace-sylphy/skills/obsidian-write

# Claude Code skill 目录
# ~/.claude/skills/designing-skills/  ← 本 skill 所在
```

**规则：** 只在 main workspace 编辑，symlink 只读。

---

## Skill 生命周期

```
设计 → 红测（无 skill 失败）→ 绿测（有 skill 通过）→ 重构（堵漏洞）→ 部署 → 按需更新
```

更新 skill 时：**先删改动的部分，重新跑红测，确认还会失败，再修。**  
否则你不知道改动是否真的修复了问题。
