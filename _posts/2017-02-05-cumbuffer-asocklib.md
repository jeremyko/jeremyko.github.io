---
layout: post
title: 'Cumbuffer 를 이용한 예제 : ASockLib'
date: '2017-02-05T18:36:00.003+09:00'
tags:
    - c++
    - epoll
    - CumBuffer
    - kqueue
    - network
modified_time: '2022-03-26T20:58:10.030+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-3624777678136013329
blogger_orig_url: https://jeremyko.blogspot.com/2017/02/cumbuffer-asocklib.html
---

[https://github.com/jeremyko/ASockLib](https://github.com/jeremyko/ASockLib)

[CumBuffer](https://github.com/jeremyko/CumBuffer) 를 사용하는 예제 겸 TCP/UDP/Domain socket 네트워크 라이브러리를 작성해 보았다. 비록 허접하지만 cross platform 에서 동일한 interface 를 지원해보자는 생각이 있어서....

linux 에서는 epoll, os x 에서는 kqueue 를 사용해서 구현 되었다.  
cmake를 사용해서 cross platform compile 을 지원하게도 해보았다.

클래스 상속과 포함(composition) 2가지 방식으로 사용 가능하다.

그리고 비동기 send 호출시 block 되는 경우에는 큐에 저장되었다가 전송 가능한 시점에 재개되는 방식으로 처리했다.  
지금 수행중인 SI 과금 프로젝트에서 이걸 적용하면서 개발중이다.
