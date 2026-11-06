# 05 - 分阶段实施计划

> [I] 建议路线

## Phase 0：预研（本轮完成）
- Fork 仓库
- 源码与文档调研
- 本地环境评估（Docker 受限已记录）
- 输出本目录文档

## Phase 1：MVP 底座（1-2 周）
1. 在 Fork 中维护 `config/model-router.yaml`（不修改上游核心）
2. 配置 economy / balanced / quality 三个 model groups
3. 接入已有 API Key（环境变量）
4. 启用 Auto Router（complexity_router）
5. 验证：多模型别名、Fallback、费用记录、Admin UI
6. 基础测试：流式、工具调用（用现有 Key）

## Phase 2：Agent 适配（2-3 周）
- 33 Coding Agent：session_affinity + 失败升级策略
- 数搭 Shuda：白名单 + 应用层校验接口
- 通用 Agent：四种模式切换（Header 或 Virtual Key）
- 路由决策日志增强（如需要，通过 proxy hook）

## Phase 3：高级能力（后续）
- 自定义 classifier（任务类型识别）
- 成本/质量权衡策略优化
- 国产模型深度适配与价格表维护
- 监控与告警集成

## 技术原则
- 优先配置，其次扩展点，最后才考虑 fork 修改核心
- 所有密钥走环境变量，不提交
- 保持与上游可同步

## 成功标准（MVP）
- Proxy 稳定运行
- 三种 tier 可配置且可切换
- 费用可追踪
- 失败可回退
- Coding Agent 基本工具调用可用
