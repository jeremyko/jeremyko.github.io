---
layout: post
title: localtime_r 은 thread safe 하지 않다? deadlock발생 경우.
date: '2017-12-03T17:25:00.000+09:00'
tags:
    - fork
    - localtime_r
    - linux
    - c
modified_time: '2022-03-26T20:57:36.541+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-9051402071414545648
blogger_orig_url: https://jeremyko.blogspot.com/2017/12/localtimer-thread-safe-deadlock.html
---

며칠 전 개발하면서 겪었던 상황인데,
fork 된 child process에서는 localtime_r 과 같은 시간 관련 함수를 사용할때 유의해야 한다.

localtime 함수는 시간을 구하는 함수이다.
이 함수는 내부적으로 static 변수에 값을 저장하기 때문에 thread safe하지 않다.
그래서 멀티 쓰레드 환경에서는 localtime_r 을 사용하게 된다.
여기까지는 일반적으로 다들 알고 있는 내용일 것이다.

그런데 localtime_r 함수를 사용해도 문제가 되는 경우가 발생했다.
fork 함수와 multi thread 가 만나면 문제가 된다.

다음 코드처럼, run_child_process 라는 함수가 멀티쓰레드에서 호출되며,
함수 내부에서는 fork를 이용해서 자식 프로세스를 생성하고 있는 경우를 생각해보자.

만약 fork 된 자식 process에서 logging이 발생되는 경우, deadlock 이 발생할수도 있다
(logging 을 처리하는 LOGD는 내부적으로 localtime_r 함수를 사용 중이다).

```cpp
void SomeUtilClass::run_child_process(const char* command)
{
    LOGD("run this command [%s]", command); //logging macro이다.내부적으로 localtime_r 을 호출한다.
    pid_t pid = fork(command);
    if(pid >0) {
        //parent
    } else if(pid == 0 ) {
        //child
        LOGD("child started..."); //XXX fork 된 child에서 logging 호출!
        ....
    }
    .....
}
```

멀티쓰레드 문제가 대부분 그렇듯.. 항상 deadlock 이 발생하는게 아니고,
운이 나쁘면 종종 발생한다는것이 큰 문제다.

이 코드가 문제가 되는 이유는, localtime_r 호출 시 내부적으로 lock이 걸리는데 fork와 같이 사용되면서
lock 이 잠긴 상태에서 다시 lock을 잡게 되는 경우가 발생 가능하기 때문이다(즉 deadlock).

<span style="color:{{site.span_emphasis_color}}">
즉, 다음 순서가 재현되면 deadlock 이 발생한다.
</span>

1. 부모 프로세스가 멀티 쓰레드를 생성하고 각 쓰레드가 run_child_process 를 호출한다

2. 부모 프로세스가 logging을 호출하게 된다.

3. fork된 자식 프로세스도 logging 을 호출한다. 그런데 fork된 자식 프로세스는 부모의 lock도 상속받는다.
   (The entire virtual address space of the parent is replicated in the child,
   including the states of mutexes, condition variables, and other pthreads objects;)

4. 이때 운 나쁘게 부모가 잡은 lock이 풀리기 전(localtime_r 호출이 종료되기 전) 에
   자식이 다시 lock을 얻는 시도(localtime_r 호출)를 하게되면 deadlock에 빠진다.

fork 를 사용하는 경우에는, 부모 로부터 뭔가를 상속 받는다는 것을 유념하고, 조심해야겠다..
