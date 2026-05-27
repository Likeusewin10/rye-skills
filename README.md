<div align="right">
  <a href="README.zh-CN.md">中文</a> | <strong>English</strong>
</div>

# rye-skills

> A collection of Claude Code skills distilled from real-world development experience.

Each skill is a single `.md` file. Drop it into `~/.claude/commands/` and invoke it directly in Claude Code.

---

## Origin

This started from an interview question: *"Have you ever written a standardized skill or MCP?"*

It made me think — I'd been accumulating patterns and workflows from real projects for a while. Why not formalize them as skills? That question was the push I needed to start packaging what I'd learned into something reusable and shareable.

---

## Prerequisites

Some skills depend on [Everything Claude Code (ECC)](https://github.com/affaan-m/ECC) for agents and coding standards. Install ECC first if a skill lists it as a dependency.

---

## Skills

| Skill | Description | Requires ECC |
|-------|-------------|:------------:|
| [multi-phase-feature-dev](skills/multi-phase-feature-dev/) | Orchestrator pattern for multi-phase features — parallel frontend/backend subagents, frozen interface contracts, non-blocking code review | ✅ |

---

## Quick Install

```bash
mkdir -p ~/.claude/skills

# multi-phase-feature-dev
curl -o ~/.claude/commands/multi-phase-feature-dev.md \
  https://raw.githubusercontent.com/Likeusewin10/rye-skills/main/skills/multi-phase-feature-dev/multi-phase-feature-dev.md
```

Then invoke in Claude Code:

```
/multi-phase-feature-dev
```

---

## Contributing

Skills in this repo are extracted from real projects. If you have a pattern worth sharing, open an issue or PR.

---

## License

[MIT](LICENSE)
