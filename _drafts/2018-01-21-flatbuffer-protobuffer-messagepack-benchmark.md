---
layout: post
title: flatbuffer, protocolbuffer, messagepack 성능 비교
---

이 포스팅을 읽기전에 이전 두 포스팅을 읽는 것을 권장한다.

[플랫버퍼 cpu 사용량 이슈](https://battlecook.github.io/2017/08/12/flatbuffer-cpu-usage-issue.html)

[플랫버퍼,프로토콜버퍼,메시지펙 사용법](https://battlecook.github.io/2018/01/21/how-to-use-flatbuffer-protobuffer-messagepack.html)

2가지로 나눠서 성능을 비교해보려고 한다.

1) 얼마나 패킷 크기를 줄일수 있는가 ( 패킷이 작을수록 레이턴시가 짧아진다. )

2) 직렬화/역직렬화 시 서버의 자원을 얼마나 먹는가 ( cpu 사용량 혹은 메모리 )

일단 어떤식으로 측정을 해볼까 하다가

실제 사용될 법한 꽤 패킷을 하나 만들어서 3가지 라이브러리로 위의 1), 2) 번을 측정 해보려고 한다.

특정 방에 들어갔을 때 기존 유져들에게 이동정보를 주는 상황을 가정해 봤다.

1)번을 먼저 측정해 보기로 했다.

플랫버퍼 IDL 은 다음과 같이 작성하였다.

```
namespace flatbench;

struct Point3 {
 x: float;
 y: float;
 z: float;
}

struct Vector3 {
 x: float;
 y: float;
 z: float;
}

struct Vector4 {
 x: float;
 y: float;
 z: float;
 w: float;
}

table Movement {
 id: int;
 position: Point3;
 velocity: Vector3;
 rotation: Vector4;
 sentTime: int;
}

// Union containing all message types.
union Packet {
 JoinRoom,
}

table MessageRoot {
 packet: Packet;
 version: float;
}

// Messages ----------------------------------------------------------------------- //

table JoinRoom {
 moveList : [Movement];
}

//  End   -----------------------------------------------------------------------  //

root_type MessageRoot;
```

프로토콜버퍼의 IDL 은 다은과 같이 작성하였다.

```
syntax = "proto3";

package protobench;

enum PacketType {
  JoinRoomType = 0;
}

message Packet {
  PacketType type = 1;
  float version = 2;

  oneof Packet {
    JoinRoom JoinRoom = 3;
  }
}

message Point3 {
  float x = 1;
  float y = 2;
  float z = 3;
}

message Vector3 {
  float x = 1;
  float y = 2;
  float z = 3;
}

message Vector4 {
  float x = 1;
  float y = 2;
  float z = 3;
  float w = 4;
}

message Movement {
  int32 id = 1;
  Point3 position = 2;
  Vector3 velocity = 3;
  Vector4 rotation = 4;
  int32 sentTime = 5;
}

message JoinRoom {
  repeated Movement moveList = 1;
}
```


