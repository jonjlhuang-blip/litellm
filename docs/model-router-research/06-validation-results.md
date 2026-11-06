# 06 - 本地部署与测试记录

> [F] 实测 | [D] 文档 | [I] 说明

## 1. Fork 状态 [F]

- 上游：https://github.com/BerriAI/litellm
- Fork：https://github.com/jonjlhuang-blip/litellm
- 最新 Commit（fork 与上游同步）：`bbc7de1ccb`  
  `chore(cost-map): add the openai o1 deprecation source from the deprecations page (#50402)`
- 默认分支：`main`
- 已配置 `upstream` remote 指向官方仓库
- 本地克隆路径：`/tmp/litellm-fork`
- 研究分支：`research/model-router`

## 2. 环境限制 [F]

- 当前执行环境（sandbox）**无 Docker**：`docker: command not found`
- 无法在本轮执行 `docker compose up` 或容器内启动 Proxy
- 因此以下能力未进行本地实测：
  - 服务启动
  - OpenAI-compatible API 访问
  - 多模型别名实际调用
  - Token/费用记录实际写入
  - Fallback 实际触发
  - Admin UI 可用性

## 3. 文档与源码验证 [F]

已通过源码与官方文档确认：
- 整体架构与调用链（`ARCHITECTURE.md`）
- Router 策略目录与 Auto Router 配置方式
- 国产模型 Provider 目录存在（dashscope / deepseek / moonshot）
- OpenAI-compatible 配置方法
- Docker 启动方式（`docker-compose.yml` 与 docs.litellm.ai/docs/proxy/docker_quick_start）

## 4. 建议的本地验证步骤（后续执行）

```bash
# 生成 master key
printf 'LITELLM_MASTER_KEY=sk-%s\nLITELLM_SALT_KEY=sk-%s\n' "$(openssl rand -hex 32)" "$(openssl rand -hex 32)" > .env

# 使用官方 quickstart
curl -sSLO https://github.com/BerriAI/litellm/raw/main/docker/docker-compose.quickstart.yml
docker compose -f docker-compose.quickstart.yml up -d

# 访问 UI: http://localhost:4000/ui  (admin / LITELLM_MASTER_KEY)
# 或使用 config 文件模式启动
```

## 5. 结论 [F]

本轮因环境限制未能完成 Docker 实测。  
架构与能力结论基于源码阅读与官方文档，已明确标注 [F]/[D]/[I]。  
下一步 MVP 开发前必须在具备 Docker 的环境中完成上述验证。

---
*验证时间：2026-11-07 | 环境：无 Docker 的 sandbox*
