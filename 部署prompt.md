请将 GitHub 仓库 lhlhome/MyOptio 的最新 origin/main 部署到 Opitor 生产，尽量减少对用户的影响。

本次授权：读取生产状态、构建镜像、创建备份、执行目标版本所需迁移、切换 server 和验收。请连续执行常规步骤，无需逐步请求批准。
遇到数据破坏风险、必须清理生产资源、配置冲突或无法安全回滚时，说明具体阻碍。
不要修改 AWS、DNS、SES，不发邮件、不调用付费模型，不发布原生安装包。

一、仓库与协作规则

- 仓库：https://github.com/lhlhome/MyOptio
- 本机主工作区：C:\Users\PC\Desktop\MyOptio
- 先读当前 AGENTS.md、docs/COLLABORATION.md 的“开始与交付”，
  再读适用的部署交接资料。
- 按仓库要求由主会话协调，指定唯一集成/部署 Agent。
- 检查 HEAD、分支、未提交改动和所有 worktree。
- 不覆盖、暂存或提交其他人的改动，不 reset/clean/强制切换。
- 生产源码必须来自精确 GitHub 提交快照，不能直接打包本地工作区。
- 不新增开发 worktree或常驻预览服务，除非确有必要且符合仓库规则。

二、生产连接与最近已知基线

SSH：
  ubuntu@47.130.123.121

连接时使用已有授权密钥，并启用：
  StrictHostKeyChecking=yes
  BatchMode=yes
  ConnectTimeout=15

本机 opitor-prod 别名曾错误指向 7.130.123.121。
可使用：
  ssh -o HostName=47.130.123.121 -o StrictHostKeyChecking=yes \
      -o BatchMode=yes -o ConnectTimeout=15 opitor-prod

不要绕过主机密钥校验或打印私钥。

目录：
- Compose project-directory：/opt/adjutant/deploy
- 生产环境文件：/opt/adjutant/deploy/.env
- 发布目录：/home/ubuntu/adjutant-releases/

最近成功发布记录（2026-10-28，北京时间17:35）：
- 源码 SHA：8d2858afdd568117e958e61376e55e64ba1b1d24
- server 容器短 ID：15bba6fdd0bb
- 镜像 ID 前缀：sha256:a9634da19436…
- 发布目录：/home/ubuntu/adjutant-releases/main-8d2858af/
- 迁移：114条成功、0条未完成
- 数据代：g2
- 磁盘剩余：约4.4GB
- 最近切换窗口：约18秒

以上只是历史参考，不能直接当作当前状态。
先从运行容器、镜像标签和迁移账本重新核实。

三、版本锁定和发布前检查

1. 用 git ls-remote origin HEAD refs/heads/main 确认 GitHub 最新 SHA；
   必要时 fetch 获取对象，不覆盖本地改动。
2. 固定本次发布 SHA，记录提交标题与时间。
3. 核实线上真实 SHA；若已经相同，只验收，不重复构建。
4. 对比线上版本与目标版本：
   - package.json、各 workspace manifest、package-lock.json；
   - Prisma schema 和新增迁移；
   - deploy 文件、环境变量及启动方式；
   - 是否包含必须另发原生 App 的改动。
5. 检查当前容器健康、磁盘、内存/swap、迁移状态及近期错误摘要。
6. 读取当前 server 容器标签中的完整 Compose 覆盖链。
   生产存在多层覆盖，含 OAuth 配置和历次 image-only 覆盖。
   必须完整保留，不能只使用仓库默认 docker-compose.yml。
7. 记录当前镜像、完整 Compose 参数、各容器 ID、卷映射、
   生产 .env 哈希，供切换和回滚核对。
   不输出完整环境变量、密码、token、cookie、连接串或业务内容。

当前业务卷历史名称：
- adjutant_pgdata_g2
- adjutant_im-attachments_g2
- adjutant_drive-files_g2

仍需核实实际挂载。不得再次执行 D22 整库重来。

四、在线构建，保持旧版服务

- 为目标 SHA 创建新的独立发布目录。
- 使用 git archive 精确导出目标提交，并按生产构建允许列表打包。
- 排除 .env、密钥、备份、QA数据等。
- 核对上传前后源码包 SHA256。
- 构建平台为 Linux ARM64，Node 22。
- 最近成功流程是复用当前生产镜像的系统层进行增量构建：
  - 依赖清单和 lockfile 未变化，可以复用已验证的依赖层；
  - 有变化必须在新镜像内正确 npm ci，不能沿用旧 node_modules。
- 按需要执行 Prisma generate，并完成 Web、server 生产构建。
- 核实安装脚本和构建所需文件已复制到镜像。
- 不在生产主机直接安装依赖，不运行生产库上的测试。
- 不为本次常规发布重复跑全量测试；已有测试结果要绑定准确版本。
  包装脚本 exit 0 不等于子测试全部通过，不能隐瞒已有失败。

