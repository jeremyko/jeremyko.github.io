---
layout: post
title: string.h 없이 strtok 사용시 SIGSEGV 발생하는 경우
date: '2013-11-08T16:34:00.001+09:00'
tags:
    - gcc
    - c++
    - strtok
modified_time: '2014-02-21T18:12:41.230+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-4797062869374216962
blogger_orig_url: https://jeremyko.blogspot.com/2013/11/stringh-strtok-sigsegv.html
---

다음 코드는 컴파일도 잘되는듯 하고, 간단해서 문제 없이 실행될것 처럼 보인다.

그런데 실행을 해보면 `SIGSEGV`가 발생하면서 죽어버리는 현상이 발생한다.

```cpp
#include <stdio.h>
//#include <string.h> //!!

void testFunc ()
{
    char *pSep = "|";
    char szData [1024];
    char *p = NULL;

    snprintf( szData, sizeof(szData), "%s", "1|2,2,4");
    printf( "data[%s]\n", szData );

    p = strtok(szData, pSep );
    if(p == NULL) {
        return;
    }
    printf( "p[%s]\n", p );
}

int main()
{
    testFunc ();
    return 0;
}
```

    /USER/kojh/TestDev]gcc -Wall -g -o test test.c
    test.c: In function 'testFunc':
    test.c:18: warning: implicit declaration of function 'strtok'
    test.c:18: warning: assignment makes pointer from integer without a cast


    /USER/kojh/TestDev]gdb test
    GNU gdb (GDB) Red Hat Enterprise Linux (7.2-48.el6)
    Copyright (C) 2010 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
    and "show warranty" for details.
    This GDB was configured as "x86_64-redhat-linux-gnu".
    For bug reporting instructions, please see:
    <http://www.gnu.org/software/gdb/bugs/>...
    Reading symbols from /OFCS/USER/kojh/TestDev/test...done.
    (gdb) run
    Starting program: /USER/kojh/TestDev/test
    data[1|2,2,4|]
    Program received signal SIGSEGV, Segmentation fault.
    0x0000003262048097 in vfprintf () from /lib64/libc.so.6
    Missing separate debuginfos, use: debuginfo-install glibc-2.12-1.25.el6.x86_64
    (gdb) bt
    #0  0x0000003262048097 in vfprintf () from /lib64/libc.so.6
    #1  0x000000326204f02a in printf () from /lib64/libc.so.6
    #2  0x00000000004005ef in testFunc () at test.c:23
    #3  0x0000000000400602 in main () at test.c:28
    (gdb)

이 문제는 정확한 헤더를 포함하지 않은 경우, 컴파일러가 임의로 strtok함수의 prototype을
가정하기 때문이다. 즉, 컴파일러 경고를 보면,

    test.c:18: warning: assignment makes pointer from integer without a cast

가 출력되는데, 다음 부분에서 strtok 함수가 int를 리턴하는것으로 잘못 해석을 하고 있다.

```cpp
p = strtok(szData, pSep );
```

하지만 함수 원형은 알다시피 `char*` 를 리턴하는것이 맞다.

```cpp
char *strtok(char *str, const char *delim);
```

`#include <string.h>` 를 include하고 다시 컴파일, 실행시에는 정상처리 된다.

    /USER/kojh/TestDev]./test
    data[1|2,2,4]
    p[1]
