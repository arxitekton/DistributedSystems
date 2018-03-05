for this lab I used tutorial from
https://www.sohamkamani.com/blog/2016/06/30/docker-mongo-replica-set/

### Налаштувати реплікацію в конфігурації: Primary with Two Secondary Members (всі ноди мають бути запущені на різних фізичних/віртуальних машинах або Docker контейнерах) - 
```
docker pull mongo
```
![docker_pull_mongo](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/docker_pull_mongo.png)
```
docker network ls
```
![network_ls](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/network_ls.png)

adding a new network called lab6-mongo-cluster :
```
docker network create lab6-mongo-cluster
```
![lab6_mongo_cluster](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/lab6_mongo_cluster.png)


The new network should now be added to our list of networks :

```
docker network ls
```
![network_ls_next](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/network_ls_next.png)

To start up containers:
```
docker run -p 30001:27017 --name mongo1 --net lab6-mongo-cluster mongo mongod --replSet lab6-mongo-cluster
docker run -p 30002:27017 --name mongo2 --net lab6-mongo-cluster mongo mongod --replSet lab6-mongo-cluster
docker run -p 30003:27017 --name mongo3 --net lab6-mongo-cluster mongo mongod --replSet lab6-mongo-cluster
```

### Setting up replication

```
docker exec -it mongo1 mongo
```
first create DB:
```
db = (new Mongo('localhost:27017')).getDB('lab6')
```
![get db](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/db.png)

create our configuration :
```
config = {"_id" : "lab6-mongo-cluster", "members" : [{"_id" : 0,"host" : "mongo1:27017"},{"_id" : 1,"host" : "mongo2:27017"},{"_id" : 2,"host" : "mongo3:27017"}]}
```
Initiates the replica set:
```
rs.initiate(config)
```
![rs_initiate config](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/rs_initiate_config.png)

Let's check the current status of the replica set:

```
rs.status()
```
![rs_status](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/rs_status.png)


### Продемонструвати запис даних на primary node з різними Write Concern Levels (http://docs.mongodb.org/manual/core/write-concern/):
 - Unacknowledged
 - Acknowledged
 - Journaled
 - AcknowledgedReplica 

Unacknowledged:
```
db.inventory.insertOne(
   { item: "canvas", qty: 100, tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" } },
   { writeConcern: { w: 0} }
)
```
![unacknowledged](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/unacknowledged.png)

Acknowledged:
```
db.inventory.insertOne(
   { item: "journal", qty: 25, tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } },
   { writeConcern: { w: 1, wtimeout: 5000 } }
)
```
![acknowledged](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/acknowledged.png)

Journaled:
```
db.inventory.insertOne(
   { item: "mat", qty: 85, tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
   { writeConcern: { w: 1, j: true, wtimeout: 5000 } }
)
```
![journaled](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/journaled.png)

AcknowledgedReplica:
```
db.inventory.insertOne(
   { item: "mat", qty: 75, tags: ["blue"], size: { h: 27.9, w: 35.5, uom: "cm" } },
   { writeConcern: { w: 2, wtimeout: 5000 } }
)
```
![acknowledgedReplica](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/acknowledgedReplica.png)
 
### Продемонструвати Read Preference Modes: читання з primary і secondary node

читання з primary:
```
db.getMongo().setReadPref('primaryPreferred')
db.inventory.find()
```
![prim_primaryPreferred](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/prim_primaryPreferred.png)
```
db.getMongo().setReadPref('secondaryPreferred')
db.inventory.find()
```
![prim_secondaryPreferred](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/prim_secondaryPreferred.png)


читання з secondary node:
```
lab6-mongo-cluster:SECONDARY> db3 = (new Mongo('mongo2:27017')).getDB('lab6')
lab6-mongo-cluster:SECONDARY> db3.setSlaveOk()
```
```
db3.getMongo().setReadPref('primaryPreferred')
db3.inventory.find()
```
![sec_primaryPreferred](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/sec_primaryPreferred.png)

```
db3.getMongo().setReadPref('secondaryPreferred')
db3.inventory.find()
```
![sec_secondaryPreferred](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/sec_secondaryPreferred.png)

