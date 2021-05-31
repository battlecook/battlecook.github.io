---
layout: post
title:  도커 스웜 배포 및 노드 라벨링
comments: true
---


도커스웜을 이용해 배포하는 방법과 배포시 라벨 등록을 이용하는 경우에 대한 설명입니다.

**스웜 생성 및 노드간 ingress network 형성**

일단 도커스웜을 만들어 줍니다.

```
$ docker swarm init
Swarm initialized: current node (d4phl1t6rj5kvcpmezngrua9d) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-510716h1vbba

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

$ docker node ls
ID                            HOSTNAME         STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
d4phl1t6rj5kvcpmezngrua9d *   docker-desktop   Ready     Active         Leader           20.10.6
```

물리서버 1대당 node 가 1대라고 생각하시면 됩니다.  도커스웜은 manager node 와 worker node 로 구성됩니다. init 명령어를 사용하여 한대를  생성하였기 때문에 manager node 한대가 만들어집니다.

간단한 http 서버를 하나 만들고 docker build 로 이미지를 만듭니다.

```go
package main

import (
	"net/http"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, req *http.Request) {
		w.Write([]byte("Hello World!"))
	})

	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		return
	}
}
```

```docker
FROM golang:latest

ENV GO111MODULE=off

WORKDIR /go/src/app
COPY . /go/src/app

RUN go build -o simple_program

ENTRYPOINT ["/go/src/app/simple_program"]
```

```
docker build -t simple_program .
```

도커로 배포시 deploy mode 는 global 과 replicated 가 있습니다. global 은 노드 한대당 한개의 컨테이너를 올립니다 replicated 는 컨테이너 개수를 조절할 수 있습니다.

예를들어 현재 노드가 1개인 상황에서 global 로 지정한다면 simple_program 컨테이너를 1개 밖에 생성하지 못하지만 replicated 로 설정한다면 여러대를 생성할 수 있습니다.

docker-compose.yml

```docker
version: '3.7'

services:
  simple_program:
    image: simple_program
    deploy:
      mode: replicated
      replicas: 2

    ports:
      - 8080:8080
```

```
$ docker stack deploy --compose-file docker-compose.yml local_deploy_stack
Creating network local_deploy_stack_default
Creating service local_deploy_stack_simple_program

$ docker service ls
ID             NAME                                MODE         REPLICAS   IMAGE                   PORTS
o40ngptp1epi   local_deploy_stack_simple_program   replicated   2/2        simple_program:latest   *:8080->8080/tcp

$ docker ps
CONTAINER ID   IMAGE                   COMMAND                  CREATED          STATUS          PORTS     NAMES
a6dd528dfd5a   simple_program:latest   "/go/src/app/simple_…"   21 seconds ago   Up 19 seconds             local_deploy_stack_simple_program.2.qpnrss8vnj1nz3r291gpdv79r
7d05a56a280b   simple_program:latest   "/go/src/app/simple_…"   21 seconds ago   Up 19 seconds             local_deploy_stack_simple_program.1.yl8hbedpk7wxatrvi90wxaepc
```

컨테이너가 2대 생성된걸 알 수 있습니다. 그러면 외부에서 접근시 어떤 컨테이너로 갈지 궁금하실수 있는데요.

