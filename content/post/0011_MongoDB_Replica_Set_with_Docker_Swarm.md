---
title: "在 Docker Swarm 上製作 MongoDB Replica Set"
date: 2018-06-29T10:06:56+08:00
draft: true
author: "Jimmy Lin"
isCJKLanguage: true
tags: ["Docker", "Backend"]
categories: ["DevOps"]
---

這篇文章說明如何在 Docker Swarm 上面使用 Mongodb 自身的 Replica Set 以及環境建置，雖然 Docker Service 已經可以使用內建的 replica 了，不過使用 Mongodb 自身的支援度較高也較完整，本文參考自 Medium 的[這篇](https://medium.com/@kalahari/running-a-mongodb-replica-set-on-docker-1-12-swarm-mode-step-by-step-a5f3ba07d06e)，寫得非常清楚詳細。

{{< figure src="/images/cover/0011.png" width="" height="" >}}

```yml
version: '3.3'
services:
  website:
    image: jimmylin212/native_pm2_env:latest
    volumes:
      - "/home/ipdev/mercury:/usr/src/app"
    ports:
      - "3000:3000"
    networks:
      - mercury
    deploy:
      replicas: 2
      update_config:
        parallelism: 1
        delay: 20s
  db_1:
    image: mongo
    volumes:
      - "/home/ipdev/mongo/mercury_mongodb:/data/db"
      - "/home/ipdev/mongo/config/mercury.conf:/etc/mongo.conf"
    environment:
      - MONGO_INITDB_DATABASE=mercury
    command: mongod --config /etc/mongo.conf
    networks:
      - mercury
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.mongo.replica == 1
  db_2:
    image: mongo
    volumes:
      - "/home/ipdev/ipdev_share/mongo/mercury/db:/data/db"
      - "/home/ipdev/ipdev_share/mongo/mercury/config/mongo.conf:/etc/mongo.conf"
    environment:
      - MONGO_INITDB_DATABASE=mercury
    command: mongod --config /etc/mongo.conf
    networks:
      - mercury
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.mongo.replica == 2
networks:
  mercury:
    external:
      name: mercury

```

