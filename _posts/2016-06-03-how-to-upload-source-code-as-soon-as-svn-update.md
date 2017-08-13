---
layout: post
title: SVN Update 한 파일을 바로 upload 하고 싶을 경우
---

환경 :

phpstorm window 에 세팅 되어있음

실제 코드 virtualbox 에 upload 되어서 돌아감

협업시 내가 svn commit 을 한 파일을 동료가 svn update를 받은 경우에 바로 vm에 파일을 upload 해야하는 경우가 발생함

phpstorm 에서

Tools -> deployment -> options 에 ‘Upload external changes’ 를 선택하면 됨

참고 사이트

http://stackoverflow.com/questions/21257966/phpstorm-automatically-upload-to-default-server-after-update-from-svn