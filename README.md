# Redis Cluster Example

With the help of docker, you can easily create a Redis cluster locally and directly work with it.

Note that Redis Cluster requires at least 3 master nodes. At least 6 nodes are required if we want to have one replica.

```
docker pull redis
docker network create red_cluster
docker run -d -v $PWD/cluster-config.conf:/usr/local/etc/redis/redis.conf --name redis-1 --net red_cluster redis redis-server /usr/local/etc/redis/redis.conf
docker run -d -v $PWD/cluster-config.conf:/usr/local/etc/redis/redis.conf --name redis-2 --net red_cluster redis redis-server /usr/local/etc/redis/redis.conf
docker run -d -v $PWD/cluster-config.conf:/usr/local/etc/redis/redis.conf --name redis-3 --net red_cluster redis redis-server /usr/local/etc/redis/redis.conf
docker run -d -v $PWD/cluster-config.conf:/usr/local/etc/redis/redis.conf --name redis-4 --net red_cluster redis redis-server /usr/local/etc/redis/redis.conf
docker run -d -v $PWD/cluster-config.conf:/usr/local/etc/redis/redis.conf --name redis-5 --net red_cluster redis redis-server /usr/local/etc/redis/redis.conf
docker run -d -v $PWD/cluster-config.conf:/usr/local/etc/redis/redis.conf --name redis-6 --net red_cluster redis redis-server /usr/local/etc/redis/redis.conf
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

Get all of the container IP addresses within the network

```
docker inspect -f '{{ (index .NetworkSettings.Networks "red_cluster").IPAddress }}' redis-1
docker inspect -f '{{ (index .NetworkSettings.Networks "red_cluster").IPAddress }}' redis-2
docker inspect -f '{{ (index .NetworkSettings.Networks "red_cluster").IPAddress }}' redis-3
docker inspect -f '{{ (index .NetworkSettings.Networks "red_cluster").IPAddress }}' redis-4
docker inspect -f '{{ (index .NetworkSettings.Networks "red_cluster").IPAddress }}' redis-5
docker inspect -f '{{ (index .NetworkSettings.Networks "red_cluster").IPAddress }}' redis-6
```

Wire it all up by issuing a command through redis-cli:

```
docker run -it --net red_cluster --rm redis redis-cli --cluster create 172.20.0.2:6379 172.20.0.3:6379 172.20.0.4:6379 172.20.0.5:6379 172.20.0.6:6379 172.20.0.7:6379 --cluster-replicas 1
```

The output from this is very interestimg and gives us an idea about how the data is going to be partitioned:

```
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 172.20.0.5:6379 to 172.20.0.2:6379
Adding replica 172.20.0.6:6379 to 172.20.0.3:6379
Adding replica 172.20.0.7:6379 to 172.20.0.4:6379
M: 1ad5a315b83b55dfcacbf1ef02ccf684513151c9 172.20.0.2:6379
   slots:[0-5460] (5461 slots) master
M: aa75a42766edf123a8af89970965d0f41be40cf8 172.20.0.3:6379
   slots:[5461-10922] (5462 slots) master
M: 159ccdcd88a3b2042c4dd9bf3bb1bfb54f16cf35 172.20.0.4:6379
   slots:[10923-16383] (5461 slots) master
S: 11cce7fdb1a82e7937005a956ff7c7c9c3eb1410 172.20.0.5:6379
   replicates 1ad5a315b83b55dfcacbf1ef02ccf684513151c9
S: e463eb0b118c3292687f78244988b38d4b220b40 172.20.0.6:6379
   replicates aa75a42766edf123a8af89970965d0f41be40cf8
S: 9d4e26c55c3a96b2921ed2476caf1fa78593e5d4 172.20.0.7:6379
   replicates 159ccdcd88a3b2042c4dd9bf3bb1bfb54f16cf35
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
...
>>> Performing Cluster Check (using node 172.20.0.2:6379)
M: 1ad5a315b83b55dfcacbf1ef02ccf684513151c9 172.20.0.2:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 159ccdcd88a3b2042c4dd9bf3bb1bfb54f16cf35 172.20.0.4:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: aa75a42766edf123a8af89970965d0f41be40cf8 172.20.0.3:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 9d4e26c55c3a96b2921ed2476caf1fa78593e5d4 172.20.0.7:6379
   slots: (0 slots) slave
   replicates 159ccdcd88a3b2042c4dd9bf3bb1bfb54f16cf35
S: e463eb0b118c3292687f78244988b38d4b220b40 172.20.0.6:6379
   slots: (0 slots) slave
   replicates aa75a42766edf123a8af89970965d0f41be40cf8
S: 11cce7fdb1a82e7937005a956ff7c7c9c3eb1410 172.20.0.5:6379
   slots: (0 slots) slave
   replicates 1ad5a315b83b55dfcacbf1ef02ccf684513151c9
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
➜  redis-cluster docker exec redis-1 redis-cli CLUSTER SLOTS
10923
16383
172.20.0.4
6379
159ccdcd88a3b2042c4dd9bf3bb1bfb54f16cf35
172.20.0.7
6379
9d4e26c55c3a96b2921ed2476caf1fa78593e5d4
5461
10922
172.20.0.3
6379
aa75a42766edf123a8af89970965d0f41be40cf8
172.20.0.6
6379
e463eb0b118c3292687f78244988b38d4b220b40
0
5460
172.20.0.2
6379
1ad5a315b83b55dfcacbf1ef02ccf684513151c9
172.20.0.5
6379
11cce7fdb1a82e7937005a956ff7c7c9c3eb1410
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