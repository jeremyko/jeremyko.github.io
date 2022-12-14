---
layout: post
title: DisruptorCpp-IPC 가변길이 데이터 처리
date: '2015-09-15T16:00:00.003+09:00'
tags:
    - c++
    - Disruptor
modified_time: '2022-03-26T20:59:54.804+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-2193770637077337607
blogger_orig_url: https://jeremyko.blogspot.com/2015/09/disruptorcpp-ipc.html
---

앞서 [Disruptor동작원리를 이용해서 IPC 로 동작하는 소스]({% post_url 2015-09-06-disruptor %})를 구현해봤는데 한가지 단점은 고정된 길이의 데이터를 미리 할당해서 링버퍼에 넣어두고 사용하는 방식이라는 점이었다. 만약 저장되는 데이터 길이가 가변적인 경우엔 이용할수 없는 단점을 보완해서 약간 수정된 버전으로 작성을 해보았다.

[https://github.com/jeremyko/disruptorCpp-IPC-Arbitrary-Length-Data](https://github.com/jeremyko/disruptorCpp-IPC-Arbitrary-Length-Data)

별도의 데이터 저장용 메모리를 할당하고, 링버퍼에는 데이터 저장위치만을 관리하는 방식이다.
