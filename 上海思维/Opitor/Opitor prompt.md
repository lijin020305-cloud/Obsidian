请在 C:\Users\PC\Desktop\MyOptio 启动并验证本地虚拟演示环境。

不要启动或修改正式 BMS 环境：
- 不使用 5173
- 不使用 8787
- 不连接 BMS
- 不读取或修改 server/.env
- 不操作 opitor_dev 数据库

目标演示环境：
- 前端：http://localhost:5183/
- API：http://127.0.0.1:8840
- 数据库：opitor_local_demo_qa_20260928
- 数据库连接：postgresql://postgres:0000@127.0.0.1:5432/opitor_local_demo_qa_20260928
- 配置文件：C:\Users\PC\.codex\local-runtime\opitor\local-demo.env
- 数据目录：/tmp/opitor-local-demo-20260928
- 用户名：demo-admin
- 密码：Opitor-Demo-2026!
- Node.js：使用 Node 22，优先：
  C:\Users\PC\Downloads\node-v22.23.1-win-x64\node.exe

第一步：检查代码版本
1. 执行 git status --short --branch
2. 执行 git rev-parse HEAD
3. 不要覆盖已有未提交修改
4. 如果需要检查 GitHub，先比较 origin/main；不要自动 reset、clean 或覆盖用户修改

第二步：检查旧进程
1. 检查 5183 和 8840 是否监听
2. 只停止明确属于这个演示环境的旧 API/Vite 进程
3. 不要停止 5173、8787 或未知 PID
4. 如果端口被未知进程占用，先报告，不要强杀

第三步：检查演示数据库
1. 确认 PostgreSQL 可连接
2. 确认数据库名必须是 opitor_local_demo_qa_20260928
3. 确认数据库已经完成迁移
4. 不运行 migrate reset、db push、seed 清库或删除数据库
5. 如需确认演示数据，只允许运行幂等种子检查：
   OPITOR_LOCAL_DEMO_ENV_FILE=C:\Users\PC\.codex\local-runtime\opitor\local-demo.env
   node --import file:///C:/Users/PC/Desktop/MyOptio/node_modules/tsx/dist/loader.mjs server/src/scripts/local-demo-preview.ts --seed
   返回 preserved 属于正常结果

第四步：启动 API
必须把环境变量传给后台 API 进程：
- OPITOR_LOCAL_DEMO_ENV_FILE=C:\Users\PC\.codex\local-runtime\opitor\local-demo.env

API 启动命令：
node --import file:///C:/Users/PC/Desktop/MyOptio/node_modules/tsx/dist/loader.mjs server/src/scripts/local-demo-preview.ts

API 必须监听：
127.0.0.1:8840

API 必须满足：
- BMS configured=false
- LOCAL configured=true
- ADJUTANT_BACKGROUND_SERVICES=off
- 不连接外部邮件、AI、BMS 或生产服务

第五步：启动前端
前端必须把代理明确指向 API 8840：

ADJUTANT_API_TARGET=http://127.0.0.1:8840

在 web 目录启动：
node ../node_modules/vite/bin/vite.js --host 127.0.0.1 --port 5183 --strictPort --configLoader runner

如果用后台方式启动，必须：
- 使用明确的工作目录
- 使用明确的环境变量
- API 和 Vite 分别保存 PID
- 日志写入：
  C:\Users\PC\AppData\Local\Temp\opitor-demo-runtime\
- 启动后等待 5 秒再检查进程和端口
- 不能只看到 Vite 页面 200 就认为 API 正常

第六步：启动后验证
必须依次验证：

1. GET http://localhost:5183/
   预期：HTTP 200

2. GET http://localhost:5183/api/auth/providers
   预期：
   - BMS configured=false
   - LOCAL configured=true

3. GET http://127.0.0.1:8840/healthz
   预期：HTTP 200，返回 {"ok":true}

4. 使用 demo-admin / Opitor-Demo-2026! 登录
   预期：HTTP 200，返回 session token

5. 携带登录 token 请求：
   GET http://localhost:5183/api/im/conversations
   预期：HTTP 200，并返回演示会话数据

6. 检查 Socket.IO 代理：
   /socket.io/ 必须连接 127.0.0.1:8840，而不是 8787

如果 API 或 Socket 失败：
- 先读取 API/Vite 日志
- 检查 API 进程是否退出
- 检查 ADJUTANT_API_TARGET 是否正确
- 检查环境变量是否传给后台子进程
- 不要只重载浏览器
- 不要切换到 BMS 或 5173/8787 作为临时绕过

最后报告：
- Git HEAD
- 页面地址
- API 地址
- 数据库名
- API PID 和前端 PID
- providers、healthz、登录、IM 列表接口结果
- 日志中是否有错误
- 工作区是否有未提交修改
- 不输出数据库密码、私钥、token 或其他秘密