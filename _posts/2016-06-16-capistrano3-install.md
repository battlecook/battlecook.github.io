---
layout: post
title: capistrano3 install
---

--- 해당문서는 2018년 10월 16일에 업데이트 되었습니다. --- 


카피스트라노 는 루비로 만든 배포툴 입니다. 

일단 루비를 설치해야 하는데 루비의 버전 관리는 꽤나 복잡해서 rvm 이라는 버전 메니지 툴이 따로 있을 정도입니다.

기존에 ruby 가 있다면 삭제하고 rvm 으로 설치하시는걸 추천드립니다. 

rvm 을 이용한 루비 설치방법은 다음과 같습니다. ( rvm 공식 사이트의 첫페이지에 있는 설치 방법을 가져 왔습니다. )

```
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
``` 

```
\curl -sSL https://get.rvm.io | bash -s stable
``` 

인스톨 할 수 있는 루비의 버전을 확인해 봅시다.

```
rvm list known | grep ruby
```

![rvm-ruby-list]({{ site.url }}/assets/20160616/rvm-ruby-list.png)
 

프리뷰 버전을 제외한 최신버전은 2.5.1 이군요.

rvm 을 이용해 루비 2.5.1 버전을 설치 합니다.

```
rvm install ruby-2.5.1
``` 

![ruby-required-packaged-error]({{ site.url }}/assets/20160616/ruby-required-packaged-error.png)

미리 설치되어져야 할 패키지 목록들이 뜨는데 비번을 쳐주면 패키지들이 설치가 됩니다.

(위 스크린샷은 최초에 비번을 써야하는건지 몰라서 엔터 쳤다가 에러난 스샷입니다. ㅠㅠ)

정상적으로 설치가 되면 아래와 같은 화면을 보실수 있습니다.

![ruby-install-complete]({{ site.url }}/assets/20160616/ruby-install-complete.png)

루비가 잘 설치 되어있는지 확인합니다.

```
ruby -v 
```

이제 카피스트라노를 설치해 봅시다.

Gemfile을 임의로 만듭니다.

```
touch Gemfile
```

Gemfile 안에 들어가서 아래의 문구를 추가 합니다.

```
group :development do
  gem "capistrano", "~> 3.4"
end
```

혹은 명령어 한줄로도 가능합니다.

```
echo -e "group :development do\n   gem \"capistrano\", \"~> 3.4\"\nend" >> Gemfile
```

```
bundle install
```

```
bundle exec cap install
```

필요한 스테이지를 만듭니다. 아래의 명령어를 치면 됩니다.

```
bundle exec cap install STAGES=dev, qa, production
```


스테이지 들이 config/deploy/ 폴더 아래에 생성됩니다.

최상위 폴더로 와서

cap stage deploy 를 치면 됩니다.

간혹 sudo 명령어를 쳐야할 때가 있습니다.

이를테면 

```
execute :sudo, "service php-fpm reload"
```

이런 명령어를 쳐야 할 경우

```
sudo stderr: sudo: sorry, you must have a tty to run sudo
```

이런에러가 발생할수 있습니다.

Gemfile 에

gem "sshkit-sudo"

이걸 추가하고

bundle 명령어를 치고 ( 혹은 gem install sshkit-sudo 를 쳐서 설치합니다. )

execute! :sudo, "service php-fpm reload"

로 수정 후 Capfile 에

require "sshkit/sudo" 추가해서 사용하면 됩니다.


#### 참고 사이트

- [루비 공식 사이트](https://www.ruby-lang.org/ko/)

- [rvm 공식 사이트](https://rvm.io/)

- [카피스트라노 공식 사이트](https://capistranorb.com/documentation/getting-started/installation/)

- [rvm 설치 한국어 포스팅](http://bigmatch.i-um.net/2013/12/04/%EB%A9%98%EB%B6%95%EC%97%86%EC%9D%B4-rvm%EA%B3%BC-%EB%A3%A8%EB%B9%84-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0/)