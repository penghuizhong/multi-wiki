# Multi-Industry Wiki Skill

> 基于 `SKILL.md` 的多行业知识库系统，以 `~/wiki/` 为根目录，各行业子库严格隔离，防止知识串库。

---

## 这是什么

这是一套给 AI Agent（Claude）使用的 **Skill 指令文件**，让 Agent 能够在本地文件系统上建立和维护一个**多行业 Wiki 知识库**。

灵感来源于 [Andrej Karpathy 的 LLM Wiki 模式](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)，在其基础上扩展为多领域管理架构。

传统 RAG 每次查询都从零检索；这套系统**一次编译，持续累积**——交叉引用已建立，矛盾已标记，合成已沉淀。

---

## 目录结构

```
~/wiki/
├── README.md          ← 人类可读的子库清单（就是这个文件的运行时版本）
├── registry.md        ← 机器可读的注册表
│
├── fashion/           ← 服装 Wiki
├── code/              ← Code Wiki
├── medical/           ← 医疗 Wiki
└── [新行业]/           ← 按需扩展
```

每个子库内部结构：

```
<industry>/
├── SCHEMA.md          ← 该领域的规范、标签分类
├── index.md           ← 所有页面的目录
├── log.md             ← 操作日志（只追加）
├── raw/               ← 原始资料（不可修改）
│   ├── articles/
│   ├── papers/
│   ├── transcripts/
│   └── assets/
├── entities/          ← 实体页（人物、品牌、产品）
├── concepts/          ← 概念页
├── comparisons/       ← 对比分析页
└── queries/           ← 值得归档的查询结果
```

---

## 核心设计原则

### 1. 严格隔离，防止串库

| 规则 | 说明 |
|------|------|
| `[[wikilinks]]` 只在子库内使用 | 禁止跨库链接 |
| 标签分类各自独立 | `fashion/` 的 `brand` 与 `code/` 的 `brand` 互不相关 |
| 原始资料归属明确 | 医疗论文只能放在 `medical/raw/` |
| frontmatter 含 `wiki:` 字段 | 每个页面标注所属子库，便于审计 |

### 2. 根目录只做路由

`~/wiki/` 根目录**不存储任何领域内容**，只有：
- `README.md` — 子库清单
- `registry.md` — 机器可读注册表

### 3. 三层架构

- **Layer 1（Raw）**：原始资料，不可修改
- **Layer 2（Wiki）**：Agent 维护的 Markdown 页面，互相交叉引用
- **Layer 3（Schema）**：每个子库的规范文件，约束 Agent 行为

---

## 快速上手

### 初始化一个子库

告诉 Agent：

```
帮我初始化一个服装行业的 wiki，放在 ~/wiki/fashion/
```

Agent 会自动：
1. 创建目录结构
2. 生成适配该领域的 `SCHEMA.md`（含标签分类）
3. 初始化 `index.md` 和 `log.md`
4. 更新根目录的 `registry.md` 和 `README.md`

### 录入一个来源

```
把这篇关于 Gore-Tex 面料的文章加入 fashion wiki
```

### 查询知识库

```
fashion wiki 里有哪些关于可持续面料的内容？
```

### 跨领域查询

```
AI 在医疗诊断中的应用有哪些？
```

Agent 会分别从 `code/` 和 `medical/` 各自回答，标注来源子库，**不合并两个领域的知识**。

### 健康检查

```
帮我 lint 一下 medical wiki
```

会检查：跨库链接违规 → 断链 → 孤立页面 → 过期内容 → 样式问题

---

## 每次会话开始时

Agent 必须在操作前完成定向三步：

```bash
# 1. 确认目标子库
cat ~/wiki/registry.md

# 2. 读取该子库的规范
cat ~/wiki/<industry>/SCHEMA.md

# 3. 读取目录和近期日志
cat ~/wiki/<industry>/index.md
tail -50 ~/wiki/<industry>/log.md
```

**跳过定向会导致：重复创建已有页面、遗漏交叉引用、违反 Schema 规范。**

---

## Obsidian 使用说明

- ✅ 以各子库目录单独作为 Vault 打开（如 `~/wiki/fashion/`）
- ❌ 不要把 `~/wiki/` 根目录作为 Vault 打开——这会导致跨库链接污染
- 推荐安装 Dataview 插件，支持按标签查询页面

---

## 文件说明

| 文件 | 用途 |
|------|------|
| `SKILL.md` | Agent 的完整行为指令，放入 Skill 系统后自动触发 |
| `README.md` | 本文件，面向人类的使用说明 |

---

## 扩展新行业

直接告诉 Agent：

```
帮我新建一个 finance wiki
```

Agent 会询问领域范围定义，然后自动初始化并注册。每个子库的 `SCHEMA.md` 都会包含明确的**领域边界说明**，避免与已有子库内容重叠。