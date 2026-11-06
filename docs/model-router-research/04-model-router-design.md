# 04 - Model Router 架构初稿

> [I] 设计建议，基于 LiteLLM 已有能力

## 1. 总体思路

以 LiteLLM Proxy 为底座，通过配置 + 少量自定义策略实现统一 Model Router，避免大规模修改上游核心。

```
Agent (33 / 数搭 / 通用)
    ↓  model="auto" 或特定 alias
LiteLLM Proxy
    ↓
Auto Router / Complexity Router (已有)
    ↓  tier → model group
Model Groups:
  - economy (低成本：DeepSeek-V3 / Qwen-Flash / GPT-4o-mini)
  - balanced (主力：GPT-4o / Claude-3.5 / Qwen-Max)
  - quality (高能力：Claude-Opus / GPT-4.1 / o1 等)
    ↓
Provider (国产 + 国际)
```

## 2. 四种路由模式 [I]

| 模式 | 实现方式 | 说明 |
|------|----------|------|
| 经济 | 固定 economy group | 简单任务 |
| 平衡 | 固定 balanced group | 常规开发 |
| 质量 | 固定 quality group | 复杂架构/Bug |
| 自动 | `auto_router/complexity_router` | 根据 prompt 自动选 tier |

可通过 Virtual Key 或 Header 指定模式，或在 config 中定义多个 router model。

## 3. 针对具体 Agent

### 33 Coding Agent
- 简单编码 → economy
- 常规开发 → balanced
- 复杂架构/难修 Bug → quality
- 重点：session_affinity 锁定会话模型、tool calling 兼容性、失败自动升级（fallback）

### 数搭 Shuda 财务问数
- 简单问题 → economy
- 复杂 SQL/指标 → quality（强制）
- **确定性**：应用层必须做 SQL 与结果校验，路由不参与计算逻辑。
- 使用 model whitelist + budget。

### 通用 Agent
- 支持上述四种模式
- Budget 上限、失败回退、模型白名单
- 记录路由原因（LiteLLM 已支持 decision log）与实际费用

## 4. 扩展点

- 自定义 classifier（在 complexity_router 上扩展）
- 自定义 routing strategy（实现 `CustomRoutingStrategyBase`）
- 通过 proxy hooks 增加路由决策日志

## 5. 已有 vs 待开发

| 能力 | 状态 | 说明 |
|------|------|------|
| 多模型别名 | 已有 | model_list |
| Fallback / Retry | 已有 | 配置即可 |
| 复杂度路由 | 已有 | Auto Router v2 |
| 成本/延迟策略 | 已有 | router_strategy |
| Session 锁定 | 已有 | session_affinity |
| 费用追踪 | 已有 | Spend Logs + headers |
| 自定义任务分类器 | 部分 | 需二次开发 classifier |
| 按业务规则强制升级 | 待开发 | 可基于 hooks 或应用层 |
| 财务确定性保证 | 应用层 | 不依赖网关 |

---
*本设计以不修改上游核心为前提*
