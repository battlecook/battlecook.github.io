---
layout: post
title:  php exception 시 error trace 의 변수
---

php 에서 Exception 이 떨어져서 Exception 객체의 getTrace 함수 사용시 변수 값이 생각 했던 것과 달랐습니다.

getTrace 관련한 간단한 셈플 코드 입니다.

```php
<?php
/**
 * @param int $currentAmount
 * @param int $purchaseCost
 * @return int
 * @throws Exception
 */
function subtract(int $currentAmount, int $purchaseCost): int
{
    $currentAmount = $currentAmount - $purchaseCost;
    if($currentAmount < 0)
    {
        throw new \Exception();
    }

    return $currentAmount;
}

try
{
    subtract(5,10);
}
catch(\Exception $e)
{
    $trace = $e->getTrace();

    echo "currentAmount : " . $trace[0]['args'][0] . PHP_EOL;
    echo "purchaseCost : " . $trace[0]['args'][1] . PHP_EOL;
}
```

결과

```
currentAmount : -5
purchaseCost : 10
```

결과 값을 보면 -5, 10 인 것을 알 수 있습니다.

테스트를 하기 전엔 인자로 넘어온 값이 찍힐 거라고 생각해서 5, 10 이 찍힐 줄 알았습니다. 

하지만 인자가 넘어온 후 값을 대입 하게 된다면 대입 한 값으로 바뀌어서 찍히게 됩니다.