---
layout: post
title: 'forever loop 를 활용한 error handling '
date: '2012-09-28T12:02:00.000+09:00'
tags:
    - c++
    - error handling
modified_time: '2012-12-11T12:38:00.144+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-5181087009568138510
blogger_orig_url: https://jeremyko.blogspot.com/2012/09/forever-loop-error-handling.html
---

함수내 에러처리에 대한 방법은 여러가지가 있겠으나, 최근 흥미로운 글 하나를 읽었다.

[http://www.codeofhonor.com/blog/error-handling-using-forever-loop](http://www.codeofhonor.com/blog/error-handling-using-forever-loop)

for loop 를 활용한 기법인데 정리해보면 다음과 같다.

먼저 일반적인 함수내 에러처리 방식을 보면, 아마 다음과 같을것이다.

```cpp
bool someFuncPlain ()
{
    if (작업1 수행이 실패한경우) {
        에러 발생시의 추가 작업수행...;
        return false;
    }

    if (작업2 수행이 실패한경우) {
        return false;
    }
    ....

    // 함수내 모든 처리 완료
    return true;
}
```

goto를 사용해서 다음처럼 처리할수도 있다.

```cpp
bool someFuncGoto ()
{
    if (작업1 수행이 실패한경우) {
        goto errExit;
    }

    if (작업2 수행이 실패한경우) {
        goto errExit;
    }
    ....

    // 함수내 모든 처리 완료
    return true;

    errExit:
        에러 발생시의 추가 작업수행;
    return false;
}
```

나는 위 예제처럼 함수 내 에러 처리 용도로 goto 를 사용하지 않지만,
다중 루프내에서 에러가 발생하여, 모든 루프를 탈출할 필요가 있을때 종종 사용하곤 한다.
goto사용이 바람직하지 않고 심지어 죄악시 여겨지기도 하지만,
이 정도는 그리 해롭지 않다고 생각하기 때문이다.

그리고 이 글의 주제인 for loop를 이용한 방법이다.

```cpp
bool someFuncForLoop ()
{
    bool result = false;
    for (;;) {
        if (작업1 수행이 실패한경우) {
            result = false;
            break;
        }

        if (작업2 수행이 실패한경우) {
            result = false;
            break;
        }
        ....
        // 함수내 모든 처리 완료
        result = true;
        break;
    } //for

    if (!result) {에러 발생시의 추가 작업수행};  //cleanup
    return result;
}
```

이 기법을 사용하면 에러처리 코드가 함수의 원래 기능에 대한 코드의 가독성을 해치지 않고, cleanup 코드가 항상 수행되기 때문에 좀 더 정확한 함수 작성을 가능하게 해준다고 주장하고 있다.
