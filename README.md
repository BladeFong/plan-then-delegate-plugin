# Plan-Then-Delegate

> 主代理对齐方案 + 整理文档，子代理串行/并行落盘。Claude Code 插件，保持主代理上下文清爽。

## 为什么用插件版

| | 插件版 | 非插件版 |
|---|---|---|
| 安装 | `/plugin install` 一键 | 手动 symlink + 确保 references 目录完整 |
| 子代理启动 | 内置 agent，规则预加载，无需 Read references | 冷启动时 Read `~/.claude/skills/.../references/` |
| 子代理能力 | 预装 caveman、android-cli、jetpack-compose-m3 等 7 个 skill | 裸启动，只靠项目 CLAUDE.md |
| 并行模式 | `/plan-then-delegate:auto` + TeamCreate + 冲突检查 | 仅串行 |
| 命名空间 | `plan-then-delegate:*`，不污染用户 skill 列表 | 占用 `plan-then-delegate` 短名 |

## 特性

- **内置 agent 预加载规则**：impl-agent 预装 caveman / android-cli / jetpack-compose-m3 / edge-to-edge / navigation-3 / styles / adaptive；test-agent 预装 caveman / testing-setup / android-cli。子代理 spawn 即带完整规则，不依赖外部 references 文件
- **F-V 测试循环直连**：测试代理发现问题直接 `SendMessage` 给实现代理，主代理只起 V、告知 agent_id、收最终报告
- **auto 并行模式**：TeamCreate 团队并行分发，自动冲突检查（文件路径不重叠、逻辑无交叉）
- **主代理不下场**：主代理专注方案判断与文档整理，编译/测试由子代理各自验收

## 安装

```bash
# 添加 marketplace（如已添加可跳过）
/plugin marketplace add <path-to-marketplace.json>

# 安装
/plugin install plan-then-delegate@plan-then-delegate-local
```

安装后可用 `/plan-then-delegate`（串行）和 `/plan-then-delegate:auto`（并行）。

## 使用

### 串行模式

```
/plan-then-delegate
```

一次一个子代理，等返回再起下一个。主代理排查根因 → 对齐方案 → 串行分发 → 轻量校验 → 整理文档。

### auto 模式（并行）

```
/plan-then-delegate:auto
```

通过 TeamCreate 并行分发多个 impl-agent，自动做冲突检查。适合多个独立问题。

### 测试循环

所有 F 完成后主代理询问是否补测试。进入循环后 F ↔ V 直连——V 发现业务问题直接发给 F，F 修复直接回 V，主代理不参与中继。

## 环境要求

- Claude Code **v2.1.32+**
- auto 模式和测试循环依赖 `SendMessage`，需启用 Agent Teams：

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

## 组件

### skills/step/SKILL.md

插件主 skill（命名空间 `plan-then-delegate:step`）。5 阶段工作流：方案对齐 → 串行分发 → 收集反馈 → 测试循环 → 整理文档。

### commands/auto.md

auto 模式命令（`plan-then-delegate:auto`）。TeamCreate + 并行 spawn + 冲突检查 + 收反馈 + TeamDelete。

### agents/impl-agent.md

实现代理 F。预装 caveman / android-cli / jetpack-compose-m3 / edge-to-edge / navigation-3 / styles / adaptive。落盘 + 自跑编译。测试循环中收 V 的业务问题直接修复。

### agents/test-agent.md

测试代理 V。预装 caveman / testing-setup / android-cli。评估覆盖 + 补测 + 跑跨模块测试。发现业务问题直连 F。

## 目录结构

```
plan-then-delegate-plugin/
├── .claude-plugin/{plugin.json, marketplace.json}
├── skills/step/{SKILL.md, references/test-loop.md}
├── agents/{impl-agent.md, test-agent.md}
├── commands/auto.md
├── README.md / CHANGELOG.md / LICENSE
└── .gitignore
```

## 相关项目

- [plan-then-delegate 非插件版](https://github.com/none/plan-then-delegate) —— CC 和 Codex 通用版本，通过 symlink 安装到 `~/.claude/skills/` 或 `~/.codex/skills/`。适合不需要子代理预加载能力的简单场景

## License

MIT
