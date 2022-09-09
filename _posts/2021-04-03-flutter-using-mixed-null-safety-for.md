---
layout: post
title: 'flutter : using mixed null safety for legacy packages'
date: '2021-04-03T15:12:00.014+09:00'
author: jeremyko
tags:
    - flutter
    - breaking upgrade
    - null safety
modified_time: '2021-04-06T15:50:48.312+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-8332205775558986948
blogger_orig_url: https://jeremyko.blogspot.com/2021/04/flutter-using-mixed-null-safety-for.html
---

### null safety 가 없는 기존 package 들을 최신 flutter, dart 개발 환경에서 사용하기.

flutter 프로젝트 내에서 dart 버전을 소스파일 단위로 다르게 지정하여 오래된 package 들도 null safety 에러가 안나게 개발할수 있다.

현재 flutter 2 최신 버전을 설치하게 되면 같이 설치되는 dart 언어도 최신 버전(2.12)으로 기본적으로 sound null safety 가 동작하게 된다. 이 상태에서 프로젝트 내 모든 기능을 직접 개발하는 상황이면, 이 null safety 문제에 대해서는 고민할것이 없이 그냥 개발하면 된다.

그런데 만약 pub.dev 같은 곳에서 다른 개발자의 기존 package 를 가져다가 사용하는 경우 , 그 기존 package 들이 null safety 지원 안되는 버전 (unsound null safety) 일 수 있다.

그래서 만약에 pubspec.yaml 에 다음처럼 dart 버전을 2.12 로 지정하고 이런 package 를 dependencies 로 걸게 되면 처음부터 바로 null safety 에러를 만나게 된다.

    environment:
    sdk: '>=2.12.0 <3.0.0'

아마 개발한다면 이런 버전 충돌 문제를 피할수는 없을거 같다.  
그래서 이런 문제를 고려했는지, dart 버전 2.8 부터는 소스 파일별로 dart 언어 버전을 선택할수 있게 지원하는데 이를 **Per-library language version selection** 라고 부르고 있다.

적용하는 방법은 간단하다. 소스 파일 제일 처음에 다음과 같이 버전을 지정하는 comment 를 넣어주면 된다. 이를 **language version comment** 라고 부른다.

    // @dart=2.8

이런식으로 넣어주면 한 프로젝트 내에서 소스 파일별로 null safety 기능을 on off 할 수 있다. 예전 패키지를 사용하는 코드는 버전을 낮게 주고, 처음부터 작성하는 코드는 최신 버전으로 지정해주면 될것이다.  
물론 이런 문제에서의 정답은 모든 package 들이 현재 최신 버전의 요구에 맞게 null safety 업데이트를 해주는게 맞다. 그런 시점이 오면 더이상 이런 꼼수는 불필요 해질 것이다.

그런데 이렇게 해서 다 끝난게 아니고, 실행할때 인자를 추가 해줘야 한다.

    flutter run --no-sound-null-safety

하지만 이렇게 매번 실행 시마다 터미널에서 하기는 불편하므로, VS Code 예를 들면, 추가되는 --no-sound-null-safety 인자를 .vscode/settings.json 에 추가해서 기본인자로 주게 해야 한다.

ctrl + shift +p 를 눌러서 ==> Preference: Open Workspace Settings(JSON) 수행해서 .vscode/settings.json 을 연다. 그리고 다음처럼 기본 인자를 추가.

    {
      "dart.flutterRunAdditionalArgs": [
        "--no-sound-null-safety"
      ],
      "dart.vmAdditionalArgs": [
        "--no-sound-null-safety"
      ]
    }

그리고 VS Code 에서 평상시 대로 마우스로 실행버튼을 클릭하면 된다.

그리고 한가지 더, flutter build 명령을 사용할때도 동일하게 인자를 주고 실행해야 한다. 다음처럼 말이다.

    flutter build apk --no-sound-null-safety

<h3> <span style="color:orange"> 
참고
</span> </h3>

[https://dart.dev/null-safety/unsound-null-safety](https://dart.dev/null-safety/unsound-null-safety)

[https://dart.dev/guides/language/evolution#per-library-language-version-selection](https://dart.dev/guides/language/evolution#per-library-language-version-selection)
