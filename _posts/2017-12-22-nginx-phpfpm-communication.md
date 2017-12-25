---
layout: post
title: nginx, php-fpm 간 통신
---

기본적으로 웹서버 구조는


![webserver_cgi]({{ site.url }}/assets/webserver_cgi.png)

와 같다.

nginx 와 php 를 사용하는 경우

![nginx_php_fpm]({{ site.url }}/assets/nginx_php_fpm.png)

와 같이 된다.

FastCGI는 웹 서버와 프로그램이 상호작용(데이터를 주고 받기 위한) 인터페이스 개발을 위한 프로토콜이다.

FastCGI는 CGI(Common Gateway Interface)를 개선한 인터페이스다.

PHP 는 FastCgi 구현채인 php-fpm 을 사용해서 nginx 와 통신한다.

php-fpm 설정 ( 기본경로 : /etc/php-fpm.d/www.conf )

```
listen = 127.0.0.1:9000
```

9000 번 포트로 tcp/ip socket 통신을 하는걸 알 수 있다.


nginx 설정 에서 php-fpm 과 통신할 수 있도록 설정 해보자

nginx 설정 ( 기본경로 : /etc/nginx/nginx.conf )

```
server {
    location ~ \.(php)$ {
        fastcgi_pass  127.0.0.1:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param QUERY_STRING    $query_string;
    }

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```

만일 nginx 와 php-fpm 이 물리적으로 동일한 서버에 있다면

unix socket 을 사용해서 속도를 향상 시킬 수 있다.

(주의 : unix socket 은 프로세스간 통신 이기 때문에 물리적으로 다른 서버에 있다면 사용 할 수 없다.)

다음과 같이 설정을 바꾼다.


php-fpm 설정

```
listen = /var/run/php7-fpm.sock
```

nginx 설정

```
fastcgi_pass   unix:/var/run/php7-fpm.sock;
```

설정 후

![unixsocket_path]({{ site.url }}/assets/unixsocket_path.png)

nginx 와 php-fpm 소켓통신의 경우 접근이 잦기 때문에

/dev/shm 경로 아래에 넣어주면 약간의 속도 향상 효과를 얻을 수 있다.

/dev/shm 은 실제로 메모리를 점유하는건 아니지만 사용하는 만큼 램을 사용한다.

결국

php-fpm 설정

```
listen = /dev/shm/php7-fpm.sock
```

nginx 설정

```
fastcgi_pass   unix:/dev/shm/php7-fpm.sock;
```

으로 수정하면 된다.

아주 간단하게 벤치마킹을 해보자.

server 코드

```
<?php

echo "test";
echo PHP_EOL;
```

client 코드

```
<?php

$message = 'test';
$header = array();
$url = "서버 주소";

$start = microtime(true);

for($i=0; $i<100; $i++)
{
    $curlSession = curl_init();
    curl_setopt($curlSession, CURLOPT_URL, $url);
    curl_setopt($curlSession, CURLOPT_HTTPHEADER, $header);
    curl_setopt($curlSession, CURLOPT_POST, TRUE);
    curl_setopt($curlSession, CURLOPT_POSTFIELDS, $message);
    curl_setopt($curlSession, CURLOPT_RETURNTRANSFER, TRUE);

    $curlResult = curl_exec($curlSession);
    curl_close($curlSession);
}

$end = microtime(true);

print_r($end - $start);

```

5회 돌려본 결과

| | 1회 | 2회 | 3회 | 4회 | 5회 |
| --- | --- | --- | --- | --- | --- |
| tcp/ip socket |  2.2519| 2.5132|2.3198 |2.4347 | 2.4336 |
| unix socket | 2.4471 | 2.3182 | 2.1452 | 2.1803 | 1.9307 |





## 참고 사이트

- [shared memory](http://bahndal.egloos.com/465571)

- [fast cgi](https://www.joinc.co.kr/w/man/12/fastcgi)

- [nginx](http://nginx.org/en/docs/beginners_guide.html)

- [phpfpm fastcgi_pass](https://easyengine.io/tutorials/php/fpm-sysctl-tweaking/)