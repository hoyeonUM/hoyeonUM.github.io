---
layout: post
title: "MongoDB 를 docker로 세팅해보자"
author: "hoyeonUm"
---
{% highlight markdown %}
몽고디비를 로컬환경에 설치를 해야하는데 라우트서버와 샤드서버 콘피그서버를 설치하려니까 이것저것 해야할 부분들이 너무많아서 docker 로 설치하는 방법이 제일 빠를거 같아 docker 를 올려보았다.
{% endhighlight %}

### 기본 세팅
{% highlight shell bash %}
#docker  설치
yum -y install docker

#docker compose 설치
sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
docker-compose version 1.22.0, build 1719ceb
{% endhighlight %}

### compose yaml 파일 생성 
{% highlight shell bash %}
#디렉토리 생성
mkdir /home/mongodb

#파일생성
vim docker-compose.yaml

version: '2'
services:
  mongorsn1:
    container_name: mongors1n1
    image: mongo
    command: mongod --shardsvr --replSet mongors1 --dbpath /data/db --port 27017
    ports:
      - 27017:27017
    expose:
      - "27017"
    environment:
      TERM: xterm
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /mongo_cluster/data1:/data/db
  mongors1n2:
    container_name: mongors1n2
    image: mongo
    command: mongod --shardsvr --replSet mongors1 --dbpath /data/db --port 27017
    ports:
      - 27027:27017
    expose:
      - "27017"
    environment:
      TERM: xterm
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /mongo_cluster/data2:/data/db
  mongors1n3:
    container_name: mongors1n3
    image: mongo
    command: mongod --shardsvr --replSet mongors1 --dbpath /data/db --port 27017
    ports:
      - 27037:27017
    expose:
      - "27017"
    environment:
      TERM: xterm
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /mongo_cluster/data3:/data/db
  mongocfg1:
    container_name: mongocfg1
    image: mongo
    command: mongod --configsvr --replSet mongors1conf --dbpath /data/db --port 27017
    environment:
      TERM: xterm
    expose:
      - "27017"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /mongo_cluster/config1:/data/db
  mongocfg2:
    container_name: mongocfg2
    image: mongo
    command: mongod --configsvr --replSet mongors1conf --dbpath /data/db --port 27017
    environment:
      TERM: xterm
    expose:
      - "27017"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /mongo_cluster/config2:/data/db
  mongocfg3:
    container_name: mongocfg3
    image: mongo
    command: mongod --configsvr --replSet mongors1conf --dbpath /data/db --port 27017
    environment:
      TERM: xterm
    expose:
      - "27017"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /mongo_cluster/config3:/data/db
  mongos1:
    container_name: mongos1
    image: mongo
    depends_on:
      - mongocfg1
      - mongocfg2
    command: mongos --configdb mongors1conf/mongocfg1:27017,mongocfg2:27017,mongocfg3:27017 --port 27017 --bind_ip_all
    ports:
      - 27019:27017
    expose:
      - "27017"
    volumes:
      - /etc/localtime:/etc/localtime:ro
  mongos2:
    container_name: mongos2
    image: mongo
    depends_on:
      - mongocfg1
      - mongocfg2
    command: mongos --configdb mongors1conf/mongocfg1:27017,mongocfg2:27017,mongocfg3:27017 --port 27017 --bind_ip_all
    ports:
      - 27020:27017
    expose:
      - "27017"
    volumes:
      - /etc/localtime:/etc/localtime:ro
{% endhighlight %}

### 실행
{% highlight shell bash %}
docker-compose up -d
`
docker ps -a
CONTAINER ID  IMAGE    COMMAND                  CREATED      STATUS       PORTS                      NAMES
051d3f0e87b0  mongo    "docker-entrypoint..."   2 hours ago  Up 2 hours   0.0.0.0:27020->27017/tcp   mongos2
e8e29446fa17  mongo    "docker-entrypoint..."   2 hours ago  Up 2 hours   0.0.0.0:27019->27017/tcp   mongos1
a031815d22a   mongo    "docker-entrypoint..."   4 hours ago  Up 3 hours   27017/tcp                  mongocfg2
e56c1111482   mongo    "docker-entrypoint..."   4 hours ago  Up 3 hours   27017/tcp                  mongocfg1
858dc7463fa   mongo    "docker-entrypoint..."   4 hours ago  Up 3 hours   27017/tcp                  mongocfg3
934b4fe8f8a8  mongo    "docker-entrypoint..."   4 hours ago  Up 3 hours   0.0.0.0:27037->27017/tcp   mongors1n3
0181076d2845  mongo    "docker-entrypoint..."   4 hours ago  Up 3 hours   0.0.0.0:27027->27017/tcp   mongors1n2
cfdea54a34c9  mongo    "docker-entrypoint..."   4 hours ago  Up 3 hours   0.0.0.0:27017->27017/tcp   mongors1n1
`

