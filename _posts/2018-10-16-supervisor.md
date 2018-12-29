---
layout: post
title:  supervisor 사용
---

어플리케이션을 개발하고 배포시에 데몬이 어떤 이유로 죽은 경우 

바로 되살려서 데몬이 떠있는 걸 유지하고 싶은 경우가 있습니다. 

예를들어 서버 프로그램들이 그렇습니다. ( 서비스가 죽어도 바로 살아서 요청을 받아야겠죠.. )

위의 기능을 가능하게 해주는 프로그램이 슈퍼바이저 입니다. 


# 슈퍼바이저 설치

일단 슈퍼 바이저를 설치해 봅시다. 

epel repo 설치 후

```
yum install supervisor
```

아래의 명령어를 사용하면 슈퍼 바이저가 떠 있는 것을 확인 할 수 있습니다.

```
ps -ef | grep supervisor
```

# 슈퍼바이저 사용
 
슈퍼바이저 환경설정 파일의 기본 경로 입니다. 
 
 ```
/etc/supervisord.conf
 ```
 
 마지막 줄에 보시면 여느 환경설정 파일들과 마찬가지로 
 
```
[include]
files = supervisord.d/*.ini
``` 
 
다른 환경설정 파일을 만들어 인클루드 할 수 있습니다.  

간단한 프로그램을 만들어 동작을 확인해 봅시다.
 
server_program.php

```php
<?php

while(1)
{
   echo "I'm server program 1 \n";
   echo "I'm alive \n";
   sleep(3);
}
```
라는 프로그램을 만듭니다.

myconf.ini 
``` 
[program:test_program] (원하시는 프로그램명을 쓰시면 됩니다.)
command=/usr/bin/php server_program.php
```

사용할 환경설정 파일을 만들고 

```
/etc/supervisord.d/
```

아래에 둡니다. 슈퍼바이저를 재시작 해서 동작을 확인합니다.

``` 
systemctl restart supervisor
```
 
 
 
# 슈퍼바이저 명령어 