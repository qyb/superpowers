# 在 Codex CLI 里安装 Skill 的操作手册

> 本文基于实测经验整理（2026-06-15），覆盖 Windows 和 WSL 两种环境。
> 记录了正确的安装方式，以及踩过的坑，避免下次重复。

---

## 一、Skill 的两种形态

Codex 里"skill"有两种来源，加载机制不同：

| 形态 | 来源 | 加载方式 | 例子 |
|------|------|----------|------|
| **散装 skill** | 用户自己写的 `SKILL.md` 目录 | 自动扫描 | `~/.codex/skills/cn-style/` |
| **plugin skill** | 通过 Codex plugin 系统安装的仓库 | 随 plugin 加载 | `~/.codex/plugins/cache/openai-curated/superpowers/` |

两种可以共存。同名时行为未明确文档化——建议避免同名，或用 `[[skills.config]]` 显式禁用其中一个（见下文）。

---

## 二、散装 Skill：正确安装方式（推荐）

这是最简单、最可靠的方式。**不需要改 config.toml，不需要注册。**

### 目录结构

```
~/.codex/skills/
├── .system/                    ← Codex 内置，不要动
├── my-skill-1/
│   └── SKILL.md
└── my-skill-2/
    ├── SKILL.md
    ├── references/             ← 可选，skill 引用的辅助文件
    └── scripts/                ← 可选
```

### SKILL.md 格式

```markdown
---
name: my-skill-name
description: Use when [具体触发条件和症状]。这句话直接决定自动委派能否命中。
---

# Skill 标题

正文内容……
```

**frontmatter 只有两个字段是必需的：**
- `name`：小写字母 + 连字符（如 `brainstorming`、`using-aq-workflow`）
- `description`：**只写触发条件，不要写流程摘要**。模型会拿这句话判断何时自动调用，写得太笼统或太详细都会影响命中率。

### 发现机制（已验证）

Codex 启动时自动扫描 `~/.codex/skills/**/SKILL.md`。
- 文件名**必须**是精确的 `SKILL.md`（大小写敏感，`SKILL.MD` 会被忽略）
- **不需要**在 config.toml 里注册
- **不需要**写 manifest 或 plugin.json

### 调用方式

- **自动委派**：模型根据 description 判断是否调用
- **手动调用**：用 `$` 前缀，如 `$brainstorming`、`$using-aq-workflow`

> ⚠️ **Windows 版 Codex 用 `$` 前缀，不是 `/`。** 这是跟 Claude Code 不同的地方，容易搞混。

### 项目级 skill（可选）

如果想只让某个项目用某 skill，放到项目目录下：
```
<project>/.codex/skills/<name>/SKILL.md
```
项目级会覆盖同名的用户级（`~/.codex/skills/`）。

---

## 三、子智能体（Agents）：安装方式

子智能体（subagent）是另一套机制，**不要跟 skill 混淆**。

### 目录结构

```
~/.codex/agents/
├── my-agent-1.md
└── my-agent-2.md
```

### Agent 文件格式

**Markdown + YAML frontmatter**（跟 skill 格式相似，但字段不同）：

```markdown
---
name: my-agent
description: |
  Use this agent when [具体触发条件]。带 example 块能提高自动委派命中率。
model: inherit                  ← 可选：inherit（跟会话）/ 具体模型名
---

你是 [角色]。你的职责是 [具体职责]。

[系统提示词正文……]
```

**frontmatter 字段：**
- `name`：必需
- `description`：必需，**建议用 YAML 多行字符串（`|`）+ `<example>` 块**，参考 superpowers 的 `code-reviewer.md` 写法
- `model`：可选，默认 `inherit`

### 发现机制（已验证）

Codex 启动时自动扫描 `~/.codex/agents/`（**仅顶层**）。
- **不要**把 agent 文件嵌套在子目录里，比如 `~/.codex/aq-workflow/agents/` —— 这个路径**不会**被扫描
- **不需要**在 config.toml 里注册
- plugin 内部的 `agents/` 目录随 plugin 自动加载（如 superpowers 的 `code-reviewer.md`）

### 调用方式

通过 `spawn_agent` 工具调用，或在对话里说"让 my-agent 检查……"。

---

## 四、Plugin Skill：安装方式（可选，较重）

如果想用 superpowers 这类完整仓库作为 plugin：

### 方式 A：通过 marketplace 安装（推荐）

在 config.toml 里声明：
```toml
[plugins."superpowers@openai-curated"]
enabled = true
```
Codex 会自动缓存到 `~/.codex/plugins/cache/<marketplace>/<plugin>/`，里面的 skill 随 plugin 加载。

plugin 目录结构需要 `.codex-plugin/plugin.json` manifest：
```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "skills": "./skills/",
  "interface": { ... }
}
```

### 方式 B：本地路径安装

把整个仓库 clone 到 `~/.codex/<plugin-name>/`，然后：
- 用 `[[skills.config]]` 逐个注册（见下文坑点）
- 或通过 plugin 系统注册（参考 Codex plugin 文档）

---

## 五、config.toml 里 `[[skills.config]]` 的正确用法

### 唯一已验证的用法：禁用 skill

```toml
[[skills.config]]
path = "/home/qyb/.codex/superpowers/skills/using-git-worktrees/SKILL.md"
enabled = false
```

这是**禁用**已存在的 skill（通常是 plugin 提供的），不删除文件。

### ⚠️ 坑：用 `[[skills.config]]` 启用独立路径的 skill 不可靠

