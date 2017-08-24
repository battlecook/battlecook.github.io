---
layout: post
title: google flatbuffer 의 cpu 사용량 이슈 
---

aws 에 띄워놓은 php 소켓 서버가 cpu 사용률이 100% 를 먹는 이슈가 있었다.

php socket 서버는 <a href = https://github.com/walkor/Workerman >Workerman</a> 기반으로 동작 중이 였고

클라이언트/서버간 통신에는 <a href = https://google.github.io/flatbuffers>flatbuffer</a> 를 사용하고 있었다.

 ![phpstorm screenshot]({{ site.url }}/assets/xhprof_maintain_term.png)
 
"게임 개발 과 다른 성능에 크리티컬한 어플리케이션" 을 위해 만든 구글의 라이브러리를 

의심하지 않고 기반이되는 workerman 을 의심함

특별한점 찾지 못함

php 의 가비지 컬렉터를 의시함

역시 특별한점 찾지 못함 

flatbuffer 에 대해 의심이 들기 시작함 










 

