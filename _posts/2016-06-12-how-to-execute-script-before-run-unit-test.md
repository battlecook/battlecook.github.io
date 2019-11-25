---
layout: post
title: unit 테스트 전에 어떤 스크립트를 돌리고 싶은 경우
---

PHPUnit 클레스를 상속받아서 Setup 함수를 만들면 클레스 시작전에 행동해야 할 것들을 정의할 수 있습니다.

디렉토리 단위로 테스트를 진행할 경우 테스트 시작전 한번만 도는 스크립트를 만들고 싶은경우가 있습니다. (나의 경우에 디비 스키마를 셋업 하는 경우)

phpunit.xml을 만들고 phpunit.xml 에

bootstrap="실행시킬 파일( ex) db_init.php )"  를 넣으면 됩니다.

예시는 다음과 같습니다.

```
<phpunit
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="https://schema.phpunit.de/6.3/phpunit.xsd"
        backupGlobals="false"
        backupStaticAttributes="false"
        bootstrap="bootstrap/db_init.php">

    <testsuites>
        <testsuite name="My App Test Suites">
            <directory>./tests/</directory>
        </testsuite>
    </testsuites>

    <filter>
        <whitelist>
            <directory suffix=".php">./tests</directory>
        </whitelist>
    </filter>
</phpunit>
```

그 후에 phpstorm 에서 Run/Debug Configurations 에서 Test Runner -> Use alternative configuration file: 에 만들어놓은 phpunit.xml 를 선택하면 됩니다.

<br>

관련 링크

[http://stackoverflow.com/questions/8085641/with-phpunit-how-does-one-run-initialization-code-before-any-tests-are-run](http://stackoverflow.com/questions/8085641/with-phpunit-how-does-one-run-initialization-code-before-any-tests-are-run)

[https://phpunit.de/manual/5.1/en/textui.html](https://phpunit.de/manual/5.1/en/textui.html)