#콘피그 서버 replica set 구성
docker exec -it mongocfg1 bash -c "echo 'rs.initiate({_id: \"mongors1conf\",configsvr: true, members: [{ _id : 0, host : \"mongocfg1\" },{ _id : 1, host : \"mongocfg2\" }, { _id : 2, host : \"mongocfg3\" }]})' | mongo"
#콘피그서버 체크
docker exec -it mongocfg1 bash -c "echo 'rs.status()' | mongo"
#위와 같은명령어로 확인시 members 노드에 PRIMARY 1개와 SECONDARY 2개로 멤버가 구성되있는것을 확인할수있다.

#샤드 서버 replica set 구성
docker exec -it mongors1n1 bash -c "echo 'rs.initiate({_id : \"mongors1\", members: [{ _id : 0, host : \"mongors1n1\" },{ _id : 1, host : \"mongors1n2\" },{ _id : 2, host : \"mongors1n3\" }]})' | mongo"
#샤드서버 체크
docker exec -it mongors1n1 bash -c "echo 'rs.status()' | mongo"
#위와 같은 명령어로 확인시 members 노드에 PRIMARY 1개와 SECONDARY 2개로 멤버가 구성되있는것을 확인할수있다.

#라우트 서버에 샤드서버 추가
docker exec -it mongos1 bash -c "echo 'sh.addShard(\"mongors1/mongors1n1\")' | mongo "
 
#라우트 서버 확인
docker exec -it mongos1 bash -c "echo 'sh.status()' | mongo "
`
 shards:
        {  "_id" : "mongors1",  "host" : "mongors1/mongors1n1:27017,mongors1n2:27017,mongors1n3:27017",  "state" : 1 }
`
#테스트를 하던도중  세컨드리에서 검색을 하는 도중 다음과같은 오류가 발생하였다.
`
mongors1:SECONDARY> db.test.count()

2018-06-26T14:04:40.682+0900 E QUERY    [js] Error: count failed: {
        "operationTime" : Timestamp(1533099879, 1),
        "ok" : 0,
        "errmsg" : "not master and slaveOk=false",
        "code" : 13435,
        "codeName" : "NotMasterNoSlaveOk",
        "$gleStats" : {
                "lastOpTime" : Timestamp(0, 0),
                "electionId" : ObjectId("000000000000000000000000")
        },
        "lastCommittedOpTime" : Timestamp(1533099879, 1),
        "$configServerState" : {
                "opTime" : {
                        "ts" : Timestamp(1533099870, 2),
                        "t" : NumberLong(3)
                }
        },
        "$clusterTime" : {
                "clusterTime" : Timestamp(1533099879, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }

`
#rs.slaveOk()
#해당 명령어로 해결할수있다.
{% endhighlight %}
{% highlight php %}

mongos 1,2 는 라우터서버다.
라우트서버에서는 모든 요청을 받는다.
php 에서  https://packagist.org/packages/mongodb/mongodb 해당 라이브러리를 사용한다면 다음과 같은 코드를 사용한다
해당 코드 순서의 위치와는 상관없이 랜덤하게 access 하며 27020 포트가 죽었을때는 27019 로 요청하는것을 확인하였다. 아래 ngrep sample 참조.

mongocfg 1,2,3 은 컨피그 서버다.
컨피그서버에서는 shard 에 어떻게 데이터가 분산되어있는지에 대한 메타정보를 가지고있는 서버이다. 메타정보가 있기때문에 라우터에서 컨피그서버에 요청후 샤드서버에서 데이터를 가져올수 있게된다.

mongors1n1,2,3 은 샤드서버다.
샤드서버는 데이터를 수직 혹은 수평으로 데이터를 분산처리하여 저장한다. 해당 서버늘리면 scale-out 이 가능해지고, chunk 라는 단위로 데이터가 파일이 쪼개진다.

<?php
use MongoDB\Client;
new Client('mongodb://127.0.0.1:27020,127.0.0.1:27019');

{% endhighlight %}

