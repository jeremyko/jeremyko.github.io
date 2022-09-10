---
layout: post
title: ASockLib  windows 를 지원
date: '2019-12-27T22:31:00.002+09:00'
tags:
    - c++
    - ASock
modified_time: '2022-03-26T20:56:03.761+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-62587157318904030
blogger_orig_url: https://jeremyko.blogspot.com/2019/12/asocklib-windows_27.html
---

[https://github.com/jeremyko/ASockLib](https://github.com/jeremyko/ASockLib)

ASock이라는 이름으로 socket 관련 라이브러리를 만들어서 프로젝트에서 사용하고 있었는데,  
linux, os x 만 지원하던것을 이번에 며칠 손봐서 windows 에서도 돌아가게 구현해봤다.  
만들고 보니, proactive 방식과 reactive 방식이 뒤섞인 결과물.  
portable 한 networking library가 일단 완성된것에만 일단 만족...
