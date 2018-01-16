---
layout: post
title: virtual box nat network
---

virtual box 에서 vm 여러개를 띄어 놓을때

외부에서 각각의 vm 에 접속하고 싶은경우

모두 nat 으로 네트워크를 설정해놓으면 ip가 동일해서 포트 포워딩할 수가 없다.

이럴 경우 nat 을 nat network 로 수정하면 ip를 다르게 할당 할 수 있다.

virtual box 관리자 설정 -> 네트워크 에서 Nat 네트워크 설정시

이름 선택이 안될 수 있음

vm 을 실행시킨 상태에서

파일 -> 환경설정 -> 네트워크 -> 추가

해서 Nat 네트워크 설정후 포트포워딩 까지 완료 하면 된다.