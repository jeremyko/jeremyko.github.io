---
layout: post
title: protothread
date: '2011-09-21T17:49:00.000+09:00'
tags:
    - protothread
    - c++
modified_time: '2012-04-04T17:53:51.383+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-8924321941478283324
blogger_orig_url: https://jeremyko.blogspot.com/2012/04/protothread.html
---

protothred 는 주로 쓰레드 사용의 장점이 필요하지만, 기존 쓰레드 생성시의 부하,
즉 쓰레드별 스택 생성됨으로 인해 메모리 사용양이 문제가 되는 임베디드 시스템등에서
사용할 목적으로 작성되었다.

쓰레드별 스택이 불필요하므로 매우 경량이며 (쓰레드당 2 바이트 메모리만 필요), 소스코드의 복잡도 감소등을 장점으로 내세운다.

C 언어로 작성된 몇개의 헤더파일로만 이루어져 있으며 헤더 파일 내부는
C 매크로와 먼저 설명한 switch 구문의 마법을 활용하고 있다.

일반적인 쓰레드는 커널에 의해 스케쥴링이 수행되지만, protothread는 프로그램 내부에서
각 쓰레드들을 스케쥴링하는 부분을 필요로 한다.

protothread 스케쥴링이란 쓰레드 우선 순위에 따라, 직접 수행 순서를 지정해서
어플리케이션에서 루프를 돌면서 함수를 차례로 호출하는 것이다.

기존 쓰레드에서는 커널에 의해 CPU 처리시간을 할당받으면서 쓰레드간 context 전환이
발생되면서 작업이 수행되는 반면, protothread는 어플리케이션이 직접 반복적인 함수호출을 하고
함수 호출의 전환(이것이 기존 쓰레드에서의 context전환 역활)을 통해서 여러작업을 같이
수행하게 되는 차이점이 있다.

이 덕분에 쓰레드별 스택은 불필요한데, 또 이런 이유로 쓰레드 내부의 지역변수 사용은 제한받게 된다.
즉, 변수값이 유지되지 않기 때문에 정적 지역변수등을 이용하던지 해야한다.

문서를 보면, 이러한 스케쥴링에 대한 특별한 지침같은 것은 없고, protothread를 사용하는
프로그램에 전적으로 맡기고 있다.

예를 들어, 이벤트 처리방식의 시스템에서 protothread를 사용하는 경우라면, 특정 이벤트 처리시점에 스케쥴링을 해주고, 스케쥴링 루프를 빠져나오기 위해서
모든 쓰레드 처리 완료를 확인하는등의 절차가 필요 할것이다.

그리고 모든 쓰레드 작업을 대체할수 없기 때문에 이를 고려해야한다.

예를 들어 소켓을 이용한 수신작업이 쓰레드 작업인 경우, 이를 protothread로 완전히 대체할수는 없다. (쓰레드 내에서 이벤트를
기다리는 경우라면 사용이 제한된다)

protothread의 기본적인 사용법은 다음과 같다. (protothread홈페이지에 있는 소스를 그대로 가져왔다)

```cpp
#include "pt.h"

static int counter;
static struct pt example_pt;

static PT_THREAD(example(struct pt *pt))
{
    PT_BEGIN(pt);
    printf("This should be executed just once!\n");  //이부분은 테스트목적으로 추가함.
    while(1) {
        PT_WAIT_UNTIL(pt, counter == 1000);
        printf("Threshold reached\n");
        counter = 0;
    }

    PT_END(pt);
}

int main(void)
{
    counter = 0;
    PT_INIT(&example_pt);
    while(1) { // 직접 스케쥴링을 해줘야한다.
        example(&example_pt);
        counter++;
    }
    return 0;
}
```

이 예제에서 가장 중요한 것은 헤더파일에 정의된 PT\_ 시작되는 매크로들로서,
파일을 열어보면 약간의 indirection을 걸치지만 실제 정의는 본질적으로 아래와 같다.

```cpp
struct pt { unsigned short lc; }; //이것이 2byte의 정체...
#define PT_THREAD(name_args)  char name_args
#define PT_BEGIN(pt)          switch(pt->lc) { case 0:
#define PT_WAIT_UNTIL(pt, c)  pt->lc = __LINE__; case __LINE__: \
                              if(!(c)) return 0
#define PT_END(pt)            } pt->lc = 0; return 2
#define PT_INIT(pt)           pt->lc = 0
```

자 그럼 이 매크로들이 적용되어 확장되는 코드는 아래와 같다.

```cpp
//example 함수의 예
static char example(struct pt *pt)
{
    switch(pt->lc) { case 0:
    printf("This should be executed just once!\n");  // --- (1)
    while(1) {
        pt->lc = 12; case 12:
        if(!(counter == 1000)) return 0;
            printf("Threshold reached\n");
            counter = 0;
        }
    } pt->lc = 0; return 2;
}
```

switch 마법이 다시 등장했다. 이함수가 처음 호출될때는 case 0을 수행하고
그이후부터는 case 12 부분을 수행하게 된다.

위의 (1) 부분은 의도한대로 한번만 수행이 된다.
그런데 12 란 숫자는 하드코딩된 숫자가 아니고 `__LINE__` 전처리 매크로를 활용해서
일반적인 로직으로 처리되게끔 교묘하게 작성된 것이다.

이러한 함수가 여러번 호출될때마다, 수행되는 위치가 달라지는것은 coroutine과 동일하다.

protothread를 사용하는데 있어서의 주의점은 다음과 같다.

1. 앞서 설명한, 지역변수 사용 제한

2. protothread와 switch 구문을 동시에 사용할수 없다. 사용시 컴파일 타임 에러가 누락되는 경우가 발생한다고 함. 그리고 현재까지 해결책은 없다고 한다.
