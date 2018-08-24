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

물론 파일이 다 바뀌고 나면 정상 동작 하겠지만 그 사이에 들어온 요청은 문제가 생기겠죠.

이 때 사용할 옵션이 opcache.ini 의 opcache.validate_timestamps 입니다.

![opcache_validate_timestamps_option]({{ site.url }}/assets/opcache_validate_timestamps_option.png)

true 또는 1 일 경우엔 파일이 바뀌면 자동으로 캐싱되어있는 php 파일을 바꾸고 

false 또는 0 일 경우엔 php-fpm 을 restart 했을 시에 바뀐 파일이 일괄 적으로 바뀌게 됩니다.

