## Set up Sharding using Docker Containers

### Config servers
Start config servers (3 member replica set)
```
docker-compose -f config-server/docker-compose.yaml up -d
```
Initiate replica set
```
mongo mongodb://ip:27017
```
```
rs.initiate(
  {
    _id: "clustermongo",
    configsvr: true,
    members: [
      { _id : 0, host : "ip:27017" },
      { _id : 1, host : "ip:27018" },
      { _id : 2, host : "ip:27019" }
    ]
  }
)

rs.status()
```

### Shard 1 servers
Start shard 1 servers (3 member replicas set)
```
docker-compose -f shard1/docker-compose.yaml up -d
```
Initiate replica set
```
mongo mongodb://ip:50001
```
```
rs.initiate(
  {
    _id: "shardRepl",
    members: [
      { _id : 0, host : "ip:50001" },
      { _id : 1, host : "ip:50002" },
      { _id : 2, host : "ip:50003" }
    ]
  }
)

rs.status()
```

### Mongos Router
Start mongos query router
```
docker-compose -f mongos/docker-compose.yaml up -d
```

### Add shard to the cluster
Connect to mongos
```
mongo mongodb://ip:27020
```
Add shard
```
mongos> sh.addShard("shardRepl/ip:50001, ip:50002, ip:50003")
mongos> sh.status()
```
## Adding another shard
### Shard 2 servers
Start shard 2 servers (3 member replicas set)
```
docker-compose -f shard2/docker-compose.yaml up -d
```
Initiate replica set
```
mongo mongodb://ip:50004
```
```
rs.initiate(
  {
    _id: "shard2Repl",
    members: [
      { _id : 0, host : "ip:50004" },
      { _id : 1, host : "ip:50005" },
      { _id : 2, host : "ip:50006" }
    ]
  }
)

rs.status()
```
### Add shard to the cluster
Connect to mongos
```
mongo mongodb://ip:27020
```
Add shard
```
mongos> sh.addShard("shard2Repl/ip:50004, ip:50005, ip:50006")
mongos> sh.status()
```



¿How to config Shard Collection?

### Connect to Mongos Server.
mongo mongodb://ip:27020 

### Create Database.
mongos> use dbtest

### Create Collection.
mongos> db.createCollection("movies")

### Enable sharding
mongos> sh.enableSharding("dbtest")

### Select a Key with High Cardinality and Shard a Collection
mongos> sh.ShardCollection("dbtest.movies", {"Key" : "hashed"})


### Check this with -> ###

### Run on bash >
root> for i in {1..50} ; do echo -e "use sharddemo \n db.movies.insertOne({\"title\": \"Spiderman $i\", \"Lenguage\": \"English\"})" | mongosh mongodb://ip:27020; done

### Run on mongos
mongos> db.movies.getShardDistribution()