```
<?php

use flatbench\MessageRoot;
use Google\FlatBuffers\FlatbufferBuilder;

use Protobench\JoinRoom;
use Protobench\Movement;
use Protobench\Packet;
use Protobench\PacketType;
use Protobench\Point3;
use Protobench\Vector3;
use Protobench\Vector4;

require 'vendor/autoload.php';


//protocol buffer large data

$sentTime = 10;
$movement = new Movement();
$movement->setSentTime($sentTime);

$movementList = array();
for($id=0; $id<20; $id++)
{
    $position1 = mt_rand(-100, 100);
    $position2 = mt_rand(-100, 100);
    $position3 = mt_rand(-100, 100);

    $velocity1 = mt_rand(-100, 100);
    $velocity2 = mt_rand(-100, 100);
    $velocity3 = mt_rand(-100, 100);

    $rotation1 = mt_rand(-100, 100);
    $rotation2 = mt_rand(-100, 100);
    $rotation3 = mt_rand(-100, 100);
    $rotation4 = mt_rand(-100, 100);

    $vector4 = new Vector4();
    $vector4->setX($rotation1);
    $vector4->setY($rotation2);
    $vector4->setZ($rotation3);
    $vector4->setW($rotation4);

    $vector3 = new Vector3();
    $vector3->setX($velocity1);
    $vector3->setY($velocity2);
    $vector3->setZ($velocity3);

    $point3 = new Point3();
    $point3->setX($position1);
    $point3->setY($position2);
    $point3->setZ($position3);

    $movement = new Movement();
    $movement->setId($id);
    $movement->setSentTime($sentTime);
    $movement->setRotation($vector4);
    $movement->setVelocity($vector3);
    $movement->setPosition($point3);

    $movementList[] = $movement;
}

$joinRoom = new JoinRoom();
$joinRoom->setMoveList($movementList);

$packet = new Packet();
$packet->setType(PacketType::JoinRoomType);
$packet->setJoinRoom($joinRoom);

$protocolBufferSerializedPacket = $packet->serializeToString();

print_r("protocol buffer size : ");
print_r(strlen($protocolBufferSerializedPacket));
print_r(PHP_EOL);


//flat buffer large data

$fbb = new FlatbufferBuilder(1);

$data = array();
for($id=0; $id<20; $id++)
{
    $position1 = mt_rand(-100, 100);
    $position2 = mt_rand(-100, 100);
    $position3 = mt_rand(-100, 100);

    $velocity1 = mt_rand(-100, 100);
    $velocity2 = mt_rand(-100, 100);
    $velocity3 = mt_rand(-100, 100);

    $rotation1 = mt_rand(-100, 100);
    $rotation2 = mt_rand(-100, 100);
    $rotation3 = mt_rand(-100, 100);
    $rotation4 = mt_rand(-100, 100);

    \flatbench\Movement::startMovement($fbb);
    \flatbench\Movement::addId($fbb, $id);
    \flatbench\Movement::addPosition($fbb, \flatbench\Vector3::createVector3($fbb, $position1, $position2, $position3));
    \flatbench\Movement::addVelocity($fbb, \flatbench\Vector3::createVector3($fbb, $velocity1, $velocity2, $velocity3));
    \flatbench\Movement::addRotation($fbb, \flatbench\Vector4::createVector4($fbb, $rotation1, $rotation2, $rotation3, $rotation4));
    $movement = \flatbench\Movement::endMovement($fbb);

    $data[] = $movement;
}


$responsePacketType = \flatbench\Packet::JoinRoom;

//--data
$offset = \flatbench\JoinRoom::createMoveListVector($fbb, $data);
\flatbench\JoinRoom::startJoinRoom($fbb);
\flatbench\JoinRoom::addMoveList($fbb, $offset);
$dataOffset = \flatbench\JoinRoom::endJoinRoom($fbb);
//--end of data
MessageRoot::startMessageRoot($fbb);
MessageRoot::addPacketType($fbb, $responsePacketType);
MessageRoot::addPacket($fbb, $dataOffset);
$messageOffset = MessageRoot::endMessageRoot($fbb);
$fbb->finish($messageOffset);

$flatBufferSerializedPacket = $fbb->sizedByteArray();

print_r("flat buffer size : ");
print_r(strlen($flatBufferSerializedPacket));
print_r(PHP_EOL);

//message pack large data

$movementList = array();
for($id=0; $id<20; $id++)
{
    $position1 = mt_rand(-100, 100);
    $position2 = mt_rand(-100, 100);
    $position3 = mt_rand(-100, 100);

    $velocity1 = mt_rand(-100, 100);
    $velocity2 = mt_rand(-100, 100);
    $velocity3 = mt_rand(-100, 100);

    $rotation1 = mt_rand(-100, 100);
    $rotation2 = mt_rand(-100, 100);
    $rotation3 = mt_rand(-100, 100);
    $rotation4 = mt_rand(-100, 100);

    $movement = array();
    $movement['id'] = $id;
    $movement['sentTime'] = $sentTime;
    $movement['rotation'] = array('x' => $rotation1, 'y' => $rotation2, 'z' => $rotation3, 'w' => $rotation4);
    $movement['velocity'] = array('x' => $velocity1, 'y' => $velocity2, 'z' => $velocity3);
    $movement['position'] = array('x' => $position1, 'y' => $position2, 'z' => $position3);
    $movementList[] = $movement;
}

$packet = array('movementList' => $movementList);

$messagePackPackedPacket = msgpack_pack($packet);

print_r("message pack size : ");
print_r(strlen($messagePackPackedPacket));
print_r(PHP_EOL);

```

결과 값

```
protocol buffer size : 1241
flat buffer size : 1100
```