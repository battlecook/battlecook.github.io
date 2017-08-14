---
layout: post
title: Capistrano3 install
---

rvm 으로 루비를 설치한후 (rvm으로 반드시 설치해야함)

Gemfile 임의로 만들고

group :development do
  gem "capistrano", "~> 3.4"
end
bundle install

bundle exec cap install

//필요한 stage 만들고

bundle exec cap install STAGES=dev, qa, production, admin

stage 들이

config/deploy/ 밑에 생김

최상위 폴더로 와서

cap stage deploy 를 치면 됨

간혹 sudo 명령어를 쳐야할 때가 있음

이를테면

execute :sudo, “service php-fpm reload”

이런경우

sudo stderr: sudo: sorry, you must have a tty to run sudo

이런에러가 발생할수 있음

Gemfile 에

gem “sshkit-sudo”

이걸 추가하고

bundle 명령어를 치고

execute! :sudo, “service php-fpm reload”

로 수정

Capfile 에

require ‘sshkit/sudo’ 추가해서 사용하면 됨

참고 사이트

https://github.com/capistrano/capistrano/blob/master/README.md
https://github.com/kentaroi/sshkit-sudo