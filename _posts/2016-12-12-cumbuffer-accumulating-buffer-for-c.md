---
layout: post
title: 'CumBuffer : accumulating byte buffer for c++ '
date: '2016-12-12T22:55:00.002+09:00'
tags:
    - c++
    - CumBuffer
modified_time: '2022-03-26T21:02:08.367+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-1865585647397958715
blogger_orig_url: https://jeremyko.blogspot.com/2016/12/cumbuffer-accumulating-buffer-for-c.html
---

네트워크 TCP 프로그램인 경우에 임의 길이의 작은 데이터 조각을 수신해서 누적(버퍼링)시키다가 일정 길이가 수신되면 완전한 하나의 packet으로 보고 해당 데이터를 처리하는 경우가 꼭 필요하다.

지금까지는 그냥 new, calloc 등으로 데이터를 수신할때마다 동적으로 할당해서 기존 데이터를 복사하는 방식으로 해왔는데 (동적할당 제거에 의한 성능개선이 크지 않았기에), 이번에 동적 할당없이 사용할 수 있는 버퍼를 구현해 보았다. 일종의 ring buffer 라고도 할수 있다.

[https://github.com/jeremyko/CumBuffer](https://github.com/jeremyko/CumBuffer)

<h3> <span style="color:{{site.span_h3_color}}"> 
기본 사용법
</span> </h3>

```cpp
#include "CumBuffer.h"

CumBuffer buffering;

if(cumbuffer_defines::OP_RSLT_OK == buffering.Init(9)) //버퍼길이 9 byte로 초기화
{
    return false;
}

char data   [100];
char dataOut[100];

//3 byte 저장
memset(data, 0x00, sizeof(data));
memcpy(data, (void*)"aaa", 3);
if(cumbuffer_defines::OP_RSLT_OK != buffering.Append(3, data)) {
    return false;
}

//4 byte 저장
memset(data, 0x00, sizeof(data));
memcpy(data, (void*)"abbb", 4);
if(cumbuffer_defines::OP_RSLT_OK != buffering.Append(4, data)) {
    return false;
}

if(buffering.GetCumulatedLen()!=7) //현재까지 저장된 데이터 길이 7
{
    return false;
}

//4 byte 추출
memset(dataOut, 0x00, sizeof(dataOut));
if(cumbuffer_defines::OP_RSLT_OK != buffering.GetData(4, dataOut)) {
    return false;
}

if( strcmp("aaaa", dataOut)!=0) {
    return false;
}

//3 byte 추출
memset(dataOut, 0x00, sizeof(dataOut));
if(cumbuffer_defines::OP_RSLT_OK != buffering.GetData(3, dataOut)) {
    return false;
}

if( strcmp("bbb", dataOut)!=0) {
    return false;
}
```