### Спробувати зробити запис з однією відключеною нодою та write concern рівнім 3 та нескінченім таймаутом. Спробувати під час таймаута включити відключену ноду 
```
docker container stop mongo3
```
![stop_mongo3](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/stop_mongo3.png)

після цього:
```
db.inventory.insertOne(
   { item: "mat", qty: 99, tags: ["green"], size: { h: 27.9, w: 35.5, uom: "cm" } },
   { writeConcern: { w: 3, wtimeout: 900000} }
)
```
![insert_w3_mongo3stop](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/insert_w3_mongo3stop.png)

після цього:
```
docker container start mongo3
```
отримуємо:
![insert_after_start_mongo3](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/insert_after_start_mongo3.png)

### Продемонстрував перевибори primary node в разі виходу з ладу поточної primary (Replica Set Elections) - 

Let's check how is master
```
rs.isMaster()
```
![check_how_is_master](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/check_how_is_master.png)

let's stop mongo1 (master):
```
docker container stop mongo1
```

```
rs.isMaster()
```
or
```
db2.isMaster()
```
![mongo2_new_master_after_election](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/mongo2_new_master_after_election.png)

```
db2.inventory.insertOne(
   { item: "mat", qty: 77, tags: ["yellow"], size: { h: 27.9, w: 35.5, uom: "cm" } },
   { writeConcern: { w: 1, wtimeout: 5000 } }
)
```
![insert_new_one](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/insert_new_one.png)

```
docker container start mongo1
```
 після відновлення роботи старої primary на неї реплікуються нові дані, які з'явилися під час її простою:
```
rs.slaveOk()
db.inventory.find()
```
![after_restart_find](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/after_restart_find.png)

### Привести кластер до неконсистентного стану користуючись моментом часу коли primary node не відразу помічає відсутність secondary node
```
docker container stop mongo1
```

```
db3.inventory.insertOne(
   { item: "zzz10", qty: 1, tags: ["red"], size: { h: 27.9, w: 35.5, uom: "cm" } }
)
```
```
docker container stop mongo3
```

```
docker container start mongo1
```
![restart_mongo1](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/restart_mongo1.png)

### Показати відмінності в поведінці між рівнями readConcern: {level: <"majority"|"local"| "linearizable">}
local:
```
docker container stop mongo3
```
```
db2.inventory.insertOne(
   { item: "q1", qty: 1, tags: ["red"], size: { h: 27.9, w: 35.5, uom: "cm" } }
)
```

lost mongo2, mongo1

```
docker container start mongo3
```

```
db3.inventory.find().readConcern("local")
```
mongo3 dont known nothing about item: "q1"

![mongo3_local_wo_q1](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/mongo3_local_wo_q1.png)

```
docker container start mongo1
```


linearizable when 3 node running:
![linearizable_when_3running](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/linearizable_when_3running.png)

when one node running:

![linearizable_when_one_running](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/linearizable_when_one_running.png)


The linearizable mode provides much stronger guarantees. once a write completes all later reads should return the value of that write or the value of a later write.

### Земулювати eventual consistency за допомогою установки затримки реплікації для репліки 

initial status:
![initial_status_before_delay](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/initial_status_before_delay.png)

check on the SECOND:
![second_before_delay](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/second_before_delay.png)

To set the delay:
```
cfg = rs.conf()
cfg.members[1].priority = 0
cfg.members[1].hidden = true
cfg.members[1].slaveDelay = 600
rs.reconfig(cfg)
```

insert new one item:
```
db.order.insertOne({ item: "c4", qty: 1 })
```
![insert_c4](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/insert_c4.png)

let's check on PRIMARY:
![c4_on_primary](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/c4_on_primary.png)

let' check on another SECONDARY:
![c4_on_mongo3.png](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/c4_on_mongo3.png)

let's check on 'delayed' SECONDARY:
![c4_on_delayed](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/c4_on_delayed.png)

and 10 minutes later on 'delayed' SECONDARY:
![10_minutes_later_on_delayed](https://github.com/arxitekton/DistributedSystems/blob/master/Lab_6_MongoDB_Replication/screenshot/10_minutes_later_on_delayed.png)

