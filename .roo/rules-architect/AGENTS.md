# AGENTS.md

This file provides guidance to agents when working with code in this repository.

- 启动生命周期耦合：必须按顺序初始化存储并注入到设置与令牌管理，再触发二次加载（[main.py](main.py:21) 到 [main.py](main.py:31)）。调整顺序会导致配置与 token 状态不同步。
- 静态资源挂载顺序：必须在 API 路由之后挂载，否则会拦截接口（[main.py](main.py:60) 到 [main.py](main.py:62)）。
- 阻塞 I O 线程池卸载：对 Grok 的 HTTP 同步请求通过 asyncio.to_thread 下放到线程池，避免阻塞事件循环；新增同步 I O 必须复用该模式（[GrokClient._send_request()](app/services/grok/client.py:169)）。
- 存储抽象与回落：StorageManager 会根据 STORAGE_MODE 装配 File MySQL Redis 并统一通过文件层做持久化同步，切换到 mysql redis 缺少 DATABASE_URL 会在启动阶段短路失败（[StorageManager.init()](app/core/storage.py:451) 到 [app/core/storage.py](app/core/storage.py:475)、[app/core/storage.py](app/core/storage.py:466)）。
- 令牌限速更新为异步旁路：响应成功后以任务方式更新限速信息，调用方不要同步依赖该结果（[GrokClient._process_response()](app/services/grok/client.py:231) 到 [app/services/grok/client.py](app/services/grok/client.py:237)、[token_manager.check_limits()](app/services/grok/token.py:244)）。
- 流式过滤与标签：服务端在流式阶段会根据 filtered_tags 丢弃含特定标签的 token，错误配置会造成内容缺失（[process_stream()](app/services/grok/processer.py:231)、[process_stream()](app/services/grok/processer.py:428)）。
- 媒体缓存与回源：图片 视频统一本地缓存并生成 /images 回源链接，反代部署时 base_url 必填以确保可达（[image_cache_service](app/services/grok/cache.py:178)、[video_cache_service](app/services/grok/cache.py:179)、[process_normal()](app/services/grok/processer.py:176)、[process_stream()](app/services/grok/processer.py:323)、[get_image()](app/api/v1/images.py:13)）。
- 视频生成契约：video 模型仅接受 1 张图，文本强制追加模式标记并以进度事件驱动完成态输出 video 标签（[GrokClient.openai_to_grok()](app/services/grok/client.py:47)、[app/services/grok/client.py](app/services/grok/client.py:56)、[process_stream()](app/services/grok/processer.py:298)、[process_stream()](app/services/grok/processer.py:320)）。
- 代理一致性与远程 DNS：配置层将 socks5 自动升级为 socks5h，HTTP 客户端按该代理透传，避免本地 DNS 造成访问异常（[ConfigManager.load()](app/core/config.py:31)、[GrokClient._send_request()](app/services/grok/client.py:153)、[ImageUploadManager.upload()](app/services/grok/upload.py:65)、[GrokTokenManager.check_limits()](app/services/grok/token.py:258)）。