---
layout: post
title: 'ios6 개발: Autorotation, viewDidUnload'
date: '2012-10-16T16:59:00.004+09:00'
tags:
    - FeelingNote
modified_time: '2013-06-07T16:08:51.996+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-3376543294470239870
blogger_orig_url: https://jeremyko.blogspot.com/2012/10/ios6-autorotation-viewdidunload.html
---

FeelingNote app에 ios6을 적용하다가 알게 된것을 간단히 정리.

## Autorotation 변경

ios6.0 이전에는 app이 처음 기동될때, 다음 순서를 따랐지만

-   shouldAutorotateToInterfaceOrientation:
-   viewDidLoad
-   viewWillAppear:
-   shouldAutorotateToInterfaceOrientation:
-   viewDidAppear:

ios6.0에서는 다음 순서로 app이 기동이 됬다.

-   viewDidLoad
-   viewWillAppear:
-   viewDidAppear:

즉, Autorotation 변경에 대한 메시지가 먼저 불려지지 않는다. 그리고, shouldAutorotateToInterfaceOrientation: 는 deprecated 되어 버렸고, 다음 3개가 추가되었다.

    -(BOOL)shouldAutorotate;
    -(NSUInteger)supportedInterfaceOrientations;
    -(UIInterfaceOrientation)preferredInterfaceOrientationForPresentation;

여기서 문제는 기존 shouldAutorotateToInterfaceOrientation 에 담겨져 있던 portrait, landscape에 맞춰 뷰의 버튼이나 컨트롤을 화면에 맞게 변경하는 부분을 ios6에서는 어떻게 수정하느냐 하는 것이었다. 그래서 다음처럼 수정하였다.

```objective-c
-(BOOL)shouldAutorotate
{
    return YES;
}

-(NSUInteger)supportedInterfaceOrientations
{
    //한번만 호출됨!!!
    return UIInterfaceOrientationMaskAll;//모든 방향 회전 허용
}

-(void)willAnimateRotationToInterfaceOrientation:(UIInterfaceOrientation)interfaceOrientation duration:(NSTimeInterval)duration
{
    //기존 뷰 배치 로직 수행...
}
```

## viewWillUnload and viewDidUnload

이것들 또한 deprecated 되버렸다. 기존에 viewDidUnload 에서 메모리 부족 상황을 판단해서 처리하던것들은 (정말로 메모리 부족으로 뷰가 내려지는 상황에서 처리할것들) didReceiveMemoryWarning 으로 옮겨야 했다.

한가지 흥미로운것은 기존 ios버전에서는 카메라 API를 사용하면 십중 팔구 메모리 부족경고를 받았으나, ios6.0에서는 지금까지 실기기(3GS)테스트 하면서 한번도 메모리 부족 메시지가 발생하지 않았다. 메모리 관리에서 내부적인 변경이 있는것 같다고 추측된다. 좀 좋아진듯~~.

## 20130607 추가함

**viewDidLoad**

ios 6에서는 메모리 부족 상황에서는 viewDidUnload 뿐 아니라 viewDidLoad 도 다시 호출이 되지 않는다. 때문에 viewDidUnload 에서 사용중인 뷰를 릴리즈 했다면, 이를 다시 재 생성까지 해주는 수고가 필요하다.
