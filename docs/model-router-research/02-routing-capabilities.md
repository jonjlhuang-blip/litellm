# 02 - 现有路由能力与源码依据

> [F] 源码确认 | [D] 文档 | [I] 推断

## 1. 已有路由策略 [F]

位于 `litellm/router_strategy/`：

| 策略 | 文件 | 说明 |
|------|------|------|
| simple-shuffle | `simple_shuffle.py` | 默认，加权随机 |
| lowest-latency | `lowest_latency.py` | 最低延迟 |
| lowest-cost | `lowest_cost.py` | 最低成本 |
| least-busy | `least_busy.py` | 最少忙碌 |
| lowest-tpm-rpm | `lowest_tpm_rpm*.py` | 最低 TPM/RPM 使用 |
| complexity_router | `complexity_router/` | 复杂度分类（启发式/LLM） |
| quality_router | `quality_router/` | 质量档位 |
| adaptive_router | `adaptive_router/` | 自适应（bandit） |
| auto_router | `auto_router/` | 语义/示例匹配 |
| tag-based | `tag_based_routing.py` | 标签路由 |

**Auto Router v2**（文档 [D]）：支持 heuristic、LLM classifier、keyword rules、tier pools、session affinity。

配置示例见 README 与 docs.litellm.ai/docs/auto_router/。

## 2. Fallback / Retry / Load Balancing [F][D]

- **Model Group**：同一 `model_name` 下多个 deployment 可负载均衡。
- **Fallbacks**：跨 group 失败回退（`router_settings.fallbacks`）。
- **Retries**：同 group 内重试。
- **Cooldown**：失败 deployment 进入冷却（Redis 共享）。
- **Routing Groups**：可为不同 group 指定不同策略。

源码：`router.py` 注释 + `ARCHITECTURE.md` 序列图。

## 3. 如何扩展自定义路由分类器/策略 [F][I]

1. 实现 `CustomRoutingStrategyBase`（`litellm/types/router.py`）。
2. 在 `router_strategy/` 下新增策略类。
3. 通过 `router_settings.routing_strategy` 或插件注册。
4. Auto Router 可通过 `complexity_router_config` 自定义 tiers 与 classifier_type。

**建议 [I]**：对于我们的 Model Router，优先复用 complexity_router + 自定义 classifier，而非重写核心。

## 4. 会话锁定与流式/工具调用 [D]

- Session affinity：Auto Router 支持 `session_affinity` 锁定首次选择模型。
- 流式：SDK 与 Proxy 均支持（`streaming_handler.py`）。
- Tools / Function Calling：通过 Provider Transformation 层统一。
- Cache-Aware Routing：支持 Kubernetes pod 级 session affinity（文档）。

## 5. 按任务类型/成本/延迟/质量选择 [F][D]

已支持：
- 成本：`lowest-cost`
- 延迟：`lowest-latency`
- 复杂度：`complexity_router`（SIMPLE/MEDIUM/COMPLEX/REASONING tiers）
- 质量：`quality_router`
- 自适应：`adaptive_router`

可组合使用 routing groups + fallbacks。

---
*基于源码 litellm/router_strategy/ 与官方文档*
