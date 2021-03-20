---
layout: post
title:  wsl2 에서 레디스 클러스터 설치
comments: true
---

레디스 설치

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install redis-server
redis-cli -v
```

3대의 레디스에 대한 config 파일을 각각 만들어 줍니다.

레디스를 설치하면 기본적으로 /etc/redis/ 경로에 redis.conf 가 생성이 됩니다.

```
sudo cp /etc/redis/redis.conf /etc/redis/redis1.conf
sudo cp /etc/redis/redis.conf /etc/redis/redis2.conf
sudo cp /etc/redis/redis.conf /etc/redis/redis3.conf
```

redis.conf 파일을 개별로 만들어준 후

cluster-enabled 설정값을 yes 로 변경 합니다. ( 주석 해제 )
cluster-config-file 값은 redis1.conf, redis2.conf, redis3.conf 에서 각각 다르게 설정합니다.
cluster-config-file 값이 동일하면 아래와 같은 에러가 발생합니다.

```
Sorry, the cluster configuration file nodes.conf is already used by a different Redis Cluster node. Please make sure that different nodes use different cluster configuration files.
```

다음과 같이 3개의 레디스 설정값을 따로 만들어 줍니다.

redis1.conf

```
port 7001
clustered-enabled yes
cluster-config-file nodes-7001.conf 
```

redis2.conf

```
port 7002
clustered-enabled yes
cluster-config-file nodes-7002.conf 
```

redis3.conf

```
port 7003
clustered-enabled yes
cluster-config-file nodes-7003.conf 
```

레디스 실행

```
sudo redis-server /etc/redis/redis1.conf
sudo redis-server /etc/redis/redis2.conf
sudo redis-server /etc/redis/redis3.conf
```

클러스터 생성

```
redis-cli -p 7001 --cluster create 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003
```

```
>>> Performing hash slots allocation on 3 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
M: ab7e52889d09650296587c9df3915114d49070f8 127.0.0.1:7001
   slots:[0-5460] (5461 slots) master
M: 3471493cf7efad886642022cfce1867006eee48a 127.0.0.1:7002
   slots:[5461-10922] (5462 slots) master
M: 713094a030762cf66cf4e2a500ee7805e5764101 127.0.0.1:7003
   slots:[10923-16383] (5461 slots) master
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.
>>> Performing Cluster Check (using node 127.0.0.1:7001)
M: ab7e52889d09650296587c9df3915114d49070f8 127.0.0.1:7001
   slots:[0-5460] (5461 slots) master
M: 713094a030762cf66cf4e2a500ee7805e5764101 127.0.0.1:7003
   slots:[10923-16383] (5461 slots) master
M: 3471493cf7efad886642022cfce1867006eee48a 127.0.0.1:7002

   slots:[5461-10922] (5462 slots) master
[OK] All nodes agree about slots configuration.
```

클러스터 된 노드에 접속하기

```
redis-cli -c -p 7001
127.0.0.1:7001> cluster nodes
```

클러스터 정보 확인

```
127.0.0.1:7001> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:3
cluster_size:3
cluster_current_epoch:3
cluster_my_epoch:1
cluster_stats_messages_ping_sent:70
cluster_stats_messages_pong_sent:71
cluster_stats_messages_sent:141
cluster_stats_messages_ping_received:69
cluster_stats_messages_pong_received:70
cluster_stats_messages_meet_received:2
cluster_stats_messages_received:141
```

클러스터 노드 정보 확인

```
127.0.0.1:7001> cluster nodes
4aa9cfff772153b66d98fad2f86b3a04b0f6d9b1 127.0.0.1:7003@17003 master - 0 1616207961542 3 connected 10923-16383
54a7fb2f8614dd71eb1e5831ad6de815f257769f 127.0.0.1:7002@17002 master - 0 1616207962547 2 connected 5461-10922
95e280be7cb511ec6b06d8d4529c9d62c9dccba6 127.0.0.1:7001@17001 myself,master - 0 1616207959000 1 connected 0-5460
```



레디스 클러스터 테스트 golang 예제 코드

```go
package main

import (
	"encoding/json"
	"github.com/go-redis/redis"
	"log"
	"strings"
	"time"
)

type valueEx struct {
	Name  string
	Email string
}

var (
	client = &redisClusterClient{}
)

//RedisClusterClient struct
type redisClusterClient struct {
	c *redis.ClusterClient
}

//GetClient get the redis client
func initialize(hostnames string) *redisClusterClient {
	addr := strings.Split(hostnames, ",")
	c := redis.NewClusterClient(&redis.ClusterOptions{
		Addrs: addr,
	})
	if err := c.Ping().Err(); err != nil {
		panic("Unable to connect to redis " + err.Error())
	}
	client.c = c
	return client
}

//GetKey get key
func (client *redisClusterClient) getKey(key string, src interface{}) error {
	val, err := client.c.Get(key).Result()
	if err == redis.Nil || err != nil {
		return err
	}
	err = json.Unmarshal([]byte(val), &src)
	if err != nil {
		return err
	}
	return nil
}

//SetKey set key
func (client *redisClusterClient) setKey(key string, value interface{}, expiration time.Duration) error {
	cacheEntry, err := json.Marshal(value)
	if err != nil {
		return err
	}
	err = client.c.Set(key, cacheEntry, expiration).Err()
	if err != nil {
		return err
	}
	return nil
}

func main() {
	redisClusterClient := initialize("127.0.0.1:7001,127.0.0.1:7002,127.0.0.1:7003,")
	key1 := "sampleKey"
	value1 := &valueEx{Name: "someName", Email: "someemail@abc.com"}
	err := redisClusterClient.setKey(key1, value1, time.Minute*1)
	if err != nil {
		log.Fatalf("Error: %v", err.Error())
	}
	value2 := &valueEx{}
	err = redisClusterClient.getKey(key1, value2)
	if err != nil {
		log.Fatalf("Error: %v", err.Error())
	}
	log.Printf("Name: %s", value2.Name)
	log.Printf("Email: %s", value2.Email)
}
```

output

```
2021/03/19 14:20:28 Name: someName
2021/03/19 14:20:28 Email: someemail@abc.com
```

참고 

https://dev.to/adityakanekar/creating-local-redis-cluster-in-minutes-on-wsl2-k63

https://golangbyexample.com/golang-redis-cluster-client-example/