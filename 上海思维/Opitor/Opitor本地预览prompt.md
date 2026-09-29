请启动并检查本项目的“本地虚拟演示环境”，不要启动 BMS 生产环境。

项目目录：
C:\Users\PC\Desktop\MyOptio

目标地址：
http://localhost:5183/

固定架构：
- 前端 Vite：127.0.0.1:5183
- 演示 API：127.0.0.1:8840
- API 必须代理到：http://127.0.0.1:8840
- 数据库：opitor_local_demo_qa_20260928
- 配置文件：
  C:\Users\PC\.codex\local-runtime\opitor\local-demo.env
- 演示账号：demo-admin
- 演示密码：Opitor-Demo-2026!
- BMS：关闭
- 不得连接 5173、8787、server/.env、opitor_dev 或任何生产数据库

启动要求：
1. 先检查 5183 和 8840 是否已有进程。
2. 已有且健康的进程直接复用，不重复启动。
3. API 不健康时，使用 Node.js 22 和上述 local-demo.env 启动 8840。
4. 前端必须在 web 目录启动，并设置：
   ADJUTANT_API_TARGET=http://127.0.0.1:8840
5. Vite 必须使用 `--host 127.0.0.1 --port 5183 --strictPort`。
6. 不要因为数据已有内容而重新 seed、清库、reset 或 migrate reset。
7. 只有明确要求时，才运行演示数据 seed；运行前必须确认不会覆盖现有数据。
8. 启动后检查：
   - http://localhost:5183/ 返回 200
   - http://localhost:5183/api/auth/providers 返回 200
   - http://127.0.0.1:8840/healthz 返回 200
9. 如果页面打不开，先判断是前端 5183、API 8840 还是代理配置问题，只重启属于本演示环境的进程。
10. 不要停止 5173/8787，也不要杀未知进程。
11. 保留 Git 工作区现有改动，不执行 git reset、git clean、git checkout、git add .。

最后请报告：
- 前端是否已启动
- API 是否健康
- 实际使用的端口和数据库
- 登录账号
- 当前 Git HEAD
- 是否发现其他异常

Prompt 只能让启动过程更清楚，不能完全保证长期稳定。现在演示前端还是手动启动的，重启电脑后需要重新拉起。更稳定的做法是把 `5183/8840` 封装成独立的 `scripts/dev-demo.mjs`，提供：

```
node scripts/dev-demo.mjs start
node scripts/dev-demo.mjs status
node scripts/dev-demo.mjs restart
node scripts/dev-demo.mjs stop
```