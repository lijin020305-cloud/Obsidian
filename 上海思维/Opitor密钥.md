LIJIN-Opitor 服务器交接说明

AWS 账户：662914073148
AWS 控制台用户名：LIJIN-Opitor
控制台入口：https://662914073148.signin.aws.amazon.com/console
临时密码见同目录登录信息文件；首次登录必须改密。建议随后在自己的安全凭证页面绑定 MFA。
EC2 区域：新加坡 ap-southeast-1
实例：i-0ef4877a7104b26bd（adjutant-prod）
服务器公网 IP：47.130.123.121
SSH 用户：ubuntu
SSH 私钥：本目录 adjutant-prod.pem
私钥公钥指纹：SHA256:Q3Doi1PpWogfAO39NvuMgzSdX7mB/Ywm35Kx4GeWFdQ

连接步骤：
1. 在 AWS 控制台选择新加坡区域，进入 EC2 → 安全组 → sg-0dd055b352e35f3f6。
2. 入站规则中添加 SSH / TCP 22，来源选择“我的 IP”（当前公网 IPv4 /32），描述注明 LIJIN 和用途。保留已有规则。
3. macOS/Linux 终端在私钥文件所在目录执行：
   chmod 600 adjutant-prod.pem
   ssh -i ./adjutant-prod.pem ubuntu@47.130.123.121
4. 运维结束后删除自己新增的临时 SSH 规则。公网 IP 变化后需要相应调整自己的规则。

说明：
- IAM 账户用于 AWS 控制台；SSH 使用 ubuntu 和本私钥，两者是不同登录方式。
- 当前实例未接入 Session Manager。EIC 实例端未验证，请以上述 SSH 方式连接。
- 已核对本私钥公钥指纹与 AWS key pair 一致；本机当前 SSH 连接超时，尚未验证实际登录及 sudo。由操作者放行自己的 IP 后登录验证。
- 数据库逻辑备份、附件打包在服务器内执行；EBS 快照是另一层保护，不等于完整数据库恢复演练。
- 历史部署目录 /opt/adjutant，历史备份目录 /home/ubuntu/adjutant-backups；实际操作前在服务器确认当前容器、数据路径、空间及配置，勿直接沿用历史路径执行破坏性操作。
- 本次仅交付账户及访问材料，没有升级应用、迁移数据库、设置备份计划或更改现有安全组规则。
- 私钥与临时密码仅通过安全渠道交接，不上传仓库。

权限使用提示：
- 创建 EBS 快照时必须添加标签 Project=Opitor；当前授权对应卷 vol-0c24a6b5e77ebfb6e。
- 绑定 MFA 时将设备命名为 LIJIN-Opitor，或使用 LIJIN-Opitor- 开头的名称。
- 服务器写操作权限限定到 Opitor；EC2 控制台可读取新加坡区域的资源列表。


AWS控制台：https://662914073148.signin.aws.amazon.com/console/
账号ID：662914073148
IAM用户名：LIJIN-Opitor
临时密码：Aa1!A0wqbNON+hZPnuhJ+X76C-#x_YYz
首次登录强制修改密码。
区域：ap-southeast-1（新加坡）
MFA设备名称必须为 LIJIN-Opitor 或以 LIJIN-Opitor- 开头。
创建EBS快照必须添加标签 Project=Opitor。
没有创建长期访问密钥。
