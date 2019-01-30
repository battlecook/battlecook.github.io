---
layout: post
title:  php warning 및 fatal error 핸들링
---

php 로 코드를 작성하다보면 warning 에러와 fatal 에러를 핸들링 하고싶은 경우가 있습니다.

저의 경우는 warning 에러와 fatal 에러가 난 경우에 에러를 잡기 위해서 db 등에 적거나 알림을 받거나 등의 작업을 하기 위함 이였습니다.

사용법을 알아 보도록 하겠습니다.

일단 warning 을 잡는 경우의 함수 입니다.

```
set_error_handler
```

공식 문서는 [이곳](http://php.net/manual/en/function.set-error-handler.php) 에서 확인 하실 수 있습니다.

없는 변수를 접근하려고 한 경우의 간단한 예제 코드를 작성 해 보겠습니다. 

```php
<?php
set_error_handler( "warningErrorHandler" );

function warningErrorHandler( $errorNum, $errorString, $errorFile, $errorLine, $errorContext = null )
{
    echo "errorNum : $errorNum" . PHP_EOL;
    echo "errorString : $errorString" . PHP_EOL;
    echo "errorFile : $errorFile" . PHP_EOL;
    echo "errorLine : $errorLine" . PHP_EOL;
}

print_r($a[0]);
```

결과

```
errorNum : 8
errorString : Undefined variable: a
errorFile : /var/www/danceville/local/tests/learning_api/php/errorHandling/warningErrorHandling.php
errorLine : 12
```

다음은 fatal 에러를 잡기위한 함수입니다. 

```
register_shutdown_function
```

공식 문서는 [이곳](http://php.net/manual/en/function.register-shutdown-function.php) 에서 확인 하실 수 있습니다.

위의 함수는 정확히 fatal 에러를 잡기 위함은 아니고 스크립트가 완료됐거나 exit() 함수 사용시에 호출 됩니다.

하지만 fatal 에러가 발생시에 등록한 콜백함수에서 잡을 수 있기때문에 fatal 에러를 잡기 위해서도 사용할 수 있습니다.

저의 경우는 register_shutdown_function 에 콜백함수를 등록할때 error_get_last() 함수와 debug_backtrace() 함수를 함께 사용하였습니다.

간단한 예제 코드 입니다.

```php
<?php

register_shutdown_function('shutdown');

function shutdown()
{
    $lastError = error_get_last();
    if($lastError !== null)
    {
        $log = array();
        $log['error_message'] = $lastError;
        $log['debug'] = debug_backtrace();

        print_r($log);
    }
}
```

한가지 알아 둬야 할 점은 register_shutdown_function 함수의 순서 종속성 입니다.

공식문서의 내용은 아래와 같습니다.

```
Multiple calls to register_shutdown_function() can be made, and each will be called in the same order as they were registered. 
If you call exit() within one registered shutdown function, processing will stop completely and no other registered shutdown functions will be called.
```

만약 register_shutdown_function 함수를 여러번 등록 하게 된다면 등록된 순서대로 호출 된다는 내용입니다.


```php
<?php

register_shutdown_function('shutdown3');
register_shutdown_function('shutdown2');
register_shutdown_function('shutdown1');

function shutdown1()
{
    echo "shutdown1 function" . PHP_EOL;
}

function shutdown2()
{
    echo "shutdown2 function" . PHP_EOL;
}

function shutdown3()
{
    echo "shutdown3 function" . PHP_EOL;
}
```

결과

```
shutdown3 function
shutdown2 function
shutdown1 function
```