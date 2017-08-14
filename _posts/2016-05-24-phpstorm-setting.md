---
layout: post
title: phpstorm 에서 설정한 서버에 바로 upload 하는 방법
---

phpstorm 에서 ctrl +  s 누르면 자동으로 설정한 ftp 서버로 저장하고 싶은 경우

Deployment -> options 에 Upload changed files automatically to the default server 를 On explicit save action (Ctrl + S ) 를 누르면 되는데

({{ site.url }}/assets/automatically_upload_setting.png)

가끔 밑에 Default server is not configured 가 뜰경우가 있다..

이런경우 Tools -> deployment -> configuration

Mappings 탭 밑에 Use this server as default 를 눌러주면 된다.