### 프라이머리 선출 테스트
{% highlight shell bash %}
docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
051d3f0e87b0        mongo               "docker-entrypoint..."   3 hours ago         Up 3 hours          0.0.0.0:27020->27017/tcp   mongos2
e8e29446fa17        mongo               "docker-entrypoint..."   3 hours ago         Up 3 hours          0.0.0.0:27019->27017/tcp   mongos1
a031815d22a9        mongo               "docker-entrypoint..."   5 hours ago         Up 4 hours          27017/tcp                  mongocfg2
e56c11114823        mongo               "docker-entrypoint..."   5 hours ago         Up 4 hours          27017/tcp                  mongocfg1
858dc7463f0a        mongo               "docker-entrypoint..."   5 hours ago         Up 4 hours          27017/tcp                  mongocfg3
934b4fe8f8a8        mongo               "docker-entrypoint..."   5 hours ago         Up 4 hours          0.0.0.0:27037->27017/tcp   mongors1n3
0181076d2845        mongo               "docker-entrypoint..."   5 hours ago         Up 4 hours          0.0.0.0:27027->27017/tcp   mongors1n2
cfdea54a34c9        mongo               "docker-entrypoint..."   5 hours ago         Up 4 hours          0.0.0.0:27017->27017/tcp   mongors1n1
3d6f0cc60b47        centos              "/bin/bash"              15 hours ago        Up 5 hours                                     forcommit

# mongors1n1  얘가 프라이머리 샤드
docker kill mongors1n1

docker ps -a
docker ps -a         
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                       PORTS                      NAMES
051d3f0e87b0        mongo               "docker-entrypoint..."   3 hours ago         Up 3 hours                   0.0.0.0:27020->27017/tcp   mongos2
e8e29446fa17        mongo               "docker-entrypoint..."   3 hours ago         Up 3 hours                   0.0.0.0:27019->27017/tcp   mongos1
a031815d22a9        mongo               "docker-entrypoint..."   5 hours ago         Up 4 hours                   27017/tcp                  mongocfg2
e56c11114823        mongo               "docker-entrypoint..."   5 hours ago         Up 4 hours                   27017/tcp                  mongocfg1
858dc7463f0a        mongo               "docker-entrypoint..."   5 hours ago         Up 4 hours                   27017/tcp                  mongocfg3
934b4fe8f8a8        mongo               "docker-entrypoint..."   5 hours ago         Up 4 hours                   0.0.0.0:27037->27017/tcp   mongors1n3
0181076d2845        mongo               "docker-entrypoint..."   5 hours ago         Up 4 hours                   0.0.0.0:27027->27017/tcp   mongors1n2
cfdea54a34c9        mongo               "docker-entrypoint..."   5 hours ago         Exited (137) 6 seconds ago                              mongors1n1
3d6f0cc60b47        centos              "/bin/bash"              15 hours ago        Up 5 hours                                              forcommit
 
 
#mongo --port 27027 (mongors1n2 접속)
2018-06-26T11:03:58.842+0900 I CONTROL  [initandlisten]
mongors1:PRIMARY

 
SECONDARY에서 PRIMARY로 바뀌어있는걸 확인할수있다.
{% endhighlight %}

### 라우터 테스트
{% highlight php %}
use MongoDB\Client
 
 
$host = 'mongodb://xxxx:27020,xxxx:27019;
$this->client = new Client($host);
$this->collection = $this->client->selectCollection('testDB', 'testCollection');
 
 
for ($i = 0; $i <= 1000; $i++) {
    $this->collection->count();
}
{% endhighlight %}

#### 다음은 ngrep 으로 서버로 요청오는 패킷을 잡아봤다.

{% highlight shell bash %}
yum install -y ngrep #미설치시
`
ngrep port 27019
 
interface: enp0s25 (xxx.xxx.xxx.xxx/255.255.255.255)
filter: ( port 27019 ) and ((ip || ip6) || (vlan && (ip || ip6)))
###
T xxx.xxx.xxx.xxx:58114 -> xxx.xxx.xxx.xxx:27019 [AP]
  O...................admin.$cmd.........(....isMaster......client......driver.C....name.....mongoc / ext-mongodb:PHP..version.....1.9.2 / 1.4.0...os.Y....type.....Windows
  ..name.....Windows..version.....10.0 (17134)..architecture.....x86...platform.@...cfg=0x402a0e9 CC=MSVC 1912 CFLAGS="" LDFLAGS="" / PHP 7.2.1-dev...compression.......  
##
T xxx.xxx.xxx.xxx:27019 -> xxx.xxx.xxx.xxx:58114 [AP]
  s...................................O....ismaster...msg.....isdbgrid..maxBsonObjectSize......maxMessageSizeBytes..l...maxWriteBatchSize......localTime.....d....logicalSe
  ssionTimeoutMinutes......maxWireVersion......minWireVersion......ok........?.operationTime.......b[.$clusterTime.X....clusterTime.......b[.signature.3....hash...........
  ................keyId............ 
.......생략
 
 
 
 
ngrep port 27020
 
interface: enp0s25 (xxx.xxx.xxx.xxx/255.255.255.255)
filter: ( port 27020 ) and ((ip || ip6) || (vlan && (ip || ip6)))
###
T xxx.xxx.xxx.xxx:58113 -> xxx.xxx.xxx.xxx:27020 [A]
  ......                                                                                                                                                                  
#
T xxx.xxx.xxx.xxx:58113 -> xxx.xxx.xxx.xxx:27020 [AP]
  O...................admin.$cmd.........(....isMaster......client......driver.C....name.....mongoc / ext-mongodb:PHP..version.....1.9.2 / 1.4.0...os.Y....type.....Windows
  ..name.....Windows..version.....10.0 (17134)..architecture.....x86...platform.@...cfg=0x402a0e9 CC=MSVC 1912 CFLAGS="" LDFLAGS="" / PHP 7.2.1-dev...compression.......  
##
.......생략
`
{% endhighlight %}
#### 양쪽 라우터로 번갈아가면서 요청이 오는것을 확인할수있었다.

