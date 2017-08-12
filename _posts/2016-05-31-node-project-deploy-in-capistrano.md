---
layout: post
title: Capistrano 3 로 node project deploy 하기
---

보통 nodejs 프로젝트는 모듈의 의존성을 따로 관리함

node 프로젝트는 외부 모듈을 node_module 디렉토리 아래로 인스톨 해서 관리함

외부 모듈목록을 package.json 에 적어 두고 소스를 직접 올리지 않음

프로젝트에서 npm install 로 package.json에 적힌 외부 모듈들을 node_module 아래로 모두 설치함

capistrano 에서 npm install을 하기위해서

gem install capistrano-npm 으로 capistrano-npm 모듈을 설치함

Capfile 에

require ‘capistrano/npm’

추가

1)

set :scm, :git

set :repo_url, “git 주소를 쓰면되는데 http의 경우 http:// ID : PASSWORD @ 주소

이렇게 해줘야 id password를 등록해 줄 수 있음

2)

운영해본 결과 node_module 을 미리 다 다운로드 받은후 받아놓은 소스를 배포하는게 더 효율적으로 보임

reference url

https://semaphoreci.com/community/tutorials/how-to-deploy-node-js-applications-with-capistrano

https://github.com/capistrano/npm