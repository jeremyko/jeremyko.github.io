---
layout: post
title: g++ std::thread -pthread option?
date: '2014-05-15T10:47:00.001+09:00'
tags:
    - c++
    - '-pthread'
    - g++
    - std::thread
modified_time: '2022-03-26T21:00:43.971+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-1658289456420714993
blogger_orig_url: https://jeremyko.blogspot.com/2014/05/g-stdthread-pthread.html
---

std::thread 를 이용한 프로그램을 리눅스 환경에서 g++ 을 이용, 컴파일 하는경우 -pthread 옵션이 필요하다.
다음과 같은 샘플 프로그램이 있다고 해보자.

```cpp
//std_thread_test.cpp
#include <thread>
#include <iostream>

void thread_func()
{
    std::cout<<"hello, I am a thread!"<<std::endl;
}

int main()
{
    std::thread t(thread_func);
    t.join();
}
```

컴파일시에 다음처럼 -pthread 옵션을 설정해야한다.

    g++ -std=c++11 -pthread -o std_thread_test std_thread_test.cpp

옵션을 주지 않는 경우엔 컴파일은 되지만 실행시 core dump가 발생한다.
사용된 g++의 버전은 g++ 4.8.2 으로 c++11 의 기능을 대부분 지원한다.
그렇다면 의문점이 생긴다. c++11 을 지원한다는 g++에 왜 -pthread 란 옵션이 필요한가?
언어 차원에서 지원된다는데 왜 pthread 라이브러리 이런게 보이는건지..?

좀 생각해보니 spec과 구현의 문제라는걸 알게 되었다.

c++11 의 thread specification을 g++에서는 pthread 라이브러리를 사용해서 구현한 것일뿐이다.
(쓰고 보니 너무나 당연한 말인 듯 - - )

이건 g++ -v 을 실행해보면 확인할수 있다.

![blog-image](/assets/img/20140515-1.PNG)

Thread model로 posix 를 사용한다 (좀 허무^^)
