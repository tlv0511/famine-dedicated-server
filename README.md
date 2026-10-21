# 饥荒联机版专用服务器搭建：先算配置再开服，地上洞穴、Mod 与更新一并讲清

饥荒联机版专用服务器搭建的关键有三样：一台能长期在线的机器、一份有效的服务器令牌、一套没写错的存档配置。Linux 云服务器适合 24 小时开服，Windows 本机适合偶尔和朋友玩；部署方式则分为 SteamCMD 手动安装和 Docker 封装两种。

> **选购速览**：如果打算地上和洞穴都开，2GB 内存起步的 Linux 服务器是相对稳妥的起点——双世界空载约 1.2GB，余量留给玩家和 Mod；只开地上世界，1GB 也能跑。CPU 看高主频，不用堆核心。 👉 [查看腾讯云服务器当前可选配置](https://bit.ly/yunhuodong)

## 先算配置：这套服务端到底吃多少资源

官方给出的参考是每名玩家 65MB 内存、8KB/s 带宽，但世界本身才是内存大头：实测空载时每个世界占 600–700MB，地上和洞穴同时开，大约 1.2GB。双世界的机器，2GB 是底线，4GB 更从容。

CPU 方面，饥荒服务端没有多线程优化，高主频比核心数更有意义；核心数不少于世界数即可，双世界至少给两个核。带宽压力不大，游戏走 UDP，延迟和丢包比带宽更影响体验。服务器默认使用 UDP 11000 端口，租机器时要确认这个端口能放行。

按开服规模粗算，可以分成三档：

| 开服场景 | 世界数 | 建议内存 | CPU | 入口 |
| --- | --- | --- | --- | --- |
| 只开地上，2–4 人朋友局 | 1 | 1–2GB | 高主频双核 | [在腾讯云选这一档](https://bit.ly/yunhuodong) |
| 地上+洞穴，常规玩法 | 2 | 2–4GB | 高主频双核 | [在腾讯云选这一档](https://bit.ly/yunhuodong) |
| 多 Mod 或十人以上 | 2 | 4GB 起 | 高主频 | [在腾讯云选这一档](https://bit.ly/yunhuodong) |

表里是按饥荒服务端实际占用推算的档位，不是某款机型的固定参数；具体机型、价格和计费周期以腾讯云页面实时显示为准。拿不准就先按双世界 2GB 准备，这是最容易被低估的地方，后期换机器还要迁存档。 👉 [按双世界 2GB 起步挑一台](https://bit.ly/yunhuodong)

## 三条搭建路线，先选适合自己的

Linux 云服务器加 SteamCMD 最通用，Docker 最省配置，Windows 本机最省钱但不适合长期在线。三条路都能开出完整的地上加洞穴服务器，差别在维护成本。

### Docker：不想碰依赖库就选它

社区项目 dst-dedicated-server 把运行环境打包好了，宿主机装 Git 和 Docker 就够。流程是克隆仓库、把令牌写进 `DSTClusterConfig/cluster_token.txt`、按项目文档启动。项目主要在 Debian 9 上测试，Linux 是最推荐的环境。令牌文件里不能有空格或换行，这是最容易出错的一步。

### SteamCMD：云服务器上的常规做法

Ubuntu 上可以直接用包管理器装 steamcmd，也可以用科雷官方的一键脚本完成安装、验证和启动。想自己控制安装目录和更新时机，就用 steamcmd 手动执行 `app_update 343050 validate`。下面按这条路线讲完整步骤。

### Windows 本机：偶尔开黑够用

Steam 库切到“工具”，下载 Don't Starve Together Dedicated Server；游戏里按 `~` 打开控制台，输入 `TheNet:GenerateClusterToken()` 生成令牌；把文档目录下的 Cluster_x 复制出来，改 `launch_preconfigured_servers.bat` 里的 `-cluster` 参数，分别启动 Master 和 Caves。缺点也明显：电脑要一直开着，Windows 自动更新还可能把服务器踢下线。

## Linux 手动搭建：从零到服务器出现在列表里

手动搭建可以拆成五步：建独立用户、装依赖和 SteamCMD、下载服务端、放令牌和存档、用 screen 启动两个世界。全程不要用 root 跑服务端，权限问题多半来自这里。

1. **建用户**。`useradd -m steam`，然后 `su - steam`。独立用户能把 SteamCMD 和系统其他部分隔开。
2. **装 32 位依赖**。64 位系统先 `dpkg --add-architecture i386`，再装 `libstdc++6:i386`、`libgcc1:i386`、`libcurl4-gnutls-dev:i386`。
3. **装 SteamCMD 并下载服务端**。`apt install steamcmd` 之后登录匿名账户，用 `force_install_dir` 指定目录，执行 `app_update 343050 validate`。这一步会下载几百 MB。
4. **准备令牌**。游戏内控制台执行 `TheNet:GenerateClusterToken()`，令牌文件在本地文档的 Klei 目录下；也可以在科雷官网新建服务器，下载配置压缩包，里面已经包含 `cluster_token.txt`。文件内容只能是令牌本身，不能有空格或其他字符。
5. **放存档**。把 MyDediServer 整个目录放到 `~/.klei/DoNotStarveTogether/`，确认 Master、Caves、`cluster.ini`、`cluster_token.txt` 都在。
6. **启动**。进入服务端 `bin` 目录，执行 `./dontstarve_dedicated_server_nullrenderer -console -cluster MyDediServer -shard Master`；洞穴用同样的命令，把 `-shard` 换成 `Caves`。两个进程分别放在 screen 会话里，按 Ctrl+A 再按 D 切到后台。
7. **放行端口**。在腾讯云控制台的安全组里放行 UDP 11000；改过端口就以实际配置为准。

> 启动报 `libcurl-gnutls.so.4: cannot open shared object file`，是依赖路径问题：用 `dpkg -L libcurl4-gnutls-dev:i386` 找到 libcurl-gnutls.so 的实际路径，再软链到 `/usr/lib/libcurl-gnutls.so.4`。服务端必须在 `bin` 目录下启动，否则会出现 databundles 加载失败后退出。

## 配置文件：哪些必须改，哪些别碰

必须改的只有 `cluster.ini` 里的服务器名和密码，其他都能沿用默认值；世界设定不要手写，用本地游戏创建世界后导出的文件最稳。

`cluster.ini` 里值得关注的几项：`cluster_name` 和 `cluster_password` 决定服务器在列表里的名字和进服门槛；`max_players` 控制人数上限；`pause_when_empty` 决定没人时是否暂停；`cluster_intention` 是合作、竞争、生存等模式。`autosaver_enabled = true` 时，游戏会在清晨自动保存一次。

目录结构大致是这样：


MyDediServer/
  Caves/
    leveldataoverride.lua
  Master/
    leveldataoverride.lua
  mods/
    dedicated_server_mods_setup.lua
    modoverrides.lua
  adminlist.txt
  blocklist.txt
  cluster.ini
  whitelist.txt
  cluster_token.txt


世界设定用本地游戏生成最省事：创建世界时把地形、资源、生物调好，退出到选人界面，把 Cluster_x 下 Master 和 Caves 的 `leveldataoverride.lua` 分别复制到服务器对应目录。洞穴文件里要保留 `id="DST_CAVE"` 和 `location="Cave"`，否则地上和洞穴的通讯会出问题。管理员名单 `adminlist.txt` 每行一个 Klei ID，黑名单和白名单同理。

## Mod：服务端只认服务器 Mod

服务器只加载服务器 Mod，客户端 Mod 不用管。配置分两个文件：`modoverrides.lua` 决定启用哪些 Mod 和参数，`dedicated_server_mods_setup.lua` 告诉服务器去哪里下载。

本地游戏勾选服务器 Mod 并创建世界后，`Master/modoverrides.lua` 里会出现 `workshop-数字` 的条目。把这些数字写成 `ServerModSetup("数字")`，一行一个，放进 `dedicated_server_mods_setup.lua`。这个文件要放到两处：服务端安装目录的 `mods` 下，以及存档目录下。

还有一个容易忽略的点：更新服务端会覆盖 `dedicated_server_mods_setup.lua`。更新脚本里先备份、更新完再还原，不然每次更新都要重新填一遍 Mod ID。

## 更新与日常维护

更新服务端就是重新执行一次 `app_update 343050 validate`，更新前备份 Mod 下载清单，更新后还原。日常维护主要是看内存、留快照，别在玩家在线时硬重启。

一个可用的更新脚本长这样：

sh
#!/bin/sh
server_path=<服务端目录>
backup_time=$(date +%s)
mkdir /tmp/$backup_time
cp $server_path/mods/dedicated_server_mods_setup.lua /tmp/$backup_time/

~/steamcmd/steamcmd.sh +login anonymous \
  +force_install_dir $server_path \
  +app_update 343050 validate +quit

cp /tmp/$backup_time/dedicated_server_mods_setup.lua $server_path/mods/


`cluster.ini` 里的 `max_snapshots` 控制可回滚的快照数量，数值越大能回退的天数越多，占用的磁盘也会增加。关服直接在服务端窗口按 Ctrl+C，确认保存后再关。存档目录建议定期打包备份，换机器时直接搬过去就能继续用。

## 常见问题

**服务器列表搜不到怎么办？** 先查三件事：令牌是否有效、UDP 11000 是否放行、机器能不能访问科雷服务器。用 `-lan` 参数启动的服务器不会出现在列表里，只能通过 IP 直连，这个模式也不适合公开开服。

**不开洞穴行不行？** 行，删掉 Caves 目录只跑 Master 即可，内存能省一半左右——双世界空载约 1.2GB，单世界 600–700MB。

**2GB 内存能带几个人？** 按官方 65MB/玩家估算，双世界空载 1.2GB 后还剩约 800MB，理论上十来人；实际带 Mod、后期实体变多都会推高占用，人数多就上 4GB。

**Windows 和 Linux 选哪个？** 长期在线选 Linux，Windows 的自动更新和休眠策略都可能让服务器掉线；只是周末开黑，Windows 本机最省事。

**带宽要多大？** 官方参考是 8KB/s/玩家，要求不高，但 UDP 对延迟和丢包敏感。选离玩家近的地域，比选大带宽更有用。

## 最后怎么选

只在周末和朋友玩，用 Windows 本机加 Steam 专用服务器工具，不用租机器。想 24 小时在线、开洞穴，选 Linux 云服务器，2GB 内存起步，SteamCMD 部署；Mod 多或十人以上升到 4GB。不想折腾依赖库，就用 Docker 方案。

在腾讯云下单前确认两件事：内存是不是 2GB 起步，安全组能不能放行 UDP 11000。机器定下来再照着上面的步骤走，顺序就不会乱。 👉 [查看腾讯云服务器当前配置](https://bit.ly/yunhuodong)
