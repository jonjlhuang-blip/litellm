# 01 - LiteLLM 整体架构与核心模块

> 标记说明：  
> [F] 已验证事实（源码或本地验证）  
> [D] 官方文档说明但未实测  
> [I] 分析推断或设计建议

## 1. 整体架构 [F]

LiteLLM 由两层组成：

1. **Python SDK** (`litellm/`)：负责与各 LLM Provider 的实际通信、请求/响应格式转换、流式处理、成本计算。
2. **AI Gateway / Proxy Server** (`litellm/proxy/`)：在 SDK 之上提供认证、限流、预算、虚拟 Key、管理界面、路由等网关能力。

```
Client (OpenAI SDK / curl / Anthropic SDK)
    ↓
LiteLLM Proxy (proxy/proxy_server.py)
    ↓ 认证 + 限流 + 预算检查
LiteLLM Router (router.py → RouterCore)
    ↓ 选择 deployment + fallback/retry
LiteLLM SDK (main.py → llms/{provider}/)
    ↓ 格式转换 + HTTP 调用
LLM Provider API
```

**源码依据**：
- 根目录 `ARCHITECTURE.md`（详细序列图与模块说明）
- `litellm/router.py`：`Router` 类将逻辑委托给 `RouterCore`（现有 Rust 核心）
- `litellm/proxy/proxy_server.py`：主要 API 端点

## 2. 请求完整调用链 [F][D]

根据 `ARCHITECTURE.md` 与官方文档 [Life of a Request](https://docs.litellm.ai/docs/proxy/architecture)：

1. 客户端 POST `/v1/chat/completions`
2. `proxy_server.py` → `user_api_key_auth()`（缓存优先，miss 查 Postgres）
3. Hooks：并行请求限流器、缓存检查等
4. Router 选择 deployment（策略 + 健康检查 + cooldown）
5. `litellm.acompletion()` → Provider 特定 Transformation → HTTP 请求
6. 响应返回后异步计算成本、写 spend log、更新预算

**关键文件**：
- `proxy/auth/user_api_key_auth.py`
- `router.py` / `router_core.py`
- `llms/custom_httpx/llm_http_handler.py`
- `cost_calculator.py` + `proxy/hooks/proxy_track_cost_callback.py`

## 3. 核心模块 [F]

| 模块 | 路径 | 职责 |
|------|------|------|
| Proxy Server | `litellm/proxy/proxy_server.py` | HTTP 端点、生命周期 |
| Auth | `litellm/proxy/auth/` | Virtual Key、JWT、OAuth |
| Router | `litellm/router.py` + `router_strategy/` | 负载均衡、Fallback、策略 |
| Provider Adapters | `litellm/llms/{provider}/` | 格式转换 |
| Cost & Logging | `cost_calculator.py`, spend logs | Token/费用追踪 |
| Admin UI | `ui/` | 管理界面 |

## 4. 基础设施依赖 [D]

- PostgreSQL：API Key、团队、Spend Logs（schema.prisma）
- Redis：限流计数器、Key 缓存、Cooldown、TPM/RPM（可选，生产推荐）

无数据库时仍可运行纯配置文件模式，但无 Virtual Key 与 Spend Tracking。

---
*生成时间：2026-11-07 | 基于 commit bbc7de1ccb*
