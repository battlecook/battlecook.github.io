---
layout: post
title: google flatbuffer 의 cpu 사용량 이슈 
---

php 소켓 서버로 사용한 라이브러리 : <a href = "https://github.com/walkor/Workerman"> Workerman </a>


클라이언트와 데이터 통신시 사용한 직렬화 라이브러리 : <a href =  "https://google.github.io/flatbuffers/" > flatbuffers </a>

workerman, flatbuffer php 라이브러리 설치

composer.json

```
{
    "require": {
        "php": "^7.1.0",
        "workerman/workerman": "^3.3.9",
        "google/flatbuffers": "^1.5"
    }
}

```

을 만든후 아래의 명령어 실행

```
composer install
```

서버쪽 코드를 짜보자

server.php

```
<?php

require_once __DIR__ . '/vendor/autoload.php';

use Google\FlatBuffers\FlatbufferBuilder;
use Workerman\Worker;
use Workerman\Lib\Timer;

$worker = new Worker("tcp://0.0.0.0:1234");
$worker->count = 1; //프로세스는 한개만 올린다.
$worker->onWorkerStart = function($task)
{
    $timeInterval = 0.100; // 타이머는 0.1초로 한다.
    $timerId = Timer::add($timeInterval, function()
        {
            update();
        }
    );
};

function update()
{
    // 이곳에 방전체에 클라이언트와 시간 동기화를 하는 패킷을
    // 플랫버퍼로 주고받는 코드를 넣을 것이다.
}
Worker::runAll();
```

이코드를 실행후 cpu 사용량을 확인해 보았다.

```
php server.php start
```

위의 명령어를 실행한다.

![workerman start screenshot]({{ site.url }}/assets/workerman_start.png)

정상동작 하는 것을 볼수있다.

아마존에서 프리티어로 주는 ec2 에 위에 작성한 코드를 올린후 cpu 사용량을 측정해 보았다.

![workerman_cpu_usage screenshot]({{ site.url }}/assets/workerman_cpu_usage.png)

0.6% 정도 사용하는 것을 볼 수 있다.

이제 플렛버퍼를 사용해 보자.

플랫버퍼 설치는 윈도우 os 에서 binary 를 아래의 경로에서 다운받아서 실행하면 된다.

https://github.com/google/flatbuffers/releases

플렛버퍼는 멀티 플랫폼을 지원하는데

IDL(Interface Description Language) 이라고 불리는 스키마를 작성해 두고

여러 플랫폼의 코드를 생성 해서 사용한다.

예를들어 클라이언트로 유니티를 사용하고 서버로 php를 사용한다면

IDL을 작성하고 C# 코드와 php 코드를 생성해서 사용하는 방식이다.

간단하게 흐른 시간을 보내주는 SyncTime IDL 을 작성한후 php 코드를 생성해 보자.

protocol.fbs (idl 파일)

```
namespace protocol;

// Union containing all message types.
union Packet {
 SyncTime,
}

table MessageRoot {
 packet: Packet;
 version: float;
}

// Messages ----------------------------------------------------------------------- //

table SyncTime {
 elapsedTime: int;
}

//  End   -----------------------------------------------------------------------  //

root_type MessageRoot;
```

패킷 루트에 패킷타입과 버전을 붙인다. ( 물론 현재는 테스트기 때문에 의미가 있진 않다. )

바디에 SyncTime 패킷을 붙인다. SyncTime 패킷은 흐른 시간에 대한 정보만 넣었다.

플렛버퍼 바이너리 flatc 와 같은 경로에 protocol.fbs 파일을 넣는다

![flatbuffer_before screenshot]({{ site.url }}/assets/flatbuffer_before.png)

아래의 명령어를 실행하면

```
flatc --php protocol.fbs
```

![flatbuffer_after screenshot]({{ site.url }}/assets/flatbuffer_after.png)

protocol 디렉토리 안을 보면

![generated_php_code screenshot]({{ site.url }}/assets/generated_php_code.png)

다음과 같이 코드가 생성된것을 볼 수 있다.

이제 기존 서버 코드를 수정해 보자.

생성된 디렉토리 ( protocol 디렉토리 ) 전체를 기존 server.php 디렉토리에 같이 넣는다.

server.php 코드를 아래와 같이 수정해보자

```
<?php

require_once __DIR__ . '/vendor/autoload.php';

use Google\FlatBuffers\FlatbufferBuilder;
use protocol\MessageRoot;
use protocol\Packet;
use protocol\SyncTime;
use Workerman\Worker;
use Workerman\Lib\Timer;

$worker = new Worker("tcp://0.0.0.0:1234");
$worker->count = 1;
$worker->onWorkerStart = function($task)
{
    $timeInterval = 0.100;
    $timerId = Timer::add($timeInterval, function()
        {
            update();
        }
    );
};

function update()
{
    $roomCount = 100;
    $playerNum = 2;

    for($i=0; $i<$roomCount; $i++)
    {
        for($j=0; $j<$playerNum; $j++)
        {
            $elapsedTime = 10;

            $fbb = new FlatbufferBuilder(1);
            $responsePacketType = Packet::SyncTime;
            //--data
            SyncTime::startSyncTime($fbb);
            SyncTime::addElapsedTime($fbb, $elapsedTime);
            $dataOffset = SyncTime::endSyncTime($fbb);
            //--end of data
            MessageRoot::startMessageRoot($fbb);
            MessageRoot::addPacketType($fbb, $responsePacketType);
            MessageRoot::addPacket($fbb, $dataOffset);
            $messageOffset = MessageRoot::endMessageRoot($fbb);
            $fbb->finish($messageOffset);
            $body = $fbb->sizedByteArray();
        }
    }
}
Worker::runAll();
```

한서버에 모든 유져가 들어왔을경우 방이 100개 라고 가정하였다.

한방엔 최대 2명의 유져가 들어간다고 가정하였다.

마찬 가지로 위에 작성한 코드를 아마존에서 프리티어로 주는 ec2 에 올린후

CPU 사용량을 측정해 보았다.

![flatbuffer_cpu_usage screenshot]({{ site.url }}/assets/flatbuffer_cpu_usage.png)

7% 정도 사용하는 것을 볼 수 있다.

단지 흐른시간 데이터를 보냈을 뿐인데 7% cpu 사용량을 보여주고 있다.

생각한 것 보다는 꽤 많은 서버 리소스를 사용하고 있어서 사용을 하기 위해서는

조금 더 R&D 를 해 봐야 할 것 같다.



















