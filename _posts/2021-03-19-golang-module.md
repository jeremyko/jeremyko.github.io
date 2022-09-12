---
layout: post
title: golang module 작성, 타 모듈에서 로컬 테스트 및 배포 개념 정리
date: '2021-03-19T02:06:00.025+09:00'
tags:
    - golang
modified_time: '2022-03-24T15:03:14.060+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-3508658223351720723
blogger_orig_url: https://jeremyko.blogspot.com/2021/03/golang-module.html
---

golang 모듈을 만들고 타 모듈에서 사용하는 것을 한번 정리해보았다.

내가 만들려고 하는 모듈이 <span style="color:{{site.span_emphasis_color}}">github.com/jeremyko/my_mod </span>이며,  
모듈 폴더는 <span style="color:{{site.span_emphasis_color}}">~/mydev/my_mod</span> 라고 가정한다.

<h3> <span style="color:{{site.span_h3_color}}"> 
작업중인 모듈 폴더에서 go mod init 을 수행한다.
</span> </h3>

    cd ~/mydev/my_mod

    go mod init github.com/jeremyko/my_mod

수행하면, 내가 만드는 모듈에서 사용되는 모든 외부 코드들의 의존성(dependency)을 추적하기 위한 용도의 go.mod 이 생성된다.

내용은 다음과 같다.

    module github.com/jeremyko/my_mod
    go 1.15

지금은 내가 만드는 모듈에 대한 정보만 존재하지만, 다른 개발자의 모듈을 import 하는 경우에는 그 의존성 정보들도 추가될것이다.

<h3> <span style="color:{{site.span_h3_color}}"> 
모듈 코드를 작성한다
</span> </h3>

my_mod.go 파일을 생성하고 다음 내용을 추가한다.

```go
package my_mod

func MyModTest() string {
    return "This is from MyModTest in my_mod"
}
```

<h3> <span style="color:{{site.span_h3_color}}"> 
모듈을 사용하는 코드 (모듈)작성
</span> </h3>

이제, 내가 작성한 my_mod 를 다른 모듈에서 사용하는 측면에서 살펴본다.

이것은 개발중인 모듈을 로컬에서 테스트 하는 용도로도 활용 될 것이다.

main package를 사용하기 위한 용도로 테스트 모듈을 새로 만드는것이다.

이 모듈은 github.com/jeremyko/sample 로 하기로 한다.

이를 위해 sample 디렉토리를 생성하고 이동.

    cd ~/mydev
    mkdir sample
    cd sample

sample 폴더 안에서 다음을 수행한다.

    go mod init github.com/jeremyko/sample

sample.go 를 새로 만든다.

```go
package main

import (
    "fmt"
    "github.com/jeremyko/my_mod" //테스트 해 볼 모듈을 import
)

func main() {
    message := my_mod.MyModTest()
    fmt.Println(message)
}
```

현재 디렉토리 구조는 다음과 같다.

    jeremyko:~/mydev$ tree
    .

    ├── my_mod
    │ ├── go.mod
    │ └── my_mod.go
    └── sample
    ├── go.mod
    └── sample.go

이제, my_mod 이 배포되어 공개된 모듈이라면 이 상태에서 바로 sample.go 를 실행하면 되겠지만,

지금은 my_mod 모듈이 아직 발행(publish) 되기 전이므로, go 를 사용하여 정식 다운로드를 할수 없다.

즉 `github.com/jeremyko/my_mod` 와 같은 경로를 현재 사용할수 없다.

그러므로 개발 단계에서는 sample.go 를 실행하기 위해서, 로컬 위치에서 my_mod 모듈을 찾을수 있게 임시로 조정을 해줘야 한다.

이를 위해서는 <span style="color:{{site.span_emphasis_color}}">go mod edit -replace</span> 명령을 사용한다.

sample 디렉토리 안에서 다음을 실행

    go mod edit -replace github.com/jeremyko/my_mod=../my_mod

이 명령을 실행하면, go.mod 파일은 다음처럼 변경된다.

    module github.com/jeremyko/sample
    go 1.15
    replace github.com/jeremyko/my_mod => ../my_mod

<h3> <span style="color:{{site.span_h3_color}}"> 
누락된 모듈을 연결
</span> </h3>

sample 모듈이 my_mod 모듈을 사용한다는것을 설정하기 위해 다음을 sample 폴더에서 수행한다.

즉 의존성을 정리하기 위한 명령을 수행하는 것이다.

    go mod tidy

명령을 실행하면 로컬에서 my_mod 코드를 찾아서 go.mod 파일에 다음처럼 마지막에 require 지시자가 추가된다.

require 지시자는 현재 모듈이 필요로 하는 모듈을 선언하는 용도이다.

    module github.com/jeremyko/sample
    go 1.15
    replace github.com/jeremyko/my_mod => ../my_mod
    require github.com/jeremyko/my_mod v0.0.0-00010101000000-000000000000

모듈 경로 다음에 나오는 숫자(v0.0.0-00010101000000-000000000000)는 유사 버전 넘버라고 불리며, 현재 배포되지 않은 모듈을 사용하기 때문에 임시로 생성된 버전 넘버라고 보면 된다.

실제 모듈이 배포가 되는 시점에는 공식 버전 관리 지침대로 버전 넘버를 설정해주면 된다. 물론 이 경우에는 replace 지시자 부분은 제거해 줘야 한다.

<h3> <span style="color:{{site.span_h3_color}}"> 
실행
</span> </h3>

sample 폴더에서 다음을 수행하여 결과를 확인한다.

    jeremyko:~/mydev/sample$ go run sample.go
    This is from MyModTest in my_mod

<h3> <span style="color:{{site.span_h3_color}}"> 
배포
</span> </h3>

이제 테스트가 완료된 모듈은 github.com/jeremyko/my_mod 리포지토리에 push 한다.

<h3> <span style="color:{{site.span_h3_color}}"> 
참고
</span> </h3>

[https://golang.org/doc/tutorial/create-module](https://golang.org/doc/tutorial/create-module)  
[https://golang.org/doc/modules/version-numbers](https://golang.org/doc/modules/version-numbers)

<h3> <span style="color:{{site.span_h3_color}}"> 
2022-03-24 update
</span> </h3>

go 1.18부터 도입된 [workspace]({% post_url 2022-03-24-golang-118-workspace-mode %}) 기능을 사용하면 mod replace 를 대체 가능하다