资源限制：
- 主机内存较小，构建会使用 swap，最近构建约16–24分钟。
- 构建期间旧服务继续运行；定时检查公开 health 和构建活性。
- 长时间无输出时核进程 CPU 增量、内存、磁盘和错误，
  不盲目重跑。
- 先评估空间峰值。目前空间不宽裕。
- 不擅自删除旧镜像、卷、备份或执行 prune。
  如果空间不足，给出具体占用与可选处理方案。

五、准备切换与回滚

参考最近成功发布目录中的：
- activate.py
- 验收脚本
- rollback-command.json
- deployment-evidence.json
- acceptance.json

复用机制时必须新建本次目录，更新：
- 目标 SHA 和新镜像；
- 当前实际生产基线及完整 Compose 链；
- 本次关键源码文件指纹；
- 容器 ID、环境文件哈希和迁移预期；
- 备份路径、验收和回滚命令。

不能直接重跑旧 activate.py。

切换前确认：
- 镜像已完整构建成功；
- 新旧渲染后的 Compose 配置符合本次范围；
- 现有环境和 OAuth 配置全部保留；
- 只新增本次 server 镜像覆盖；
- 旧镜像存在；
- 新增迁移是否兼容旧代码，回滚边界已明确；
- 备份、切换、验收脚本全部准备好。

用户身份背景：
- 李进：当前用户本人
- Frank：老板
- 马斌：同事

不要默认将 Frank 当作当前用户。
若用户要求择空闲窗口，可只读查询有效未撤销会话的 lastSeenAt。
该字段也可能被后台请求刷新，不能等同人在屏幕前操作。
用户明确说直接切换时，不再自行等待活动窗口。
不要踢用户、修改会话或发送通知消息，除非用户另行授权。

六、短停备份和切换

所有准备完成后才开始停服窗口：

1. 停止 server，冻结业务写入。
2. 创建同窗口数据库逻辑备份及 IM、Drive 文件备份。
3. 校验备份可读性、数据库 TOC，记录 SHA256。
4. 沿当前完整 Compose 链，仅追加新 image-only 覆盖，
   使用 --no-deps、--no-build、--pull never 等适合实际链的参数，
   只重建 server。
5. 按现有入口运行 migrate deploy → isolation → 启动应用，
   不绕过失败。
6. 立即等待新服务健康并验收。

禁止：
- 停止或重建 db、caddy、官网容器；
- --remove-orphans；
- 用本地不完整 .env 覆盖生产；
- 清库、seed、db push、伪造 migrate resolve；
- 改变现有数据代、密钥或数据卷；
- 为回滚直接用旧备份覆盖已有新写入。

记录准确时间：
- 发出停止请求；
- 旧容器停止；
- 新容器启动；
- 首次观察到 healthy。

历史切换约16–18秒，只作参考，不承诺固定时长。
不能用“备份结束到健康”的时间冒充完整中断时间。

失败时按已确认的迁移兼容性执行回滚。
涉及不可逆迁移或新数据时，先判断恢复边界，不能盲目只换旧镜像。

七、一次上线验收

至少确认：
- server running/healthy；
- 迁移成功数与目标发布目录一致，未完成为0；
- app.opitor.com 首页和 /healthz 返回200；
- 匿名 /api/auth/session 返回401；
- /api/auth/providers：BMS=false、LOCAL=true；
- cowork.lhlhome.com 同类公开检查正常；
- opitor.com、www.opitor.com 跟随正常跳转后返回200；
- db、caddy、两官网容器 ID 与切换前一致；
- 生产 .env 哈希、数据代和卷映射保持预期；
- 本次关键源码和依赖清单指纹与目标快照一致；
- 新 server 启动日志无新增错误。

只输出必要状态和错误摘要，不输出业务正文或秘密。
健康检查和匿名认证检查不能冒称真实账号登录、发信或模型功能已验证。

八、备份和交接

- 保留本次服务器备份、SHA清单、TOC、旧镜像及配对回滚命令。
- 可将三份备份下载到受保护的本机临时目录并核双端 SHA。
- 大文件 SSH 传输曾出现停滞；只做有界尝试，
  失败如实记录，不因传输收尾反复干扰生产。
- 不自动清理旧发布目录、镜像或数据卷。

最终简短报告：
1. 实际上线 SHA、镜像和完成时间（北京时间）；
2. 构建与验收实际执行次数、耗时和结果；
3. 完整切换窗口；
4. 数据、配置和其他容器是否保持；
5. 备份、证据与回滚路径；
6. 未完成事项、本地改动及工作树/服务交接。

原生 App 改动需要另发安装包，本次只发布 server 和 Web。
执行期间约每60秒简短报告有意义的进度，不用重复大段日志。