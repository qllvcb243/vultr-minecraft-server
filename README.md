# vultr minecraft：10 分钟内开好服务器，套餐怎么选、端口怎么放行、朋友怎么进

在 Vultr 上开 Minecraft 服务器，最省事的路子是官方 Marketplace 里的 Minecraft (Vanilla) 一键镜像：部署时勾上它，Java 和原版服务端自动装好，最长 10 分钟就能开服。真正需要你做决定的只有三件事：选哪档配置、25565 端口怎么放行、朋友用什么地址进服。这篇文章按这个顺序讲，价格和几个容易踩的坑也一并说清楚。

> **选购速览**：四五个人玩原版，High Performance 的 2 vCPU / 4 GB（$24/月）就够；人数更多或打算装插件，上 4 vCPU / 8 GB（$48/月）。Vultr 按小时计费，新账号注册后 30 天内有 $300 试用金，头一个月基本不花钱。👉 [去 Vultr 部署 Minecraft 服务器，领新用户 $300 试用金](https://bit.ly/VulTR)

## Vultr 开 Minecraft 服务器要花多少钱？

入门档是 $24/月（High Performance，2 vCPU、4 GB 内存、100 GB NVMe），大多数小服务器在 $48/月这档就到头了。下表价格整理自 2026 年 9 月底的第三方 sizing 指南，均为税前月价，实际按小时计费、以月价为上限。

| 方案 | CPU / 内存 | NVMe 硬盘 | 月价（美元） | 适合规模 | 购买链接 |
| --- | --- | --- | --- | --- | --- |
| 2 vCPU / 4 GB | 2 核 / 4 GB | 100 GB | $24 | 原版或 Paper，1-4 人 | [ 以此配置开服](https://www.vultr.com/marketplace/apps/minecraft-vanilla/?ref=7268283) |
| 4 vCPU / 8 GB | 4 核 / 8 GB | 180 GB | $48 | 原版 5-20 人；带插件或少量模组 1-10 人 | [ 以此配置开服](https://www.vultr.com/marketplace/apps/minecraft-vanilla/?ref=7268283) |
| 4 vCPU / 12 GB | 4 核 / 12 GB | 260 GB | $72 | 原版 21-40 人；插件服 11-20 人；整合包 1-4 人 | [ 以此配置开服](https://www.vultr.com/marketplace/apps/minecraft-vanilla/?ref=7268283) |
| 8 vCPU / 16 GB | 8 核 / 16 GB | 350 GB | $96 | 插件或少量模组，21-40 人 | [ 以此配置开服](https://www.vultr.com/marketplace/apps/minecraft-vanilla/?ref=7268283) |
| 12 vCPU / 24 GB | 12 核 / 24 GB | 500 GB | $144 | 大型整合包，5-40 人 | [ 以此配置开服](https://www.vultr.com/marketplace/apps/minecraft-vanilla/?ref=7268283) |

适合规模一栏取自 Playkeeper 的 sizing 指南，标准偏保守；Vultr 官方文档给的人数估算更宽松，两套标准怎么取舍，下一节单独说。

两个直接影响账单的点：

- 按小时计费意味着临时服很划算。开三天的活动服只算三天的钱，不会因为你开了一台机器就按整月扣。
- $300 试用金有条件：只给新账号，30 天内有效，注册时要绑卡或 PayPal，没用完的额度到期作废。

备份另算钱。Vultr 自带的整机自动备份是可选付费项，不开的话世界数据就只存在这台机器上。想异地留一份副本，可以把备份推到 S3 兼容的对象存储，Vultr 自家的 Object Storage 就支持这种接法。

## 配置怎么选：先看人数，再看玩法

原版生存按人数选内存就行，要装插件或整合包就在原版基础上加一档。原因是 Minecraft 的世界逻辑主要跑在一个核心上，单核性能和内存容量比核心总数更重要。这也是 Vultr 几个系列里更推荐 High Performance 的理由：它用最新一代的 Intel Xeon 或 AMD EPYC 核心，配 NVMe 硬盘；Regular Performance 是老核心加普通 SSD；High Frequency 单核不弱，但同样价钱能买到的核心数不如 High Performance。

Vultr 官方文档在 Minecraft 部署页给过一版内存对人数的参考：

- 1 GB 内存 / 1 核：1-5 人，适合测试和小团体
- 2 GB / 2 核：5-10 人，官方称为建议的最低配置
- 4 GB / 2 核：10-20 人
- 8 GB / 4 核：20 人以上

这版数字偏理想。同样 4 GB，Playkeeper 只建议 1-4 个朋友。差距来自假设：官方按服务端最低需求算，第三方把系统开销、插件和视距都算了进去。官方自己的手动教程也提醒过，别把全部内存分给 Java，系统本身要占掉一块。所以更稳妥的选法是：三五个人短期开原版服，照官方参考选没问题；打算长期运营、要装插件，按第三方的余量选，卡在下限上迟早要升配。

一句话版：原版 1-4 人选 4 GB（$24），5-20 人或带插件选 8 GB（$48），大型整合包从 12 GB（$72）起步，模组吃内存比什么都凶。👉 [查看 Vultr 各档方案实时价格](https://bit.ly/VulTR)

## 开服步骤：一键镜像为主，手动安装兜底

绝大多数人直接用一键镜像，部署全程就是点几次鼠标；手动安装留给镜像覆盖不了的特殊需求。

### 用 Minecraft (Vanilla) 一键镜像（推荐）

在控制台走正常部署流程：选离玩家最近的机房（Vultr 六大洲都有节点，ping 差别很实在），选好套餐，然后在 Marketplace Apps 里搜索 Minecraft (Vanilla) 勾上。Additional Features 建议选 Limited Login，会创建一个带 sudo 权限的受限用户，比全程用 root 干活稳妥。点 Deploy Now，等状态变成 Running，安装 Minecraft 和 Java 最长要等 10 分钟。

部署完成后，服务器的 Server Overview 页面能看到 IP、SSH 连接信息和 root 密码。上台先确认三件事：

console
$ sudo systemctl status minecraft.service   # 应显示 active (running)
$ java -version                              # OpenJDK 21
$ ls -la /home/minecraft/                    # 能看到 server.jar、server.properties、world/


这个镜像真正省心的是管理命令。SSH 登录后敲 `minecraft` 一个词，就能启动、停止、查状态、进实时控制台、翻日志，不用自己记一串 systemctl 加 screen 的组合。注意镜像默认以 root 账号登录，这正是前面建议勾 Limited Login 的原因。

更新也简单：先 `minecraft stop`，再运行镜像自带的 update.sh（官方文档给的位置是 /home/minecraft/server/update.sh），脚本会顺带把 Java 升到系统支持的最新版本。升级前先停服，别在世界正在跑的时候动它。

👉 [打开 Minecraft (Vanilla) 一键镜像页面](https://www.vultr.com/marketplace/apps/minecraft-vanilla/?ref=7268283)

### 手动安装什么时候值得

只有一种情况：一键镜像给的原版服务端满足不了你，比如要跑 Paper、插件或整合包环境。流程不复杂，建一个 minecraft 用户，装 OpenJDK 和 screen，下载对应的服务端 jar，首次启动改 eula.txt 同意协议，再写个 start.sh 挂在 screen 里后台跑。最容易翻车的是内存参数：官方教程给的参考是 4 GB 内存的机器用 -Xms3072M -Xmx3072M，给系统留出余量。

## 玩家进不来？大概率是 25565 没放行

Vultr 的 Ubuntu 镜像默认开启 ufw 防火墙，而且只放行了 SSH，这是“服务部署成功了、客户端却连不上”的头号原因。放行 Minecraft 端口用三条命令：

console
$ sudo ufw allow OpenSSH
$ sudo ufw allow 25565/tcp
$ sudo ufw enable


ufw 多数时候已经在跑，你真正要补的往往只有 25565 那一条，但三条都执行一遍也无妨，然后用 `sudo ufw status` 确认规则生效。如果在 Vultr 后台另外配了 Firewall 组，组里也要放行 TCP 25565：后台防火墙和系统防火墙是两道闸，缺一道照样连不上。

想让朋友记个好记的地址，给你的域名加一条 A 记录指向服务器 IP，比如 mc.example.com。进服时在客户端选多人游戏 → 添加服务器，填 IP:25565 或域名就行。

## 服主日常：白名单、OP 和几个关键配置

不想让陌生人进来就开白名单：server.properties 里把 white-list 和 enforce-whitelist 都设为 true，重启后在控制台用 `whitelist add 玩家ID` 加人，名单存在 /home/minecraft/whitelist.json。给朋友管理权限用 `op 玩家ID`。

改 server.properties 之前先 `systemctl stop minecraft.service`，改完再 start，服务运行中直接改配置不生效。几项对体验影响最大的：

- max-players：最大同时在线人数
- view-distance：视距，范围 2-32 区块，服务器卡顿先调它，效果最立竿见影
- difficulty / gamemode：难度和游戏模式
- online-mode：保持 true 强制正版验证，这是最基础的一道防线

## 常见问题

**只在周末开服能省钱吗？** 能，计费精确到小时，上限就是月价。销毁实例前先把世界备份下来，机器没了数据就没了。

**需要正版账号吗？** 默认 online-mode=true，玩家要有正版 Minecraft 账号才能进。这个开关能改，但关掉等于向所有陌生人敞开大门，真要关就配合白名单一起用。

**内存怎么分给 Java？** 一键镜像不用你管；手动装的话，4 GB 机器给 3072 MB 左右，剩下的留给系统。

**服务端怎么更新？** 先 `minecraft stop`，再跑 update.sh，Java 会一起升到系统支持的最新版本。

**能跑 Paper 和模组吗？** Marketplace 镜像只提供原版服务端，要插件和模组就走手动安装路线装对应的服务端，配置按前面“带插件”那一栏留足内存。

## 按规模下单，别多花钱

三五个人原版生存，2 vCPU / 4 GB（$24/月）绰绰有余；十来个人、要装插件，4 vCPU / 8 GB（$48/月）；动大型整合包，直接从 12 GB（$72/月）起步。下单前确认两件事：机房选离队友最近的那个；部署时在 Marketplace Apps 里勾上 Minecraft (Vanilla)，能省掉一整套手装流程。$300 试用金只有 30 天有效期，注册了就尽早开服。👉 [按你的规模挑好配置，去 Vultr 开服](https://bit.ly/VulTR)
