---
title: "在 Docker Swarm 上製作 MongoDB Replica Set"
date: 2018-07-04T10:06:56+08:00
draft: false
author: "Jimmy Lin"
isCJKLanguage: true
tags: ["Docker", "Backend"]
categories: ["DevOps"]
image: "/images/cover/0011.png"
---

這篇文章說明如何在 Docker Swarm 上面使用 Mongodb 自身的 Replica Set 以及環境建置，雖然 Docker Service 已經可以使用內建的 replica 了，不過使用 Mongodb 自身的支援度較高也較完整，本文參考自 Medium 的[這篇](https://medium.com/@kalahari/running-a-mongodb-replica-set-on-docker-1-12-swarm-mode-step-by-step-a5f3ba07d06e)，寫得非常清楚詳細。

{{< figure src="/images/cover/0011.png" width="" height="" >}}

## 初始化 Docker Swarm Mode
我們的架構長這樣，一台 manager _(primary)_，兩台 worker _(secondary)_，三台機器在同一個 swarm 。如果還沒有初始化 swarm，請在 manager 的機器上先初始化，另外再將兩台 worker 的機器加入這個 manager，成為一組 swarm。

```shell
## 在 manager 機器上初始化 swarm mode 
docker swarm init --listen-addr MANAGER_IP_ADDRESS:2377 --advertise-addr MANAGER_IP_ADDRESS:2377

## 輸出 
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-3fhjqnlewg78kcb0hkoc2v9o3zwmotqgurhy42jgkrnhpvmgh8-5qylz0finkxjpaiulli0yfs4w MANAGER_IP_ADDRESS:2377

## ----------
## 在 worker_1 機器上加入 swarm，成為 node
docker swarm join --token SWMTKN-1-3fhjqnlewg78kcb0hkoc2v9o3zwmotqgurhy42jgkrnhpvmgh8-5qylz0finkxjpaiulli0yfs4w --listen-addr WORKER_1_IP_ADDRESS:2377 --advertise-addr=WORKER_1_IP_ADDRESS:2377 MANAGER_IP_ADDRESS:2377

## ----------
## 在 worker_2 機器上加入 swarm，成為 node
docker swarm join --token SWMTKN-1-3fhjqnlewg78kcb0hkoc2v9o3zwmotqgurhy42jgkrnhpvmgh8-5qylz0finkxjpaiulli0yfs4w --listen-addr WORKER_2_IP_ADDRESS:2377 --advertise-addr=WORKER_2_IP_ADDRESS:2377 MANAGER_IP_ADDRESS:2377
```

這時候切回 manager 機器，利用指令
```shell
## 在 manager 上查看 docker 資訊
docker info
```
可以看到 swarm node 的資訊，如果看到下面的資訊代表 swarm 已經建立成功了。
```
Swarm: active
 NodeID: 3nrg063zylzxfe6ij92c6safp
 Is Manager: true  <---------------- 註明這個 node 為 manager
 ClusterID: s3rgodac8w7qdp3la0gwuu620
 Managers: 1
 Nodes: 3 <--------------- 有三個 node 在這個 swarm 中
 Orchestration:
  Task History Retention Limit: 5
 Raft:
  Snapshot Interval: 10000
  Number of Old Snapshots to Retain: 0
  Heartbeat Tick: 1
  Election Tick: 10
 Dispatcher:
  Heartbeat Period: 5 seconds
 CA Configuration:
  Expiry Duration: 3 months
  Force Rotate: 0
 Autolock Managers: false
 Root Rotation In Progress: false
 Node Address: MANAGER_IP_ADDRESS
 Manager Addresses:
  MANAGER_IP_ADDRESS:2377
```
***

## 初始化 MongaDB 環境
為了要更好的控制或是讓不同的 mongodb 部署在不同機器上，可以在 node 上設定 label。

```shell
## 在 manager 機器上設定 node 的 label

## 先查看 node 資訊，找出 node ID
docker node ls
## 輸出
ID                            HOSTNAME  STATUS  AVAILABILITY   MANAGER STATUS   ENGINE VERSION
sv8gp5uyfe471iohkfwu8cyv2     WORKER_2  Ready   Active                          18.06.0-ce-rc1
34dizj0kic58kplkjsdhpgjv8     WORKER_1  Ready   Active                          17.06.0-ce
3nrg063zylzxfe6ij92c6safp *   MANAGER   Ready   Active         Leader           18.03.1-ce

## 為 manager 設定 label mongo.replica=1
docker node update --label-add mongo.replica=1 3nrg063zylzxfe6ij92c6safp
## 為 worker_1 設定 label mongo.replica=2
docker node update --label-add mongo.replica=2 34dizj0kic58kplkjsdhpgjv8
## 為 worker_2 設定 label mongo.replica=3
docker node update --label-add mongo.replica=3 sv8gp5uyfe471iohkfwu8cyv2
```

Replica Set 需要一個 overlap 的網路環境互相溝通，建立新的網路環境
```shell
## 在 manager 機器上新增網路
docker network create --driver overlay mongo

## 查看目前網路
docker network ls
## 輸出
NETWORK ID          NAME                DRIVER              SCOPE
2ef63bb7d7ba        bridge              bridge              local
6469b890cf65        docker_gwbridge     bridge              local
567d39d1ac4c        host                host                local
kgt6db91iko1        ingress             overlay             swarm
l136q8h5hzfl        mongo               overlay             swarm  ## <---- 這個是新增的
e0bf52c71f72        none                null                local
```

另外這邊我們可以決定要把 db 放在哪邊：

- 放在 docker 環境下，利用 volume 掛載一個 docker 空間給 container
{{< figure src="/images/0011/types-of-mounts-volume.png" width="50%" height="" >}} 

- 放在硬碟中，利用 bind mount 掛載一個實體位置給 container
{{< figure src="/images/0011/types-of-mounts-bind.png" width="50%" height="" >}}

詳細的差異可以參考[官方文件](https://docs.docker.com/storage/volumes/)

我們選擇的是第二個方式，想要在硬碟中可以看到實際的 db raw data，另外我們也把 mongodb config 放在硬碟中，以便日後的修改；如果選擇的是第一種方式，那就要在三台機器上面分別新增 volume：

```bash
## 在 manager 機器上新增 db volume 以及 config volume
docker volume create --name data
docker volume create --name config

## 在 worker_1 機器上新增 db volume 以及 config volume
docker volume create --name data
docker volume create --name config

## 在 worker_2 機器上新增 db volume 以及 config volume
docker volume create --name data
docker volume create --name config
```

當然也可以混用，在 manager 上面採用第二種做法，直接 bind mount 一個硬碟空間，在 worker 上採用第二種做法，mount volume 到 container 中。

***
## 啟動 Mongodb service
我們使用 `docker-compose.yml` 啟動 mongodb service，這份 `yml` 使用混合模式，檔案內容如下：

```yml
version: '3.3'
services:
  db_manager:
    image: mongo:3.6.4
    volumes:
      ## 利用 bind mount 掛載硬碟空間以及存放在硬碟上的 config 給 container
      - "PATH_TO_DIRECTORY_FOR_DB:/data/db"
      - "PATH_TO_CONFIG_FILE:/etc/mongo.conf"
    environment:
      - MONGO_INITDB_DATABASE=YOUR_DB_NAME
    command: mongod --config /etc/mongo.conf
    networks:
      - mongo
    deploy:
      replicas: 1
      placement:
        constraints:
          ## 指定這個 service 被 deploy 至 label mongo.replica == 1 的 node 
          - node.labels.mongo.replica == 1
  db_worker_1:
    image: mongo:3.6.4
    volumes:
      ## 利用 volume 掛載 docker 空間給 container
      - type: volume
        source: data
        target: /data/db
      ## 利用 bind mount 掛載存放在硬碟上的 config 給 container
      - "PATH_TO_CONFIG_FILE:/etc/mongo.conf"
    environment:
      - MONGO_INITDB_DATABASE=YOUR_DB_NAME
    command: mongod --config /etc/mongo.conf
    networks:
      - mongo
    deploy:
      replicas: 1
      placement:
        constraints:
          ## 指定這個 service 被 deploy 至 label mongo.replica == 2 的 node
          - node.labels.mongo.replica == 2
  db_worker_2:
    image: mongo:3.6.4
    volumes:
      ## 利用 volume 掛載 docker 空間給 container
      - type: volume
        source: data
        target: /data/db
      ## 利用 bind mount 掛載存放在硬碟上的 config 給 container
      - "PATH_TO_CONFIG_FILE:/etc/mongo.conf"
    environment:
      - MONGO_INITDB_DATABASE=YOUR_DB_NAME
    command: mongod --config /etc/mongo.conf
    networks:
      - mongo
    deploy:
      replicas: 1
      placement:
        constraints:
          ## 指定這個 service 被 deploy 至 label mongo.replica == 2 的 node
          - node.labels.mongo.replica == 3

networks:
  mercury:
    external:
      ## 使用 external 的網路，否則 docker stack 會自動產生網路，網路名稱前綴 service 名字
      name: mongo

volumes:
  data:
```

另外我們的 `mongo.conf` 如下，最重要的設定就是 **replSetName**，等等在 mongodb console 會用到，按照上面的 `yml` ，請記得把這個 config 放到三台機器的指定路徑 _(PATH_TO_CONFIG_FILE)_ 下。

```conf
# mongod.conf

# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# Where and how to store data.
storage:
  dbPath: /data/db
#  journal:
#    enabled: true
#  engine:
#  mmapv1:
#  wiredTiger:

# where to write logging data.
#systemLog:
#  destination: file
#  logAppend: true
#  path: /var/log/mongodb/mongod.log

# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0

# how the process runs
processManagement:
  timeZoneInfo: /usr/share/zoneinfo

#security:
#  keyFile: /etc/mongodb_keyfile
#operationProfiling:

replication:
  replSetName: rs0
#sharding:

## Enterprise-Only Options:

#auditLog:
```

啟動 `docker-compose.yml` 中的 service：

```bash
## 啟動 service
docker stack deploy -c docker-compose.yml SERVICE_NAME

## 查看 service 資訊
docker service ls 
## 輸出
ID             NAME          MODE         REPLICAS   IMAGE         PORTS
z6q4z2646b20   db_manager    replicated   1/1        mongo:3.6.4                         
7uptth7xnpuj   db_worker_1   replicated   1/1        mongo:3.6.4                         
kuq8djlsoszc   db_worker_2   replicated   1/1        mongo:3.6.4                         
```

看到上面資訊代表 service 都已經成功起來了，最後就是設定 replica set 啦。

***
## 初始化 Replica Set
為求方便起見，我們直接進到 mongodb console 操作，先查看一下 manager 上面的 mongodb container 資訊

```shell
## 在 manager 機器上查看 container 資訊
docker ps 
## 輸出
CONTAINER ID   IMAGE         COMMAND                  CREATED     STATUS     PORTS      NAMES
b578b7f6c539   mongo:3.6.4   "docker-entrypoint.s…"   2 days ago  Up 2 days  27017/tcp  db_manager.1.mim1o3ojh92jhcxvc2ugboam7

## 進入 container mongodb console
docker exec -it db_manager.1.mim1o3ojh92jhcxvc2ugboam7 mongo
```

在 mongodb console 中，我們利用 mongodb 的指令來初始化以及管理 replica set，詳細可以參考[官方文件](https://docs.mongodb.com/manual/tutorial/deploy-replica-set/)。使用下面指令可以初始化 replica set，方便的是可以直接用 service name 當作 host name，docker 會自動連結到正確的機器上。

```javascript
rs.initiate( {
   _id : "rs0",
   members: [
      { _id: 0, host: "db_manager:27017" },
      { _id: 1, host: "db_worker_1:27017" },
      { _id: 2, host: "db_worker_2:27017" }
   ]
})
```

再來可以在 manager 上面查看 replica set 資訊

```javascript
// 在 manager (primary) 上查看 replica set 資訊
rs.status()
```

直接看 members 那欄，可以看到三個 service 已經被加到一組 replica set，一個 Primary，兩個 secondary。
```json
// rs.status() 的輸出
{
	"set" : "rs0",
	"date" : ISODate("2018-07-04T06:46:49.280Z"),
	"myState" : 1,
	"term" : NumberLong(5),
	"heartbeatIntervalMillis" : NumberLong(2000),
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(1530686803, 1),
			"t" : NumberLong(5)
		},
		"readConcernMajorityOpTime" : {
			"ts" : Timestamp(1530686803, 1),
			"t" : NumberLong(5)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1530686803, 1),
			"t" : NumberLong(5)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1530686803, 1),
			"t" : NumberLong(5)
		}
	},
	"members" : [
		{
			"_id" : 1,
			"name" : "db_manager:27017",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 175074,
			"optime" : {
				"ts" : Timestamp(1530686803, 1),
				"t" : NumberLong(5)
			},
			"optimeDate" : ISODate("2018-07-04T06:46:43Z"),
			"electionTime" : Timestamp(1530531998, 2),
			"electionDate" : ISODate("2018-07-02T11:46:38Z"),
			"configVersion" : 1,
			"self" : true
		},
		{
			"_id" : 2,
			"name" : "db_worker_1:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 175071,
			"optime" : {
				"ts" : Timestamp(1530686803, 1),
				"t" : NumberLong(5)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1530686803, 1),
				"t" : NumberLong(5)
			},
			"optimeDate" : ISODate("2018-07-04T06:46:43Z"),
			"optimeDurableDate" : ISODate("2018-07-04T06:46:43Z"),
			"lastHeartbeat" : ISODate("2018-07-04T06:46:48.850Z"),
			"lastHeartbeatRecv" : ISODate("2018-07-04T06:46:47.593Z"),
			"pingMs" : NumberLong(0),
			"syncingTo" : "db_manager:27017",
			"configVersion" : 1
		},
		{
			"_id" : 3,
			"name" : "db_worker_2:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 175071,
			"optime" : {
				"ts" : Timestamp(1530686803, 1),
				"t" : NumberLong(5)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1530686803, 1),
				"t" : NumberLong(5)
			},
			"optimeDate" : ISODate("2018-07-04T06:46:43Z"),
			"optimeDurableDate" : ISODate("2018-07-04T06:46:43Z"),
			"lastHeartbeat" : ISODate("2018-07-04T06:46:49.078Z"),
			"lastHeartbeatRecv" : ISODate("2018-07-04T06:46:49Z"),
			"pingMs" : NumberLong(0),
			"syncingTo" : "db_manager:27017",
			"configVersion" : 1
		}
	],
	"ok" : 1,
	"operationTime" : Timestamp(1530686803, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1530686803, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
```

不放心的話可以測試一下，新增一筆資料，看看 secondary 有沒有也同時新增，另外多久 sync 一次也都可以直接在這邊設定。

***
這次剛好有找到不錯的文章，在資訊蒐集上面省了很多時間，不過也是第一次設定，相信之後還有機會可做一些最佳化，或是資料量增加之後可能可以做 sharding，學到很多東西，Docker 這東西越用越覺得厲害，應該還有很多功能是還沒有碰到的。有問題請發問，看到都會回。

