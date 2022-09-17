---
layout: post
title: Expression template
date: '2009-08-03T18:03:00.000+09:00'
tags:
    - c++
    - RVO
    - 템플릿
    - Expression template
modified_time: '2016-09-09T17:52:18.993+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-6899605960372039055
blogger_orig_url: https://jeremyko.blogspot.com/2009/08/expression-template.html
---

Expression template 은 과학, 수치 계산용 프로그램 같은, 고도의 성능이 중요시되는 코드를 작성하기 위한 방법으로 C++의 템플릿 을 활용한 것이다.

템플릿은 컴파일 시점에 코드가 생성된다는것을 이용해서, 여러번의 순회(loop),대입이 필요한 연산을 컴파일 시점에 하나의 수식처럼 변환하고, 불필요한 임시객체 생성을 막기 위해 RVO (Return Value Optimization) 최적화도 이루에 질수 있게끔 코드를 만들었다.

만약 우리가 행렬을 표현하기 위한 클래스와, 행렬간 덧셈을 위한 연산자 오버로딩을 다음과 같이 구현했다 라고 가정하면,

```cpp
template<typename T, size_t n, size_t m>
class Matrix
{
public:
    Matrix(){}

    Matrix(const Matrix& rhs)
    {
        for(int i=0; i<n; ++i)
            for(int j=0; j<m; ++j)
                ElementAt(i,j) = rhs.ElementAt(i,j);
    }

    Matrix& operator=(const Matrix& rhs)
    {
        if( this != &rhs )
            for(int i=0; i<n; ++i)
                for(int j=0; j<m; ++j)
                    ElementAt(i,j) = rhs.ElementAt(i,j);

        return *this;
    }

    virtual ~Matrix() {}

    const T& ElementAt(size_t n, size_t m) const
    { return arrData[n][m]; }

    T& ElementAt(size_t n, size_t m)
    { return arrData[n][m]; }

private:
    // C-style array for efficiency and locality of reference
    T arrData[n][m];
};

template<typename T, size_t n, size_t m>
Matrix<T, n, m> operator+( const Matrix<T, n, m>& lhs,
const Matrix<T, n, m>& rhs )
{
    Matrix<T, n, m> matSum;
    for(int i=0; i<n; ++i)
    for(int j=0; j<m; ++j)
    matSum.ElementAt(i,j) = lhs.ElementAt(i,j) + rhs.ElementAt(i,j);
    return matSum;
}
```

그리고 동일한 행과 열을 가진 두 행렬 A, B 를 더하는 경우를 생각해본다.

```cpp
Matrix<double, n, m> S = A + B;
```

최적화 없는 경우 임시 객체가 생성되어, S 를 위한 복사생성자를 호출하게 될것이다. 하지만 대부분의 컴파일러는 RVO 최적화를 통해서 임시 객체 대신 S의 저장공간을 활용해 직접 복사 생성 될것이다. 그럼, 컴파일러가 최적화 해주는 경우는 제외하고 다음과 같은 경우를 고려해본다.

```cpp
Matrix<double, n, m> S ; // 먼저 정의
... S 사용
S = A + B; // 임시객체 생성
```

이 경우 RVO는 작동 하지 않는다.

대입연산자는 S의 이전 내용을 다른 내용으로 변환 하는 역활을 하며 임시객체가 생성된다. 컴파일러는 먼저 임시객체를 하나 만들어서 루프를 돌면서 A,B 행렬의 값을 더해서 그 임시객체에 대입하고, 대입 연산자에서 두번째로 루프를 돌면서 임시객체로부터 S 에 대입하게 된다.

이것은 시간, 공간적으로 매우 비효율적인 작업이다. 연산이 늘어날수로 임시객체와 루프사용이 점점 늘어나게 되어 성능이 중요한 프로그램등에서는 적합하지 않다. 성능을 위해서는 모든 임시객체의 삭제뿐만 아니라, n x m 횟수만큼 1번만 순회하면서 S 행렬을 만들게 하는 방법이 필요하다.

**바로 이것이 Expression template 이 하는 일이다.**

다음처럼 클래스를 수정해보자.