#### 라우터를 강제로 죽여서 테스트를 진행해보았다.

{% highlight shell bash %}
docker kill mongos1
mongos1
docker ps -a      
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                       PORTS                      NAMES
051d3f0e87b0        mongo               "docker-entrypoint..."   4 hours ago        Up 4 hours                  0.0.0.0:27020->27017/tcp   mongos2
e8e29446fa17        mongo               "docker-entrypoint..."   4 hours ago        Exited (137) 3 seconds ago                              mongos1
a031815d22a9        mongo               "docker-entrypoint..."   4 hours ago        Up 4 hours                  27017/tcp                  mongocfg2
e56c11114823        mongo               "docker-entrypoint..."   4 hours ago        Up 4 hours                  27017/tcp                  mongocfg1
858dc7463f0a        mongo               "docker-entrypoint..."   4 hours ago        Up 4 hours                  27017/tcp                  mongocfg3
934b4fe8f8a8        mongo               "docker-entrypoint..."   4 hours ago        Up 4 hours                  0.0.0.0:27037->27017/tcp   mongors1n3
0181076d2845        mongo               "docker-entrypoint..."   4 hours ago        Up 4 hours                  0.0.0.0:27027->27017/tcp   mongors1n2
cfdea54a34c9        mongo               "docker-entrypoint..."   4 hours ago        Up 4 hours                  0.0.0.0:27017->27017/tcp   mongors1n1o


#ngrep
`
ngrep port 27020
 
 
interface: enp0s25 (xxx.xxx.xxx.xxx/255.255.255.255)
filter: ( port 27020 ) and ((ip || ip6) || (vlan && (ip || ip6)))
###
T xxx.xxx.xxx.xxx:60823 -> xxx.xxx.xxx.xxx:27020 [A]
  ......
#
T xxx.xxx.xxx.xxx:60823 -> xxx.xxx.xxx.xxx:27020 [AP]
  O...................admin.$cmd.........(....isMaster......client......driver.C....name.....mongoc / ext-mongodb:PHP..version.....1.9.2 / 1.4.0...os.Y....type.....Windows
  ..name.....Windows..version.....10.0 (17134)..architecture.....x86...platform.@...cfg=0x402a0e9 CC=MSVC 1912 CFLAGS="" LDFLAGS="" / PHP 7.2.1-dev...compression.......
##
T xxx.xxx.xxx.xxx:27020 -> xxx.xxx.xxx.xxx:60823 [AP]
  s...G...............................O....ismaster...msg.....isdbgrid..maxBsonObjectSize......maxMessageSizeBytes..l...maxWriteBatchSize......localTime.....d....logicalSe
  ssionTimeoutMinutes......maxWireVersion......minWireVersion......ok........?.operationTime.....C.b[.$clusterTime.X....clusterTime.....C.b[.signature.3....hash...........
  ................keyId............
 
 
 
 
ngrep port 27019
interface: enp0s25 (xxx.xxx.xxx.xxx/255.255.255.255)
filter: ( port 27019 ) and ((ip || ip6) || (vlan && (ip || ip6)))
####################################################################################
`
 
#위와같이 mongos1 라우터로는 접근을 시도하나 접속되지 않아 mongos2 라우터로 요청하는것을 확인할수있다.
{% endhighlight %}
#### 비고
##### hash mark (#) 는 ngrep 에서 요청이 왔을때 표시해줌 ngrep (-q is be quiet (don't print packet reception hash marks.)
[참조링크](https://dzone.com/articles/composing-a-sharded-mongodb-on-docker)
