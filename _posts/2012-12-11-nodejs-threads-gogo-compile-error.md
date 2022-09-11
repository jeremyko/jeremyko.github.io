---
layout: post
title: 'node.js : threads-a-gogo compile error'
date: '2012-12-11T01:09:00.000+09:00'
tags:
    - c++
    - node.js
modified_time: '2012-12-11T01:11:17.051+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-6492509203588761622
blogger_orig_url: https://jeremyko.blogspot.com/2012/12/nodejs-threads-gogo-compile-error.html
---

<h3> <span style="color:{{site.span_h3_color}}"> 
node.js를 위한 thread, threads_a_gogo 컴파일시 오류 해결 방법
</span> </h3>

[threads_a_gogo](https://github.com/xk/node-threads-a-gogo)

koui-MacBook-Pro:node-threads-a-gogo kojunghyun$ `node-waf configure install`

    ******** THREADS_A_GOGO ********
    **** Executing the configuration

    Checking for program gcc or cc : /usr/bin/gcc
    Checking for program cpp : /usr/bin/cpp
    Checking for program ar : /usr/bin/ar
    Checking for program ranlib : /usr/bin/ranlib
    Checking for program g++ or c++ : /usr/bin/g++
    Checking for program ar : /usr/bin/ar
    Checking for program ranlib : /usr/bin/ranlib
    Checking for g++ : ok

    **** Checking for node_addon

    Checking for node path : not found
    Checking for node prefix : ok /usr/local

    **** Compiling minify.c
    gcc /Users/kojunghyun/kojh/MyDev_MBP/NodeJs_Dev/node-threads-a-gogo/deps/minifier/src/minify.c -o /Users/kojunghyun/kojh/MyDev_MBP/NodeJs_Dev/node-threads-a-gogo/deps/minifier/bin/minify

    'configure' finished successfully (0.210s)
    Waf: Entering directory `/Users/kojunghyun/kojh/MyDev_MBP/NodeJs_Dev/node-threads-a-gogo/build'

    *** Building the project

    *** Minifying .js source files
    cat /Users/kojunghyun/kojh/MyDev_MBP/NodeJs_Dev/node-threads-a-gogo/src/createPool.js | /Users/kojunghyun/kojh/MyDev_MBP/NodeJs_Dev/node-threads-a-gogo/deps/minifier/bin/minify kCreatePool_js > /Users/kojunghyun/kojh/MyDev_MBP/NodeJs_Dev/node-threads-a-gogo/src/createPool.js.c
    ......
    [1/2] cxx: src/threads_a_gogo.cc -> build/Release/src/threads_a_gogo_1.o
    ../src/threads_a_gogo.cc: In function ‘void_ aThread(void*)’:
    ../src/threads_a_gogo.cc:202: error: cannot convert ‘ev_async*’ to ‘ev*loop*’ for argument ‘1’ to ‘void ev_async_send(ev_loop*, ev_async*)’
    ../src/threads_a_gogo.cc: In function ‘void eventLoop(typeThread*)’:
    ../src/threads_a_gogo.cc:281: error: cannot convert ‘ev_async*’ to ‘ev_loop*’ for argument ‘1’ to ‘void ev_async_send(ev_loop*, ev_async*)’
    ../src/threads_a_gogo.cc: In function ‘void destroyaThread(typeThread*)’:
    ../src/threads_a_gogo.cc:362: error: cannot convert ‘ev_async*’ to ‘ev_loop*’ for argument ‘1’ to ‘void ev_async_stop(ev_loop*, ev_async*)’
    ../src/threads_a_gogo.cc: In function ‘void Callback(ev_async*, int)’:
    ../src/threads_a_gogo.cc:423: error: cannot convert ‘ev_async*’ to ‘ev_loop*’ for argument ‘1’ to ‘void ev_async_send(ev_loop*, ev_async*)’
    ../src/threads_a_gogo.cc: In function ‘v8::Handle<v8::Value> threadEmit(const v8::Arguments&)’:
    ../src/threads_a_gogo.cc:646: error: cannot convert ‘ev_async*’ to ‘ev_loop*’ for argument ‘1’ to ‘void ev_async_send(ev_loop*, ev_async*)’
    ../src/threads_a_gogo.cc: In function ‘v8::Handle<v8::Value> Create(const v8::Arguments&)’:
    ../src/threads_a_gogo.cc:685: error: invalid conversion from ‘void (*)(ev*async*, int)’ to ‘void (_)(ev_loop_, ev_async*, int)’
    ../src/threads_a_gogo.cc:686: error: cannot convert ‘ev_async*’ to ‘ev_loop*’ for argument ‘1’ to ‘void ev_async_start(ev_loop*, ev_async\*)’
    Waf: Leaving directory `/Users/kojunghyun/kojh/MyDev_MBP/NodeJs_Dev/node-threads-a-gogo/build'
    Build failed: -> task failed (err #1):
    {task: cxx threads_a_gogo.cc -> threads_a_gogo_1.o}
    koui-MacBook-Pro:node-threads-a-gogo kojunghyun$

<h3> <span style="color:{{site.span_h3_color}}"> 
해결  
</span> </h3>

threads_a_gogo.cc 파일 제일 위쪽 부분에 다음을 선언한다.

```cpp
#define NODE_WANT_INTERNALS 0
```

<h3> <span style="color:{{site.span_h3_color}}"> 
Why?
</span> </h3>

ev 소스는 잘 모르지만 왜 컴파일 오류가 나는지만 알아보자.
먼저 오류가 나는 함수의 정의는 다음과 같다.

```cpp

void
ev_async_send (EV_P_  ev_async *w)
{
    w->sent = 1;
    evpipe_write (EV_A_  &async_pending);
}
```

그리고 다음처럼 정의되어 있다.

```cpp
# define EV_P struct ev_loop *loop

# define EV_P* EV_P,
```

즉, 위의 함수는 `ev_async_send (struct ev_loop *loop, ev_async *w)` 원형을 가진다.
그런데, node.h 의 하단부에 다음과 같은 전처리문이 존재한다.

```cpp
#if !defined(NODE_WANT_INTERNALS) && !defined(\_WIN32)

# include "ev-emul.h"

# include "eio-emul.h"

#endif
```

`NODE_WANT_INTERNALS` 가 정의 안되있는 경우, `ev-emul.h`, `eio-emul.h` 헤더를 포함하게 되고, `ev-emul.h` 에서는 기존 정의를 무시하게 만들고 있다 (뭔지?).

```cpp
#undef EV_P
#undef EV_P_
...
#define EV_P void
#define EV_P_
```

NODE_WANT_INTERNALS 이게 어떤 용도인지 궁금... 아무튼 컴파일은 성공~
