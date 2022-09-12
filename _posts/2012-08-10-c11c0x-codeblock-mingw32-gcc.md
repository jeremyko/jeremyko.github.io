---
layout: post
title: C++11(C++0x) 과 CodeBlock , mingw32-gcc
date: '2012-08-10T13:13:00.002+09:00'
tags:
    - c++
    - CodeBlock
    - mingw32-gcc
modified_time: '2016-09-09T17:52:18.962+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-3518309587729136999
blogger_orig_url: https://jeremyko.blogspot.com/2012/08/c11c0x-codeblock-mingw32-gcc.html
---

C++11 (C++0x) 기능을 CodeBlock, mingw32-gcc 를 이용해서 Windows에서 시험해 보자.

새로운 c++ 표준의 기능들을 시험해보기 위해서는 VC++ express 2010 를 비롯한 다양한 컴파일러를 이용해서 해볼수 있지만,
[https://wiki.apache.org/stdcxx/C++0xCompilerSupport](https://wiki.apache.org/stdcxx/C++0xCompilerSupport)
에서 확인 할수 있듯, 현재 MSVC보다는 GCC 가 새로운 feature들을 더 많이 구현해놓았다.

또한 테스트 코드를 생성해서 컴파일 명령을 직접 수행해서 테스트 해볼 수도 있지만, 공개 IDE 소프트웨어인 CodeBlock을 사용해서 테스트 할수 있는 환경을 만들어보자.

http://www.codeblocks.org/downloads/binaries 에서 codeblocks-10.05-setup.exe 를 다운로드 받는다.

mingw32-gcc 가 포함된 버전도 있지만, 여기 포함된 있는 mingw32-gcc 버전은 최신 버전이 아니다.

우리는 별도로 C++11 기능을 더 잘 지원하는 mingw32-gcc를 다운로드 받아서 설치 할것이다.

[http://code.google.com/p/mingw-builds/downloads/list](http://code.google.com/p/mingw-builds/downloads/list)
에서 `mingw32-gcc-4.6.3-release-c,c++,fortran-sjlj.zip` 를 다운로드 한다. 그리고 적당한 위치에 압축을 푼다.

C++11 의 간단한 예제로는 Range-based for-loop 기능을 한번 테스트 해보기로 하자.

설치된 CodeBlock을 실행하고, File -> New -> Project -> Console Application 을 생성한다.
사용언어는 CPP 설정. title은 "cpp11x_rangeForLoop" 로 한다.

이제, 다운받은 mingw32-gcc 컴파일러를 사용하게끔 환경 설정이 필요하다.

Setting -> Compiler and debugger 를 연다. 기본 컴파일러로 GNU GCC Compiler가 설정이 되어있을것이다.

![blog-image](/assets/img/20120810-1.jpg)

새로운 컴파일러 환경을 만들기 위해서 Copy를 누른다.

그럼 새로운 컴파일러 이름을 설정하는 부분이 나오는데, 적당한 이름을 설정하고 확인을 누른다.

여기서는 CPP11 GNU GCC Compiler 라고 이름을 만들었다.
![blog-image](/assets/img/20120810-2.jpg)

Toolchain executables설정을 변경 하는것을 잊지 말라는 친절한 메시지가 출력된다.
![blog-image](/assets/img/20120810-3.jpg)

Toolchain executables 설정 버튼을 눌러서, 컴파일러 설정을 해줘야 한다.
![blog-image](/assets/img/20120810-4.jpg)

우선, 컴파일러 설치 디렉토리를 변경한다. 우리가 다운로드 받은 mingw32-gcc 의 위치를 지정해야한다. 그리고 다음처럼 Program Files 설정을 해준다. 혹시 Linker 설정에 ld 를 설정하는 실수를 할수도 있다 (바로 나 --;). 이러면 컴파일이 안된다. 링크 작업을 ld로 직접하는게 아니라 g++ 을 통해서 수행되도록, 반드시 g++로 설정해야 한다.
![blog-image](/assets/img/20120810-5.jpg)

새로만든 이컴파일러 설정을 기본 컴파일러로 설정하기 위해 Set as default를 클릭한다.

main.cpp 에 다음처럼 코딩하고, Build를 한번 수행해보자.

```cpp
#include <iostream>
#include <vector>

using namespace std;

int main()
{
    vector <int> data = { 1, 2, 3, 4 };

    //Range-based for-loop 테스트
    for ( int datum : data )  {
        cout << datum << endl;
    }
    return 0;
}
```

그런데 제대로 컴파일되지 않는다.

![blog-image](/assets/img/20120810-8.jpg)
c++0x 의 기능을 사용하기 위한 컴파일러 flag ( -std=c++0X )를 설정해줘야 하기 때문이다. 다시 Setting -> Compiler and debugger 를 연다. 그리고 Compiler flag항목에서 -std=c++0X 설정하는 부분을 찾아서 체크해준다.
![blog-image](/assets/img/20120810-9.jpg)

다시 확인을 한번 해보자.

프로젝트에서 오른쪽 버튼을 클릭해서 properties메뉴를 클릭한다. 그리고 project's build options 항목을 열고, Selected compiler가 우리가 새로 만든 컴파일러임을 다시 확인해 본다.
![blog-image](/assets/img/20120810-10.jpg)

![blog-image](/assets/img/20120810-11.jpg)

이상 없다면 F9를 눌러서 프로그램을 실행해보자.

![blog-image](/assets/img/20120810-12.jpg)

CodeBlock 과 mingw32-gcc 을 이용해서 C++11 의 기능들을 편리하게 시험해볼수 있다. 그런데, mingw32-gcc 은 아직 버전이 4.6.3 이다. 때문에 override and final 과 같은 기능들은 시험해 볼 수 없다 (4.7에서만 가능).

하지만 시간이 갈수록 컴파일러들가 지원하는 기능들이 많아질터이니 https://wiki.apache.org/stdcxx/C++0xCompilerSupport 를 계속 모니터링 하면서 스터디 해보면 좋을것 같다.
