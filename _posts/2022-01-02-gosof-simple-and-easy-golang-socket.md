---
layout: post
title: 'GOSOF : A simple yet quite practical socket framework made with golang.'
date: '2022-01-02T00:46:00.018+09:00'
tags:
    - golang
    - socket
    - tcp
modified_time: '2022-09-01T22:25:45.137+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-6401537824391051793
blogger_orig_url: https://jeremyko.blogspot.com/2022/01/gosof-simple-and-easy-golang-socket.html
---

[https://github.com/jeremyko/gosof](https://github.com/jeremyko/gosof)

Instead of the simple socket echo example that comes up a lot if you search, I tried to implement it so that it can properly handle the buffering of tcp data that can be split into pieces or transmitted all at once. And to increase reusability, I implemented it so that it can be used like a framework.

-   The framework calls the user-specified callback.
-   For TCP, the total size of user data is passed to the framework via a callback, and the framework does TCP buffering automatically.
-   Byte data is exchanged.
-   Supports TCP, UDP and Domain Socket.

When sending a byte message composed of a fixed-length header and variable-length body data, it may be useful as it can reduce the work a little.

udp and domain socket are also supported, but this is not much different from using the existing net package as it is, so there is no special point.

<h3> <span style="color:{{site.span_h3_color}}"> 
golang 으로 만든 간단하면서도 꽤^^ 실용적인 socket 프레임워크.
</span> </h3>

검색하면 많이 나오는 단순한 socket echo 예제 대신, 조각으로 쪼개지거나, 한번에 다 전송될수 있는 tcp 의 데이터 버퍼링 처리를 제대로 처리 할 수 있게 끔 구현해 봤다. 그리고 재활용성을 높이기 위해 프레임워크 처럼 사용할수 있게 구현 해 봤다. 몇가지 특징을 나열하면,

-   프레임워크가 사용자 지정 콜백을 호출한다.
-   TCP의 경우 사용자 데이터의 전체 크기는 콜백을 통해 프레임워크에 전달되고 프레임워크는 TCP 버퍼링을 자동으로 수행 해준다.
-   바이트 데이터를 주고 받는다
-   TCP, UDP 및 도메인 소켓을 지원

고정길이 헤더와 가변길이 body 데이터로 구성된 byte 메시지를 전송할때 조금이나마 작업을 줄일수 있어서, 나름 유용할거 같다.

udp 나 domain socket 도 지원하지만, 이건 뭐 기존 net package 를 그대로 사용하는것과 거의 차이가 없기 때문에 별다른 점은 없겠다.

[https://github.com/jeremyko/ASockLib](https://github.com/jeremyko/ASockLib) 과 비슷한 사용법을 가지고 있다.
