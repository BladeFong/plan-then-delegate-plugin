---
description: >
  plan-then-delegate 的 auto 模式。主代理通过 TeamCreate 团队模式并行分发多个实现子代理。
  任务拆分需做冲突检查。要求 TeamCreate 和 SendMessage 工具可用。
---

# Plan-Then-Delegate（auto 模式）

## 前置检查

主代理首先确认工具列表中存在 `TeamCreate` 和 `SendMessage`。缺任一则告知用户并停止。

## 核心原则

主代理只做两件事：**和用户对齐方案** + **整理文档**。具体代码改动通过插件内置 agent 落实，**通过 TeamCreate 团队模式并行分发**。

主代理**不**：
- 给子代理的落实细节提意见
- 自己接手问题和下场写代码改文件（除非用户明确要求"这次你直接改"）
- 自己看子代理处理问题后的 log
- 用 `run_in_background=true` 起做写操作的子代理
- 让子代理来写文档
- 自己跑编译 / 测试验证（编译归 F，测试归 V）

## 工作流

1. **方案对齐**（第 1 节）：主代理排查根因，与用户确认方案
2. **并行分发**（第 2 节）：冲突检查 → TeamCreate → 并行 spawn 多个 impl-agent → 等待全部返回 → TeamDelete
3. **收集反馈**（第 3 节）：主代理对每个子代理返回做轻量校验
4. **补充测试循环**（第 4 节，可选）：所有 F 完成后询问用户；同意则进入 test-agent ↔ impl-agent 循环
5. **统一整理文档**（第 5 节）

测试循环保持串行（每对 F-V 轮流），不受 auto 模式影响。

### 1. 排查与方案对齐

通过 Read / Grep / Glob 等只读工具排查根因，呈现方案给用户，**等用户明确确认**后再分发。

### 2. 并行分发

#### 冲突检查（分发前必做）

拆分任务时确保并行 agent 间：
- 改动文件路径不重叠
- 改动逻辑无交叉
- 每个 agent prompt 中明确列出「不要碰的文件」

#### 分发流程

1. **TeamCreate**：创建团队（如 `ptd-auto`）
2. **并行 spawn**：一条消息中并发多个 Agent 调用（`subagent_type: plan-then-delegate:impl-agent`，`team_name: ptd-auto`），每个带完整 prompt 和 `name`
3. **保存 agent_id**：记录每个 F 的 name ↔ agent_id
4. **等待全部返回**：进入第 3 节
5. **TeamDelete**：收集反馈后清理

#### prompt 模板

```
## 任务背景  [必填]
<一句话说明问题>

## 根因  [必填]
<主代理已排查的根因>

## 已确认方案  [必填]
- 改动文件：<file_path>
- 关键代码 / 函数：<片段或定义>
- 命名 / 风格：<用户特别强调的偏好>

## 不要碰的文件  [auto 模式必填]
- <其他子代理正在处理的文件>

## 特殊编译命令  [可选]
```

返工时用 SendMessage 续接同一 agent_id。

### 3. 收集反馈并继续

轻量校验（偏离方案？遗留点？），**不**重读文件验证。正常则继续，否则返工。

### 4. 补充测试循环（可选）

所有 F 完成后询问用户。同意则进入 test-agent ↔ impl-agent 循环，每对严格串行。

主代理冷启动 test-agent 时传入 F 的 agent_id，之后 F ↔ V 通过 SendMessage 直连。主代理不参与中继，只等 V 最终报告。

详见插件 `skills/step/references/test-loop.md`。

### 5. 统一整理文档

询问用户是否整合文档，按项目惯例，不跑编译/测试。

## 何时不用

- 单个简单改动：直接改更快
- 用户说"这次你直接改"
- 纯研究/探索任务
- TeamCreate 或 SendMessage 不可用时改用 `/plan-then-delegate`
