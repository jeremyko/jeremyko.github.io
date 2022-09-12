---
layout: post
title: Lvalue 와 Rvalue 정리
date: '2012-08-10T17:56:00.000+09:00'
tags:
    - c++
    - Lvalue 와 Rvalue
modified_time: '2022-03-26T21:03:39.850+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-4825013922714553472
blogger_orig_url: https://jeremyko.blogspot.com/2012/08/lvalue-rvalue.html
---

c++11 의 기능중에 Rvalue references 을 보다 보니, Lvalue 와 Rvalue 의 판단 기준을 다시 한번 정리해야 할것 같다.

내가 지금까지 이것들을 구분하는 기준은 대입연산자 왼쪽, 오른쪽으로 기준으로 원시적 판단을 했었는데, C++ 의 새로운 기능을 보다보니...머릿속이 헷갈리기 시작한다.

먼저, 일반적이고 간단한 예를 들어보면 :

```cpp
int i = 3;
```

이경우엔, Lvalue = i, Rvalue = 3 이다.. 지금까지 판단 기준이 통하는군...

그런데 다음 경우는 어떠한가?

```cpp
int i = 1;
int j = 2;
i = j; // Lvalue = i, Rvalue = j ???
```

위 경우에는 Rvalue 는 없다.

`i`,`j`는 모두 Lvalue 이며, 컴파일러에 의해 `lvalue-to-rvalue conversion` 이 발생하면서 `j` 가 마치 Rvalue 처럼 작동할 뿐이다.

