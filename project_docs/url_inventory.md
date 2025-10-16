# 项目 URL 索引

说明：聚焦代码与模板中显式写死的 URL，标注精确文件与行号。已排除 .git 目录内部记录。

## 核心运行时外呼端点
- https://grok.com/rest/app-chat/conversations/new — [Python.GROK_API_ENDPOINT](app/services/grok/client.py:19)
- https://grok.com/rest/app-chat/upload-file — [Python.UPLOAD_ENDPOINT](app/services/grok/upload.py:16)
- https://grok.com/rest/rate-limits — [Python.RATE_LIMIT_ENDPOINT](app/services/grok/token.py:18)
- https://grok.com — 用作 Origin/Referer
  - [app/services/grok/statsig.py](app/services/grok/statsig.py:30)
  - [app/services/grok/cache.py](app/services/grok/cache.py:45)

## 资源域名 assets.grok.com
- https://assets.grok.com/{video_url} — [app/services/grok/processer.py](app/services/grok/processer.py:106)
- https://assets.grok.com/{img} — [app/services/grok/processer.py](app/services/grok/processer.py:171), [app/services/grok/processer.py](app/services/grok/processer.py:181), [app/services/grok/processer.py](app/services/grok/processer.py:184), [app/services/grok/processer.py](app/services/grok/processer.py:399), [app/services/grok/processer.py](app/services/grok/processer.py:411)
- https://assets.grok.com/{v_url} — [app/services/grok/processer.py](app/services/grok/processer.py:317)
- https://assets.grok.com{file_path} — [app/services/grok/cache.py](app/services/grok/cache.py:56), [app/services/grok/cache.py](app/services/grok/cache.py:58)

## 站内服务与本地示例
- http://localhost:8000/health — [AGENTS.md](AGENTS.md:26)
- http://localhost:8000/v1/chat/completions — [AGENTS.md](AGENTS.md:29), [AGENTS.md](AGENTS.md:36), [project_docs/agents_init_plan.md](project_docs/agents_init_plan.md:21)
- http://localhost:8000 — 占位示例输入 — [app/template/admin.html](app/template/admin.html:302)

## 前端模板与第三方 CDN
- https://cdn.tailwindcss.com — [app/template/login.html](app/template/login.html:8), [app/template/admin.html](app/template/admin.html:8)

## 文档与示例中的占位 URL
- https://你的服务器地址/v1/chat/completions — [readme.md](readme.md:28)
- https://your-image.jpg — [readme.md](readme.md:44)
- https://yourdomain.com — [app/template/admin.html](app/template/admin.html:252)

## 维护与仓库相关链接
- https://github.com/chenyme/grok2api/issues — [app/template/admin.html](app/template/admin.html:56)
- https://linux.do — [readme.md](readme.md:181)
- https://github.com/VeroFess/grok2api — [readme.md](readme.md:181)
- https://github.com/xLmiler/grok2api_python — [readme.md](readme.md:181)
- https://python-poetry.org/docs/basic-usage/#commit-your-poetrylock-file-to-version-control — [.gitignore](.gitignore:108)
- https://pdm.fming.dev/latest/usage/project/#working-with-version-control — [.gitignore](.gitignore:116)
- https://github.com/github/gitignore/blob/main/Global/JetBrains.gitignore — [.gitignore](.gitignore:168)
- https://docs.cursor.com/context/ignore-files — [.gitignore](.gitignore:182)