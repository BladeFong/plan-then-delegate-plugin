# Plan-Then-Delegate

> 主代理对齐方案 + 整理文档，子代理串行/并行落盘。一个保持主代理上下文清爽的 Claude Code 工作流插件。

## 特性

- **轻量实现**：1 个 skill + 2 个内置 agent + 1 个 auto 命令
- **串行模式**（`/plan-then-delegate`）：主代理排查根因、与用户对齐方案后，串行分发子代理落盘
- **auto 模式**（`/plan-then-delegate:auto`）：TeamCreate 团队并行分发，含冲突检查
- **F-V 测试循环直连**：测试代理发现问题直接发给实现代理，主代理不参与中继
- **主代理上下文清爽**：主代理不下场写代码、不重复跑编译，专注方案判断与文档整理

## 安装

### Marketplace 安装

```bash
# 添加本地 marketplace
/plugin marketplace add /home/lanef/projects_skills/plan-then-delegate/repo/claude-code/.claude-plugin/marketplace.json

# 安装插件
/plugin install plan-then-delegate@plan-then-delegate-local
```

### 手动安装（开发调试）

```bash
ln -s "$(pwd)" ~/.claude/plugins/cache/plan-then-delegate-local/plan-then-delegate/0.1.0
```

## 使用

### 串行模式

```
/plan-then-delegate
```

一次一个子代理，等返回再起下一个。适合改动之间有依赖关系的场景。

### auto 模式（并行）

```
/plan-then-delegate:auto
```

通过 TeamCreate 团队模式并行分发多个实现子代理。会自动做冲突检查（文件路径不重叠、逻辑无交叉）。适合多个独立问题的场景。

### 工作流

```
用户报问题
       │
主代理排查 → 与用户对齐方案
       │
       ▼
[串行/并行] 实现子代理落盘 + 自跑编译 → 返回修改要点
       │
       ▼
（可选）测试循环：V 直连 F，主代理不参与中继
       ▼
主代理统一整理项目文档
```

## 环境要求

- Claude Code **v2.1.32+**
- auto 模式和测试循环依赖 `SendMessage` 工具，需启用 Agent Teams：

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

启用后重启 Claude Code 会话，验证工具列表中包含 `SendMessage`。

## 目录结构

```
plan-then-delegate-plugin/
├── .claude-plugin/
│   ├── plugin.json           # 插件 manifest
│   └── marketplace.json      # marketplace 入口
├── skills/
│   └── step/
│       ├── SKILL.md          # 串行模式主 skill
│       └── references/
│           └── test-loop.md  # 测试循环详细规则
├── agents/
│   ├── impl-agent.md         # 实现代理 F：落盘 + 自跑编译
│   └── test-agent.md         # 测试代理 V：评估 + 补测 + 跑测试
├── commands/
│   └── auto.md               # auto 并行模式子命令
├── README.md
├── CHANGELOG.md
├── LICENSE
└── .gitignore
```

## 组件说明

### skills/step/SKILL.md

插件主 skill（命名空间 `plan-then-delegate:step`）。用户触发后按 5 阶段执行：方案对齐 → 串行分发 → 收集反馈 → 测试循环 → 整理文档。

### commands/auto.md

auto 模式子命令（`plan-then-delegate:auto`）。通过 TeamCreate 创建团队，并行 spawn 多个 impl-agent，各 agent 间做冲突检查。

### agents/impl-agent.md

实现代理 F。落盘代码改动 + 自跑编译，通过后返回修改清单。测试循环中接收 V 的业务问题直接修复。

### agents/test-agent.md

测试代理 V。评估测试覆盖 + 新增/调整测试 + 跑跨模块测试。发现业务问题直连 F，不经过主代理。

## 相关项目

- [plan-then-delegate 非插件版](https://github.com/none/plan-then-delegate) —— CC 和 Codex 通用版本，通过 symlink 安装到 `~/.claude/skills/` 或 `~/.codex/skills/`

## License

MIT
