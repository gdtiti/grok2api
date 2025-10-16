# AGENTS.md

This file provides guidance to agents when working with code in this repository.

- 日志排查优先看文件输出：统一写入 logs/app.log；若日志量异常，检查是否重复注册 handler（参见 [app/core/logger.py](app/core/logger.py) 与日志初始化处）。
- x-statsig-id 缺失会直接导致外呼失败：定位 4xx 先核对配置与注入逻辑（参见 [app/services/grok/statsig.py](app/services/grok/statsig.py)、[data/setting.toml](data/setting.toml)）。
- 代理相关问题优先确认前缀已被规范化为 socks5h（远程 DNS）：若仍超时或 403，检查上游访问是否被本地 DNS 影响（参见 [app/core/config.py](app/core/config.py)、[app/services/grok/client.py](app/services/grok/client.py)、[app/services/grok/upload.py](app/services/grok/upload.py)）。
- 启动报错直查存储配置：当 STORAGE_MODE=mysql 或 redis 且缺少 DATABASE_URL，会在启动阶段短路失败（参见 [app/core/storage.py](app/core/storage.py)）。
- SSE 卡顿或断流：三类超时可配，复现时开启流式并减小超时阈值定位问题（参见 [app/services/grok/processer.py](app/services/grok/processer.py)、[data/setting.toml](data/setting.toml)）。
- 反向代理场景 404 媒体链接：响应内本地 URL 依赖 base_url；未配置会导致消费者收到不可达链接（参见 [app/services/grok/processer.py](app/services/grok/processer.py)、[app/api/v1/images.py](app/api/v1/images.py)）。
- /images 回源路径中连字符将被还原为斜杠：排查 404 时注意路径映射逻辑（参见 [app/api/v1/images.py](app/api/v1/images.py)）。
- heavy 模型限速与令牌选择异常：优先核对 token.json 键名与配额字段；heavy 仅走 Super，其他模型优先 Normal 不可用再回退（参见 [app/services/grok/token.py](app/services/grok/token.py)、[data/token.json](data/token.json)、[app/models/grok_models.py](app/models/grok_models.py)）。
- 视频生成疑难：仅接受 1 张图，多图会被截断；进度事件结束后返回包含 video 标签的最终内容（参见 [app/services/grok/client.py](app/services/grok/client.py)、[app/services/grok/processer.py](app/services/grok/processer.py)）。
- 认证与权限问题定位：/v1 路由均启用鉴权，/images 不鉴权；出现 401/403 时分别从凭据与限速角度排查（参见 [app/api/v1/chat.py](app/api/v1/chat.py)、[app/api/v1/images.py](app/api/v1/images.py)）。
- 设置热更新与持久化：经管理接口更新设置后需通过 setting 抽象持久化保存，否则重启丢失（参见 [app/api/admin/manage.py](app/api/admin/manage.py)、[app/core/config.py](app/core/config.py)）。