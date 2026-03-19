# 个人开发者使用指南

Claude Code 最佳实践仓库 — 个人开发者快速上手与深度使用手册

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 目录

- [这个项目是什么](#这个项目是什么)
- [快速开始（5 分钟）](#快速开始5-分钟)
- [核心概念速览](#核心概念速览)
- [分层使用路线图](#分层使用路线图)
  - [Level 1 — 基础配置（立即见效）](#level-1--基础配置立即见效)
  - [Level 2 — 工作流自动化（日常提效）](#level-2--工作流自动化日常提效)
  - [Level 3 — 编排系统（复杂项目）](#level-3--编排系统复杂项目)
- [实战模板](#实战模板)
- [常见问题](#常见问题)
- [调试与维护](#调试与维护)

---

## 这个项目是什么

这是一个 **Claude Code 配置参考模板**，不是一个可运行的应用程序。它展示了 Claude Code 的各种配置模式（Memory、Commands、Skills、Agents、Hooks），你应该 **「学习 → 复制 → 定制」** 而非直接 fork 使用。

**对个人开发者的核心价值：** 从中提取配置模式，应用到你自己的项目中，让 Claude Code 更精准地理解你的代码库、遵循你的工作习惯。

---

## 快速开始（5 分钟）

在你自己的项目根目录下执行以下步骤：

### Step 1: 创建 CLAUDE.md

```bash
touch CLAUDE.md
```

写入你项目的核心上下文（参考本项目的 [`CLAUDE.md`](../CLAUDE.md)）：

```markdown
# CLAUDE.md

## 项目概述
[一句话描述你的项目做什么]

## 技术栈
- 语言: [例: TypeScript]
- 框架: [例: Next.js 14]
- 数据库: [例: PostgreSQL + Prisma]
- 测试: [例: Vitest]

## 关键命令
- `npm run dev` — 启动开发服务器
- `npm test` — 运行测试
- `npm run build` — 构建生产版本
- `npm run lint` — 代码检查

## 项目结构
- `src/app/` — 页面路由
- `src/lib/` — 共享工具函数
- `src/components/` — UI 组件
- `prisma/` — 数据库 schema 和迁移

## 编码规范
- 使用函数式组件 + hooks
- API 路由返回 NextResponse
- 错误处理统一使用 AppError 类
```

> **关键原则：保持在 200 行以内。** 过长的 CLAUDE.md 会导致 Claude 不可靠地遗漏指令。

### Step 2: 创建权限配置

```bash
mkdir -p .claude
```

创建 `.claude/settings.json`：

```json
{
  "permissions": {
    "allow": [
      "Edit(*)",
      "Write(*)",
      "Bash(npm run *)",
      "Bash(npx *)",
      "Bash(node *)",
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git log *)",
      "Bash(ls *)",
      "Bash(cat *)"
    ],
    "deny": [
      "Read(.env)",
      "Read(.env.*)"
    ]
  }
}
```

这样 Claude 可以自由编辑文件和运行开发命令，但不会读取你的环境变量文件。破坏性命令（`rm`、`git push`）会弹窗确认。

### Step 3: 开始使用

```bash
claude
```

至此你已经有了一个对你项目有基本理解的 Claude Code 环境。

---

## 核心概念速览

本项目演示了 Claude Code 的六大配置概念。下表按从简到繁排序：

| 概念 | 位置 | 本质 | 适合个人开发者？ |
|------|------|------|:---:|
| **Memory** | `CLAUDE.md`、`.claude/rules/` | 持久化上下文，每次启动自动加载 | 必须 |
| **Settings** | `.claude/settings.json` | 权限、hooks、MCP 的配置中心 | 必须 |
| **Commands** | `.claude/commands/*.md` | 斜杠命令 — 注入到当前上下文的提示词模板 | 推荐 |
| **Skills** | `.claude/skills/*/SKILL.md` | 结构化知识模块 — 可被预加载或按需调用 | 按需 |
| **Agents** | `.claude/agents/*.md` | 自治代理 — 独立上下文、独立权限、独立模型 | 按需 |
| **Hooks** | `.claude/hooks/` | 事件驱动脚本 — 工具调用前后自动执行 | 进阶 |

### 关键区别：Command vs Skill vs Agent

```
Command（命令）          Skill（技能）            Agent（代理）
─────────────         ─────────────          ─────────────
注入当前上下文           注入当前上下文            独立隔离上下文
无 frontmatter          有 frontmatter           有 frontmatter
用户通过 / 调用         可被 agent 预加载        通过 Agent 工具调用
最轻量                  中等                     最重量级
────────────────────────────────────────────────────────────
80% 的需求               15% 的需求               5% 的需求
```

---

## 分层使用路线图

### Level 1 — 基础配置（立即见效）

> 适用：所有项目，5 分钟内完成

#### 1.1 CLAUDE.md — 项目记忆

你在[快速开始](#快速开始5-分钟)中已经创建了。以下是优化建议：

**该写什么：**
- 项目做什么（一句话）
- 技术栈和关键依赖
- 常用命令（dev、test、build、lint）
- 目录结构和职责划分
- 编码规范和关键设计模式
- 调试技巧

**不该写什么：**
- 所有 API 端点的详细文档（太长）
- 每个文件的说明（Claude 会自己读）
- 通用编程知识（Claude 已经知道）

#### 1.2 条件规则 — `.claude/rules/`

条件规则是「按需加载的 CLAUDE.md」—— 只在 Claude 操作匹配特定 glob 模式的文件时生效。

```bash
mkdir -p .claude/rules
```

本项目中的例子（参考 [`.claude/rules/markdown-docs.md`](../.claude/rules/markdown-docs.md)）：

```markdown
# Glob: **/*.md

## 文档规范
- 一个文件一个主题
- 使用相对链接
- 标题层级不跳级
```

**个人开发者实用示例：**

`.claude/rules/api-routes.md`：
```markdown
# Glob: src/app/api/**/*.ts

## API 路由规范
- 所有路由使用 NextResponse 返回
- 错误码统一用 AppError 抛出
- 必须包含输入验证（用 zod）
- 数据库操作用 prisma，不写裸 SQL
```

`.claude/rules/components.md`：
```markdown
# Glob: src/components/**/*.tsx

## 组件规范
- 使用函数式组件
- Props 接口定义在组件文件顶部
- 样式用 Tailwind CSS，不用 inline style
- 可复用组件放 src/components/ui/
```

#### 1.3 权限配置细化

本项目的 `.claude/settings.json` 配置了详细的权限分层（参考 [`.claude/settings.json`](../.claude/settings.json)）。

**个人开发者推荐配置模板：**

```json
{
  "permissions": {
    "allow": [
      "Edit(*)",
      "Write(*)",
      "Bash(npm run *)",
      "Bash(npx *)",
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git log *)"
    ],
    "ask": [
      "Bash(rm *)",
      "Bash(git push *)",
      "Bash(git commit *)",
      "Bash(git checkout *)",
      "Bash(npm install *)",
      "Bash(docker *)"
    ],
    "deny": [
      "Read(.env)",
      "Read(.env.*)"
    ]
  }
}
```

**分层原则：**
- `allow` — 安全的只读/编辑操作，无需确认
- `ask` — 有副作用的操作（安装依赖、git 写操作），弹窗确认
- `deny` — 敏感文件，完全禁止

---

### Level 2 — 工作流自动化（日常提效）

> 适用：有重复性操作的项目

#### 2.1 创建 Slash Commands

Slash commands 是最简单的自动化方式 —— 本质就是一个 `.md` 文件，通过 `/命令名` 调用。

本项目的例子（参考 [`.claude/commands/weather-orchestrator.md`](../.claude/commands/weather-orchestrator.md)）展示了一个完整的编排流程。但个人开发者通常只需要简单命令：

**常用命令模板：**

`.claude/commands/review.md` — 代码审查：
```markdown
审查当前分支相对于 main 的所有更改。

## 审查维度
1. **安全性** — 是否有注入、XSS、敏感信息泄露
2. **性能** — 是否有 N+1 查询、不必要的重渲染
3. **可维护性** — 命名是否清晰、逻辑是否过于复杂

## 输出格式
按严重程度分类列出问题，每个问题给出：
- 文件和行号
- 问题描述
- 修复建议
```

`.claude/commands/test-coverage.md` — 测试覆盖：
```markdown
分析 $ARGUMENTS 模块的测试覆盖情况。

## 步骤
1. 找到该模块的所有源文件
2. 找到对应的测试文件
3. 分析哪些函数/分支缺少测试
4. 为缺失的场景编写测试
```

`.claude/commands/refactor.md` — 重构：
```markdown
重构 $ARGUMENTS。

## 原则
- 不改变外部行为
- 每次只做一种类型的重构
- 重构前先确认测试通过
- 重构后运行测试确认未引入 bug
```

使用方式：
```
claude
> /review
> /test-coverage src/lib/auth
> /refactor src/utils/date.ts
```

> **`$ARGUMENTS`** 是 Claude Code 的内置变量，会被替换为用户在斜杠命令后输入的内容。

#### 2.2 创建 Skills

当你发现某个知识点需要 **跨多个 command 或 agent 复用** 时，把它提取为 Skill。

本项目的 [`weather-fetcher`](../.claude/skills/weather-fetcher/SKILL.md) 是一个典型例子：它封装了「如何调用 Open-Meteo API」的知识，被 `weather-agent` 预加载使用。

**个人开发者实用示例：**

`.claude/skills/deploy/SKILL.md` — 部署流程：
```yaml
---
name: deploy
description: 项目部署到生产环境的完整流程
user-invocable: true
allowed-tools:
  - "Bash(npm run *)"
  - "Bash(git *)"
---
```
```markdown
# 部署技能

## 部署步骤
1. 确认当前在 main 分支且无未提交更改
2. 运行 `npm run build` 确认构建成功
3. 运行 `npm test` 确认测试通过
4. 执行 `npm run deploy`
5. 验证部署是否成功
```

`.claude/skills/db-migration/SKILL.md` — 数据库迁移：
```yaml
---
name: db-migration
description: 创建和执行数据库迁移
user-invocable: true
allowed-tools:
  - "Bash(npx prisma *)"
---
```
```markdown
# 数据库迁移技能

## 创建迁移
1. 修改 prisma/schema.prisma
2. 运行 `npx prisma migrate dev --name <描述性名称>`
3. 检查生成的 SQL 迁移文件
4. 如有种子数据需要更新，修改 prisma/seed.ts
```

#### 2.3 MCP 服务器连接

在项目根目录创建 `.mcp.json` 连接外部工具：

```json
{
  "mcpServers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["@anthropic-ai/mcp-server-playwright"]
    }
  }
}
```

常用 MCP 服务器：
- **Playwright** — 浏览器自动化、UI 测试
- **GitHub** — PR/Issue 操作
- **PostgreSQL/SQLite** — 直接查询数据库
- **Context7** — 获取最新库文档

---

### Level 3 — 编排系统（复杂项目）

> 适用：需要多步骤自动化的大型项目

#### 3.1 自定义 Agents

当某类任务足够复杂且重复出现时，创建专用 Agent。

本项目的 [`weather-agent`](../.claude/agents/weather-agent.md) 展示了完整的 agent 定义，包括工具权限、模型选择、预加载技能和 hooks。

**个人开发者实用示例：**

`.claude/agents/test-writer.md`：
```yaml
---
name: test-writer
description: 为指定模块编写单元测试
allowedTools:
  - "Read"
  - "Write(*)"
  - "Edit(*)"
  - "Bash(npm test *)"
  - "Glob"
  - "Grep"
model: sonnet
maxTurns: 10
---
```
```markdown
# 测试编写代理

你是一个专门编写单元测试的代理。

## 工作流程
1. 读取目标源文件，理解所有导出函数
2. 检查是否已有测试文件
3. 分析每个函数的输入/输出/边界条件
4. 编写测试，覆盖正常路径和边界情况
5. 运行测试确认全部通过
```

#### 3.2 Command → Agent → Skill 编排

本项目的核心演示是 **Command → Agent → Skill** 模式（参考 [`orchestration-workflow`](../orchestration-workflow/orchestration-workflow.md)）：

```
用户调用 /weather-orchestrator（Command）
    ↓
Command 询问用户偏好（摄氏/华氏）
    ↓
Command 通过 Agent 工具调用 weather-agent（Agent）
    ├── Agent 预加载了 weather-fetcher（Skill，作为知识）
    └── Agent 调用 API 获取温度数据
    ↓
Command 通过 Skill 工具调用 weather-svg-creator（Skill，作为动作）
    └── Skill 生成 SVG 天气卡片
```

**关键设计原则：**
- **Command** 负责编排流程和用户交互
- **Agent** 在隔离上下文中执行专业任务
- **Skill** 提供可复用的知识或动作
- Agent 不能通过 Bash 调用其他 Agent — 必须用 `Agent(...)` 工具

---

## 实战模板

### 模板 A：前端项目（React/Next.js）

```
your-project/
├── CLAUDE.md                          # 项目概述、技术栈、关键命令
├── .claude/
│   ├── settings.json                  # 权限配置
│   ├── rules/
│   │   ├── components.md              # Glob: src/components/**/*.tsx
│   │   ├── api-routes.md              # Glob: src/app/api/**/*.ts
│   │   └── styles.md                  # Glob: **/*.css
│   └── commands/
│       ├── review.md                  # /review — 代码审查
│       ├── new-page.md                # /new-page — 创建新页面
│       └── test-coverage.md           # /test-coverage — 测试覆盖分析
└── .mcp.json                          # Playwright MCP（可选）
```

### 模板 B：后端 API 项目（Node.js/Python）

```
your-project/
├── CLAUDE.md                          # 项目概述、API 架构、数据库 schema
├── .claude/
│   ├── settings.json                  # 权限（含数据库命令）
│   ├── rules/
│   │   ├── routes.md                  # Glob: src/routes/**/*.ts
│   │   ├── models.md                  # Glob: src/models/**/*.ts
│   │   └── migrations.md             # Glob: prisma/migrations/**/*
│   ├── commands/
│   │   ├── review.md                  # /review
│   │   └── new-endpoint.md            # /new-endpoint — 创建新 API 端点
│   └── skills/
│       └── db-migration/SKILL.md      # 数据库迁移知识
└── .mcp.json                          # 数据库 MCP（可选）
```

### 模板 C：全栈项目（Monorepo）

```
your-project/
├── CLAUDE.md                          # 顶层概述（简短，< 50 行）
├── frontend/
│   └── CLAUDE.md                      # 前端特有的上下文
├── backend/
│   └── CLAUDE.md                      # 后端特有的上下文
├── .claude/
│   ├── settings.json
│   ├── rules/
│   │   ├── frontend.md                # Glob: frontend/**/*.tsx
│   │   └── backend.md                 # Glob: backend/**/*.ts
│   ├── commands/
│   │   ├── review.md
│   │   └── deploy.md
│   └── agents/
│       └── test-writer.md             # 测试编写代理
└── .mcp.json
```

> **Monorepo 提示：** 子目录的 CLAUDE.md 是懒加载的 — 只有当 Claude 读取该目录下的文件时才会加载。把全局信息放根目录，把特有信息放子目录。

---

## 常见问题

### Q: 我只有一个小项目，需要这么多配置吗？

**不需要。** 只需要：
1. 一个 `CLAUDE.md`（5-20 行就够）
2. 一个 `.claude/settings.json`（权限配置）

其他一切都是可选的。

### Q: Command 和 Skill 怎么选？

- **用 Command** 如果：只需要一个提示词模板，用户通过 `/命令` 调用
- **用 Skill** 如果：需要被 agent 预加载，或需要 frontmatter 元数据（allowed-tools、model 等）

大多数个人开发者只需要 Command。

### Q: Agent 什么时候值得创建？

当满足以下 **全部** 条件时：
1. 任务足够复杂（需要多轮工具调用）
2. 任务需要与主对话隔离（避免污染上下文）
3. 任务会重复出现
4. 任务需要不同的权限或模型配置

### Q: 如何防止 CLAUDE.md 变得过长？

- 核心 CLAUDE.md 保持 < 200 行
- 把文件类型相关的规范移到 `.claude/rules/`（按 glob 条件加载）
- 把具体操作知识移到 `.claude/skills/`
- 用 `@path` 引用外部文件而非内联

### Q: `.claude/settings.json` 和 `.claude/settings.local.json` 的区别？

- `settings.json` — 提交到 git，团队共享（个人开发者也建议提交，作为配置版本记录）
- `settings.local.json` — git-ignored，个人覆盖。对个人开发者来说通常不需要

### Q: 如何调试 Claude 没有遵循 CLAUDE.md 的指令？

1. 确认文件不超过 200 行
2. 用 `/doctor` 检查 Claude Code 环境
3. 在 ~50% 上下文使用量时手动 `/compact`
4. 把关键指令放在文件顶部（更不容易被忽略）

---

## 调试与维护

### 日常维护清单

- [ ] 定期更新 CLAUDE.md 反映项目变化（新增模块、新工具、新规范）
- [ ] 上下文使用到 ~50% 时执行 `/compact`
- [ ] 复杂任务先用 plan mode（`/plan`）
- [ ] 大任务拆解为小子任务，每个在 50% 上下文内可完成

### 有用的内置命令

| 命令 | 用途 |
|------|------|
| `/help` | 查看帮助 |
| `/doctor` | 诊断 Claude Code 环境 |
| `/compact` | 压缩上下文 |
| `/plan` | 切换到计划模式 |
| `/rewind` | 回退到之前的检查点 |
| `/voice` | 语音输入模式 |
| `/btw` | 不中断当前任务的旁路对话 |

### 推荐工作流

```
1. 启动 Claude Code
2. 用 /plan 规划复杂任务
3. 确认计划后执行
4. 在 ~50% 上下文时 /compact
5. 完成后 /review 审查更改
6. 确认无误后提交
```

---

## 延伸阅读

- [Claude Code 官方文档](https://code.claude.com/docs)
- [Claude Code 最佳实践](https://code.claude.com/docs/en/best-practices)
- [Writing a Good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
- [本项目 — Memory 最佳实践](../best-practice/claude-memory.md)
- [本项目 — Commands 最佳实践](../best-practice/claude-commands.md)
- [本项目 — Skills 最佳实践](../best-practice/claude-skills.md)
- [本项目 — Subagents 最佳实践](../best-practice/claude-subagents.md)
- [本项目 — Settings 最佳实践](../best-practice/claude-settings.md)
- [本项目 — 编排工作流](../orchestration-workflow/orchestration-workflow.md)
