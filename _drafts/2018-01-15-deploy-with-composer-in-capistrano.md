---
layout: post
title: capistrano 배포시 composer 를 이용한 의존성 해결
---

php 도 node 의 npm 과 비슷한 의존성 관리 툴이 있다.

composer 라는 툴인데 이를 이용해서 외부 라이브러리 의존성을 관리할 수 있다.

capistrano 를 사용해서 배포시에 매번 의존성있는 라이브러리들을 install 하거나 update 한다면

비효율 적이므로 고민을 좀 해봤다.

