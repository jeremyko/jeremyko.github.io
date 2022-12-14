---
layout: post
title: 'AngularJS : Controller들간 데이터 공유, $broadcast , $on or service?'
date: '2014-12-24T15:08:00.000+09:00'
tags:
    - javascript
    - AngularJS
modified_time: '2017-03-02T14:37:40.695+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-4416925100014719675
blogger_orig_url: https://jeremyko.blogspot.com/2014/12/angularjs-controller-broadcaston-or.html
---

<h3> <span style="color:{{site.span_h3_color}}"> 
AngularJS sharing data between controllers : $broadcast/$on or service
</span> </h3>

예를 들어 A, B, C 컨트롤러간에 데이터를 공유해야 하는 경우가 있다라고 가정 해보자.

즉, A 컨트롤러가 변경시키는 어떠한 데이터를 다른 컨트롤러들에서 추적해서 화면등에 갱신이 필요한 경우라면 어떤 방법이 좋을까..

이러한 Controller들간의 데이터를 공유하는 방법에 대한 내용을 인터넷을 찾아보면
`$broadcast` , `$on` 을 사용해서 구현한 경우를 자주 볼수 있다.

그런데 이곳저곳 살펴보고 내린 나름의 결론은 서비스를 사용하는것이 정석이란 것이다.
서비스의 본래 목적은 어플리케이션의 비지니스 로직을 담기 위한 장소이지만, singleton의 특징을 활용,
이 서비스에 데이터를 보관하고 컨트롤러에서 inject하면 각 컨트롤러에서 동일한 데이터를 참조하게 할수 있는것이다.

[https://jsbin.com/xebife/3/edit?html,js,output](https://jsbin.com/xebife/3/edit?html,js,output)

물론 `$broadcast` , `$on` 를 사용하는 경우에는 그 이벤트 발생 시점이 명확하다는 장점이 있다.

<h3> <span style="color:{{site.span_h3_color}}"> 
결론
</span> </h3>

컨트롤러가 데이터 공유를 위해 `$broadcast` , `$on` 가 반드시 필요한지 먼저 생각해볼 필요가 있다. 단순한 데이터 공유 차원이라면 서비스를 활용하면 된다.