```cpp
template<typename T, size_t n, size_t m>
class Matrix;

template<typename T, size_t n, size_t m>
class EtMatrixAdd
{
public:
    EtMatrixAdd( const Matrix<T, n, m>& lhs,
    const Matrix<T, n, m>& rhs) : m_lhs(lhs), m_rhs(rhs
    )
    {
        printf("EtMatrixAdd Constructor! \n");
    }

    T ElementAt(size_t nn, size_t mm) const
    {
        printf("EtMatrixAdd ElementAt! \n");
        return m_lhs.ElementAt(nn, mm) + m_rhs.ElementAt(nn, mm);
    }

private:
    const Matrix<T, n, m>& m_lhs;
    const Matrix<T, n, m>& m_rhs;
};

template<typename T, size_t n, size_t m>
class Matrix
{
public:
    Matrix(){}
    Matrix(const Matrix& rhs)
    {
        printf("Copy Constructor! \n");

        for(int i=0; i<n; ++i)
            for(int j=0; j<m; ++j)
                ElementAt(i,j) = rhs.ElementAt(i,j);
    }

    Matrix& operator=(const Matrix& rhs)
    {
        printf("operator = #1! \n");

        if( this != &rhs )
            for(int i=0; i<n; ++i)
                for(int j=0; j<m; ++j)
                    ElementAt(i,j) = rhs.ElementAt(i,j);

        return *this;
    }

    Matrix<T, n, m>& operator=( const EtMatrixAdd<T, n, m>& rhs)
    {
        printf("operator = #2! \n");

        for(int i=0; i<n; ++i)
            for(int j=0; j<m; ++j)
                ElementAt(i,j) = rhs.ElementAt(i,j);

        return *this;
    }

    virtual ~Matrix() {}

    const T& ElementAt(size_t nn, size_t mm) const
    { return arrData[nn][mm]; }
    T& ElementAt(size_t nn, size_t mm)
    { return arrData[nn][mm]; }

private:
    // C-style array for efficiency and locality of reference
    T arrData[n][m];
};

template<typename T, size_t n, size_t m>
inline EtMatrixAdd<T, n, m> operator+( const Matrix<T, n, m>& lhs,
const Matrix<T, n, m>& rhs)
{
printf("EtMatrixAdd operator +! \n");

    return EtMatrixAdd<T, n, m>(lhs, rhs);

}
```

이제 아래의 연산을 수행해보면,

```cpp
Matrix<double, n, m> S ; // 먼저 정의
... S 사용
S = A + B;
```

새로 정의된 + 연산자는 이제 A,B 행렬의 참조자를 가진 EtMatrixAdd 임시객체를 리턴한다. 대입 연산자에서는 순회를 통해 EtMatrixAdd 클래스의 ElementAt 메소드를 호출하여 더하기 연산의 결과를 구한다.

ElementAt 메소드는 A,B 행렬의 각 요소의 합을 리턴하고 있다. 이렇게 하면 RVO가 동작함으로서 임시객체도 생성되지 않고, 루프 순회도 1번 동작으로 원하는 더하기 연산이 수행된다.

**하지만 아직은 해결책이라고 할수없다.**

다음의 경우를 고려해 보자.

```cpp
Matrix<double, n, m> S ; // 먼저 정의
... S 사용
S = A + B + C;
```

이경우, 컴파일러는 A,B행렬의 더하기에 새로운 + 연산자를 적용할것이다. 그리고 그 결과로 EtMatrixAdd 임시 객체가 생성될것이다. 즉, 아래의 코드를 작성한것과 동일하게 동작할 것이다.

```cpp
EtMatrixAdd<double, n, m> Temp1 = A + B;
```

그리고 컴파일러는 Temp1 +C 를 위한 코드를 만들려 할것이다. 그결과는 Temp1 ,C 를 참조자로 갖는 Temp2 임시 객체가 되야할것이다. 하지만, 지금 EtMatrixAdd 와 matrix 를 위한 + 연산자는 정의된것이 없기때문에 가능하지 않다. 또한 EtMatrixAdd 의 생성자또한 EtMatrixAdd 와 matrix 를 위한것은 정의되있지 않다.

이것을 해결하기 위해, 템플릿 인자를 활용해서 클래스를 수정해보면,

