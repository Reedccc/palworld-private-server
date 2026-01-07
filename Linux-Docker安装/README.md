# 感谢灰灰大佬提供的镜像

需要安装两个Docker镜像，第一个是装了帕鲁服务 + PalDefender插件的镜像，第二个是服务端管理工具的镜像。
PalDefender插件提供的命令清单：https://ultimeit.github.io/PalDefender/zh/Commands/

## 拉取镜像

docker pull swr.cn-north-4.myhuaweicloud.com/huihui-cloud/palworld-docker-wine:latest

docker pull swr.cn-north-4.myhuaweicloud.com/huihui-cloud/palworld-docker-tool:latest

## 重命名
docker tag swr.cn-north-4.myhuaweicloud.com/huihui-cloud/palworld-docker-wine:latest pserver-docker-wine:latest

docker tag swr.cn-north-4.myhuaweicloud.com/huihui-cloud/palworld-docker-tool:latest pserver-docker-tool:latest

## 保存镜像文件，以备后用

docker save -o palworld-server-docker.tar  pserver-docker-wine:latest

docker save -o palworld-manage-tool-docker.tar  pserver-docker-tool:latest

## docker-compose.yml文件内容如下

~~~ yaml
services:
   palworld-server-wine:
      build: .
      image: pserver-docker-wine:latest
      restart: unless-stopped
      container_name: palworld-server-wine
      stop_grace_period: 30s # Set to however long you are willing to wait for the container to gracefully stop
      ports:
        - 8805:8211/udp  #按需进行端口映射
        - 25575:25575
        - 8212:8212/tcp
      environment:
         - PORT=8211 # Optional but recommended
         - PLAYERS=16 # Optional but recommended
         - SERVER_PASSWORD= # Optional but recommended
         - MULTITHREADING=true
         - RCON_ENABLED=true
         - RCON_PORT=25575
         - TZ=UTC
         - ADMIN_PASSWORD=hahaha
         - COMMUNITY=false  # Enable this if you want your server to show up in the community servers tab, USE WITH SERVER_PASSWORD!
         - SERVER_NAME=
         - SERVER_DESCRIPTION=
         - ENABLE_MULTITHREAD=true  # enable multithreading
      volumes:
         - ./palworld:/home/wineuser/.wine/drive_c/Steam/steamapps/common/PalServer/Pal/Saved
         - ./PalDefender:/home/wineuser/.wine/drive_c/Steam/steamapps/common/PalServer/Pal/Binaries/Win64/PalDefender
   palworld-server-tool:
     image: pserver-docker-tool:latest
     container_name: palworld-server-tool
     restart: unless-stopped
     ports:
       - "8802:8080"  # Web管理界面端口
     volumes:
       - ./palworld:/game  # 挂载游戏数据目录
       - ./backups:/app/backups  # 挂载备份目录
     environment:
       - WEB__PASSWORD=hahaha
       - RCON__ADDRESS=palworld-server-wine:25575
       - RCON__PASSWORD=hahaha  #服务器配置文件中的 AdminPassword
       - REST__ADDRESS=http://palworld-server-wine:8212
       - REST__PASSWORD=hahaha  #服务器配置文件中的 AdminPassword
       - SAVE__PATH=/game
       - SAVE__SYNC_INTERVAL=120
       - TASK__PLAYER_LOGGING=true
     depends_on:
      - palworld-server-wine
~~~
## 运行前准备
~~~ bash
mkdir pserver
cd pserver
vim docker-compose.yml # 把上面内容复制进去
mkdir palworld
mkdir PalDefender
mkdir backups
chown -R 1000:1000 ./palworld
chown -R 1000:1000 ./PalDefender
~~~

## 运行
docker compose up -d

