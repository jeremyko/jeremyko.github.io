---
layout: post
title: ubuntu server virtualbox remote ssh root login 설정
date: '2017-10-21T12:52:00.001+09:00'
tags:
    - ssh
    - ubuntu
modified_time: '2018-10-12T13:04:07.085+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-4297843491227108010
blogger_orig_url: https://jeremyko.blogspot.com/2017/10/ubuntu-server-virtualbox-remote-ssh.html
---

<h3> <span style="color:{{site.span_h3_color}}"> 
ubuntu 서버 virtualbox 원격 ssh 로그인 설정
</span> </h3>

-   root 계정 생성

```
    sudo passwd root 수행 후 su 수행.
```

-   ubuntu server 에 sshd 동작 확인

```
    ps -ef | grep sshd

    설치 안된 경우엔

    apt-get install openssh-server -y
```

-   ssh root login 설정

```
    vi /etc/ssh/sshd_config , PermitRootLogin yes 변경.
```

-   virtualbox 의 네트워크 환경 어댑터 2 는 불필요, 기본 어댑터의 포트 포워딩에 추가.
    예를 들어, os x 터미널에서 `ssh root@127.0.0.1 -p1234` 처럼 접속하기 위해서는 다음처럼 설정한다.

    ![virtualbox-config](/assets/img/20171021-virtualbox.png)

-   ubuntu 재시동 후 `ssh root@127.0.0.1 -p1234` 로그인 확인
