---
layout: post
title: 'close(0) 로 인한 stdin close : getline, getchars 동작 이상'
date: '2017-02-10T00:43:00.001+09:00'
tags:
    - c++
    - 삽질기록
modified_time: '2017-02-13T21:12:00.992+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-3988169717043903330
blogger_orig_url: https://jeremyko.blogspot.com/2017/02/close0-stdin-close-getline-getchars.html
---

왜 getline() 함수가 즉시 리턴되는가? 에 대한 삽질의 기억...

```cpp
class SomeClass
{
    ...
    int nSockFd_  {0}; //초기화를 0 으로 한 경우.
    ...
};

void SomeClass::SomeMethod1()
{
    ....
    //코드 어딘가에서 다음을 호출했는데... 아직 nSockFd_ 가 할당안된 경우
    close(nSockFd_); //--> close(0) --> stdin 을 닫아버림 - - ;;
    ...
}

void SomeClass::SomeMethod2()
{
   ....
   std::string line="";
    while(true)
    {
        std::cin.clear();
        getline(std::cin, line); //여기서 문제 발생, 즉시 리턴됨
        std::cout << "msg:" << line << std::endl;
    }
}
```
