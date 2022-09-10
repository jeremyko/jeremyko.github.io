---
layout: post
title: c++ 코드에서 언더스코어 ('_')  사용시 주의
date: '2019-11-10T23:02:00.003+09:00'
tags:
    - c++
    - 니맘대로하면안된다
modified_time: '2022-03-26T20:56:17.877+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-566961743518724250
blogger_orig_url: https://jeremyko.blogspot.com/2019/11/c.html
---

오늘 유튜브에서 흥미로운 주제에 대한 [짧은 동영상](https://www.youtube.com/watch?v=0vJXQ2VOzEs)을 봤는데,  
c++에서 underscore ('\_') 가 포함된 [identifier](https://eel.is/c++draft/lex.name) 사용시 유의할 점에 대한 내용이었다.  
컴파일러가 파싱할때 적용되는 내용이므로, 그 대상은 모든 변수, 클래스명, 함수명 등등 다 해당되는 내용이다.

구체적으로

1.  underscore + 대문자로 시작하거나
2.  double underscore 가 포함되거나
3.  underscore로 시작되는 경우는

표준에서 컴파일러 구현을 위해 예약되거나, 전역 namespace 에 예약되었으므로 사용금지.

예를 들면,

    //이렇게 사용하지 말것.
    #ifndef   __MYHEADER_HPP__
    #define  __MYHEADER_HPP__
    #endif


    //중간에 __ 들어가는 것도 하지말고
    #ifndef  MYHEADER__HPP
    #define MYHEADER__HPP
    #endif

    //이런것도 하지말것
    int _myindex;
    int _MyIndex;

    //끝에 붙이는 underscore는 상관없다.
    int myindex_ ; //OK

특히나 변수앞에 \_ 붙이는 코드는 나는 그렇게 사용을 한적은 없지만,  
다른 사람들 코드에서 그동안 수도 없이 봤는데, 그런거 볼때마다 거부감이  
들었던 이유가 역시나 근거가 있는 내용이었다는 ...  
반면 ' #ifndef' 같은 경우는 나도 그냥 생각없이 저런식으로 사용했었고..  
앞으로는 유념해서 코드작성을 해야겠다.

**MS 의 경우에는 좀 틀린데, \_ 로 시작되는 경우는 허용되는것처럼 기술 되어 있다**

[https://docs.microsoft.com/en-us/cpp/cpp/identifiers-cpp?view=vs-2019](https://docs.microsoft.com/en-us/cpp/cpp/identifiers-cpp?view=vs-2019)
[https://github.com/MicrosoftDocs/cpp-docs/blob/master/docs/cpp/identifiers-cpp.md](https://github.com/MicrosoftDocs/cpp-docs/blob/master/docs/cpp/identifiers-cpp.md)

<h3> <span style="color:{{site.span_h3_color}}"> 참고 </span> </h3>
[https://eel.is/c++draft/lex.name](https://eel.is/c++draft/lex.name)
