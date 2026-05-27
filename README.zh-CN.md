<div align="right">
  <strong>中文</strong> | <a href="README.md">English</a>
</div>

# rye-skills

> 从真实开发经验中提炼的 Claude Code skill 合集。

每个 skill 是一个独立的 `.md` 文件，放入 `~/.claude/commands/` 后即可在 Claude Code 中直接调用。

---

## 起源

这个项目的灵感来自一场面试。面试官问我：*"你有没有写过标准化的 skill 或者 MCP？"*

这个问题让我意识到——我在真实项目中积累了不少方法和工作流，为什么不把它们整理成 skill，做成可复用、可分享的东西呢？就是那个问题，推动我开始把自己总结的经验正式打包成这个仓库。

---

## 前置依赖

部分 skill 依赖 [Everything Claude Code (ECC)](https://github.com/affaan-m/ECC) 提供的 agent 和编码规范。如果 skill 标注了 ECC 依赖，请先安装 ECC。

---

## Skill 列表

| Skill | 描述 | 需要 ECC |
|-------|------|:--------:|
| [multi-phase-feature-dev](skills/multi-phase-feature-dev/) | 多阶段功能开发的 Orchestrator 模式 — 前后端子 Agent 并行、冻结接口契约、代码审查不阻断进度 | ✅ |

---

## 快速安装

```bash
mkdir -p ~/.claude/skills

# multi-phase-feature-dev
curl -o ~/.claude/commands/multi-phase-feature-dev.md \
  https://raw.githubusercontent.com/Likeusewin10/rye-skills/main/skills/multi-phase-feature-dev/multi-phase-feature-dev.md
```

在 Claude Code 中调用：

```
/multi-phase-feature-dev
```

---

## 贡献

这里的 skill 都来自真实项目的经验总结。如果你有值得分享的模式，欢迎提 issue 或 PR。

---

## 许可证

[MIT](LICENSE)
