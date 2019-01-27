---
layout: post
title: aws rds pdo exceiption 권한 이슈 
---

aws 프리티어 신청후

lvs, was, db  를 인프라를 구축해봤습니다.

aws의 elastic load balance, ec2, rds 로 구축을 하고 was에 nginx, php-fpm 으로 웹서버를 구축한 후 웹요청을 받아서 pdo 로 디비접속시 익셉션이 떨어졌습니다.

익셉션 로그는 permission denied … 권한 문제 aws 콘솔 접속후 security group 의 was의 out bound 와 rds의 in bound 방화벽을 설정확인 후 다시 실행 역시나 익셉션이 떨어집니다.

다시 rds 의 security group으로 내 로컬 컴퓨터의 방화벽을 허용 후 phpstorm db plugin으로 접속 시도 … 접속 성공

was에 접속해서 pdo 커넥션 로직을 커멘드라인으로 시도 접속 성공

웹요청 받은후 웹루트 하위의 php 파일로 pdo 실행시에만 익셉션이 발생한다고 판단 합니다.

웹요청은 nginx 권한으로 실행 nginx 는 php-fpm 으로 던져서 php파일 은 결국 php-fpm 이 실행

이제 php-fpm  과 nginx 권한을 의심하기 시작 합니다.

하위 디렉토리 권한을 상승시켜보앗으나 접속 실패

nginx, php-fpm 으로 user변경후 커멘드라인에서 접속을 하기위해 su – nginx, su – php-fpm

nginx, php-fpm 으로 유저 변경할수 없습니다.

직장동료의 iam  관련 이슈를 확인해보라는 권유에 ec2에 iam  rds full access 역할 부여 하였으나 접속 실패 (참고 url : http://wildpup.cafe24.com/archives/673 )

centos 7 이슈로 판단하고 docker 컨테이너를 이용해서 centos6 을 올리려고 알아보다가 이슈가 우선순위에서 밀림….

며칠후 직장동료의 접속 문제 해결했냐는 물음에 다시 이슈가 스타베이션에서 벗어났습니다.

직장동료의 selinux 끈후 해보라는 충고에

setenforce 0 명령어 입력후 실행 접속 성공 ..

selinux 이슈로 확인함

selinux 관련 문서 찾기 시작

https://kldp.org/files/selinux_140.pdf

http://hodol.kr/xe/index.php?mid=note&document_srl=17532&m=0

위의 두 문서 및 위키피디아를 읽은 후 selinux 를 켜야겠다고 생각함

selinux 킨 후

setsebool httpd_can_network_connect_db 1 설정 함

pdo 로 db 접속 성공 확인함

 