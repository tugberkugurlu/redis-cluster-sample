# Redis Cluster Example

With the help of docker, you can easily create a Redis cluster locally and directly work with it.

Note that Redis Cluster requires at least 3 master nodes. At least 6 nodes are required if we want to have one replica. Make sure to [assign the same IP address](https://stackoverflow.com/a/35359185/463785) to each node so that we get the cluster recovered from faiures as expected

```
docker pull redis
docker network create --subnet=172.18.0.0/16 red_cluster
docker run -d -v $PWD/cluster-config.conf:/usr/local/etc/redis/redis.conf --name redis-1 --net red_cluster --ip 172.18.0.2 redis redis-server /usr/local/etc/redis/redis.conf
docker run -d -v $PWD/cluster-config.conf:/usr/local/etc/redis/redis.conf --name redis-2 --net red_cluster --ip 172.18.0.3 redis redis-server /usr/local/etc/redis/redis.conf
docker run -d -v $PWD/cluster-config.conf:/usr/local/etc/redis/redis.conf --name redis-3 --net red_cluster --ip 172.18.0.4 redis redis-server /usr/local/etc/redis/redis.conf
docker run -d -v $PWD/cluster-config.conf:/usr/local/etc/redis/redis.conf --name redis-4 --net red_cluster --ip 172.18.0.5 redis redis-server /usr/local/etc/redis/redis.conf
docker run -d -v $PWD/cluster-config.conf:/usr/local/etc/redis/redis.conf --name redis-5 --net red_cluster --ip 172.18.0.6 redis redis-server /usr/local/etc/redis/redis.conf
docker run -d -v $PWD/cluster-config.conf:/usr/local/etc/redis/redis.conf --name redis-6 --net red_cluster --ip 172.18.0.7 redis redis-server /usr/local/etc/redis/redis.conf
```

Now we can see the running containers:

```
➜  redis-cluster git:(master) docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
aeb996679bb3        redis               "docker-entrypoint.s…"   23 hours ago        Up 23 hours         6379/tcp            redis-6
5fd4bf3b5270        redis               "docker-entrypoint.s…"   23 hours ago        Up 23 hours         6379/tcp            redis-5
6c7d1c749f98        redis               "docker-entrypoint.s…"   23 hours ago        Up 23 hours         6379/tcp            redis-4
40e5bc7e3532        redis               "docker-entrypoint.s…"   23 hours ago        Up 23 hours         6379/tcp            redis-3
192895ec8bb4        redis               "docker-entrypoint.s…"   23 hours ago        Up 23 hours         6379/tcp            redis-2
bea3645445a6        redis               "docker-entrypoint.s…"   23 hours ago        Up 23 hours         6379/tcp            redis-1
```

Wire it all up by issuing a command through redis-cli:

```
docker run -it --net red_cluster --rm redis redis-cli --cluster create 172.18.0.2:6379 172.18.0.3:6379 172.18.0.4:6379 172.18.0.5:6379 172.18.0.6:6379 172.18.0.7:6379 --cluster-replicas 1
```

The output from this is very interestimg and gives us an idea about how the data is going to be partitioned:

```
➜  redis-cluster git:(master) ✗ docker run -it --net red_cluster --rm redis redis-cli --cluster create 172.18.0.2:6379 172.18.0.3:6379 172.18.0.4:6379 172.18.0.5:6379 172.18.0.6:6379 172.18.0.7:6379 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 172.18.0.5:6379 to 172.18.0.2:6379
Adding replica 172.18.0.6:6379 to 172.18.0.3:6379
Adding replica 172.18.0.7:6379 to 172.18.0.4:6379
M: 4807a04d2c6025cd1a6ba6f713d33b979155f082 172.18.0.2:6379
   slots:[0-5460] (5461 slots) master
M: c78a52a4c76c5a87f1905144a54852f764091611 172.18.0.3:6379
   slots:[5461-10922] (5462 slots) master
M: f69af80088e17365d921de05d4f1ae14c8f03baa 172.18.0.4:6379
   slots:[10923-16383] (5461 slots) master
S: 0a7e89cde97d339e646279fa496e68de914777f2 172.18.0.5:6379
   replicates 4807a04d2c6025cd1a6ba6f713d33b979155f082
S: 3c302355697f5e7340dc4dee3016949f9b2d628d 172.18.0.6:6379
   replicates c78a52a4c76c5a87f1905144a54852f764091611
S: 4ada9ba420d16ef05ce9c41da211803289152c83 172.18.0.7:6379
   replicates f69af80088e17365d921de05d4f1ae14c8f03baa
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
....
>>> Performing Cluster Check (using node 172.18.0.2:6379)
M: 4807a04d2c6025cd1a6ba6f713d33b979155f082 172.18.0.2:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 0a7e89cde97d339e646279fa496e68de914777f2 172.18.0.5:6379
   slots: (0 slots) slave
   replicates 4807a04d2c6025cd1a6ba6f713d33b979155f082
S: 4ada9ba420d16ef05ce9c41da211803289152c83 172.18.0.7:6379
   slots: (0 slots) slave
   replicates f69af80088e17365d921de05d4f1ae14c8f03baa
M: c78a52a4c76c5a87f1905144a54852f764091611 172.18.0.3:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 3c302355697f5e7340dc4dee3016949f9b2d628d 172.18.0.6:6379
   slots: (0 slots) slave
   replicates c78a52a4c76c5a87f1905144a54852f764091611
M: f69af80088e17365d921de05d4f1ae14c8f03baa 172.18.0.4:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

There are also various CLUSTER commands which you can issue:

```
➜  redis-cluster docker exec redis-1 redis-cli cluster nodes
159ccdcd88a3b2042c4dd9bf3bb1bfb54f16cf35 172.20.0.4:6379@16379 master - 0 1552082268000 3 connected 10923-16383
aa75a42766edf123a8af89970965d0f41be40cf8 172.20.0.3:6379@16379 master - 0 1552082268000 2 connected 5461-10922
9d4e26c55c3a96b2921ed2476caf1fa78593e5d4 172.20.0.7:6379@16379 slave 159ccdcd88a3b2042c4dd9bf3bb1bfb54f16cf35 0 1552082269160 6 connected
e463eb0b118c3292687f78244988b38d4b220b40 172.20.0.6:6379@16379 slave aa75a42766edf123a8af89970965d0f41be40cf8 0 1552082269000 5 connected
11cce7fdb1a82e7937005a956ff7c7c9c3eb1410 172.20.0.5:6379@16379 slave 1ad5a315b83b55dfcacbf1ef02ccf684513151c9 0 1552082269574 4 connected
1ad5a315b83b55dfcacbf1ef02ccf684513151c9 172.20.0.2:6379@16379 myself,master - 0 1552082266000 1 connected 0-5460
```

One of the most interesting ones is the [CLUSTER SLOTS](https://redis.io/commands/cluster-slots) which clients can issue during initialization to figure out the cluster details:

```
➜  redis-cluster git:(master) ✗ docker exec redis-1 redis-cli CLUSTER SLOTS 
0
5460
172.18.0.2
6379
4807a04d2c6025cd1a6ba6f713d33b979155f082
172.18.0.5
6379
0a7e89cde97d339e646279fa496e68de914777f2
5461
10922
172.18.0.3
6379
c78a52a4c76c5a87f1905144a54852f764091611
172.18.0.6
6379
3c302355697f5e7340dc4dee3016949f9b2d628d
10923
16383
172.18.0.4
6379
f69af80088e17365d921de05d4f1ae14c8f03baa
172.18.0.7
6379
4ada9ba420d16ef05ce9c41da211803289152c83
```

## Client Example

From [the spec](https://redis.io/topics/cluster-spec):

> In Redis Cluster nodes don't proxy commands to the right node in charge for a given key, but instead they redirect clients to the right nodes serving a given portion of the key space. Eventually clients obtain an up-to-date representation of the cluster and which node serves which subset of keys, so during normal operations clients directly contact the right nodes in order to send a given command.

You can also read about the [key distribution model](https://redis.io/topics/cluster-spec#keys-distribution-model) to understand how redis distributes keys.

### redis-go-cluster

 - Handles [MOVED redirection responses](https://redis.io/topics/cluster-spec#moved-redirection) from Redis instance, see https://github.com/chasex/redis-go-cluster/blob/222d81891f1d3fa7cf8b5655020352c3e5b4ec0f/cluster.go#L254-L259

## Resources

 - https://redis.io/topics/cluster-spec
 - https://medium.com/commencis/creating-redis-cluster-using-docker-67f65545796d
 - https://redis.io/topics/cluster-tutorial
 - https://redis.io/topics/persistence
 - http://oldblog.antirez.com/post/redis-persistence-demystified.html
 - https://redis.io/commands/cluster-slots
 - https://github.com/chasex/redis-go-cluster
    - How the client initiates its state: https://github.com/chasex/redis-go-cluster/blob/222d81891f1d3fa7cf8b5655020352c3e5b4ec0f/cluster.go#L315