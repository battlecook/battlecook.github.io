---
layout: post
title: Phpstorm 재시작시 Php language level 세팅 리셋 이슈
---

개발 중 php7.1 에서 추가된 기능 사용시 phpstorm 에서 문법 에러를 내고 있었다.

![phpstorm screenshot]({{ site.url }}/assets/20180329/php_language_level_in_phpstorm.png)

php language level 이 7 로 되어있어서 7.1로 수정 후 작업을 했었는데

며칠뒤에 세팅이 다시 7로 롤백 되어있어서 원인을 좀 찾아 보았다.

File => Settings

Language & Frameworks => PHP 에

CLI Interpreter 세팅은 7.1로 되어있었다.

Composer 세팅에

![phpstorm screenshot]({{ site.url }}/assets/20180329/synchronize_ide_settings_with_composer_json.jpg)

굉장히 의심스럽게 생긴 요놈이 있어서

이 녀석 체크박스를 선택 해제 하고 스톰 재시작시 php language level 세팅이 리셋 되지 않았다.

composer.json 설정을 보니

```
"require": {
    "php": "^7.0.0"
  },

```


composer 에 명시한 php 버전을 따라 간다고 생각하고


```
"require": {
    "php": "^7.1.0"
  },

```

로 수정후 Synchronize IDE Setting with composer.json

세팅 체크박스를 다시 선택 후

재시작 했다.

php language level 7.1 로 세팅 되었다.

