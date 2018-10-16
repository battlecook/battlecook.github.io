---
layout: post
title: svn update 또는 git pull 한 파일을 바로 upload 하고 싶을 경우
---

환경 :

phpstorm 을 window 환경에서 사용중에 있습니다. 

하지만 실제 서버는 리눅스이기 때문에 실제 코드를 virtualbox 에 리눅스에서 작업합니다. 
 
phpstorm 에서 작업을 한걸 upload 해서 리눅스 서버에 올립니다.

협업시 내가 svn commit 또는 git push 한 파일을 동료가 svn update 또는 git pull 받은 경우에 바로 vm에 파일을 upload 해야하는 경우가 발생합니다. ( 사실 대부분 이렇습니다. )

phpstorm 에서

Tools -> deployment -> options 에 ‘Upload external changes’를 선택하면 됩니다.

![phpstorm screenshot]({{ site.url }}/assets/20160603/upload_external_changes.png)

<br>

관련 링크

http://stackoverflow.com/questions/21257966/phpstorm-automatically-upload-to-default-server-after-update-from-svn