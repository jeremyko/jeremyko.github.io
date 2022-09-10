---
layout: post
title: c++ thread pool
date: '2017-10-08T13:35:00.007+09:00'
tags:
    - cpp
    - c++11
modified_time: '2022-03-26T20:57:51.717+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-96266151791272400
blogger_orig_url: https://jeremyko.blogspot.com/2017/10/c-thread-pool.html
---

[https://github.com/jeremyko/kothreadpool](https://github.com/jeremyko/kothreadpool)

간단한 worker thread pool 정도만 필요해서 하나 만들어 보았다.
사용법은 다음과 같다.

```cpp
#include <iostream>
#include <string>
#include <atomic>
#include "KoThreadPool.hpp"

std::atomic<int> gSum1;
std::atomic<int> gSum2;


bool MyThreadWork() {
    gSum1++;
    return true;
}

class MyClass
{
    public:
        MyClass()  {} ;
        ~MyClass() {} ;

        bool MyThreadWork() {
            gSum2++;
            return true;
        }
};


int main()
{
    KoThreadPool tpool;

    if( ! tpool.InitThreadPool() ) {
        std::cerr << "Error : Init" << "\n";
        exit(1);
    }
    tpool.SetWaitingCnt(2); //set total work count
    gSum1 = 0;
    gSum2 = 0;
    MyClass myclass;

    //class member
    std::function<void()> temp_func1 = std::bind( &MyClass::MyThreadWork, &myclass)  ;
    tpool.AssignTask(temp_func1 )  ;

    //free function
    std::function<void()> temp_func2 = std::bind( &MyThreadWork )  ;
    tpool.AssignTask(temp_func2 )  ;

    //wait all 2 works done.
    tpool.WaitAllWorkDone(); // --> XXX blocking call.

    //At the end of the program, exit the thread pool.
    tpool.Terminate(); //graceful terminate : wait until all work done
    //tpool.Terminate(true); //terminate immediately

    std::cout << "gSum1 =" << gSum1 << "\n";
    std::cout << "gSum2 =" << gSum2 << "\n";

    return 0;
}
```
