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
         - TZ=Asia/Shanghai
         - PORT=8211 # Optional but recommended
         - MULTITHREADING=true
         - ADMIN_PASSWORD=hahaha
         - ENABLE_MULTITHREAD=true  # enable multithreading
      volumes:
         - ./palworld:/home/wineuser/.wine/drive_c/Steam/steamapps/common/PalServer/Pal/Saved
         - ./PalDefender:/home/wineuser/.wine/drive_c/Steam/steamapps/common/PalServer/Pal/Binaries/Win64/PalDefender

   palworld-server-tool:
     image: pserver-docker-tool:latest
     container_name: palworld-server-tool
     restart: unless-stopped
     ports:
       - "8805:8080"  # Web管理界面端口
     volumes:
       - ./palworld:/game  # 挂载游戏数据目录
       - ./backups:/app/backups  # 挂载备份目录
     environment:
       - TZ=Asia/Shanghai
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
chown -R 1000:1000 ./palworld
chown -R 1000:1000 ./PalDefender
~~~

## 运行
docker compose up -d

## 自定义配置  
使用服务器管理工具的过程中，发现服务器信息加载失败。


查看日志发现管理工具docker调用服务器docker的8212服务失败。原因是未开启RestfulAPI，查看/path/to/pserver/palworld/Config/WindowsServer/PalWorldSettings.ini配置文件，发现RESRFULAPI=False。需要自行去修改配置文件，然后重启服务。如果想进一步自定义配置，也需要自行修改配置文件。。如果有其他参数需要另行修改，也遵循此操作，尽量直接修改配置文件，不要通过docker-compose.yml指定。  
vim /path/to/pserver/palworld/Config/WindowsServer/PalWorldSettings.ini  #修改RESTAPIEnabled=True  
docker compose down  
docker compose up -d  


## 参考参数
 [/Script/Pal.PalGameWorldSettings]
OptionSettings=(Difficulty=None,RandomizerType=None,RandomizerSeed="",bIsRandomizerPalLevelRandom=False,DayTimeSpeedRate=1.000000,NightTimeSpeedRate=1.000000,ExpRate=1.000000,PalCaptureRate=1.000000,PalSpawnNumRate=1.000000,PalDamageRateAttack=1.000000,PalDamageRateDefense=1.000000,PlayerDamageRateAttack=1.000000,PlayerDamageRateDefense=1.000000,PlayerStomachDecreaceRate=1.000000,PlayerStaminaDecreaceRate=1.000000,PlayerAutoHPRegeneRate=1.000000,PlayerAutoHpRegeneRateInSleep=1.000000,PalStomachDecreaceRate=1.000000,PalStaminaDecreaceRate=1.000000,PalAutoHPRegeneRate=1.000000,PalAutoHpRegeneRateInSleep=1.000000,BuildObjectHpRate=1.000000,BuildObjectDamageRate=1.000000,BuildObjectDeteriorationDamageRate=1.000000,CollectionDropRate=1.000000,CollectionObjectHpRate=1.000000,CollectionObjectRespawnSpeedRate=1.000000,EnemyDropItemRate=1.000000,DeathPenalty=All,bEnablePlayerToPlayerDamage=False,bEnableFriendlyFire=False,bEnableInvaderEnemy=True,bActiveUNKO=False,bEnableAimAssistPad=True,bEnableAimAssistKeyboard=False,DropItemMaxNum=3000,DropItemMaxNum_UNKO=100,BaseCampMaxNum=128,BaseCampWorkerMaxNum=15,DropItemAliveMaxHours=1.000000,bAutoResetGuildNoOnlinePlayers=False,AutoResetGuildTimeNoOnlinePlayers=72.000000,GuildPlayerMaxNum=20,BaseCampMaxNumInGuild=4,PalEggDefaultHatchingTime=72.000000,WorkSpeedRate=1.000000,AutoSaveSpan=30.000000,bIsMultiplay=False,bIsPvP=False,bHardcore=False,bPalLost=False,bCharacterRecreateInHardcore=False,bCanPickupOtherGuildDeathPenaltyDrop=False,bEnableNonLoginPenalty=True,bEnableFastTravel=True,bEnableFastTravelOnlyBaseCamp=False,bIsStartLocationSelectByMap=True,bExistPlayerAfterLogout=False,bEnableDefenseOtherGuildPlayer=False,bInvisibleOtherGuildBaseCampAreaFX=False,bBuildAreaLimit=False,ItemWeightRate=1.000000,CoopPlayerMaxNum=4,ServerPlayerMaxNum=32,ServerName="666666",ServerDescription="Default Palworld Server",AdminPassword="hahaha",ServerPassword="",bAllowClientMod=True,PublicPort=8211,PublicIP="",RCONEnabled=True,RCONPort=25575,Region="",bUseAuth=True,BanListURL="https://api.palworldgame.com/api/banlist.txt",RESTAPIEnabled=True,RESTAPIPort=8212,bShowPlayerList=False,ChatPostLimitPerMinute=30,CrossplayPlatforms=(Steam,Xbox,PS5,Mac),bIsUseBackupSaveData=True,LogFormatType=Text,bIsShowJoinLeftMessage=True,SupplyDropSpan=180,EnablePredatorBossPal=True,MaxBuildingLimitNum=0,ServerReplicatePawnCullDistance=15000.000000,bAllowGlobalPalboxExport=True,bAllowGlobalPalboxImport=False,EquipmentDurabilityDamageRate=1.000000,ItemContainerForceMarkDirtyInterval=1.000000,ItemCorruptionMultiplier=1.000000,DenyTechnologyList=,GuildRejoinCooldownMinutes=0,BlockRespawnTime=5.000000,RespawnPenaltyDurationThreshold=0.000000,RespawnPenaltyTimeScale=2.000000,bDisplayPvPItemNumOnWorldMap_BaseCamp=False,bDisplayPvPItemNumOnWorldMap_Player=False,AdditionalDropItemWhenPlayerKillingInPvPMode="PlayerDropItem",AdditionalDropItemNumWhenPlayerKillingInPvPMode=1,bAdditionalDropItemWhenPlayerKillingInPvPMode=False,bAllowEnhanceStat_Health=True,bAllowEnhanceStat_Attack=True,bAllowEnhanceStat_Stamina=True,bAllowEnhanceStat_Weight=True,bAllowEnhanceStat_WorkSpeed=True)











