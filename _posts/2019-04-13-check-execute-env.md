---
layout: post
title:  php 개발시 현재 실행되고 있는 환경 체크 
---

php 로 프로그래밍을 할때 실행되는 프로그램이 동작하는 환경을 체크하고 싶을때가 있습니다.

예를들면 현재 프로그램을 실행되는 운영체제가 윈도우인지 리눅스인지를 체크하고 싶거나

혹은 프로그램이 실행되는 환경이 cli 모드인지 cgi 모드인지 등입니다.

먼저 운영체제 환경은 디렉토리 구분자로 알수 있습니다. 

php 에서 [미리 등록되어있는 상수값](https://www.php.net/manual/en/dir.constants.php) 중 DIRECTORY_SEPARATOR 값을 이용하면 됩니다.

 ```php
<?php
if (DIRECTORY_SEPARATOR === '\\') {
    print_r("윈도우 운영체제 입니다.");
} else {
    print_r("리눅스 운영체제 입니다.");
}
```

두번째로 실행되는 모드 는 php_sapi_name 함수로 구분 할 수 있습니다.

 ```php
<?php
if (php_sapi_name() == "cli") {
    print_r("cli 모드 입니다.");
} else if (php_sapi_name() == "fpm-fcgi") {
    print_r("fast cgi 모드 입니다.");
}
```

#### 참고한 오픈소스

[workerman](https://github.com/walkor/Workerman/blob/master/Worker.php) checkSapiEnv 함수