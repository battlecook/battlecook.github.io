---
layout: post
title:  프로그램을 Dockerfile 로 빌드시, 소스코드를 COPY 한 경우와 REPO 에서 클론한 경우의 동작 차이점
comments: true
---

**TL;DR ( Too Long, Didn't Read )**

```
프로그램을 Dockerfile 로 빌드시, 소스코드를 COPY 한 경우는 소스코드를 항상 빌드 해서 변경사항을 바로 적용 할 수 있다. 

하지만 REPO 에서 소스코드를 클론 받아서 빌드하는 경우는 클론하는 부분이 CACHED 되기 때문에 바로 적용이 되지 않는다. 

도커 빌드시 —no-cache 옵션을 줘서 해결 할 수 있다.
```


다음은 8080 포트로 요청이 들어오면 "Hello World!" 를 응답하는 http 서버 코드 입니다.

```go
package main

import (
	"net/http"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, req *http.Request) {
		w.Write([]byte("Hello World!"))
	})

	http.ListenAndServe(":8080", nil)
}
```

프로그램을 도커이미지로 만들어서 실행을 하는 경우에 소스코드를 Dockerfile 에 넣어 다음의 2가지 방법으로 빌드 할 수 있습니다.

1. 소스코드에 Dockerfile 을 넣고 현재 디렉토리를 COPY 명령어를 이용해 복사 후 빌드
2. Dockerfile 에 저장소의 코드를 clone 후 빌드

두 가지 경우를 도커파일로 빌드해 이미지를 만들어 보겠습니다.

**1. 소스코드에 Dockerfile 을 넣고 현재 디렉토리를 COPY 명령어를 이용해 복사 후 빌드**

Dockerfile

```docker
FROM golang:latest

ENV GO111MODULE=off

WORKDIR /go/src/app
COPY . /go/src/app

RUN go build -o simple_web_server

ENTRYPOINT ["/go/src/app/simple_web_server"]
```

참고로 ENV GO111MODULE=off 부분을 추가하지 않으면 아래와 같은 에러가 발생합니다.

```
> [5/5] RUN go build -o simple_web_server:
#8 0.463 go: cannot find main module, but found 
#8 0.463        to create a module there, run:
#8 0.463        go mod init
```

도커 빌드 명령어를 실행 해 이미지를 만들어 봅시다.

```docker
$ docker build -t simple_web_server .
[+] Building 3.0s (9/9) FINISHED
 => [internal] load build definition from Dockerfile
 => => transferring dockerfile: 1.04kB 
 => [internal] load .dockerignore 
 => => transferring context: 2B
 => [internal] load metadata for docker.io/library/golang:latest 
 => [1/4] FROM docker.io/library/golang:latest@sha256:6f0b0a314b158ff6caf8f12d7f6f3a966500ec6afb533e986eca7375e2f7560f
 => [internal] load build context 
 => => transferring context: 4.21kB 
 => CACHED [2/4] WORKDIR /go/src/app
 => [3/4] COPY . /go/src/app
 => [4/4] RUN go build -o simple_web_server
 => exporting to image
 => => exporting layers
 => => writing image sha256:76941c58aafa91537ece5133a2ea981c68c1d11ce62dc2f7279212262caf04a5 
 => => naming to docker.io/library/simple_web_server
```

도커 이미지가 잘 생성됐는지 확인합니다.

```docker
$ docker images;
REPOSITORY                 TAG              IMAGE ID           CREATED          SIZE
simple_web_server          latest           **8f825582151f**   45 seconds ago   868MB
```

생성된 이미지를 컨테이너로 실행하고 동작을 확인합니다.

```docker
$ docker run -d --rm -p 8080:8080 --name simple_web_server simple_web_server
d9c98853d ( 생성된 컨테이너 id ) 
$ curl localhost:8080
Hello World!
```

이제 소스코드를 아래와 같이 변경 후 위의 과정을 반복해 봅니다.

```go
package main

import (
	"net/http"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, req *http.Request) {
		w.Write([]byte("Hello World.")) // ! 를 . 로 변경합니다.
	})

	http.ListenAndServe(":8080", nil)
}
```

```docker
$ docker build -t simple_web_server .
[+] Building 3.2s (9/9) FINISHED
 => [internal] load build definition from Dockerfile
 => => transferring dockerfile: 32B
 => [internal] load .dockerignore
 => => transferring context: 2B
 => [internal] load metadata for docker.io/library/golang:latest
 => [1/4] FROM docker.io/library/golang:latest@sha256:6f0b0a314b158ff6caf8f12d7f6f3a966500ec6afb533e986eca7375e2f7560f
 => [internal] load build context
 => => transferring context: 3.40kB
 => CACHED [2/4] WORKDIR /go/src/app
 => [3/4] COPY . /go/src/app
 => [4/4] RUN go build -o simple_web_server
 => exporting to image
 => => exporting layers
 => => writing image sha256:80daeb738daf713b3cfc69d75476e6338ab567fcb0ffde04b120a583da3e8cac
 => => naming to docker.io/library/simple_web_server
```

```docker
$ docker images;
REPOSITORY                 TAG              IMAGE ID           CREATED          SIZE
simple_web_server          latest           **80daeb738daf**   38 seconds ago   868MB
```

잘 적용된 것을 확인 할 수 있습니다.

```docker
$ docker run -d --rm -p 8080:8080 --name simple_web_server simple_web_server
238c5cd35 ( 생성된 컨테이너 id ) 
$ curl localhost:8080
Hello World.   <<<<<<< 적용됨을 알 수 있음
```

**2. Dockerfile 에 저장소의 코드를 clone 후 빌드**

Dockerfile

```docker
FROM golang:latest

ENV GO111MODULE=off

WORKDIR /go/src/app
ARG USER
ARG PASSWORD
RUN git clone https://$USER:$PASSWORD@github.com/battlecook/프로그램.git

WORKDIR /go/src/app/프로그램/
RUN go build -o simple_web_server

ENTRYPOINT ["/go/src/app/프로그램/simple_web_server"]
```

저장소의 id 와 password 는 도커파일 빌드시 주입받도록 작성합니다.

( --build-arg 명령어를 사용하면 run 명령어 시 인자로 주입 할 수 있습니다. )

```
docker build --build-arg USER="저장소 아이디" --build-arg PASSWORD="저장소 비번" -t simple_web_server .
```

```
$ docker build --build-arg USER="저장소 아이디" --build-arg PASSWORD="저장소 비번" -t simple_web_server .
[+] Building 3.4s (9/9) FINISHED
 => [internal] load build definition from Dockerfile
 => => transferring dockerfile: 1.04kB
 => [internal] load .dockerignore
 => => transferring context: 2B
 => [internal] load metadata for docker.io/library/golang:latest
 => [1/5] FROM docker.io/library/golang:latest@sha256:6f0b0a314b158ff6caf8f12d7f6f3a966500ec6afb533e986eca7375e2f7560f
 => CACHED [2/5] WORKDIR /go/src/app
 => [3/5] RUN git clone https://아이디:비번@github.com/battlecook/프로그램.git
 => [4/5] WORKDIR /go/src/app/프로그램/
 => [5/5] RUN go build -o simple_web_server
 => exporting to image
 => => exporting layers
 => => writing image sha256:a12721d7905f7ff73a6efeb945a2cfc7dd8117ddade57786a0b32ba09cf01de3
 => => naming to docker.io/library/simple_web_server
```

```
$ docker images;
REPOSITORY                 TAG              IMAGE ID           CREATED         SIZE
simple_web_server          latest           a12721d7905f   3 minutes ago   868MB
```

```
$ docker run -d --rm -p 8080:8080 --name simple_web_server simple_web_server
2c81ce05226c05a30 ( 생성된 컨테이너 id )
$ curl localhost:8080
Hello World!
```

마찬가지로 코드를 변경하고 저장소에 push 한 후 다시 빌드해 보도록 합니다.

```
$ docker build --build-arg USER="저장소 아이디" --build-arg PASSWORD="저장소 비번" -t simple_web_server .
[+] Building 3.4s (9/9) FINISHED
 => [internal] load build definition from Dockerfile
 => => transferring dockerfile: 1.04kB
 => [internal] load .dockerignore
 => => transferring context: 2B
 => [internal] load metadata for docker.io/library/golang:latest
 => [1/5] FROM docker.io/library/golang:latest@sha256:6f0b0a314b158ff6caf8f12d7f6f3a966500ec6afb533e986eca7375e2f7560f
 => CACHED [2/5] WORKDIR /go/src/app
 => [3/5] RUN git clone https://아이디:비번@github.com/battlecook/프로그램.git
 => CACHED [4/5] WORKDIR /go/src/app/프로그램/
 => CACHED [5/5] RUN go build -o simple_web_server  <<<<<< 해당 라인이 CACHED 됨을 알 수 있음
 => exporting to image
 => => exporting layers
 => => writing image sha256:a12721d7905f7ff73a6efeb945a2cfc7dd8117ddade57786a0b32ba09cf01de3
 => => naming to docker.io/library/simple_web_server
```

go build 시에 CACHED 된 걸 사용하며 IMAGE ID 가 변경되지 않은걸 알 수 있습니다.

```
$ docker images;
REPOSITORY                 TAG              IMAGE ID       CREATED         SIZE
simple_web_server          latest           a12721d7905f   3 minutes ago   868MB
```

확인을 해보면 여전히 이전 코드로 실행됨을 알 수 있습니다.

```
$ curl localhost:8080
Hello World!
```

해결책 중 한가지는 go build 시에 -no-cache 옵션을 주는 방법이 있습니다.

```
$ docker build --build-arg USER="저장소 아이디" --build-arg PASSWORD="저장소 비번" -t simple_web_server . --no-cache
[+] Building 3.4s (9/9) FINISHED
 => [internal] load build definition from Dockerfile
 => => transferring dockerfile: 1.04kB
 => [internal] load .dockerignore
 => => transferring context: 2B
 => [internal] load metadata for docker.io/library/golang:latest
 => [1/5] FROM docker.io/library/golang:latest@sha256:6f0b0a314b158ff6caf8f12d7f6f3a966500ec6afb533e986eca7375e2f7560f
 => CACHED [2/5] WORKDIR /go/src/app
 => [3/5] RUN git clone https://아이디:비번@github.com/battlecook/프로그램.git
 => [4/5] WORKDIR /go/src/app/프로그램/
 => [5/5] RUN go build -o simple_web_server <<<<<< 해당 라인이 CACHE 된 걸 사용하지 않음
 => exporting to image
 => => exporting layers
 => => writing image sha256:36be0be7d2373152463a222cc2c035ae70077443ed77060d0835d6ee948eb68a
 => => naming to docker.io/library/simple_web_server
```

IMAGE ID 가 변경 된 걸 확인 할 수 있습니다.

```
$ docker images
REPOSITORY                 TAG              IMAGE ID       CREATED          SIZE
simple_web_server          latest           36be0be7d237   2 minutes ago    868MB
```

바뀐 코드가 적용된 걸 확인 할 수 있습니다.

```
$  docker run -d --rm -p 8080:8080 --name simple_web_server simple_web_server
aa4ef000dc8bc67f3b0e0c ( 생성된 컨테이너 id )
$ curl localhost:8080
Hello World.
```