[https://docs.docker.com/engine/swarm/ingress/](https://docs.docker.com/engine/swarm/ingress/)



해당 문서를 보면 스웜에서 노드끼리 내부네트워크를 형성하고 로드밸런스가 있다는것을 알 수 있습니다.

![ingress network]({{ site.url }}/assets/20210530/ingress_network.png)

관련된 stackoverflow 질문도 추가합니다.

[https://stackoverflow.com/questions/42510944/how-is-load-balancing-done-in-docker-swarm-mode](https://stackoverflow.com/questions/42510944/how-is-load-balancing-done-in-docker-swarm-mode)

만약에 외부 라우터를 사용한다면 endpoint_mode 값을 dns 라운드로빈으로 설정을 할 수도 있습니다. 

endpoint_mode 값은 디폴트가 vip 이고 vip, dnsrr 두개 설정값을 가집니다.

```docker
deploy:
      mode: replicated
      replicas: 2
      endpoint_mode: dnsrr 
```

**도커스웜 노드간 포트 공유**

ingress network 로 형성이 되어있기때문에 포트를 binding 하게 되면 컨테이너가 떠있지 않은 node 에서도 해당 포트는 사용 할 수 없습니다.

예를들어 node 1, node 2 를 스웜으로 형성하고 node 1 에는 8000번 포트 node 2 에는 9000번 포트를 사용하고 두포트를 host 포트와 binding 을 시킨다면 node 1에서 9000번 포트를 사용할수 없고 node 2 에서 8000번 포트를 사용할 수 없습니다.

제가 했던 실수는 node 1, node 2 에 레디스를 네이티브로 설치해두었었는데 node 1 에 컨테이너로 레디스를 설치한 순간 port already in use 에러가 떴습니다.

**도커 스웜 노드 라벨링**

개별 도커노드들을 서버 종류에 따라서 배포하고 싶을 수 있습니다.

이를테면 api 서버, redis 서버 등등 으로 구분하고 배포하고 싶을 수 있는데 이때 사용할 수 있는 방법 중 하나는 도커 노드들에 라벨을 붙이는 것 입니다.

라벨 추가

docker node update --label-add 라벨key=라벨value 호스트명

ex) docker node update --label-add server_type=api docker-desktop

라벨 삭제

docker node update --label-rm 라벨key 호스트명

ex) docker node update --label-rm server_type docker-desktop

라벨링이 잘 되었는지 확인을 하려면  아래 명령어를 사용하면 됩니다.

docker node inspect 호스트명 --pretty

```
$ docker node update --label-add server_type=api docker-desktop
docker-desktop

$ docker node inspect docker-desktop --pretty
ID:                     d4phl1t
Labels:
 - server_type=api
Hostname:               docker-desktop
```

라벨링이 되어있다면 아래 명령어를 이용해 배포할 노드에 원하는 종류의 서버를 배포할 수 있습니다.

```
version: '3.7'

services:
  api_server:
    image: simple_program
    deploy:
      mode: replicated
      replicas: 2
      placement:
        constraints:
          - node.labels.server_type == api
    ports:
      - 8080:8080
```

만약 mode 를 global 로 한다면 server_type 이 api 인 노드들에 각각 한대씩 배포하게 됩니다.

```
deploy:
      mode: global
      placement:
        constraints:
          - node.labels.server_type == api
```

한가지 노드에 2가지 이상의 역활을 맡기는 경우에 노드들에 라벨링을 여러개 줍니다.

```
$ docker node update --label-add aggregator_role=true docker-desktop
docker-desktop

$ docker node update --label-add log_server_role=true docker-desktop
docker-desktop

$  docker node inspect docker-desktop --pretty
ID:                     d4phl1t6rj5kvcpmezngrua9d
Labels:
 - aggregator_role=true
 - log_server_role=true
```

이런식으로 라벨링을 하면 한 노드에 여러가지 역할을 부여할 수 있습니다.


constraints 을 여러개 걸었을 경우 모두 만족하는 node 에만 배포 됩니다.

```
deploy:
      mode: global
      placement:
        constraints:
          - node.labels.aggregator_role=true
          - node.labels.log_server_role=true
```

위와 같이 2가지 제약조건을 건다면 aggregator_role 도 true 이고 log_server_role 도 true 인 node 에만 배포하게 됩니다.

처음에 조건이 and 가 아닌 or 라고 생각해서 stackoverflow 에 질문을 올렸었는데 docs 에 모든조건 만족이라고 나와있다고 친절한 답변을 받았습니다. ( docs 를 좀 더 꼼꼼히 읽어봤어야 했네요 )

[https://stackoverflow.com/questions/67761268/docker-swarm-a-problem-that-does-not-work-when-deploying-with-multiple-labels-i/67762721#67762721](https://stackoverflow.com/questions/67761268/docker-swarm-a-problem-that-does-not-work-when-deploying-with-multiple-labels-i/67762721#67762721)