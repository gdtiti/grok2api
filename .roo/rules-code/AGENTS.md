# AGENTS.md

This file provides guidance to agents when working with code in this repository.

- 新增 v1 路由必须显式继承认证依赖，保持与现有实现一致，避免出现未鉴权入口（参见 [app/api/v1/chat.py](app/api/v1/chat.py)、[app/api/v1/images.py](app/api/v1/images.py)）。
- 所有对 Grok 或外部服务的 HTTP 同步调用一律下放线程池执行，避免阻塞事件循环；复用现有客户端封装，勿在协程内直接发同步请求（参见 [app/services/grok/client.py](app/services/grok/client.py)）。
- 任何外呼 Header 注入必须通过 statsig 工具统一生成，禁止手写 x-statsig-id 以免缺失或不一致（参见 [app/services/grok/statsig.py](app/services/grok/statsig.py)、[data/setting.toml](data/setting.toml)）。
- 不自建重复 Logger；统一使用全局 Logger，避免重复 handler 造成日志膨胀（参见 [app/core/logger.py](app/core/logger.py)）。
- 面向上游的错误统一抛出项目自定义异常，由全局处理器映射为 OpenAI 兼容结构；禁止直接把下游错误原样透出（参见 [app/core/exception.py](app/core/exception.py)）。
- Token 使用约束：heavy 模型仅走 Super Token，其他模型优先 Normal 不可用再回退 Super；新增模型时遵循该选择策略（参见 [app/services/grok/token.py](app/services/grok/token.py)、[app/models/grok_models.py](app/models/grok_models.py)）。
- 文件存储模式下 token.json 键名必须为 ssoNormal 与 ssoSuper；如发现为 sso 与 ssoSuper，请迁移修正（参见 [app/core/storage.py](app/core/storage.py)、[data/token.json](data/token.json)）。
- 图片与视频处理走统一缓存与本地回源管线，响应内本地 URL 依赖 base_url；反代部署未配置将导致消费者拿到 404（参见 [app/services/grok/cache.py](app/services/grok/cache.py)、[app/api/v1/images.py](app/api/v1/images.py)、[app/services/grok/processer.py](app/services/grok/processer.py)）。
- 视频生成模型仅接受 1 张图片且需追加模式标记；多图输入会被截断，调用方需自行明确取舍（参见 [app/services/grok/client.py](app/services/grok/client.py)、[app/services/grok/processer.py](app/services/grok/processer.py)）。
- 代理必须透传并统一使用 socks5h 以保证远程 DNS 解析；勿在业务代码内自行拼装代理前缀（参见 [app/core/config.py](app/core/config.py)、[app/services/grok/upload.py](app/services/grok/upload.py)、[app/services/grok/client.py](app/services/grok/client.py)）。
- 配置与存储的耦合：所有动态配置变更需通过 setting 抽象进行持久化保存，避免绕过存储造成下次启动丢失（参见 [app/core/config.py](app/core/config.py)、[app/api/admin/manage.py](app/api/admin/manage.py)）。
- 流式输出必须复用统一的处理管线与超时管理，避免自建 SSE 逻辑与错误的超时控制（参见 [app/services/grok/processer.py](app/services/grok/processer.py)、[data/setting.toml](data/setting.toml)）。