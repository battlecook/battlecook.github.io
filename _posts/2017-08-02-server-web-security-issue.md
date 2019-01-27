---
layout: post
title: 서버 웹 보안 이슈 
---

밤에 AWS 에 띄워놓은 웹서버에서 작업중 nginx 의 access.log 에 웹스케닝 공격이 들어왔습니다.

관련 로그는 아래와 같습니다.

![nginx access log]({{ site.url }}/assets/20170802/webserver_hacking.png)

위와 같이 웹서버 에서 보편적으로 사용하는 admin 페이지나 혹은 db 관련 정보를 얻을 수 있을법한 경로를 무작위로 뒤지는 공격입니다. 

그것도 끊임없이 계속… 인터넷을 뒤져보니 Jorgee라는 웹 취약점 찾는 도구라고 합니다.

신경이 쓰여서 하던 개발을 멈추고 대응을 좀 해보았습니다.

일단 웹서버에

<pre><code>
if ($http_user_agent ~* (Jorgee|npbot)) {
    return 403;
}
</code></pre>

를 추가해서 user_agent가 Jorgee 혹은 npbot이면 무조건 403으로 내려주도록 추가 하였습니다.

그리고  현재 HTTP METHOD 는 Get,Post 밖에 사용하지 않았으므로 nginx 의 location 아래에

<pre><code>
if ( $request_method !~ ^(GET|POST)$ ) {
    return 405;
}
</code></pre>

도 추가하였습니다.

이왕 이렇게 된거 추가적으로 몇가지 대응을 더 해 보았습니다.

일단 개인 vm에서 내 서버로 namp을 사용해서 포트 스캔이 되는지 확인했습니다.

사용한 명령어 : nmap -sT myip

포트스캔이 확인되는 것을 확인하고 현재 사용하지 않는 포트를 모두 제거 했습니다.

AWS에 security group에 ICMP  프로토콜을 제거 하니 nmap 자체가 되지 않았다. nmap이 icmp 프로토콜을 이용한다는걸 확인하였습니다.

추가로 permission 관련 몇가지 대응을 더 했습니다.

조만간 보안회사에 다니는 지인에게 오늘 이슈 관련해서 좀더 문의를 해봐야겠습니다.

<br>

관련 링크

http://blog.radup.net/34

http://coffeenix.net/doc/security/nmap.html

http://ask.xmodulo.com/block-specific-user-agents-nginx-web-server.html

http://coffeenix.net/doc/security/nmap.html

https://www.bjornjohansen.no/restrict-allowed-http-methods-in-nginx

http://sarc.io/index.php/nginx/325-nginx-http-method

+

제한할 user agent 추가 목록을 구함

<pre><code>
if ($http_user_agent ~* (Jorgee|npbot|ZmEu|paros|sqlmap|nikto|dirbuster|w3af|openvas|Morfeus|JCE|Zollard)) {
    return 404;
}
</code></pre>

nginx 서버 버전 노출을 금지하는 세팅 추가

http 태그 아래에  server_tokens off;   세팅을 추가