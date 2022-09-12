---
layout: post
title: Windows 에서 Ruby on Rails (RoR) 3.2.8 + Oracle XE 11g
date: '2012-08-14T12:39:00.000+09:00'
tags:
    - Ruby on Rails
    - Oracle XE
modified_time: '2012-12-04T20:54:49.616+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-2293831300062100982
blogger_orig_url: https://jeremyko.blogspot.com/2012/08/windows-ruby-on-rails-ror-oracle-xe-11g.html
---

RoR 은 기본적으로 Sqlite 를 Database로 제공해주고 있다. 나는 Oracle XE를 개발용으로 사용중이라서, 여기에 맞춰서 설정을 해보기로 했다. 개발 환경은 다음과 같다.

-   Ruby 1.9.3
-   RoR 3.2.8
-   Oracle XE 11g

### RoR 설치

설치 관련해서는 이미 많은 글들이 존재하므로 다시 내가 뭘 다시 써야할 내용은 없을 것 같다. 간단하게 말해보면, 먼저 Ruby를 설치하고 [(http://rubyinstaller.org/downloads/)](http://rubyinstaller.org/downloads/), 이때 Windows 에서 gem 설치를 위해서 DevKit도 같이 다운로드 해서 설치한다. 그리고 명령 프롬프트 상에서 다음을 실행해서 RoR을 설치한다.

    gem install rails

DevKit설치는 다운받아 압축을 푼 디렉토리 내에서 다음을 수행한다.

    ruby dk.rb init
    ruby dk.rb install

([https://github.com/oneclick/rubyinstaller/wiki/Development-Kit](https://github.com/oneclick/rubyinstaller/wiki/Development-Kit) 을 참조)

아니면, 간단하게 [http://railsinstaller.org/](http://railsinstaller.org/) 에서 통합 패키지를 다운로드 받아서 설치할수도 있다.

### 오라클 관련 gem 설치

다음을 수행해서 필요한 gem 들을 설치한다.

    gem install ruby-oci8
    gem install activerecord-oracle_enhanced-adapter

제대로 설치됬는지 테스트를 해보자. 명령 프롬프트 상에서 다음을 수행해 본다. study/study 는 내 개발 환경에서의 id/pwd이다.

    D:\MyRubyTest> ruby -r oci8 -e "OCI8.new('study', 'study', 'XE').exec('select sysdate from dual') do |r| puts  r.join(' | '); end"

    2012-08-14 12:07:40 +0900

이런 결과가 나온다면 설치는 정상적으로 된 것이다.

### 프로젝트 Gemfile에 gem 추가

RoR 설치후 rails new xxx 등으로 실제 프로젝트를 생성했다고 가정해보자.

이제 중요한 사항은, 위에서 필요한 gem 들을 install 하긴 했지만, 프로젝트에서 이를 사용하려면, Gemfile 에 명시해줘야 한다는 것이다. 그렇지 않다면, gem을 설치하라는 에러메시지를 보게 될것이다 (그럼, 나처럼.. 이런..분명히 설치 했는데 또? 라며 당황하게 될것이고).

Gemfile 에 다음을 추가한다.

    gem 'ruby-oci8'
    gem 'activerecord-oracle_enhanced-adapter'

그리고, 사용하지 않을 기존 sqlite3 부분은 주석 처리한다.

    # gem 'sqlite3'

### bundle install 재 실행

이제 bundle install 을 다시 실행해서, Gemfile의 내용을 반영한다.

### database.yml 수정

다음처럼 수정한다. 개발, 시험,배포의 설정이 구분될수도 있지만, 일단 동일한 설정으로 만들었다.

    development:
        adapter: oracle_enhanced
        database: //localhost:1521/xe
        username: study
        password: study

    test:
        adapter: oracle_enhanced
        database: //localhost:1521/xe
        username: study
        password: study

    production:
        adapter: oracle_enhanced
        database: //localhost:1521/xe
        username: study
        password: study

### db migrate 수행

다음을 수행해서 오류없이 종료되었다면, 이제 연동환경이 구성된 것이다.

    rake db:migrate

실제로 테이블이 생성되는지 확인 해본다.
