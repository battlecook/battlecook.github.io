---
layout: post
title: php-pecl-memcache 버전 이슈
---

php-pecl-memcache 2.2.7 에서 디스크 라이트시 버그가 있는 것 같습니다.

어느정도 부하가 생기면 memcache 에 읽기 쓰기가 느려지기 시작합니다.

php-pecl-memcache 3.0.8 (beta)

로 업데이트 한후 이슈가 해결되었습니다.

추가로 예전에 있었던 이슈중에 memcache client 라이브러리 버전이 다르면 헤시 함수도 다른거 같았습니다.

당연하지만 서버 설치할때 라이브러리 버전은 반드시 확인해야 합니다.