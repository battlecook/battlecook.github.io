---
layout: post
title: phpredis set 함수 timeout 인자 사용
---

익스텐션 라이브러리 redis 를 래핑한 함수로 사용중에 set 함수를 사용할 일이 있었습니다.

간단하게 코드를 써보자면

```
class RedisWrapper
{
    private $redis;

    /**
     * RedisMapper constructor.
     * @throws Exception
     */
    public function __construct()
    {
        $redis = new \Redis();

        $ret = $redis->pconnect('127.0.0.1', 6379);
        if($ret === false)
        {
            throw new \Exception("connection failed");
        }
        $ret = $redis->auth('비밀번호');
        if($ret === false)
        {
            throw new \Exception("auth failed");
        }
        $ret = $redis->ping();
        if($ret === "+PONG")
        {
            $this->redis = $redis;
        }
        else
        {
            throw new \Exception("ping failed");
        }
    }

    public function set(string $key, $value)
    {
        $this->redis->set($key, $value);
    }
}
```

이런식으로 \Redis 를 래핑 해서 사용하는 함수가 있었고

외부에서 set 함수를 사용중에 있었습니다. 

그러던 중 set 에 expire_time 옵션을 써야 하는 상황이 생겼습니다.

당연히 래핑 함수는
 
```
    public function set(string $key, $value, $expireTime)
    {
        $this->redis->set($key, $value);
    }

```

이렇게 수정이 됩니다. 

그런데 이렇게 수정을 하면 선택지가 2개가 생깁니다.

set 함수를 사용하던 클라이언트 코드들에 모두 expireTime 을 넣어 주던지

혹은 사용하던 클라이언트가 에러가 나지 않도록 디폴트 값을 넣어 주던지 

저는 후자를 선택했습니다. 

외부에서 set 을 사용중인 클라이언트값이 많기도 했고 디폴트 값을 넣어 주는게 

조금 더 안전한 방법이라고 생각했습니다. 

디폴트 값이 뭔지 확인해 보았습니다.   

[레디스 정식 사이트](https://redis.io/commands/set) 에서는 expiration 은 옵션이다. 외엔 별다른 내용이 없습니다.

사용하는 ide 인 phpstorm 에서 디폴트값을 확인해 보았습니다.

plugins\php\lib\php.jar!\stubs\redis\Redis.php 경로의 stub 을 보면

![phpredis_memcache_set_stub]({{ site.url }}/assets/phpredis_memcache_set_stub.png)

위와 같이 timeout 값은 0으로 세팅 된다고 되어있었습니다.

사용중인 레디스레핑 함수에 디폴트 expireTime 값을 0 으로 세팅하엿습니다.

```
    public function set(string $key, $value, $expireTime = 0)
    {
        $this->redis->set($key, $value);
    }
```

코드를 수정하고 테스트 케이스를 돌려보니 새로 추가한 expireTime 값이 있는 테스트를 제외한

기존 set 함수를 사용한 코드는 모두 테스트 케이스 실패가 떨어졌습니다.

일단 레디스 정식 사이트에 들아가서 문제가 될 만한 내용을 확인해 봤습니다.

레디스 정식 사이트는 레디스서버에 관련된 내용이 있고 redis 클라이언트쪽 은 따로 언급이 없었습니다.

이제 제가 설치한 php redis 라이브러리를 의심해 보기로 합니다. 

일단 php.d 에 설치되어 있으므로 익스텐션 라이브러리 라는걸 확인합니다.

![ll_redis_extension_library]({{ site.url }}/assets/ll_redis_extension_library.png)

버전이 몇인지 확인해봅니다. 

![yum_list_installed_php_pecl_redis]({{ site.url }}/assets/yum_list_installed_php_pecl_redis.png)

3.0.0 인걸 확인했습니다. pecl 코드는 [해당사이트](https://pecl.php.net/) 에서 검색해 볼 수 있습니다.

pecl redis 는 https://pecl.php.net/package/redis 여기서 확인해 볼수 있습니다. 

phpredis 라이브러리를 사용중이네요.

접근하기 쉽게 깃헙에 소스 코드가 있는걸 확인할 수 있습니다.

자 이제 깃헙 소스코드를 확인해 보도록 합시다.

3.0.0 은 [이곳](https://github.com/phpredis/phpredis/tree/3.0.0)에 소스코드가 있습니다.

redis_commands.c 파일에 redis_set_cmd 함수를 보면 문제의 원인을 찾을 수 있습니다.

php 코드에서 set 함수에 옵션을 주면 해당 로직을 타는거 같네요.

![phpredis_redis_set_cmd_optional_logic]({{ site.url }}/assets/phpredis_redis_set_cmd_optional_logic.png)

expire 가 1보다 작으면 FAILURE 를 반환하게 되어있습니다.

![set_timeout_must_do_bigger_than_1_seconds]({{ site.url }}/assets/set_timeout_must_do_bigger_than_1_seconds.png)

아마도 너무작은 값으로 set 을 주면 set 을 하자마자 지워지게 되므로 의미가 없어서 저렇게 처리 된 거 같습니다.

해당 이슈에 맞게 php 코드를 수정하였고 테스트 케이스가 실패 없이 동작 하였습니다.

ide 의 스텁도 틀릴 경우가 있고 틀렸을시 실제 구현 코드를 확인해서 이슈를 수정할 수 있었습니다.
