# AGENTS.md

This file provides guidance to agents when working with code in this repository.

- 端口差异与运行：容器暴露 8000（[Dockerfile](Dockerfile:37)、[docker-compose.yml](docker-compose.yml:5)），直接运行 [uvicorn.run](main.py:80) 默认 8001。建议本地统一 8000：
```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```
- 授权依赖：/v1 接口必须带 Authorization，/images 无需认证（[auth_manager.verify](app/api/v1/chat.py:25)、[get_image](app/api/v1/images.py:13)）。
- 必填 header：所有外呼经 [get_dynamic_headers](app/services/grok/statsig.py:9) 注入 x-statsig-id，缺失将抛错（[app/services/grok/statsig.py](app/services/grok/statsig.py:19)）；保持配置默认值（[data/setting.toml](data/setting.toml:6)）。
- 代理规范：配置层将 socks5:// 自动重写为 socks5h://（[ConfigManager.load](app/core/config.py:31)），HTTP 客户端透传该代理（[GrokClient._send_request](app/services/grok/client.py:153)）。
- Token 文件结构陷阱：运行期期望 ssoNormal ssoSuper（[app/models/grok_models.py](app/models/grok_models.py:86)、[GrokTokenManager._load_data](app/services/grok/token.py:161)），而文件存储初始写入 sso ssoSuper（[FileStorage.init_db](app/core/storage.py:64)）。请将 data/token.json 迁移为：
```json
{"ssoNormal": {}, "ssoSuper": {}}
```
- 存储模式切换：当 STORAGE_MODE 为 mysql 或 redis 时，必须提供 DATABASE_URL，否则启动短路失败（[StorageManager.init](app/core/storage.py:466)）。
- 媒体本地回源：图片 视频缓存到 data/temp，通过 [/images](app/api/v1/images.py:13) 回源；响应内本地 URL 依赖 base_url，反代未设将 404（[process_normal](app/services/grok/processer.py:176)、[process_stream](app/services/grok/processer.py:323)）。
- 流式超时：chunk 首次响应 总超时均可配（[app/services/grok/processer.py](app/services/grok/processer.py:237)、[data/setting.toml](data/setting.toml:8)）。
- Heavy 模型与令牌：grok-4-heavy 仅用 Super Token 且按 heavyremainingQueries 选择（[token_manager.select_token](app/services/grok/token.py:214)）；其他模型优先 Normal，不可用回退 Super（[token_manager.select_token](app/services/grok/token.py:222)）。
- 视频模型契约：仅接受 1 张图并强制追加 --mode=custom（[GrokClient.openai_to_grok](app/services/grok/client.py:47) 到 [app/services/grok/client.py](app/services/grok/client.py:56)）；完成后返回 video 标签（[process_stream](app/services/grok/processer.py:315)）。
- 阻塞 I O 卸载：对 Grok 的同步 HTTP 请求必须下放线程池，避免阻塞事件循环（[GrokClient._send_request](app/services/grok/client.py:169)）。
- 统一异常与错误结构：对外抛 [GrokApiException](app/core/exception.py:9)，由全局处理器映射为 OpenAI 兼容错误（[register_exception_handlers](app/core/exception.py:115)）。
- 日志：统一 RotatingFileHandler 输出到 logs/app.log，避免自建 logger 重复 handler（[setup_logger](app/core/logger.py:44)）。
- 端到端最小校验：
```bash
curl http://localhost:8000/health
```
```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d "{\"model\":\"grok-4-fast\",\"messages\":[{\"role\":\"user\",\"content\":\"ping\"}]}"
```
- 流式最小复现：
```bash
curl -N http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d "{\"model\":\"grok-4-fast\",\"messages\":[{\"role\":\"user\",\"content\":\"hi\"}],\"stream\":true}"