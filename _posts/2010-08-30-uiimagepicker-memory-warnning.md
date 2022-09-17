---
layout: post
title: UIImagePicker Memory Warnning viewDidUnload
date: '2010-08-30T17:58:00.000+09:00'
tags:
    - Memory Warnning
    - UIImagePicker
    - viewDidUnload
    - iphone
modified_time: '2012-09-12T18:01:33.730+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-2121512824639340393
blogger_orig_url: https://jeremyko.blogspot.com/2010/08/uiimagepicker-memory-warnning.html
---

UIImagePicker 를 이용해서 사진을 찍어서 일기장 어플에 삽입시키는 로직을 구현중에 골치아픈(그리고 구글에서도 해답을 속시원하게 찾을수없는) 현상을 겪게 되었다.

앱하나만을 띄워놓고 실행시에는 잘 나타나지 않지만 다른 앱이 여러개 떠있는 상태에서, UIImagePicker 로 사진을 촬영시엔 거의 100% 메모리 부족 경고를 받게되고, 모든 뷰들이 초기화 되는 현상이다.

구글에서 조언을 얻어, 사진 사이즈를 줄여도보고 했지만.. 이 메모리 부족 현상을 완전히 없앨수는 없었다. 심지어 누군가는 이미지 피커 사용하면 viewDidUnload 호출이 먼저 발생 한다고 설명하는 내용도 봤다.

항상 메모리 부족경고를 받는 프로그램에서는 항상 viewDidUnload 가 호출되니 그렇게 판단한걸로 보여진다. 정상적인 경우라면 호출되지 않는다.

다른 앱도 분명히 이런 메모리 부족을 겪었을건데라고 의심하면서 비슷한 앱들을 실행해서 메모리 부족이 일어나게끔 환경을 맞춰주고 사진을 촬영했는데, 처리를 제대로 하는 앱도있고 그렇지 않은 것들도 있었다. 그렇다면 뭔가 해결할 방법은 존재하는것 같다.

삽질끝에 얻은 나름대로의 방안은 이런식이다.

Memory Warnning 발생시 사용중이던 데이터들을 백업하고 viewDidUnload 호출되고 이후 뷰들이 초기화 되는과정에서 flag처리와 임시로 보관중인 데이터를 복원해서 마치 아무일 없었던거처럼 처리하게 하는것.

물론 아주 성가시고 지저분한 방법이다... 다른방법은 뭐가있을련지..

## 결론

메모리 부족 경고와 함께 viewDidUnload 가 발생하며,

이런 상황에서 해결방법은 데이터를 백업하고 뷰 초기화시 데이터를 복원하는것이다.
UIImagePicker 사용시 가끔 발생할수 있는 메모리 부족을 막을 방법은 없다.
