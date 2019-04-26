---
layout: post
title: flatbuffer, protocolbuffer, messagepack 사용법
comments: true
---

나의 경우에 외부 라이브러리 사용법을 익힐때 테스트 케이스를 작성해서 사용법을 익힙니다.

토비의 스프링 이라는 책을 보면 이런 테스트를 학습 테스트 라고 부른다고 합니다.

[프로토콜버퍼](https://developers.google.com/protocol-buffers/),
[플랫버퍼](https://google.github.io/flatbuffers/),
[메시지팩](https://msgpack.org/)

을 php 에서 사용하기 위한 학습 테스트를 아주 간단하게 만들어 봤습니다.

[이전 포스팅](https://battlecook.github.io/2017/08/12/flatbuffer-cpu-usage-issue.html)

에서도 잠시 언급 했다 시피 플렛버퍼와 프로토콜버퍼는 멀티 플랫폼을 지원하는데 IDL(Interface Description Language) 이라고 불리는 스키마를 작성해 두고 여러 플랫폼의 코드를 생성 해서 사용합니다.

먼저 플랫버퍼 부터 살펴 봅시다.

플랫버퍼 IDL 에서 코드를 생성하는 방법은 [이전 포스팅](https://battlecook.github.io/2017/08/12/flatbuffer-cpu-usage-issue.html) 을 참조 하면 됩니다.

flatbuffer IDL 파일

```
namespace flatProtocol;

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

flatbuffer 학습 테스트 코드

```
class FlatBufferTest extends TestCase
{
    public function testSerializeDeserialize()
    {
        //given
        $elapsedTime = 10;

        $fbb = new FlatbufferBuilder(1);
        $type = Packet::SyncTime;
        //--data
        SyncTime::startSyncTime($fbb);
        SyncTime::addElapsedTime($fbb, $elapsedTime);
        $dataOffset = SyncTime::endSyncTime($fbb);
        //--end of data
        MessageRoot::startMessageRoot($fbb);
        MessageRoot::addPacketType($fbb, $type);
        MessageRoot::addPacket($fbb, $dataOffset);
        $messageOffset = MessageRoot::endMessageRoot($fbb);
        $fbb->finish($messageOffset);

        //when
        $serializedData = $fbb->sizedByteArray();

        //then
        $byteBuffer  = ByteBuffer::wrap($serializedData);
        $messageRoot = MessageRoot::getRootAsMessageRoot($byteBuffer);

        $this->assertEquals($type, $messageRoot->getPacketType());

        $syncTime = new SyncTime();
        $packet = $messageRoot->getPacket($syncTime);
        $this->assertEquals($elapsedTime, $packet->getElapsedTime());
    }
}

```

다음으로 프로토콜버퍼를 살펴 봅시다.

내가 다운 받은 프로토콜 버퍼 버전은 3.5.1로 https://github.com/google/protobuf/releases/tag/v3.5.1 이 주소에서 아래 protoc-3.5.1-win32.zip 파일을 다운 받아서 압축을 풀어서 사용하였습니다.

protocolbuffer IDL 파일을 작성합니다.

```
syntax = "proto3";

package protoProtocol;

enum PacketType {
  SyncTimeType = 0;
}

message Packet {
  PacketType type = 1;
  float version = 2;

  oneof Packet {
    SyncTime SyncTime = 3;
  }
}

message SyncTime {
  int32 elapsedTime = 1;
}
```

protoc.exe 파일과 같은 경로에 protocol.proto 파일을 넣습니다.

![protocolbuffer_before screenshot]({{ site.url }}/assets/20180121/protocolbuffer_before.png)

아래의 명령어를 실행하면

```
protoc --php_out=./ protocol.proto
```

![protocolbuffer_after screenshot]({{ site.url }}/assets/20180121/protocolbuffer_after.png)

생성된 디렉토리 안을 보면 php 코드가 생성되어져 있는것을 확인 할 수 있습니다.

이제 학습 테스트 코드를 작성해 봅시다.

protocolbuffer 학습 테스트 코드

```
class ProtocolBufferTest extends TestCase
{
    public function testSerializeDeserialize()
    {
        //given
        $elapsedTime = 10;
        $type = PacketType::SyncTimeType;

        $syncTime = new SyncTime();
        $syncTime->setElapsedTime($elapsedTime);

        $requestPacket = new Packet();
        $requestPacket->setType($type);
        $requestPacket->setSyncTime($syncTime);

        //when
        $serializedPacket = $requestPacket->serializeToString();

        //then
        $packet = new Packet();
        $packet->mergeFromString($serializedPacket);

        $this->assertEquals($type, $packet->getType());
        $this->assertEquals($elapsedTime, $packet->getSyncTime()->getElapsedTime());
    }
}

```

마지막으로 메시지팩에 대해 살펴 봅시다.

메시지팩은 유명한 메모리 캐시 솔루션인 레디스에서 메시지를 주고받을때 사용중인 직렬화 포맷입니다.

굉장히 다양한 언어를 지원 하고 있습니다.

메시지팩의 경우 따로 IDL 을 작성할 필요가 없습니다.

MessagePack 학습 테스트 코드

```
class MessagePackTest extends TestCase
{
    public function testSerializeDeserialize()
    {
        //given
        $elapsedTime = 10;

        $type = 1;
        $packet = array('type'=> $type, 'SyncTime' => $elapsedTime);

        //when
        $msg = msgpack_pack($packet);

        //then
        $data = msgpack_unpack($msg);

        $this->assertEquals($type, $data['type']);
        $this->assertEquals($elapsedTime, $data['SyncTime']);
    }
}

```

3가지 직렬화 라이브러리 사용법을 알아보았습니다.

시간이 될 때 3가지 직렬화 라이브러리를 php 에서 사용시에 성능에 대한 테스트를 해봐야 겠습니다.