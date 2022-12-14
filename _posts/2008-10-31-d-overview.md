---
layout: post
title: D 언어 overview
date: '2008-10-31T23:37:00.000+09:00'
tags:
    - D언어
modified_time: '2012-12-03T23:38:39.322+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-1186507686067899186
blogger_orig_url: https://jeremyko.blogspot.com/2008/10/d-overview.html
---

[http://www.digitalmars.com/d/2.0/overview.html](http://www.digitalmars.com/d/2.0/overview.html)

D 언어에 대한 한글자료는 거의없는거 같아서 한번 번역을 해보았다. D 언어의 공식 사이트를 방문해보면, 상당한 진척이 있음을 알수있다. 또한 컴퍼런스 링크를 따라가보니, Andrei Alexandrescu 의 이름도 눈에 띈다. D 언어에도 많은 영향을 끼치는 인사인것 같다.

## D언어란 무엇인가?

D언어는 범용의 목적을 가진, 시스템과 어플리케이션을 위한 프로그래밍 언어다. 고수준의 언어지만, 높은 성능의 코드 작성 및 운용체제 API 및 하드웨어와 직접적인 인터페이스가 가능하다.

D는 중간 규모에서부터 수백만 라인의 대형 프로그램 작성을 위한 개발자 팀에 적합하다. D는 배우기 쉽고 프로그래머에게 많은 능력을 제공하며, 공세적인 컴파일러 최적화 기술과 잘 부합한다.

D는 scripting, interpreted 언어가 아니며, VM, 사상따위가 필요하지 않다. D언어는 작업을 신속하고, 신뢰성있게, 유지보수가 용이하게, 또한 코드 판독 용이하게 끝내고자하는 실용적인 프로그래머를 위한 실용적인 언어이다.

D는 그동안의 다양한 언어를 위한 컴파일러들의 구현 및 그 언어를 이용한 대형 프로젝트 구축 시도에 있어서의 수십년간 경험의 결과라고 할수있다.

D언어는 다른언어(특히 C++)로 부터 많은 영감을 받았고, 그것들의 현실세계의 실용성, 경험을 적절히 반영하고 있다.

## 왜 D언어인가?

정말로, 왜일까? 다른 프로그래밍 언어가 정말 필요한가? C 언어를 발명한 이후 소프트웨어 업계는 먼 길을 걸어왔다. 많은 새로운 개념들이 C++에 추가되었으나, 초기 디자인의 거의 모든 결함까지도 포함하면서 C언어와의 호환성은 계속 유지되었다.

이러한 단점을 해결하기 위해 많은 시도들이 있었지만, 호환성 문제 때문에 성공하지 못했으며, 그동안, C와 C++ 모두 계속적인 기능 추가를 겪었다. 이러한 새로운 기능들은 이전 코드를 재작성하지 않게끔 기존구조에 주의깊게 반영되어야 한다. 그 최종 결과물은 매우 복잡한것이 되고 말았다.

**C 표준안은 거의 500 페이지이며, C++ 은 거의 750 페이지나 된다!**
또한 C++은 어렵고, 구현하기에 비용이 많이들며, C++의 구현은 여러가지여서 완전히 이식가능한 C++ 코드를 작성하기는 매우 어렵다. C++ 프로그래머들은 언어의 일부분에서만 프로그래밍하는 경향이 있다. 즉, 다른 기능의 사용은 회피하면서, 어떤 특정 부분을 사용하면서 그것만 점점 더 능숙해져가는 것이다. 한편, 코드는 컴파일러끼리 이식가능한 반면 프로그래머간에 서로 이식하기는 어려울수 있다.

C++의 강력한 힘은 많은 급격한 프로그래밍 스타일의 차이를 지원해줄수 있다는 것이다(하지만 장기적 사용관점에서 중복적이고 일관적이지 않는 스타일들은 바람직하지 않다). C++은 resizable arrays 나 문자열 연결(concatenation) 등과 같은것들을 언어 자체 지원이 아니고 표준 라이브러리의 일부로서 구현한다.

언어 자체에 포함되지 않음으로서, 일부분에서는 최선의 결과가 나오지 않는다. 간단하고 견고하며 실용적인 언어로 C++ 의 힘과 능력을 추출, 재설계, 수정가능할까?

컴파일러 제작자가 정확하게 구현할수있고, 컴파일러가 최적화된 코드를 효율적으로 생성하게끔 모든것을 하나의 패키지로 만들수 있을까?

현대 컴파일러 기술은 초창기 컴파일러 기술을 위한 보완 목적의 언어 기능을 생략 가능하게 개선하는 쪽으로 이루어져왔다. (예를 들면 C언어의 'register' 키워드가 있다.

좀더 미묘한 예는 C 에서의 매크로 전처리기이다.) 우리는 만족할만한 코드 품질을 얻기위해서 초창기 컴파일러들에게 필요했던 언어적 기능없이 현대 컴파일러의 최적화 기술에 의존할수있다.

## D언어의 주요 목표

-   증명된 생산성 향상 기능들을 도입하고 언어의 기능들을 조절해서 시작 단계부터, 흔하게 발생하는 시간 소모성 버그를 제거해서 소프트웨어 개발비용을 최소한 10% 줄인다.
-   컴파일러간, machine간, 운영체계간 이식가능한 코드를 쉽게 작성하게 한다.
-   멀티 - 패러다임 프로그래밍 지원
    즉, imperative, 절차적, 객체지향적, 제네릭 프로그래밍 패러다임을 위한 최소한의 지원
-   C, C++ 에 익숙한 프로그래머들을 위한 짧은 학습 곡선
-   요구되는 저수준의 하드웨어 접근을 지원
-   C++보다 컴파일러를 구현하기 쉽게끔 한다.
-   C 어플리케이션 바이너리 인터페이스와 호환성
-   context-free 문법
-   간편한 국제화 응용 프로그램 작성 지원
-   contract 프로그래밍 및 단위 테스트 방법론을 결합
-   가볍운 독립 실행형 프로그램을 구축할 수있게한다.
-   문서 작성의 비용을 절감하게 한다.

## C/C++ 로부터 유지되는 기능들

일반적인 외형은 C 나 C++ 과 비슷하다. 이것은 D언어를 학습, 포팅하는것을 쉽게만든다.

C/C++ 로 부터 D언어로의 전환은 편안하게 느껴질것이다. 프로그래머는 이를 위해서 모든것을 새로운 방식으로 배울 필요는 없다.

D언어를 사용한다는것은 프로그래머가 자바나 스몰토크의 vm(virtual machine) 처럼 특정 런타임 vm 에 제한적이게 된다는 의미가 아니다.

D 언에에는 vm이 없다.링크가능한 오브젝트 파일을 직접 컴파일러가 생성한다. C언어에서처럼 D언어도 운영체계에 연결한다. 일반적으로 make 같이 친숙한 툴들이 D언어 개발에도 적합할것이다.

-   일반적인 외양, 느낌은 C/C++것을 유지하였다. C/C++의 그것과 표현양식, 외양이 거의 동일하다.

-   D 프로그램은 C 스타일(함수와 데이터), C++ 객체지향스타일, C++ 템플릿 메타프로그래밍, 혹은 3가지의 조합으로도 사용할수있다.

-   바이트코드로 컴파일되고 해석되어지는것을 배제한것은 아니지만, compile/link/debug 개발모델이 유지되었다,

-   예외처리 : 많은 경험에 의해서 예외처리가 C 에서 전통적으로 error 코드와 errno 전역변수들로 에러를 처리하는것보다 우월한 방법이 될것이다.

-   RTTI(Runtime Type Identification) : 이것은 일부 C++에 구현되었다. D언어에서는 완전하게 지원하며, 향상된 가비지 컬렉션, 디버거 지원, 보다 자동화된 persistence 등등을 지원하고있다.

-   D 언어는 C언어의 함수 호출 규약과 호환성을 유지한다. 이것은 D 프로그램이 운영체계의 API에 직접 접근할수있게 해준다. 기존 API에 대한 프로그래머의 지식과 경험, 패러다임은 최소한의 노력으로 D 언어에 적용가능하다.

-   연산자 오버로딩 : D 프로그램은 연산자 오버로딩이 가능하고, 이로 인해서 기본 데이터 타잎을 사용자 정의 타잎으로 확장가능하다.

-   템플릿 메타프로그래밍 : 템플릿은 제네릭 프로그래밍을 구현하기위한 한 방법이다. 다른 방법중에는 매크로를 사용하거나, variant 데이터 타잎 사용등이 있다. 매크로의 사용은 고려대상이 아니다. Variants의 사용은 수월하기는 하지만, 비효울적이고 형식 체크에 부족하다. C++템플릿의 어려움은, 그 복잡성에 있다. 그것은 언어의 문법과 잘 어울리지 않는다. D언어에서는 conversions 과
    overloading을 위한 모든 다양한 규칙들이 갖추어져있다. D언어는 템플릿을 하는데있어서 훨씬 간단한 방법을 제공한다.

-   RAII (Resource Acquisition Is Initialization) : RAII 기법은 신뢰성있는 소프트웨어를 개발하기 위한 필수요소이다.

-   Down and dirty programming : D언어는 다른 언어로 컴파일된 외부 모듈의 도움 없이도, down-and-dirty programming
    (저수준의 어려운 프로그래밍을 말하는것같다.)을 할수있다. 때때로, 시스템 작업을 위해서 포인터를 써야만 하거나, 어셈블리를 이용할 필요가 있다. D언어의 목적은 down and dirty programming 을 못하게 하는것이 아니라, 일상적인 코딩 업무를 해결하는데 있어서 그 필요성을 최소화 시키는데 있다.

## C/C++ 로부터 제거된 기능들

-   C 소스와의 호환성
    C언어 확장으로 소스코드의 호환성을 유지하려는 것은 이미 진행되어져 왔다 (C++ 과 ObjectiveC). 이영역에서의 향후 작업들은 수많은 기존 코드들로 인해 제한받게 될것이고,
    괄목할만한 개선이 이루어지기는 어려울 것이다.

-   C++과의 링크 호환성
    C++ runtime object model 은 너무 복잡하다. 만약 호환성을 유지하려한다면, 그것은 본질적으로 D 컴파일러를 완전한 C++ compiler 로도 만들어야한다는것을 의미한다.

-   C 전처리기.
    매크로 처리는 실제 존재하지 않는 기능을 추가함으로서 (symbolic 디버거로 볼수없는), 언어를 확장시키는데 쉬운 방법중의 하나이다. 조건적인 컴파일, 층층이 쌓인 #include 문장 , 매크로들, token 연결, 등등은 본질적으로 하나가 아닌, 두개가 함께 병합되어 그것들을 명확히 구분할수 없는 언어를 형성한다. 더 안좋은것은(혹은 아마 가장 훌륭한것은 ) C 전처리기가 매우 원시적인 매크로 언어란것이다. 한걸음 뒤로 물러나, 전처리기가 무엇을 위해서 사용되는지 살펴보고, 또 그것의 능력을 직접 언어안에서 지원하게끔 설계를 해야하는 시점이다.

-   다중 상속.
    복잡하며 논쟁의 여지가 있는 기능이다. 효율적인 방법으로 구현하기 매우 어렵고, 그것을 구현하면서 컴파일러는 많은 오류 발생의 여지가 있다. 거의 모든 다중상속의 효용성은 단일 상속과 인터페이스, 통합(aggregation) 조합으로도 대체 가능하며, 더이상 다중상속 구현의
    어려움을 정당화시키지는 못한다.

-   네임 스페이스.
    독립적으로 개발어진 코드들을 모두 함께 연결하는데 있어서 문제점은 이름들간의 충돌이다. Modules 사용하는 것이 훨씬 간단하며 더욱 잘 동작한다.

-   Tag name space.
    이 C언어의 잘못된 기능은, 분리되었지만 병행되어 같이 참조되는 심벌 테이블에 구조체들의 Tag 이름들이 존재하는 것에 기인한다. C++는 기존 C 코드와 호환성을 유지하면서, tag name space 를 일반 name space 와 병합하려 했다. 그결과로 불필요한 혼돈이 야기 되었다.

-   먼저 선언하기(Forward declarations)
    C 컴파일러는 현재상태 이전 내용에 대해서만 의미 파악을 할수있다. C++은 이것을 조금 확장해서, 클래스 멤버들은 이전에 참조된 클래스 멤버들에 의존할수 있다. D언어는 논리적인 결론에 도달했는데, 전방 선언은 더이상 module 단위에서 불필요 하다는것이다. 함수들은 C 프로그램에서 전방 선언을 피하기 위해 흔히 사용되는, 뒤바뀐 함수들의 순서가 아닌 자연스러운 순서로 정의되어질 수 있다.

-   Include 파일들
    각각의 컴파일 단위마다 무지막지한 양의 헤더파일을 재해석해야함으로서 컴파일이 느려지는 주원인이다. 파일들을 include하는것은 심벌 테이블을 importing 할때 처리되어야 한다.

-   Trigraphs and digraphs. Unicode가 미래다.

-   비 가상 멤버 함수들.
    C++ 에서는 클래스 설계자는 어떤 함수가 가상인지 비가상인지를 먼저 결정한다. 함수가 오버라이드 될때, 베이스 클래스 멤버함수를 가상함수로 변경하는것을 잊어버리는것은 흔하고(그리고 매우 찾기 힘든) 코딩 오류이다. 모든 멤버 함수를 가상함수로 만들고 만약 아무런 오버라이드가 없다면 그함수를 비가상함수로 전환시키는것을 컴파일러가 결정하게 하는것이 더 신뢰성있다.

-   임의 크기의 Bit fields.
    Bit fields 는 복잡하고 비효율적이며 거의 사용안되는 기능이다.

-   16비트 컴퓨터 지원.
    near/far 포인터, 훌륭한 16비트 코드를 생성하기 위해 필요한 장치에 대한 고려는 없다. D언어의 설계에서는 최소한 32비트 실수 메모리 공간을 가정한다. D언어는 향후 64비트 구조에 원활하게 적응할 것이다.

-   상호 의존적인 컴파일러 단계들.
    C++에서는 성공적인 소스 파싱은 심볼 테이블을 가지는것 그리고 전처리기의 다양한 명령에 의존한다. 이것은 C++소스를 먼저 파싱하는것을 불가능하게 만들고, 코드분석기와 문법 통제된 편집기가 제대로 동작하는것을 극도로 어렵게 만든다.

-   컴파일러 복잡성.
    컴파일러 구현을의 복잡성을 감소시키는것은, 다수의 올바른 컴파일러 구현을 가능하게 한다.

-   저 정밀도 부동 소수점.
    만약 누군가 현대의 부동 소수점을 구현한 하드웨어를 사용중이라면, 그것은 현재 하드웨어간 통용되는 가장 낮은 정밀도의 부동 소수점을 사용중인 프로그래머에게도 가능해야한다. 특별히, D 언어의 구현은 IEEE 754 계산과 만약 확장된 정밀도가 가능하다면 그것을 지원해야 한다.

-   < 과 > 부호를 이용한 템플릿 오버로딩.
    C++에서의 이 선택은 수년간의 버그들, 비탄 그리고 프로그래머들, 컴파일러 구현자들, C++소스 파싱 벤더들의 혼란의 원인이 되어 왔다. 이것은 완전한 C++ 컴파일러가 수행하는것과 거의 같은 일을 하지 않는다면, C++코드를 제대로 파싱하는것을 불가능하게 만든다. D 언어는 깔끔하고 명백한 문법으로서 !( 와 ) 를 사용한다.

## D는 누구를 위한 언어인가

-   일상적으로 컴파일되지도 않은 코드에서 버그를 제거하려, lint나 비슷한 코드분석툴을 사용하는 프로그래머.

-   경고레벨을 최고로 놓고 컴파일하고, 경고를 에러로 처리하게 컴파일러 설정하는 사람.

-   일상적인 C 버그들을 피하기 위해 프로그래밍 스타일 가이드 라인들에 의존해야만 했던 프로그래밍 매니저

-   복잡성 때문에 C++ 객체지향 프로그래밍이 약속했던것이 만족되지않는다고 결정한 사람.

-   C++ 의 풍부한 표현력을 즐기지만, 메모리 관리와 포인터 버그를 발견하는데 훨씬 많은 노력이 요구됨에 당황스러운 프로그래머들.

-   내장된 테스트와 검증이 필요한 프로젝트들.

-   수백만 라인의 어플리케이션을 제작하는 팀.

-   프로그래밍 언어란 빈번하게 포인터를 직접 다뤄야하는 필요성을 미연에 방지하는 충분한 기능을 제공해야한다고 생각하는 프로그래머.

-   수치연산 프로그래머들.
    D는 확장된 부동 소수점 정확도, 복소수와 허수 부동 타잎, NaN (Not a Number)의 정의된 행동, 무한대에 대한 핵심지원 같은, 수치연산 프로그래머들에게 필요한 기능들을 직접 지원하는 많은 기능을 가지고 있다. ( 새로운 C99 표준에는 추가되었지만 C++은 그렇지 않다 )

-   어플리케이션의 절반을 루비와 파이선등의 스크립팅 언어로 작성하고 , 나머지 절반을 병목지점의 속도개선을 위해서 C++ 을 사용한 프로그래머들.

D언어는 루비와 파이선의많은 생산성 가지고 있으며, 전체 어플리케이션을 하나의 언어로 작성가능하게 한다.

D의 어휘 분석기, 파서, 의미 분석기는 전적으로 서로에 대해 독립적이다.
이것은 D 소스를 다루는 간단한 툴들을 완전한 컴파일러를 만들지 않고도 쉽게 작성할 수 있으며, 또한 소스코드가 토큰화된 형태로서 특화된 어플리케이션으로 전송 가능것을 의미한다.

## 사용하기 적합하지 않은 경우

-   현실적으로 아무도 C혹은 C++ 로 짜여진 수백만 라인의 코드를 D언어로 변환하지는 않을것이다. D언어는 수정되지 않은 C/C++ 소스코드를 컴파일 할수 없기 때문에, 레거시 프로그램에는 적합하지 않다. (그렇지만, D는 기존 C API를 매우 잘 지원한다. D는 C 인터페이스를 가진 어떠한 코드와도 직접 연동할수있다)

-   프로그래밍 언어로 처음 사용.
    초보자에게는 자바나 베이직이 더 적합하다. D언어는 중간단계에서 고급 프로그래머로 향하는 과정에서의 훌륭한 세컨드 언어이다.

-   언어 순수주의자.
    D는 실용적인 언어이다. 그리고 각각의 기능은 이상적인것 보다는, 실용적 관점에서 평가된다. 예를들어, D는 일반적인 작업에서 실질적으로 포인터의 사용을 제거하기 위한 방법을 가지고 있지만, 가끔은 규칙이 깨어져야하는 경우가 있기때문에, 여전히 포인터는 사용할수있다. 비슷하게, 형변환(casts) 또한 형변환 체계가 오버로딩되어야하는 때를 위해서 여전히 사용할수있다.

## D 언어 주요기능

이번에는 다양한 주제로 좀더 흥미로운 기능들을 살펴본다.

### 객체지향 프로그래밍

**클래스**
D의 객체지향적인 면은 클래스에서 시작된다. 인터페이스들을 강화시킨 단일 상속모델을 사용한다. 클래스 객체가 상속계층의 꼭대기에 위치하며, 공통된 기능들은 모든 클래스들이 구현한다. 클래스들은 참조에 의해 구체화되며, 예외 발생 이후를 정리하기위한 복잡한 코드는 불필요하다.

**연산자 오버로딩**
클래스들은 새로운 타입 지원을 위한 타입시스템의 확장을 위해 기존 연산자들과 같이 작동되게 조작 가능하다. 예를들면, bignumber 클래스를 생성하고 +, -, \* 그리고 / 연산자들을 오버로딩해서, 일반적인 대수 문법을 같이 사용하게 할수있다.

### 생산성

**모듈들**
소스 파일들은 1대1 로 모듈과 일치한다. #include 문장으로 선언부를 가져오지말고, 단지 module 를 import한다. 동일한 module에대해 중복된 import를 걱정할 필요는 없다. #ifndef/#endif 나 #pragma once 등과 같은 지저분한 방법은 필요없다.

**선언 vs 정의**
C++은 대개 함수들과 클래스들이 2번 선언되어질것을 요구한다.

-   선언은 .h파일에 들어가고, .c 파일에는 정의가 들어간다.
    이것은 오류발생 여지가 많은 성가신 과정이다. 프로그래머는 그것을 오직 한번만 작성하면 되며, 컴파일러는 선언정보를 추출하여 심볼 import를 가능하게 한다. 이것이 정확하게 D언어가 하는 작업이다.

예제

    class ABC
    {
        int func() { return 7; }
        static int z = 7;
    }
    int q;

멤버함수들과, static 멤버, externs 의 별도의 정의나, 다음과 같은 어색한 문법도 더이상 필요하지 않다.

    int ABC::func() { return 7; }
    int ABC::z = 7;
    extern int q;

Note: 물론, C++에서는 { return 7; } 같은 사소한 함수들은 인라인으로 작성되지만, 복잡한 경우는 그렇지 않다. 또한 만약 어떤 전방 참조가 있다면, 그 함수는 함수원형이 필요하다. 다음과 같은 소스는 C++에서는 작동하지 않는다.

```cpp
class Foo
{
    int foo(Bar *c) { return c->bar(); }
};

class Bar
{
    public:
    int bar() { return 3; }
};
```

그러나 동일한 D소스는 작동한다:

    class Foo
    {
        int foo(Bar c) { return c.bar; }
    }
    class Bar
    {
        int bar() { return 3; }
    }

함수가 인라인되는지 여부는 최적화 설정에 떠른다.

**템플릿**

D템플릿은 부분 특화의 능력을 제공해서 명확한 방식으로 제네릭 프로그래밍을 지원한다. 가변 템츨릿 인자들과 tuples을 가지는 템플릿 클래스와 템플릿 함수들이 가능하다.

**Associative Arrays (조합배열)**
조합배열은 인덱스로서 정수형이 아닌 다양한 데이터형을 가지는 배열이다. 본질적으로, 조합배열은 빠르고, 효율적이고, 버그없는 심볼 테이블들을 쉽게 구성하게 한다.

**진정한 Typedefs**

C와C++에서의 typedefs는 새로운 타입이 정말 만들어지지 않기 때문에, 단순한 타입에 대한 별칭이었다. D는 다음처럼 진정한 typedefs을 구현한다 :

    typedef int handle;

정말로 handle이러는 새로운 타입을 생성한다.형식체크가 강제적이며, typedefs가 함수 오버로딩에 관여한다. 예를 들면:

    int foo(int i);
    int foo(handle h);

**문서화**

문서화는 전형적으로 2번 수행된다. - 한번은 함수가 무엇을 하는것인지 주석을 다는것이고, 이것이 별개의 html이나 man page로 다시한번 작성된다. 그리고 시간이 지남에 따라 자연스럽게, 코드는 업데이트되지만 별도의 문서는 변경 안되는 경향이 있다. 주석으로부터 세련된 문서를 직접 생성 할 수있는 것은 시간을 반 이상 절약 시킬뿐 아니라, 소스와 문서를 일치시키는 것을 더욱 쉽게 한다. Ddoc 은 D 문서화 생성기의 사양(specification)이다. 이 페이지 또한 Ddoc 으로 생성되었다.

C++을 위한 서드파티 툴이 존재하지만, 그것들은 몇몇 심각한 단점을 가지고있다:

-   C++ 을 100% 분석하는것이 엄청나게 어렵다. 그렇게 하기위해선 정말 거의 100% 수준의 C++ 컴파일러가 요구된다. 또한 C++의 일부만을 제대로 해석하는 경향이 있고, 그로 인해서 기능의 일부만을 사용하는 소스코드로 제한하게 할것이다.

-   상이한 C++의 버전을 지원하는 상이한 컴파일러는 C++의 상이한 확장을 가진다.

이 모든 변형에 모두 부합하지 않는 문제가 있다.

-   원하는 플랫폼에 가능하지 않을수도 있으며, 필연적으로 컴파일러와는 틀린 업데이트 주기를 가진다.

-   컴파일러에 내장된다는것은 모든 D 구현들에 있어서 표준이라는 의미이다. 언제나 사용가능한 기본 문서화도구를 가지고있음으로서 보다 더 많이 활용될 수 있다.

### 함수

D는 전역함수, 오버로딩된 함수, 인라인함수, 멤버함수, 가상함수, 함수포인터 등의 일반 함수뿐 아니라, 추가적으로 다음 함수를 지원한다.

**중첩된 함수 (Nested Functions)**

함수는 다른 함수안에서 중첩가능하다. 이것은 code factoring, locality, function closure 기법에 매우 유용하다.

**함수 어구**
익명의 함수가 수식에 직접 내장 가능하다.

**Dynamic Closures**
중첩된 함수와 클래스 멤버 함수는 클로져(대리자라고도 불리운다)를 통해서 참조될수있으며, 제네릭 프로그래밍을 보더 쉽고 형식에 안전하게 만든다.

**In Out 과 Inout 인자들**
이것은 함수 스스로를 좀더 문서화시킬뿐 아니라, 위험한 포인터의 필요성을 제거하고, 컴파일러가 코딩상의 문제점을 찾는것에 보다 더 도움을 주는 가능성을 펼쳐놓았다. 또한 다양한 외부 API와 직접 소통가능하게 한다.

인터페이스 정의 언어같은 방법은 불필요 해졌다.

### 배열

C 배열은 수정할수 있는 몇개의 오류를 가지고 있다 :

-   차원 정보가 배열에 같이 전달되지 않고, 별도로 전달되어져야한다.
    전형적인 예는 main(int argc , char \*argv []) 함수의 argc, argv 이다. (D에서는 main은 main(char[][] args ) 로 정의된다)

-   배열은 클래스 객체가 아니다. 함수로 배열이 전달될때, 그 함수 원형에서는
    혼란스럽긴해도 인자가 배열이라고 언급 했지만, 그냥 포인터로 변환된다. 이런 변환이 일어날때 모든 배열의 타입정보는 유실된다.

-   C 배열은 크기가 재조정될 수 없다. 때문에 간단한 stack 조차 복잡한 클래스로 구축되어져야한다.

-   C배열은 배열 경계를 알수없기 때문에, 경계 체크가 불가능하다.

-   배열은 구분값이후 []로 선언된다.이것은 배열에 대한 포인터를 정의하는것처럼 매우 성가신 문법을 만든다 :

    ```cpp
    int (*array)[3];
    ```

D에서는 배열에 대한 []가 왼쪽에 위치한다:

    int[3]* array; // 3개의 정수배열의 포인터를 정의
    long[] func(int x); // long 배열을 리턴하는 함수 정의

훨씬 이해하기 간단하다. D배열에는 포인터, 정적 배열 동적배열 그리고 조합배열등의 종류가있다.
[Arrays](https://digitalmars.com/d/2.0/arrays.html) 참조.

### 문자열

문자열 처리는 너무도 일상적이고, C와 C++에서는 너무 볼품없기 때문에, 언어자체 지원이 필요하다. 현대의 언어들은 문자열 연결, 복사등등을 다루고있고, D또한 그러하다. 문자열은 향상된 배열조작의 직접적인 결과이다.

### 리소스 관리

**자동 메모리 관리**
D의 메모리 할당은 완전한 가비지 회수이다.경험상, C++의 많은 복잡한 기능들은 메모리 제거를 관리하기 위해 요구된다. 가비지 컬렉션기능으로 D언어는 훨씬 간단해졌다. 가비지 컬렉션은 게으르고, 초보 프로그래머를 위한것이라는 인식이 있다. C++도 그런말을 들었던것으로 기억하지만, 결국 C나 어셈블러가 할수있는것을 C++이 못하는것은 없다. 가비지 컬렉션은 C와 C++에서처럼 성가시고, 에러발생확률이 높은 메모리 할당 추적 코드를 방지한다. 이것은 빠른 개발시간과 낮은 유지비용뿐만 아니라, 종종 완성된 프로그램이 더 빨리 동작하기도 한다! 물론, 가비지 컬렉션은 C++ 과 사용가능 하다. C++은 가비지 수집기에
친화적이지 않고 그 효율성을 방해한다. 많은 런타임 라이브러리 코드가 수집기와 같이 사용될 수 없다. 이것에 대한 논의는 garbage collection 을 참조.

**명시적인 메모리 관리**

D가 가비지 컬렉트 언어이지만, 특수한 클래스를 위해 new와 delete 연산자를 오버로딩해서 커스텀 할당자를 사용할수 있다.

**RAII**
RAII 는 리소스 할당 및 해제를 다루는데 있어 현대화된 소프트웨어 개발 기법이다, D언어는 RAII 를 통제되고, 예측가능한 방식으로 지원하며, 가비지 컬렉션 주기와는 별개이다 .

### 성능

**경량의 Aggregates**

D는 C 데이터 구조체와의 호환성과, 클래스의 모든 기능이 불필요한 경우에 유용하기 때문에, 간단한 C스타일 구조체를 지원한다.

**인라인 어셈블러**

디바이스 드라이버, 고성능의 시스템 어플리케이션, 임베디드 시스템, 특수화된 코드는 주어진 작업을 해결하기 위해 때때로 어셈블리 언어가 필요하다. D언어구현은 인라인 어셈블러를 구현하지 않아도 된다. 그것은 언어의 일부로 정의되어져있다. 어셈블러 코드의 필요성은 별도 어셈블러나 DLL 필요없이 대부분 내부적으로 처리가능하다. 많은 D언어 구현들이 C의 I/O포트, 특정 부동 소수점 연산 직접 접근등의 지원과 동등한 내재된 함수를 지원하게 될것이다.

### 신뢰성

현재적인 언어는 프로그래머가 버그를 몰아낼수있게 도움을 주는 모든 조치를 취해야한다. 이러한 도움은 좀더 견고한 기법을 쉽게 사용할수있게 하는것부터 잘못된 코드를 찾아주거나 런타임 체킹등 다양한 방법으로 가능하다.

**Contracts**
Contract 프로그래밍 (B. Meyer에 의해 개발됨) 은 프로그램의 정확성 보증을 돕는 혁신적인 기법이다. D의 DBC 버전은 함수 preconditions, 함수 postconditions, 클래스 불변식, assert contracts 을 포함하고 있다. Contracts D구현을 참조

**유닛 테스트**

단위테스트를 클래스에 추가가능하며, 프로그램 시작부터 자동적으로 동작한다. 이것은 모든 빌드마다, 클래스 구현이 부주의하게 작성되지 않았음을 증명하는것을 도와준다.

단위테스트는 클래스 소스 코드의 일부분이 된다. 최종 코드를 테스팅 그룹에 던져주는것과는 반대로, 단위시험 코드를 생성하는것은 클래스 개발과정에 있어서 자연스러운 일이된다.

단위시험은 타 언어에서도 이용가능하지만, 불편하고 편리하지 않다. 단위테스트는 D의 주요기능이다. 라이브러리 함수들에 대해서, 함수들이 실제로 동작하는지 보장하고, 어떻게 사용하는지 예시하게끔 멋지게 동작한다.

많은 C++라이브러리들이 인터넷으로 다운받은 코드에 기반한다는것을 고려해보면, 그중 단위시험은 말할것도없고, 일반 검증 테스트를 조금이라도 거친것은 얼마나 될까? 1% 미만?

일반적으로, 만약 컴파일되는것이 있다면 우리는 그것을 동작하는것이라고 보고, 컴파일러가 내뱉는 경고들을 정말 버그로 봐야할지 아니면 숨겨진 버그를 암시하는것인지 의아하게 여긴다.

Contract 프로그래밍, 단위테스트 덕분에 D는 신뢰성있고 견고한 시스템 어플리케이션을 작성하기위한 최고의 언어이다.

또한 단위테스트는 누군가가 작성한 코드의 품질을 판단하는데 임시변통의 도구 역활도 한다. 만약 그소스에 단위테스트와 contracts가 없다면 그건 받아들여질수 없는것이다.

**Debug Attributes and Statements**

이제 디버그는 언어문법의 일부이다. 코드는 전처리 명령이나 매크로 사용없이 컴파일 시간에 활성화 비활성화가 가능하다. 디버그 문법은 일관되고, 이식성있고, 이해할수있는 구분을
가능하게 해 주어서, 실제 소스코드는 디버그, 릴리즈 컴파일 생성을 할수있어야 한다.

**예외처리**

단순한 try-catch보다 나은 try-catch-finally 모델이 사용되었다. 단지 객체 파과자에서 최종의 처리를 수행하기 위한 목적으로의 더미 객체를 생성할 필요가 없다.

**동기화**

멀티쓰레드 프로그래밍은 점점더 일반적인것으로 되고있다. 동기화는 메소드나 객체단위로 가능하다.

    synchronized int func() { ... }

Synchronized 함수는 한번에 오직 하나의 쓰레드를 허용한다. synchronize 구분은 mutex 를 만들고 접근을 제어한다.

**Robust 기법 지원**

-   포인터 대신 동적 배열

-   포인터대신 참조변수

-   포인터대신 참조객체

-   명시적 메모리 관리보다 가비지 콜렉션

-   쓰레드 동기화를 위한 내장

-   매크로 지원안함으로 부주의한 코드 방지

-   매크로 대신 인라인 함수

-   포인터 필요성을 급감시킴

-   명시된 정수형 사이즈

-   더이상 char의 명확하지 않은 부호 여부 없음

-   헤더와 소스파일에서의 중복된 선언 불필요

-   디버그 코드를 추가하는것을 위한 명시적인 파싱지원

**컴파일 타임시 체크**

-   강화된 타입 체크

-   for 루프 바디안에서의 빈 ; 불가함.

-   대입이 참거짓의 결과를 반환하지 않음.

-   사용안되는 API에 대해서 Deprecated 처리

**실행시 체크**

-   assert() 표현식

-   배열 경계 체크

-   switch 에서 정의안된 case 예외

-   메모리 부족 예외

-   In, out 과 클래스 불변식 Contract Programming 지원

### 호환성

**연산자 우선순위 및 평가 규칙**

D는 C의 연산자 우선순위, 평가규칙을 유지한다.

**C API에 직접 접근**

C에 상응하는 데이터 타입을 가지는것뿐 아니라, C함수에 직접 접근할수있다.

**모든 C data types 지원**

C API나 기존 C library code 와 연계가능하다.
structs, unions, enums, pointers, 모든 C99 타입을 포함한다.

**OS 예외 처리**

D의 예외처리 메카니즘은 OS 가 어플리케이션의 예외를 다루는 근본방식으로 연결되어질것이다.

**기존 Tool 사용**

D는 코드를 표준 오브젝트 파일 형식으로 생성하기 때문에, 표준 어셈블러, 링커, 디버거, 프로파일러, exe compressors, 다른 분석기를 사용할수있으며, 다른 언어로 작성된 코드와 링크될수있다.

### 프로젝트 관리

**버전관리**

동일한 문장에서 다양한 버전을 생성하는 기능을 자체 지원하여, C 전처리기의 #if/#endif 기법을 대체한다.

**Deprecation**

코드가 시간에 따라 변화함에 따라, 일부 이전의 라이브러리코드가 새롭고, 기능개선된 것들로 교체된다. 이전 버전은 기존 코드를 지원하기 위해서 이용가능해야 하지만, 그것들은 deprecated 로 구분 되질수있다. 이전버전을 사용하는 코드는 보통 불법으로 표기되어질 것 이지만, 컴파일러 스위치에 의해 허용 되어질 수 있다. 이것은 유지보수 프로그래들이 이전 deprecated 기능에 의존하는 것들을 구분하는것을 쉽게 만들것이다.

### 간단한 D Program (sieve.d)

    /* 소수구하기 , 에라토스테네스의 체 */
    import std.stdio;

    bool[8191] flags;
    int main()
    {
    int i, count, prime, k, iter;

        writefln("10 iterations");
        for (iter = 1; iter <= 10; iter++)
        {
            count = 0;
            flags[] = 1;
            for (i = 0; i < flags.length; i++)
            {
                if (flags[i])
                {
                    prime = i + i + 3;
                    k = i + prime;
                    while (k < flags.length)
                    {
                        flags[k] = 0;
                        k += prime;
                    }
                    count += 1;
                }
            }
        }
        writefln("%d primes", count);
        return 0;

    }

## 설치 해 보기

1. [http://www.digitalmars.com/d/download.html](http://www.digitalmars.com/d/download.html) 에서 dmd.2.014.zip (dmd D 2.0 compiler) , dmc.zip ( Linker and Utilities) 를 받는다

2. 압축을 푼다 (이때 동일한 폴더안에 두개를 풀어야 나중에 바로 사용가능함. 다른 위치에 풀었다면 sc.ini 를 수정해줘야한다. 이 설정 파일안에 linker위치가 상대경로로 표시된다 )

3. 제어판 가서 환경변수 path에 ...dmd\bin 을 설정해준다.

4. dmd\samples\d 에 있는 hello.d 를 프롬프트에서 dmd hello.d 로 컴파일해보면 간단하다..

## D 언어사용 게임

[http://www.asahi-net.or.jp/~cs8k-cyu/windows/ttn_e.html](http://www.asahi-net.or.jp/~cs8k-cyu/windows/ttn_e.html)

1인 게임 개발자로 유명한 kenta cho 가 D언어를 이용해서 만든 게임들..
