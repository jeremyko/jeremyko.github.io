---
layout: post
title: UITextField becomeFirstResponder 가 NO를 리턴하는 경우
date: '2013-06-10T00:19:00.002+09:00'
tags:
    - becomeFirstResponder
    - App개발
modified_time: '2013-06-10T00:23:15.146+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-3710255112728830650
blogger_orig_url: https://jeremyko.blogspot.com/2013/06/uitextfield-becomefirstresponder-no.html
---

예전에 개발해 놓고 거의 방치하다시피 한 FeelingNote를 ios6 대응 업그레이드 하려고 보니, 한가지 이상한 점 을 발견했다.

코드상에서 동적으로 UITextField 를 생성하고, 키보드 입력이 가능하게끔 becomeFirstResponder 호출을 하던 부분이 있었고,  
제대로 동작하고 있었는데 ios6에서는 제대로 동작하지 않는다. 이건 정말 난감..

수많은 삽질과 구글링을 통해 알아낸 사실은 ios6 에서는 동적으로 생성된 UITextField 가 화면에 보인 시점에서야 becomeFirstResponder 호출이 성공한다는 것이다.

구글링을 하다보면 becomeFirstResponder호출이 viewWillAppear 에서는 안되고 viewDidAppear 에는 성공한다는 것도 다 이런 맥락인듯하다.

즉, 생성한 뷰가 화면에 보이는 시점 (즉, view hierarchy 에 존재하는 시점, view의 window 속성이 nil이 아닌 경우라고)에 성공한다.

    Apple's doc in UIResponder:
    You may call this method to make a responder object such as a view the first responder. However, you should only call it on that view if it is part of a view hierarchy. If the view’s window property holds a UIWindow object, it has been installed in a view hierarchy; if it returns nil, the view is detached from any hierarchy.

결국, viewDidAppear 에서 앞서 생성한 UITextField를 알아내서 becomeFirstResponder 를 호출했더니 키보드 입력이 처리 되었다.  
그럼 여기서 의문점..
ios5와 왜 동일하게 동작하지 않는지? 이런것 때문에 허비한 시간을 생각하니, ...참.. 어렵다 어려워...
