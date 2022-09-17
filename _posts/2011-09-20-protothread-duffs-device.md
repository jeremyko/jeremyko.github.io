---
layout: post
title: "protothread : Duff's device"
date: '2011-09-20T17:50:00.000+09:00'
tags:
    - c++
    - protothread
    - c
    - Duff's device
modified_time: '2022-03-26T21:04:01.227+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-1189253559528628555
blogger_orig_url: https://jeremyko.blogspot.com/2012/04/protothread-duffs-device.html
---

protothread 란 것을 보던중에 아래와 같은 소스를 발견.

그동안 몰랐던 이상야릇한 C언어 switch 구문형태이다~

```cpp
int counter = 0;
int n = 0;

int somefun()
{
    switch(n){
        case 0:
        while(1){
            n = 12;
            case 12:
            if(!(counter == 1000))
                return 0;
            printf("Threshold reached\n");
            counter = 0;
        }
    }
    n = 0;
    return 2;
}

int _tmain(int argc, _TCHAR* argv[])
{
    somefun(); // case 0
    somefun(); // case 12
    return 0;
}
```

찾은 정보들을 정리하면 다음과 같다. 이 마법같은 코드는 1983년도에 루카스 필름에 근무하던 프로그래머 Tom Duff 가 만든것으로... 아니.. 일단 이런건 skip하고... loop unrolling을 하기 위한 코드를 컴팩트하게 만들기 위해 등장한 기법이다.

loop unrolling은 이미 알고 있듯이, 루프를 좀더 풀어내서 점프, 비교되는 반복 횟수를 줄여서 수행속도를 올리는 최적화 기법이다.
고전적인 예로, 문자열을 복사하는 함수이다.

```cpp
void CopyArray(char* szArrSrc, char* szArrDest, int nCount)
{
    do
    {
        *szArrDest++ = *szArrSrc++;
    } while( --nCount > 0 );
}
```

이것을 loop unrolling을 하게되면

```cpp
int n = nCount / 8;
for (int i = 0; i < n; ++i)
{
    *szArrDest++ = *szArrSrc++;
    *szArrDest++ = *szArrSrc++;
    *szArrDest++ = *szArrSrc++;
    *szArrDest++ = *szArrSrc++;
    *szArrDest++ = *szArrSrc++;
    *szArrDest++ = *szArrSrc++;
    *szArrDest++ = *szArrSrc++;
    *szArrDest++ = *szArrSrc++;
}

// nCount 가 8배수 아닌경우, 남은 복사 수행
n = nCount % 8;
for (int i = 0; i < n; ++i)
{
    *szArrDest++ = *szArrSrc++;
}
```

이렇게 사용가능하다.
이것을 Tom Duff 는 두번째 루프를 없애기 위해 아래처럼 작성했다.

```cpp
// Duff's device 로 이름 붙여짐.
int mod = nCount % 8;
switch (mod)
{
    case 0: do { *szArrDest++ = *szArrSrc++;
    case 7: *szArrDest++ = *szArrSrc++;
    case 6: *szArrDest++ = *szArrSrc++;
    case 5: *szArrDest++ = *szArrSrc++;
    case 4: *szArrDest++ = *szArrSrc++;
    case 3: *szArrDest++ = *szArrSrc++;
    case 2: *szArrDest++ = *szArrSrc++;
    case 1: *szArrDest++ = *szArrSrc++;
    } while (--n > 0);
}
```

case문장에서 } while 을 만나면, 신기하게도 흐름이 상단의 do 부분으로 이동하면서 처리가 된다.
이런 기법을 이용한 coroutine 도 만들어 졌다.

## 참고

[http://goo.gl/UyxgP](http://goo.gl/UyxgP)