```cpp
template<typename T, size_t n, size_t m>
class Matrix;

template < typename T, size_t n, size_t m, typename LeftOp, typename RightOp>
class EtMatrixAdd
{
public:
    EtMatrixAdd(const LeftOp& lhs, const RightOp& rhs) : m_lhs(lhs), m_rhs(rhs)
    {
        printf("EtMatrixAdd Constructor! \n");
    }

    T ElementAt(size_t nn, size_t mm) const
    {
        printf("EtMatrixAdd ElementAt! \n");
        return m_lhs.ElementAt(nn, mm) + m_rhs.ElementAt(nn, mm);
    }

private:
    const LeftOp& m_lhs;
    const RightOp& m_rhs;
};

template<typename T, size_t n, size_t m>
class Matrix
{
public:
    Matrix(){printf("Constructor! \n");}
    Matrix(const Matrix& rhs)
    {
        printf("Copy Constructor! \n");

        for(int i=0; i<n; ++i)
            for(int j=0; j<m; ++j)
                ElementAt(i,j) = rhs.ElementAt(i,j);
    }

    Matrix& operator=(const Matrix& rhs)
    {
        printf("operator = #1! \n");

        if( this != &rhs )
            for(int i=0; i<n; ++i)
                for(int j=0; j<m; ++j)
                    ElementAt(i,j) = rhs.ElementAt(i,j);

        return *this;
    }
    template< typename TT, size_t nn, size_t mm, typename LeftOp, typename RightOp >
    Matrix<T, nn, mm>& operator=( const EtMatrixAdd<TT, nn, mm, LeftOp, RightOp>& rhs)
    {
        for(int i=0; i<nn; ++i)
            for(int j=0; j<mm; ++j)
                ElementAt(i,j) = rhs.ElementAt(i,j);

        return *this;
    }
    virtual ~Matrix() {}

    const T& ElementAt(size_t nn, size_t mm) const
    { return arrData[nn][mm]; }
    T& ElementAt(size_t nn, size_t mm)
    { return arrData[nn][mm]; }

private:
    T arrData[n][m];
};

template<typename T, size_t n, size_t m>
inline EtMatrixAdd<T, n, m, Matrix<T, n, m>, Matrix<T, n, m> >
operator+ ( const Matrix<T, n, m>& lhs, const Matrix<T, n, m>& rhs )
{
    printf("#1 operator + \n");
    return EtMatrixAdd<T, n, m, Matrix<T, n, m>, Matrix<T, n, m> >
    ( lhs, rhs );
}

// 컴파일 시점에 재귀적인 생성자 호출 코드가 생성되며, RVO가 동작하게 된다.
template< typename T, size_t n, size_t m, typename LeftOp, typename RightOp >
inline EtMatrixAdd< T, n, m, EtMatrixAdd<T, n, m, LeftOp, RightOp>, Matrix<T, n, m> >
operator+( const EtMatrixAdd<T, n, m, LeftOp, RightOp>& lhs,
const Matrix<T, n, m>& rhs )
{
    printf("#2 operator + \n");
    return EtMatrixAdd<T, n, m,
        EtMatrixAdd<T, n, m, LeftOp, RightOp>,
        Matrix<T, n, m> > (lhs, rhs) ;
}

int main( int argc, char\* argv [ ] )
{
    Matrix<double, 1, 2> A;
    Matrix<double, 1, 2> B;
    Matrix<double, 1, 2> C;
    Matrix<double, 1, 2> S ;
    S = A + B + C ;
    return 0;
}
```

EtMatrixAdd 에 LeftOp, RightOp 템플릿 인자를 추가함으로서 앞서의 문제를 해결했다.

    S = A + B + C;

이연산을 위해 컴파일러가 생성되는 코드는 아래처럼 작성한것과 동일하다.

```cpp
typedef Matrix<double, n, m> Mat;

Temp1 = A + B;
//=> EtMatrixAdd<double, n, m, Mat, Mat>

Temp2 = Temp1 + C;
//=> EtMatrixAdd<double, n, m, EtMatrixAdd<double, n, m, Mat, Mat>, Mat>

S = Temp2;
```

```cpp
S.ElementAt(i, j) = Temp2.ElementAt(i, j);
```

여기서 = 의 우측 내용은

```cpp
Temp1.ElementAt(i, j) + C.ElementAt(i, j);
```

로 해석되고, 이는 다시

```cpp
A.ElementAt(i, j) + B.ElementAt(i, j) + C.ElementAt(i, j);
```

이렇게 전개된다.

그리고 대입 연산 `S = A + B + C` 이 일어나게 된다.

즉, 임시객체 생성을 반복하는것이 아니라, 마치 재귀적인 생성자 호출로 처리되어 결국 임시 객체는 발생하지 않고 (RVO가 작동하기 때문), 대입 연산시에 1번의 순회만을 통해서 원하는 S 를 구할수 있게 된다.

연산을 템플릿을 활용하여 컴파일 시간에 단순한 수식연산으로 치환할수 있는 점은 c++의 템플릿의 강력함을 잘 보여준다.

## References

[Dr.Dobb's 저널의 (http://www.ddj.com) Expression template article](http://www.ddj.com/cpp/184401656)
[http://gpgstudy.com/gpgiki/CppExpressionTemplate](http://gpgstudy.com/gpgiki/CppExpressionTemplate)
