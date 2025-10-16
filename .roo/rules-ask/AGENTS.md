# AGENTS.md

This file provides guidance to agents when working with code in this repository.

- 模型清单与别名的唯一事实来源在 [app/models/grok_models.py](app/models/grok_models.py)，/v1/models 接口直接以此为准，新增模型需同步维护映射与 is_video_model 标记（参见 [app/api/v1/models.py](app/api/v1/models.py)）。
- 文档样例中的媒体链接依赖 base_url 拼装；若反向代理未配置 base_url，会出现文档可跑通 API 但媒体 404 的悖论（参见 [data/setting.toml](data/setting.toml)、[app/services/grok/processer.py](app/services/grok/processer.py)）。
- /images 文档路径是将请求中的连字符还原为斜杠的回源规则，撰写示例时避免直接粘贴真实子目录结构，应按连字符形式构造（参见 [app/api/v1/images.py](app/api/v1/images.py)）。
- 流式响应文档需注明三段超时可调，避免误判为模型侧超时；建议在示例中给出 -N 与最小超时阈值的复现实验（参见 [app/services/grok/processer.py](app/services/grok/processer.py)、[data/setting.toml](data/setting.toml)）。
- 所有外呼示例必须通过 statsig 生成头部，禁止在文档中手写 x-statsig-id；否则读者照抄会直接 4xx（参见 [app/services/grok/statsig.py](app/services/grok/statsig.py)）。
- 重模型额度说明：grok-4-heavy 仅使用 Super Token，其他模型优先 Normal；文档中应明确配额字段 heavyremainingQueries 的约束以免误导（参见 [app/services/grok/token.py](app/services/grok/token.py)、[data/token.json](data/token.json)）。
- 视频生成功能文档需强调仅支持 1 张图，多图会被截断且文本会追加模式标记；结尾产出包含 video 标签，读者应以此为完成信号（参见 [app/services/grok/client.py](app/services/grok/client.py)、[app/services/grok/processer.py](app/services/grok/processer.py)）。
- 管理端会话为进程内存级，不持久化；在文档中避免承诺跨重启登录态延续（参见 [app/api/admin/manage.py](app/api/admin/manage.py)）。
- 错误结构在文档中应标注为 OpenAI 兼容，由全局处理器统一映射；不要在示例中直抛底层错误堆栈（参见 [app/core/exception.py](app/core/exception.py)）。
- 日志位置固定为 logs/app.log，排障章节请指向该路径而非控制台输出，避免误导（参见 [app/core/logger.py](app/core/logger.py)）。