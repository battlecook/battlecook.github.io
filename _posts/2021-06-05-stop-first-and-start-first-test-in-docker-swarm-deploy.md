---
layout: post
title:  도커스웜 배포시 stop-first, start-first 에 따른 동작 테스트 및 종료 시그널과 우아한 종료
comments: true
---

도커스웜 배포 준비를 합니다. 배포는 docker private registry 를 이용합니다.

docker private registry 를 설치합니다.

```
docker run -d --rm -v /opt/docker_registry:/var/lib/registry/docker/registry/v2  -p 5000:5000  --name docker_registry registry:2
```

volume 을 위한 디렉토리는 원하는 곳에 해주시면 됩니다.

—rm 옵션을 사용하면 컨테이너가 종료될때 삭제가 됩니다.

테스트에 사용할 서버코드, 클라이언트코드, Dockerfile, docker-compose 파일을 준비합니다. 테스트에는 echo 프레임웍을 사용하도록 하겠습니다.

main.go ( 서버 코드 )

```go
package main

import (
	"github.com/labstack/echo"
	"github.com/labstack/gommon/log"
	"net/http"
)

func main() {

	e := echo.New()
	e.Logger.SetLevel(log.INFO)
	e.GET("/", func(c echo.Context) error {
		return c.String(http.StatusOK, "old")
	})

	if err := e.Start(":8080"); err != nil && err != http.ErrServerClosed {
		e.Logger.Fatal("shutting down the server")
	}
}
```

client.go

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	done := make(chan bool)
	for i := 0; i < 10; i++ {
		go func() {

			for {
				resp, err := http.Get("http://127.0.0.1:8080")
				if err != nil {
					fmt.Println("request error : " + err.Error())
				}

				data, httpError := ioutil.ReadAll(resp.Body)
				if httpError != nil {
					fmt.Println("http error : " + httpError.Error())
				} else {
					fmt.Printf("%s\n", string(data))
				}
			}
		}()
	}
	<-done
}
```

Dockerfile

```docker
FROM golang:latest

WORKDIR /go/src/app
COPY . /go/src/app

RUN go mod vendor
RUN go build -o simple_program

ENTRYPOINT ["/go/src/app/simple_program"]
```

docker-compose.yml

```docker
version: '3.8'

services:

  simple_program:
    image: 127.0.0.1:5000/simple_program
    deploy:
      update_config:
        delay: 3s
        order: stop-first
    ports:
      - target: 8080
        published: 8080
```

이미지를 빌드해 준후 레포지터리에 푸쉬합니다.

curl 명령어를 사용하여 레지스트리에 잘 업로드 됐는지 확인합니다.

```docker
$ curl -X GET http://127.0.0.1:5000/v2/_catalog
{"repositories":["simple_echo_server"]}

$ curl -X GET http://127.0.0.1:5000/v2/simple_echo_server/tags/list
{"name":"simple_echo_server","tags":["latest"]}
```

배포해봅니다.

```docker
$ docker stack deploy -c docker-compose.yml test
Creating network test_default
Creating service test_simple_program

$ docker stack ls
NAME      SERVICES   ORCHESTRATOR
test      1          Swarm

$ docker service ls
ID             NAME                  MODE         REPLICAS   IMAGE                                      PORTS
2gv5d6czhkv8   test_simple_program   replicated   1/1        127.0.0.1:5000/simple_echo_server:latest   *:8080->8080/tcp

$ curl localhost:8080
old program
```

**테스트 : 도커 배포 시 stop-first 와 start-first 옵션에 따른 동작 확인**

docker stack deploy 시 기본정책은 stop-first 입니다. stop-first 는 실행중인 컨테이너를 stop 시키고 새로운 컨테이너를 생성합니다.

먼저 stop-first 정책으로 client 를 실행시키는 중에 서버코드를 배포해 봅시다.

서버 소스코드는 다음과 같이 수정합니다.

```go
e.GET("/", func(c echo.Context) error {
		return c.String(http.StatusOK, "new program")
	})
```

실행 결과입니다.

```
old program
old program
old program
old program
old program
old program
old program
old program
old program
old program
old program
old program
old program
old program
request error : Get "http://127.0.0.1:8080": dial tcp 127.0.0.1:8080: connectex: Only one usage of each socket address (protocol/network address/port) is normally permitted.
panic: runtime error: invalid memory address or nil pointer dereference
[signal 0xc0000005 code=0x0 addr=0x40 pc=0x60c6f3]

