<div align="right">
  <strong>中文</strong> | <a href="README.md">English</a>
</div>

# multi-phase-feature-dev

> 使用并行子 Agent 的多阶段功能开发 Orchestrator 模式。

[![Claude Code](https://img.shields.io/badge/Claude%20Code-skill-blue)](https://github.com/Likeusewin10/rye-skills)
[![Requires ECC](https://img.shields.io/badge/requires-ECC-orange)](https://github.com/affaan-m/ECC)

---

## 依赖

本 skill 依赖 [Everything Claude Code (ECC)](https://github.com/affaan-m/ECC) 中的两个组件：

| 组件 | 类型 | 用途 |
|------|------|------|
| `code-reviewer` | agent | 每个阶段的并行代码审查 |
| `coding-standards` | skill | 注入子 Agent prompt 的编码规范 |

使用本 skill 前请先安装 ECC。

---

## 背景

在多次长任务开发过程中，我反复遇到两个问题：任务进行到一半上下文耗尽，以及 Agent 在执行后期阶段时"忘记"了前期的决策——后半段的实现和前半段承诺的接口对不上。

根本原因在于，长任务在实践中往往是顺序执行的，即使它本不需要如此。每个 Agent 等待上一个完成，上下文不断累积，到第 4、5 阶段时，Agent 已经在靠压缩摘要工作，而不是原始决策。

这个工作流的解法是：让主 Agent 专注于最关键的一件事——定义接口契约——然后让子 Agent 基于冻结的契约并行工作。每个子 Agent 只携带它需要的上下文，全新启动。主 Agent 保持轻量，因为它在编排而不是实现。代码审查与下一阶段并行进行，不阻断进度。

这不是一个框架，只是我在多个项目中反复遇到相同失败模式后，把经验写下来的一套规则。

---

## 安装

```bash
mkdir -p ~/.claude/skills
curl -o ~/.claude/skills/multi-phase-feature-dev.md \
  https://raw.githubusercontent.com/Likeusewin10/rye-skills/main/skills/multi-phase-feature-dev/multi-phase-feature-dev.md
```

在 Claude Code 中调用：

```
/multi-phase-feature-dev
```

---

## 工作原理

```
主 Agent
  │
  ├── 读取代码库 → 设计接口契约 → 冻结
  │
  ├── 并行启动：
  │     ├── 后端子 Agent（基于冻结契约实现）
  │     └── 前端子 Agent（基于冻结契约实现）
  │
  ├── 两者均完成 → 设计下一阶段接口契约
  │
  ├── 并行启动：
  │     ├── 后端子 Agent（第 N+1 阶段）
  │     ├── 前端子 Agent（第 N+1 阶段）
  │     └── code-reviewer（第 N 阶段）← 不阻断进度，来自 ECC
  │
  └── 重复直到完成 → 提交
```

核心思路：主 Agent 在任何子 Agent 启动之前，先定义完整的 API 契约（端点、请求/响应结构、类型）。前后端基于同一份冻结规范独立实现，消除最常见的失败模式——前后端类型不匹配（由一方猜测另一方的返回结构导致）。

---

## 适用场景

| ✅ 适用 | ❌ 跳过 |
|---------|---------|
| 有明确前后端分工的功能 | 单文件改动 |
| 涉及 3 个以上文件 | 纯前端或纯后端 |
| 单阶段预估超过 30 分钟 | 紧急 hotfix |

---

## 内容

完整 skill 见 [`multi-phase-feature-dev.md`](multi-phase-feature-dev.md)，包含：

- 主 Agent 规则（接口所有权、并行执行、审查优先级）
- 接口契约模板
- 后端 / 前端 / code-reviewer 子 Agent 任务模板
- 完整执行流程图
- 常见陷阱

---

← [返回所有 skill](../../README.zh-CN.md)