그래서, 명확한 정의를 찾던 중 [http://www.codeproject.com/Articles/313469/The-Notion-of-Lvalues-and-Rvalues](http://www.codeproject.com/Articles/313469/The-Notion-of-Lvalues-and-Rvalues)
에서 좋은 내용을 찾았다. 알기 쉽게 잘 설명되어 있어서 정리해본다.

---

### introduction

필자는 Lvalues and Rvalues 에 대해서 심각하게 생각해본적이 없으며, 그런것이 문제되는 경우는 컴파일시 에러가 나는 경우가 대부분이었으며, 이 또한 쉽게 에러를 수정할수 있었다.
즉 다음 코드에서처럼 말이다.

```cpp
int NextVal_1(int* p) { return *(p+1); }
int* NextVal_2(int* p) { return (p+1); }

int main()
{
    int a[] = {1,2,3,4,5};
    NextVal_1(a) = 9; // 에러. left operand must be l-value
    *NextVal_2(a) = 9; // Fine. Now a[] = {1,9,3,4,5}
}
```

위 코드를 통해서 내가 말하는것을 잘 이해했길 바란다. 그런데 내가 C++0X 의 RValue reference 부분을 읽기 시작했을때, 내 비젼과 확신이 조금씩 흔들리기 시작했다. 내가 Lvalue 로 당연히 여기던 것들이 Rvalue 로 보이기 시작했다. 이 글을 통해서 L & R value들에 관련된 다양한 개념들을 간략하게 정리하려 한다. 이것과 관련된 여러가지 정보들을 다시 구글링할 필요가 없도록 하나의 정보로 모으기 위해 노력하였음을 알아주길 바란다. 모든 크레딧은 원 저자들에게 돌린다.

### Definitions

객체는 하나의 저장 영역으로 간주되어질수 있다. 그리고 이 저장 영역은 관찰 가능하거나 변경 가능하거나, 혹은 그 두가지가 접근 지정자에 의해 결정될수 있다. 즉,

```cpp
int i; // i 에 연관된 저장영역은 Observable, Modifiable 모두 해당된다.
const int j = 8; // j 에 연관된 저장영역은 Observable 만 가능하며, 변경 불가이다.
```

더 진행하기 전에 다음 구절을 기억하기 바란다.

    "Lvalueness 혹은 Rvalueness 개념은 전적으로 표현법이며 객체와 무관하다".

이걸 좀더 간단하게 말하자면 :

```cpp
double d;
```

이제 `d` 는 단순히 `double` 타입의 객체이다 [그리고 d에 대해서 l/r 값을 따지는것은 이 시점에서는 무의미하다].

이제 다음처럼 표현된다면,

```cpp
d = 3.1414 * 2;
```

<span style="color:{{site.span_emphasis_color}}">
**모든 l/r 값 개념이 시작된다.**
</span>

여기서 우리는 수치 연산으로 임시 값을 구한후 `d` 에 대해 대입식을 쓰고 있고, 이 임시값은 세미콜론 이후에는 사라질 것이다.

여기서 구분가능한 메모리 위치를 가르키는 `'d'`는 `Lvalue` 이다. 그리고 `(3.1414*2)` 로 계산되는 임시값은 `Rvalue` 이다.

자 이시점에 L/RValue를 한번 정의해보자.

-   **Lvalue** : Lvalue 는 객체를 참조하는 표현식이다 [메모리 위치를 가지고 있다] [The C Programming Language - Kernighan and Ritchie].

-   **Rvalue** : C++ 표준은 r-value 정의할때, 제외 개념으로 처리하고있느데, 다음과 같다.

        "모든 표현식은 Lvalue 거나 Rvalue이다"

    고로, Rvalue 는 Lvalue 가 아닌 모든것이다. 정확하게 말하자면, 구분 가능한 메모리 영역을 가지는 객체를 나타낼 필요가 없는 표현식이다(임시로 존재하는것일수 있다).

### Points on Lvalues and Rvalues

1.  Numeric literals, 3 과 3.14159, 이것들은 Rvalues 이다. character literals, 예를 들면 'a' 도 마찬가지이다.

2.  enumeration 상수 구분자는 Rvalue 이다. 예를 들면:

    ```cpp
    enum Color { red, green, blue };
    Color enumColor;
    enumColor = green; // Fine
    blue = green; // Error. blue is an Rvalue
    ```

3.  binary + 연산자의 결과는 항상 Rvalue 이다.

    ```cpp
    m + 1 = n // Error. 왜냐하면 (m+1) 는 Rvalue.
    ```

4.  단항 & (address-of) 연산자는 그것의 피연산자로 Lvalue 를 필요로 한다. 즉, &n는 n이 Lvalue 인 경우에만 유효한 표현식이다. 그러므로, &3 같은 표현식은 에러이다. 다시한번 말하자면, 3 은 객체를 참조하고 있지 않다.그러므로 그것은 주소를 이용해서 불러낼수 없다. 비록 단항 & 연산자가 피연산자로 Lvalue 를 필요로 하지만, 그 결과는 Rvalue 이다.

    ```cpp
    int n, *p;
    p = &n; // Fine
    &n = p; // Error: &n is an Rvalue
    ```

5.  unary & 과는 대조적으로, unary * 는 그 결과로 lvalue 를 만들어 준다. non-null(유효한) 포인터 p 는 항상 객체를 가르킨다. 그러므로 *p 는 lvalue 이다. 예를 들면:

    ```cpp
    int a[N];
    int *p = a;
    *p = 3; // Fine.

    // 그 결과가 Lvalue 이긴 하지만, 피 연산자는 Rvalue 가 될수도 있다.
    *(p + 1) = 4; // Fine. (p+1) 는 Rvalue
    ```

6.  Pre-increment 연산자 표현식의 결과는 LValues

    ```cpp
    int nCount = 0; // nCount 는 영속 객체를 나타내며 그러므로 Lvalue 이다.
    ++nCount; // 이 표현식은 Lvalue 이다.왜냐하면
    // 이것은 변경이후 nCount 객체를 가르키기 때문이다.

    // 이것이 Lvalue인 것을 증명하기 위해, 다음 연산을 할수 있다
    ++nCount = 5; // Fine. nCount 는 5 이다.
    ```

7.  리턴타입이 오직 참조인 경우에만 함수 호출은 Lvalue 이다.

    ```cpp
    int& GetBig(int& a, int& b)    // 함수 호출을 Lvalue 로 만들기 위해 참조를 반환
    {
        return ( a > b ? a : b );
    }

    void main()
    {
        int i = 10, j = 50;
        GetBig( i, j ) *= 5;
        // 여기서, j = 250. GetBig() 은 j의 참조를 리턴한다.
        // 그리고 그것에 5가 곱해진것으로 저장된다.
    }
    ```

8.  참조는 그냥 이름이다. 그래서 Rvalue 에 묶인 참조 그 자체는 Lvalue 이다.

    ```cpp
    int GetBig(int& a, int& b) // 함수 호출을 Rvalue 로 만들기 위해 int를 리턴
    {
        return ( a > b ? a : b );
    }

    void main()
    {
        int i = 10, j = 50;
        const int& big = GetBig( i, j );
        // 'big'를 GetBig()의 리턴값(Rvalue)에 대해 Lvalue로 바인딩한다.

        int& big2 = GetBig(i, j); // Error. big2 가 const가 아니므로
        // temporary 는 non-const reference 에 바인드 불가.
    }
    ```

9.  Rvalues 는 temporaries 이고 메모리 영역을 가르킬 필요가 없다. 그러나 어떤 경우에는 메모리를 가르킬수 있다. 하지만 이런 임시값에 대해서 작업하는것은 권장되지 않는다

    ```cpp
    char* fun() { return "Hellow"; }

    int main()
    {
        char* q = fun();
        q[0]='h'; // 예외발생, fun() 이 임시 메모리를 리턴하는데 거기 접근하려 한다.
    }
    ```

10. 후위 증가(Post-increment) 연산자 표현식의 결과는 RValue 이다.

    ```cpp
    int nCount = 0; // nCount 는 영속 객체를 나타내므로 Lvalue
    nCount++ // 이 표현식은 Rvalue이다. 객체의 값을 복사하고,
    // 변경한후 임시 복사를 리턴하기 때문이다.

    // 이것이 Rvalue라는것을 증명하기 위해, 다음 연산을 할수 있다.
    nCount++ = 5; //Error
    ```

### 정리해보면 다음과 같이 말할수 있다

만약 우리가 표현식의 주소를 안전하게 얻을수 있다면, 그것은 lvalue 표현식이다. 그렇지 않다면 그것은 rvalue 표현식이다.
{: .notice--info}

**Note** : Lvalues 와 Rvalues 모두 변경가능 혹은 변경 불가일수 있다.여기 예제가 있다 :
{: .notice--warning}

```cpp
string strName("Hello"); // modifiable lvalue
const string strConstName("Hello"); // const lvalue
string JunkFunction() { return "Hellow World"; /* catch this properly */}//modifiable rvalue
const string Fun() { return "Hellow World"; } // const rvalue
```

### Conversion between Lvalues and Rvalues

Rvalue를 필요로 하는곳에 Lvalue인 것이 사용될 수 있을까? 그렇다 가능하다.예를 들면,

```cpp
int a, b;
a = 8;
b = 5;
a = b;
```

이 = 표현식은 Lvalue 인 `b` 를 Rvalue 로 사용한다. 이 경우 컴파일러는 b에 저장된 값을 얻기 위해 `lvalue-to-rvalue conversion` 라고 불리는 것을 수행한다.

그럼 Lvalue 가 필요한곳에 Rvalue 가 사용될수 있을까? **아니 그것은 불가능하다.**

```cpp
3 = a // Error. Lvalue 가 필요한곳에 3이라는 RValue가 사용됨
```

### Acknowledgments

이 정보를 취합하고 구성하는것을 흔쾌히 도와준 Clement Emerson에게 감사한다.

### External resources

[http://msdn.microsoft.com/en-us/library/f90831hc.aspx](http://msdn.microsoft.com/en-us/library/f90831hc.aspx)  
[http://www.eetimes.com/discussion/programming-pointers/4023341/Lvalues-and-Rvalues](http://www.eetimes.com/discussion/programming-pointers/4023341/Lvalues-and-Rvalues)
