---
layout: post
title: include 함수 사용시 ./ 경로 이슈
---

최근 들어서 composer autoloader 를 쓰고 있어서 파일을 직접 include 할일이 잘 없었는데

동료의 코드를 보다가 include 함수 사용시에 몰랐던 점이 있어서 블로깅 한다.

현재 파일 경로는 다음과 같다.

```
/tmp/php_include_test/project/test.php

/tmp/php_include_test/project/vars.php

/tmp/php_include_test/vars.php

```

/tmp/php_include_test/project/test.php

의 코드는 다음과 같다.

```
<?php

include './vars.php';

echo "A $color $fruit";

```


/tmp/php_include_test/project/vars.php

의 코드는 다음과 같다.

```
<?php

$color = 'green';
$fruit = 'apple';

```

/tmp/php_include_test/vars.php

의 코드는 다음과 같다.

```
<?php

$color = 'yellow';
$fruit = 'strawberry';

```

/tmp/php_include_test/project/ 위치에서

```
php test.php
```

명령어 실행시 결과는 다음과 같다.

```
A green apple
```

/tmp/php_include_test/ 위치에서

```
php project/test.php
```

명령어 실행시 결과는 다음과 같다.

```
A yellow strawberry
```

/tmp/php_include_test/ 위치의 vars.php 를 제거하고

/tmp/php_include_test/ 위치에서

```
php project/test.php
```

위의 명령어 실행시 include 할 vars.php 파일을 찾을 수 없다는 에러 메시지가 나왔다.

```
PHP Warning:  include(./vars.php): failed to open stream: No such file or directory in /tmp/php_include_test/project/test.php on line 3

이하 생략
```

include 시에 ./ 의 의미가 현재 경로로 알고 있었는데

현재 경로라는 것이 include 함수를 실행 시키는 .php 파일이 있는 위치가 아니라

php 명령어를 실행 시키는 파일의 위치라는 것을 알 수 있었다.
