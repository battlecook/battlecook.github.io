---
layout: post
title: 배포를 위한 php-fpm 및 opcache 설정
---

php-fpm 을 사용해서 개발시 배포시에 주의할 점이 한가지 있습니다. 

php-fpm 은 기본적으로 코드를 변경시에 바뀐 파일이 바로 적용 되는데 이 부분이 문제가 됩니다.

예를 들어 보겠습니다. 

main.php 파일이 있고

```
<?php

include "lib.php";

echo "world";

```

lib.php 파일이 있다고 합시다.

```
<?php

echo "hello ";
```

출력 결과는 아래와 같습니다.  

```
hello world
```

해당 파일을 아래와 같이 수정해서 배포한다고 합시다.

main.php

```
<?php

include "lib.php";

echo "an idealist.";

```

lib.php

```
<?php

echo "I am ";
```

배포자의 의도는 아래의 출력결과를 의도 했겠지만

```
I am an idealist.
```

바뀐 파일이 바로 적용 되기 때문에 모든 파일이 바뀌기 전까지  

```
I am world 
```

혹은 

```
hello an idealist.
```

이런 결과값을 응답 할 수 있습니다. 

물론 파일이 다 바뀌고 나면 정상 동작 하겠지만 그 사이에 들어온 요청은 문제가 생깁니다.

이 때 사용할 옵션이 opcache.ini 의 opcache.validate_timestamps 입니다.

![opcache_validate_timestamps_option]({{ site.url }}/assets/opcache_validate_timestamps_option.png)

true 또는 1 일 경우엔 파일이 바뀌면 자동으로 캐싱되어있는 php 파일을 바꾸고 

false 또는 0 일 경우엔 php-fpm 을 restart 했을 시에 바뀐 파일이 일괄 적으로 바뀌게 됩니다.

상용 서버의 경우 0 또는 false 로 바꿔서 사용 하시는게 좋습니다.

opcache.validate_timestamps 옵션을 false 로 하더라도 한가지 문제가 더 있습니다.

php-fpm restart 를 하는 경우 기존에 떠 있던 프로세스는 모두 강제로 죽어 버리기 때문에 기존 요청 들은 문제가 됩니다.

그래서 보통 php-fpm restart 보다는 php-fpm reload 를 하게 됩니다. 

둘의 차이는 

restart 는 서비스를 내렸다가 다시 시작 하는 것이고 

reload 는 서비스를 죽이지 않고 config 를 다시 읽습니다.  

그럼 여기서 한가지 의문이 있는데요 언제 config 를 다시 읽을 것이냐 입니다. 

그때 필요한 옵션이 php-fpm.conf 의 process_control_timeout 입니다.

 ![phpfpm_process_control_timeout_option]({{ site.url }}/assets/phpfpm_process_control_timeout_option.png)
 
 이 값을 0으로 하면 기존에 떠 있던 php-fpm 프로세스를 기다리지 않기 때문에 기껏 restart 를 reload 로 한 의미가 없어 집니다.
 
 기존 프로세스가 다 처리 될 정도의 시간 값을 세팅 해 두고 사용 하시길 추천 합니다.
 
  
 