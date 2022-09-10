---
layout: post
title: ' flutter : package plugin (using swift and kotlin) proximity_sensor '
date: '2021-03-28T20:16:00.012+09:00'
tags:
    - swift
    - flutter
    - dart
    - kotlin
modified_time: '2021-04-03T15:53:47.584+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-7668011564603300066
blogger_orig_url: https://jeremyko.blogspot.com/2021/03/proximitysensor-flutter-package-plugin.html
---

[https://pub.dev/packages/proximity_sensor](https://pub.dev/packages/proximity_sensor)

[https://github.com/jeremyko/flutter-proximity-sensor-plugin](https://github.com/jeremyko/flutter-proximity-sensor-plugin)

오래전 proximity pushup counter 라는 ios app 을 작성 했을때를 생각하면서 근접센서를 Flutter 에서 사용할수 있는 plugin을 개발해 보았다. plugin 을 만드는 것이라서 ios, andoid 플랫폼에 종속적인 개발과 동시에 Flutter 를 다뤄야 하는 부분도 있는것이 흥미로운 점이다.

flutter 에서 plugin 을 새로 만들면 ios는 swift, android는 kotlin 언어로  
기본 골격을 만들어 주긴 하는데, 이건 ios, android 의 method 를 호출하는 샘플이다.

event stream 으로 센서 정보를 계속 가져와야 하는데  
pub.dev 라는 공식 package 리포지토리에서 찾아보니  
proximity sensor 관련된 pacakge들은 몇개 있긴 하지만  
예전에 작성된것들이라서 java, object c 로 구현된거 밖에 없는거 같다.  
swift와 kotlin 으로 작성된 예제는 잘 찾을수 없었다.  
그래서 swift 와 kotlin으로 생성되는 기본 plugin skeleton 을 그대로 두고,  
여기에 event stream 을 처리하는것을 추가하는 것으로 만들었다 :-)
