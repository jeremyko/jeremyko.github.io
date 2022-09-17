---
layout: post
title: Member Function Pointers 활용
date: '2009-10-20T17:55:00.000+09:00'
tags:
    - c++
    - Member Function Pointers
modified_time: '2016-09-09T17:52:19.014+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-8197493198030544601
blogger_orig_url: https://jeremyko.blogspot.com/2009/10/member-function-pointers.html
---

어떤 변수의 값에 따라서 클래스 멤버함수를 각각 호출해야하는 상황이라면, 아마 다음처럼 코딩을 하게 될것이다.

```cpp
SomeClass::SomeMethod( int Somevalue  )
{
    ...
    if( 0 == Somevalue ) {
        memFunc_01( SomeArg ) ; // 입력인자 SomeArg 로 멤버함수1 호출
    }
    else if( 1 == Somevalue ) {
        memFunc_02( SomeArg ) ; // 입력인자 SomeArg 로 멤버함수2 호출
    }
    ... 생략
    else if( 9 == Somevalue ) {
        memFunc_10( SomeArg ) ; // 입력인자 SomeArg 로 멤버함수10 호출
    }
    ...
}
```

혹은 다른 방법(switch..case)을 이용할수 있다.

하지만 클래스 멤버 함수 포인터를 활용하면, 계속 반복되는 비교문을 없애고 효율적인 코드 작성이 가능하다.

```cpp
SomeClass::SomeMethod(int Somevalue )
{    ...
    // fPAryMemFunc 라는 멤버함수 포인터 배열을 선언한다.
    // 사용시에 1번만 초기화 된다
    static BOOL( SomeClass::* fPAryMemFunc[] )( SomeArg ) =
    {
        & SomeClass:: memFunc_01,
        & SomeClass:: memFunc_02,
        & SomeClass:: memFunc_03,
        ...
        & SomeClass:: memFunc_10
    }

    //실제 멤버 함수 호출시에는 다음처럼 한다.
    (this->* fPAryMemFunc[ Somevalue ]) (SomeArg );
}
```

이걸 채팅서버에 적용해봤는데 , 클라이언트의 요청이 들어오면 헤더의 사용목적을 보고 일일이 구분 처리하는게 아니고 헤더의 메시지 용도에 배열의 인덱스를 저장하게 하고 바로 해당 인덱스의 멤버 함수를 호출하게 하는것이다.
