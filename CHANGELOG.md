# Changelog

## 0.1.0 (2026-05-29)

- 插件初始版本
- `skills/step/`：串行模式主 skill，5 阶段工作流（方案对齐 → 串行分发 → 收集反馈 → 测试循环 → 整理文档）
- `agents/impl-agent.md`：实现代理 F，落盘 + 自跑编译，F-V 直连修复-复验
- `agents/test-agent.md`：测试代理 V，评估覆盖 + 补测 + 跨模块跑测试，直连 F
- `commands/auto.md`：auto 并行模式，TeamCreate + 冲突检查
- F-V 测试循环直连：主代理只起 V、告知 agent_id、收报告，不参与中继
