---
layout: post
title: _UIViewServiceInterfaceErrorDomain Code=3
date: '2014-10-02T22:31:00.000+09:00'
tags:
    - objectC
    - ios8
modified_time: '2015-09-20T12:26:52.710+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-5989678619720567271
blogger_orig_url: https://jeremyko.blogspot.com/2014/10/uiviewserviceinterfaceerrordomain-code3.html
---

Xcode 6.0.1 IOS8 simulator 에서 MFMailComposeViewController 클래스를 이용한 메일 전송시 다음과 같은 에러 발생됨.

    viewServiceDidTerminateWithError: Error Domain=_UIViewServiceInterfaceErrorDomain Code=3 "The operation couldn’t be completed. (_UIViewServiceInterfaceErrorDomain error 3.)" UserInfo=0x7fcb9ca007a0 {Message=Service Connection Interrupted}
    2014-10-02 22:10:41.518 PushUpCounter[1241:22189] <MFMailComposeRemoteViewController: 0x7fcb9c86aa70> timed out waiting for fence barrier from com.apple.MailCompositionService

이게 IOS8 시뮬에서만 발생되는 버그인데, 실제 device로 테스트 시에는 정상 동작한다.
애플이 좀 급하게 IOS8 SDK를 내놓은 모양이네 ..^^
