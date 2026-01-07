在腾讯云/阿里云/华为云租一个服务器，安装Linux操作系统，建议Ubuntu22.04。连接到服务器

# **第一步：服务器上创建一个新用户**

sudo useradd -m steam  //添加新用户，名字叫steam

sudo passwd steam   //设置密码，需要重复输入两次。我的密码是Dc@2026

sudo adduser steam sudo   //给予用户sudo权限（执行后续命令需要）

# **第二步：安装steam服务器工具**

sudo -u steam -s

cd /home/steam

sudo add-apt-repository multiverse

sudo dpkg --add-architecture i386

sudo apt update

sudo apt install steamcmd -y

export PATH="$PATH:/usr/games"

# **第三步：借助steam服务器工具安装幻兽帕鲁服务**

steamcmd   //进入steam交互命令窗口

login anonymous  //匿名登录steam（方便省事，也可以用自己的steam账号登录，如有需要自行查攻略）

app_update 2394010

app_update 1007

quit  //退出steam交互命令窗口

# **第四步：启动幻兽帕鲁服务器**

cd ~/Steam/steamapps/common/PalServer

./PalServer.sh   //这条命令用于启动服务器，启用后会一直占用终端（锁定光标）。如果按CTRL+C，或者退出终端，服务器会关闭

nohup ./PalServer.sh  > pal_server.log 2>&1 &  //后台启动服务器，这条命令可以避免占用终端的问题。

如果采用后台启动的话，停止服务需要先找到后台进程号（用netstat或者ps命令找进程号）

然后用kill -9 进程号停止进程

# **附1：修改端口**

nohup ./PalServer.sh -port=8805 > pal_server.log 2>&1 &  //用8805端口启动幻兽帕鲁，幻兽帕鲁服务器默认用8211端口，如果想用其他端口可以在启动时用-port自定义。

# **附2：修改游戏参数**

默认参数在这个文件当中，直接在这个文件中修改不会生效。把这个文件里的参数复制出来

~/Steam/steamapps/common/PalServer/DefaultPalWorldSettings.ini

在这个文件修改游戏参数，把刚才复制的参数粘贴过来（这个文件需要先启动一次服务器才会生成）

~/Steam/steamapps/common/PalServer/Pal/Saved/Config/LinuxServer/PalWorldSettings.ini

参数对应关系如图：

![img](https://github.com/Reedccc/palworld-private-server/blob/main/images/%E6%9C%8D%E5%8A%A1%E7%AB%AF%E5%8F%82%E6%95%B0.png)



# 附3 游戏内管理员操作

前提：修改帕鲁服务器参数文件~/Steam/steamapps/common/PalServer/Pal/Saved/Config/LinuxServer/PalWorldSettings.ini，开启管理员密码

AdminPassword="hahaha"

| 用管理员密码登录服务器            | /AdminPassword 管理员密码         |
| --------------------------------- | --------------------------------- |
| 显示当前玩家                      | /ShowPlayers                      |
| 踢出                              | /KickPlayer [UserID]/[SteamID64]  |
| 封禁                              | /BanPlayer [UserID]/[SteamID64]   |
| 传送（传送到目标玩家身边）        | /TeleportToPlayer {SteamID}       |
| 拉人（将玩家传送到我身边）        | /TeleportToMe {SteamID}           |
| 存档                              | /Save                             |
| 通知服务器在线玩家n秒后服务器关闭 | /Shutdown {Seconds} {MessageText} |



# 附4 管理面板

## 安装管理工具

修改帕鲁服务器参数文件~/Steam/steamapps/common/PalServer/Pal/Saved/Config/LinuxServer/PalWorldSettings.ini，开启Rcon和RESTAPI：

RCONEnabled=True

RESTAPIEnabled=True

设置好admin密码：

AdminPassword="hahaha"

到这里下载帕鲁服务器管理工具：https://github.com/zaigie/palworld-server-tool

把压缩包传到steam用户的文件夹下，注意版本

```
mkdir -p pst && tar -xzf pst_v0.9.12_linux_x86_64.tar.gz -C pst
cd pst
chmod +x pst sav_cli
```

找到其中的 `config.yaml` 文件并按照说明修改。

```yaml
# WebUI 设置
web:
  # WebUI 管理员密码
  password: ""
  # WebUI 访问端口  按需要修改
  port: 8080
  # 是否开启使用 HTTPS TLS 访问
  tls: false
  # TLS Cert 如果开启使用 HTTPS 请输入证书文件路径
  cert_path: ""
  # TLS Key 如果开启使用 HTTPS 请输入证书密钥文件路径
  key_path: ""
  # 若开启 HTTPS 访问请填写你的 HTTPS 证书绑定的域名 eg. https://yourdomain.com
  public_url: ""

# 任务相关设置
task:
  # 定时向游戏服务获取玩家在线情况的间隔，单位秒
  sync_interval: 60
  # 玩家进入/离开服务器通知
  player_logging: true
  # 玩家进入服务器消息
  player_login_message: "玩家 {username} 加入服务器!\n当前在线人数: {online_num}"
  # 玩家离开服务器消息
  player_logout_message: "玩家 {username} 离开服务器!\n当前在线人数: {online_num}"

# RCON 相关设置
rcon:
  # RCON 的地址和端口
  address: "127.0.0.1:25575"
  # 服务端设置的 AdminPassword
  password: "hahaha"
  # 服务器是否已开启 PalGuard 功能插件的 Base64 RCON 功能(需自行安装)
  use_base64: false
  # RCON 通信超时时间，推荐 <= 5
  timeout: 5

# REST API 相关配置
rest:
  # REST 的地址
  address: "http://127.0.0.1:8212"
  # Base Auth 的用户名，固定为 admin
  username: "admin"
  # 服务端设置的 AdminPassword
  password: "hahaha"
  # 通信超时时间，推荐 <= 5
  timeout: 5

# sav_cli Config 存档文件解析相关配置
save:
  # 存档文件路径，按实际位置修改
  path: "/path/to/your/Pal/Saved"
  # 存档解析工具路径，一般和 pst 在同一目录，可以为空
  decode_path: ""
  # 定时从存档获取数据的间隔，单位秒，推荐 >= 120
  sync_interval: 120
  # 存档定时备份间隔，单位秒，设置为0时禁用
  backup_interval: 14400
  # 存档定时备份保留天数，默认为7天
  backup_keep_days: 7

# Automation Config 自动化管理相关
manage:
  # 玩家不在白名单是否自动踢出
  kick_non_whitelist: false
```

运行管理面板

```
./pst
```



## 使用Rcon执行命令

使用rcon执行命令，无需登录游戏即可执行管理员命令，命令清单。命令同附3 游戏内管理员操作



## 附5 安装mod

Linux服务器不能安装windows版本的mod，例如https://github.com/Ultimeit/PalDefender。
帕鲁的mod很多都是仅支持windows。如果希望支持mod，可以参考Linux-Docker安装方式（docker中装了wine）