goroutine 10 [running]:
main.main.func1()
	go/src/deploy_system/client/main.go:20 +0x73
created by main.main
	go/src/deploy_system/client/main.go:12 +0x70
```

프로그램이 죽었고 다시 실행을 해보면 바뀐 프로그램으로 응답을 하는것을 알 수 있습니다.

```
new program
new program
new program
new program
new program
new program
new program
new program
new program
new program
new program
new program
```

그럼 이제 start-first 로 배포해보도록 합시다.

docker-compose 파일을 수정합니다.

```docker
simple_program:
    image: 127.0.0.1:5000/simple_echo_server
    deploy:
      update_config:
        delay: 3s
        order: start-first
    ports:
      - target: 8080
        published: 8080
```

start-first 로 옵션을 바꾸고 배포를 하면 배포중에 replicas 가 일시적으로 2개가 되는 것을 확인할 수 있습니다.

```docker
Every 2.0s: docker service ls                                                                                      

ID             NAME                  MODE         REPLICAS   IMAGE                                      PORTS
lkygo8elsykw   test_simple_program   replicated   2/1        127.0.0.1:5000/simple_echo_server:latest   *:8080->8080/tcp
```

배포중 클라이언트 출력값 입니다.

```docker
new program
old program
old program
old program
old program
old program
new program
new program
new program
new program
new program
new program
new program
old program
old program
new program
old program
old program
```

배포 전 프로그램과 배포 후 프로그램이 모두 응답한 것을 알 수 있습니다.

**테스트 : 도커스웜 배포시 스웜에서 어플리케이션에 주는 인터럽트 시그널 확인**

[https://docs.docker.com/engine/reference/commandline/stop/](https://docs.docker.com/engine/reference/commandline/stop/)

해당 문서를 보면 docker stop 시에 컨테이너에 SIGTERM 시그널을 보내는 것을 알 수 있습니다.

```text
The main process inside the container will receive SIGTERM, and after a grace period, SIGKILL. 
The first signal can be changed with the STOPSIGNAL instruction in the container’s Dockerfile, or the --stop-signal option to docker run.
```
docker stack deploy 시에 도커에서보내는 시그널을 확인해 봅시다.

시그널을 확인할 수 있는 코드를 작성 합니다.

```go
package main

import (
	"fmt"
	"os"
	"os/signal"
	"syscall"
)

func main() {
	fmt.Println("simple program")
	signalChannel := make(chan os.Signal, 1)
	signal.Notify(signalChannel, os.Interrupt, syscall.SIGTERM, syscall.SIGKILL)

	sig := <-signalChannel
	fmt.Println("sig type : " + sig.String())
}
```

stop-first 로 배포시 로그 출력값 입니다.

```docker
$ docker service logs test_signal_simple_program
test_signal_simple_program.1.jat4pb6hb0rf@docker-desktop    | simple program old
test_signal_simple_program.1.jat4pb6hb0rf@docker-desktop    | sig type : terminated
test_signal_simple_program.1.f8fpxg3yx5y2@docker-desktop    | simple program new
```

start-first 로 배포시 로그 출력값 입니다.

```docker
$ docker service logs test_signal_simple_program
test_signal_simple_program.1.f8fpxg3yx5y2@docker-desktop    | simple program new
test_signal_simple_program.1.jat4pb6hb0rf@docker-desktop    | simple program old
test_signal_simple_program.1.jat4pb6hb0rf@docker-desktop    | sig type : terminated
```

문서에 나와있는 대로 terminate 시그널을 보내는 것을 알 수 있습니다.

**우아한 종료**

위의 2가지 테스트를 진행하면서 생각해보면 컨테이너에 terminate 시그널을 보내게 된다면 서버의 경우 들어온 요청을 처리해줘야 합니다.

golang 에코프레임웍 깃헙 이슈에 보면 사용자가 시그널이 들어왔을때 적절히 종료하지 않는다는 문의가 있습니다. 

답변에 보면 우아한 종료를 제공한다는 답변을 볼수 있습니다.

[https://github.com/labstack/echo/issues/1067](https://github.com/labstack/echo/issues/1067)

go언어의 echo 프레임웍 우아한 종료에 관한 문서는 아래의 문서를 참고하시면 됩니다.

[https://echo.labstack.com/cookbook/graceful-shutdown/](https://echo.labstack.com/cookbook/graceful-shutdown/)