理论上 `enabled = true` + 自定义 path 可以注册任意位置的 skill，但实测：
- 文档对这种用法描述模糊（"to disable a skill without deleting it"）
- 实际运行时 skill 可能不被识别
- **不要**依赖这种方式启用 skill

**正确做法**：把 skill 放到 `~/.codex/skills/` 下，靠自动扫描。

### ⚠️ 坑：`enabled` 字段是必需的

如果只写 `path` 不写 `enabled`，Codex 启动会报错：
```
Error loading config.toml: missing field `enabled` in `skills.config`
```
**每个 `[[skills.config]]` 块都必须有 `enabled` 字段**（true 或 false）。

---

## 六、踩过的坑（汇总）

### 坑 1：把 skill 放错目录

**错误**：放到 `~/.codex/aq-workflow/skills/`（自定义子目录），期望靠 `[[skills.config]]` 注册。

**结果**：Codex 启动正常，但 skill 不被识别。

**正确**：放到 `~/.codex/skills/<name>/SKILL.md`，靠自动扫描。

### 坑 2：漏写 `enabled` 字段

**错误**：
```toml
[[skills.config]]
path = "C:\\Users\\...\\SKILL.md"
```

**结果**：`Error loading config.toml: missing field 'enabled' in 'skills.config'`，Codex 无法启动。

**正确**：
```toml
[[skills.config]]
path = "C:\\Users\\...\\SKILL.md"
enabled = true
```

### 坑 3：用 TOML 格式定义子智能体

**错误**：参考网络搜索结论，用 `.toml` 文件定义子智能体。

**事实**：实际运行的 Codex 子智能体（如 superpowers 的 code-reviewer）用的是 **Markdown + YAML frontmatter** 格式（`.md`）。

**正确**：用 `.md` 格式，frontmatter 含 `name`/`description`/`model`，正文是系统提示词。

### 坑 4：把子智能体嵌套在子目录

**错误**：放到 `~/.codex/aq-workflow/agents/`。

**事实**：Codex 只扫描 `~/.codex/agents/` 顶层，子目录不被扫描。

**正确**：放到 `~/.codex/agents/<name>.md`（顶层）。

### 坑 5：混淆 `$` 和 `/` 前缀

**错误**：用 `/brainstorming` 调用 skill（Claude Code 的习惯）。

**事实**：Codex 用 `$` 前缀。

**正确**：`$brainstorming`。

### 坑 6：Skill 文件名大小写

**错误**：文件名写成 `SKILL.MD`。

**事实**：发现机制大小写敏感，`SKILL.MD` 被静默忽略。

**正确**：`SKILL.md`（精确大小写）。

---

## 七、Windows 路径写法（config.toml）

TOML 字符串里反斜杠要转义：

```toml
# 正确（双反斜杠）
path = "C:\\Users\\qiuyi\\.codex\\skills\\brainstorming\\SKILL.md"

# 或者用正斜杠（也接受）
path = "C:/Users/qiyi/.codex/skills/brainstorming/SKILL.md"
```

散装 skill 路径（`~/.codex/skills/`）不需要写进 config.toml，所以一般不会碰到这个问题。只有用 `[[skills.config]]` 禁用 plugin skill 时才需要。

---

## 八、验证安装是否成功

### 检查 skill 是否被发现

1. 启动 Codex，看启动信息有无 skill 加载提示
2. 在对话里试调用：`$<skill-name>`
3. 或问 Codex："你能用哪些 skill？"

### 检查子智能体是否被发现

1. 在对话里说"让 <agent-name> 检查一下……"
2. 或用 `spawn_agent` 工具显式调用

### 常见排查

| 症状 | 可能原因 |
|------|----------|
| Codex 无法启动 | config.toml 语法错（如漏 `enabled` 字段） |
| 启动正常但 skill 不识别 | skill 不在 `~/.codex/skills/` 下；或文件名大小写错；或 feature flag 没开 |
| skill 识别但不自动触发 | `description` 写得不好——只写触发条件，别写流程 |
| 子智能体不识别 | 不在 `~/.codex/agents/` 顶层；或 frontmatter 缺 `name`/`description` |

---

## 九、当前环境实际安装情况（参考）

### Windows 本机（`C:\Users\qiuyi\.codex\`）

```
skills/
├── .system/                    ← Codex 内置
├── brainstorming/              ┐
├── dispatching-parallel-agents/│
├── executing-plans/            │
├── finishing-a-development-branch/│
├── receiving-code-review/      │  14 个 aq-workflow skill
├── requesting-code-review/     │  （superpowers fork，已定制）
├── subagent-driven-development/│
├── systematic-debugging/       │
├── test-driven-development/    │
├── using-aq-workflow/          │
├── using-git-worktrees/        │
├── verification-before-completion/│
├── writing-plans/              │
└── writing-skills/             ┘

agents/
├── aq-implementer.md           ┐
├── aq-spec-reviewer.md         │  3 个自定义子智能体
└── aq-code-reviewer.md         ┘
```

config.toml 里**没有** skills 相关配置（全靠自动扫描）。

### WSL noble（`/home/qyb/.codex/`）

- superpowers 通过 plugin 安装（`[plugins."superpowers@openai-curated"]`）
- `[[skills.config]]` 用来禁用其中 2 个 skill（`enabled = false`）
- `~/.codex/skills/` 下有自定义的 `cn-style`、`git-change-reviewer`

---

## 十、参考链接

- [Agent Skills – Codex 官方文档](https://developers.openai.com/codex/skills)
- [Subagents – Codex 官方文档](https://developers.openai.com/codex/subagents)
- [Configuration Reference – Codex 官方文档](https://developers.openai.com/codex/config-reference)

---

*最后更新：2026-06-15*
