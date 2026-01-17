完整项目名：1995chen/dnf

docker compose

```
services:
  dnf:
    # 基础镜像（个人维护的 CentOS 7 版 DNF 服务端）
    image: 1995chen/dnf:centos7-latest
    # 容器名称（固定名称，方便后续操作容器）
    container_name: dnf
    # 容器内部主机名（用于容器内服务间通信标识）
    hostname: dnf
    # 授予容器网络管理权限（DNF 运行必需）
    cap_add:
      - NET_ADMIN
    # 自定义环境变量（需根据实际环境修改）
    environment:
      - PUBLIC_IP=192.168.32.10       # 替换为部署主机的内网/公网 IP
      - WEB_USER=root                   # 后台管理系统登录用户名（可自定义）
      - WEB_PASS=Wu188917                 # 后台管理系统登录密码（建议修改为强密码）
      - DNF_DB_ROOT_PASSWORD=Wu188917   # MySQL root 密码（需牢记，与后续配置对应）
      - GM_ACCOUNT=gmuser               # 游戏管理员（GM）账号（可自定义）
      - GM_PASSWORD=gmpass              # GM 账号密码（建议修改为强密码）
      - CLIENT_POOL_SIZE=10             # 客户端连接池大小（默认 10 满足小型部署）
      - GM_CONNECT_KEY=NX9O7UVPPWCQRSMWXF2K
    # 端口映射（主机端口:容器端口，TCP 默认，UDP 显式标注）
    ports:
      - "180:180"                       # supervisor web（进程管理后台）
      - "3306:3306"                     # mysql（外部工具连接数据库端口）
      - "7600:7600"                     # 统一登陆器（客户端登录验证核心端口）
      - "881:881"                       # 统一网关（客户端与服务端通信网关）
      - "7001:7001"                     # df_channel_r（频道服务 TCP 端口）
      - "7001:7001/udp"                 # df_channel_r（频道服务 UDP 端口）
      - "30011:30011"                   # df_game_r[ch.11]（11 频道游戏服务 TCP 端口）
      - "31011:31011/udp"               # df_game_r[ch.11]（11 频道游戏服务 UDP 端口）
      - "30052:30052"                   # df_game_r[ch.52]（52 频道游戏服务 TCP 端口）
      - "31052:31052/udp"               # df_game_r[ch.52]（52 频道游戏服务 UDP 端口）
      - "7300:7300"                     # df_relay_r（中继服务 TCP 端口）
      - "7300:7300/udp"                 # df_relay_r（中继服务 UDP 端口）
      - "2311-2313:2311-2313/udp"       # df_stun_r（STUN 服务端口，用于网络穿透）
    # 数据卷挂载（实现数据持久化，避免容器销毁数据丢失）
    volumes:
      - /vol2/1000/DNF/log:/home/neople/game/log   # 游戏运行日志持久化
      - /vol2/1000/DNF/mysql:/var/lib/mysql         # MySQL 数据库数据持久化
      - /vol2/1000/DNF/data:/data                   # DNF 核心配置/角色数据持久化
    # 共享内存配置（多进程通信必需，保障服务流畅运行）
    shm_size: 8g
    # 自动重启策略（保障服务高可用性）
    restart: always
```
首先是「DO补丁大合集V7.6」文件夹，其中有三个文件。打开后将其中的DNF.toml文件进行编辑，将IP地址更换为上文PUBLIC_IP一致。建议不要用记事本直接打开编辑，推荐使用VScode。

<img width="1080" height="662" alt="Image" src="https://github.com/user-attachments/assets/693f4fc6-8d96-4b5b-908c-5947d4335583" />

接着将三个文件，直接复制到「地下城与勇士」文件夹中，有重复文件就选择直接替换。

<img width="1080" height="646" alt="Image" src="https://github.com/user-attachments/assets/0dddb256-fee7-4ba6-9950-7d12c3adda84" />

接着打开「统一网管在线管理工具」文件夹，双击运行统一网关在线管理工具v6.4.exe。这也算是半个GM工具，发装备（增幅/强化），发材料啥的都能操作，大家部署完了可以自己用用看。
首先如下图，点击「网关设置」。网关地址与PUBLIC_IP一致，网关端口881，登陆账号密码与部署时的一致，通信密钥763WXRBW3PFTC3IXPFWH，登录器端口7600。填写完毕后，点击网关端口右侧的连接，不出问题的话就会连上，接着再点击底部的「参数设置内容立刻生效」。如果连不上，请检查参数是否有误。

<img width="1080" height="801" alt="Image" src="https://github.com/user-attachments/assets/1fdd27de-eb1d-495c-a467-f8d2756d725d" />

如下图。然后顶部栏再点击「登陆器设置」。服务器名称自定义，登陆器版本 20180307。接着继续填写：线路名自定义，登录器端口7600，游戏地址、网关地址与PUBLIC_IP一致。这四个填写完后点击右侧的「添加」。添加完毕后，填入通信密钥763WXRBW3PFTC3IXPFWH，接着再点击「生成登陆器」。

<img width="1080" height="801" alt="Image" src="https://github.com/user-attachments/assets/7314b4d6-81c7-4260-ae0f-246b32bca86f" />

生成完毕后，会弹出登录器已生成界面。此时我们再关掉统一网关在线管理工具v6.4.exe这个进程，可以看到又多出来一个Config.ini配置文件。将「统一网管在线管理工具」文件夹中新生成的这两个文件也放入「地下城与勇士」文件夹中。也是有重复就直接替换。至此，所有工作完成，可以开始玩了。

<img width="1080" height="664" alt="Image" src="https://github.com/user-attachments/assets/ec88b343-d90a-49f0-a3ef-df8a879219d0" />