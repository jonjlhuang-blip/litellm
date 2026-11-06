# 03 - 国产模型及 Coding Agent 兼容性

> [F] 源码/目录确认 | [D] 文档 | [I] 建议

## 1. 已支持国产/相关 Provider [F]

源码目录 `litellm/llms/` 下存在：

- `dashscope/` — 阿里云 DashScope（Qwen 系列）
- `deepseek/` — DeepSeek
- `moonshot/` — Moonshot（Kimi）
- `openai_like/` / `openai/` — 通用 OpenAI-compatible

文档 [D]：
- Inference APIs 聚合了 DeepSeek、Qwen、GLM、Kimi 等。
- 支持 `-latest` 别名（如 `dashscope/qwen-max-latest`）。
- Qwen Code CLI 有专门教程。

## 2. 自定义 OpenAI-compatible API [F][D]

配置方式（`config.yaml`）：

```yaml
model_list:
  - model_name: my-qwen
    litellm_params:
      model: openai/qwen-plus          # openai/ 前缀
      api_base: https://dashscope.aliyuncs.com/compatible-mode/v1
      api_key: os.environ/DASHSCOPE_API_KEY
```

或使用 `custom_openai/` 前缀。  
注意 `api_base` 需带 `/v1`。

源码：`llms/openai_like/` 与文档 OpenAI-Compatible Endpoints。

## 3. Coding Agent 支持 [D]

- 流式输出：原生支持。
- 工具调用（function calling / tools）：通过统一 Transformation 层。
- 会话连续性：
  - Auto Router `session_affinity`
  - Cache-Aware Routing（K8s）
  - 客户端可自行传递 session id。
- 官方提供 Agent Harness（Claude Code、Codex、OpenCode 等），可走 Proxy。

**风险 [I]**：部分国产模型 tool calling 实现与 OpenAI 不完全一致，需实测。

## 4. 财务确定性保障建议 [I]

LiteLLM 路由本身是概率性的。对于数搭 Shuda：

- 复杂 SQL/指标分析必须走固定高能力模型（白名单 + 强制 tier）。
- 或在应用层做二次校验（数据口径、SQL 校验），不依赖路由结果的确定性。
- 使用 Virtual Key + Budget + Logging 追踪每一次路由决策与费用。

---
*国产模型支持以源码目录与官方文档为准，具体模型价格与可用性需以 Provider 文档为准*
