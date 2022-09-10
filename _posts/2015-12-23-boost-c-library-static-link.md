---
layout: post
title: boost c++ library static link 컴파일 방법
date: '2015-12-23T13:47:00.001+09:00'
tags:
    - c++
    - boost
modified_time: '2022-03-26T20:58:42.625+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-4764141462947394704
blogger_orig_url: https://jeremyko.blogspot.com/2015/12/boost-c-library-static-link.html
---

막상 필요해서 찾다보니 잘 정리가 안되있어서 나름 정리해본다. boost 라이브러리를 이용해서 작성된 프로그램을 boost 가 설치 안된 환경에서 사용해야 하는 경우, static link 로 컴파일 해서 사용할수 있다.

`.bash_profile` 설정은 다음과 같다.

    export BOOST_ROOT=/CG/USER/kojh/MyApps/boost_1_57_0
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$BOOST_ROOT/stage/lib

<h3> <span style="color:{{site.span_h3_color}}"> 
static library 생성
</span> </h3>

`b2` 실행 옵션을 다음 처럼 주면, 기존에 동적 라이브러리가 위치하던 곳(boost 설치시 결정된)에 static 라이브러리 파일등이 생성된다.

    b2 link=static

<h3> <span style="color:{{site.span_h3_color}}"> 
컴파일
</span> </h3>

자신이 사용한 boost lib 를 static 하게 link 해서 컴파일 하면 된다( 다음 예는 boost thread 를 사용하는 경우이다).

    g++ -pthread -O2 -I $BOOST_ROOT boostQueue.cpp -o boostQueue $BOOST_ROOT/stage/lib/libboost_system.a $BOOST_ROOT/stage/lib/libboost_thread.a -lrt
