---
layout: post
title: golang 1.18의 workspace mode 알아보기
date: '2022-03-24T14:51:00.013+09:00'
tags:
    - golang
modified_time: '2022-09-01T22:24:13.100+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-1621826929243890544
blogger_orig_url: https://jeremyko.blogspot.com/2022/03/golang-118-workspace-mode.html
---

이번에 1.18 버전이 나오면서 크게 변경된 것 (generic, fuzzing, 성능 향상, workspace 모드) 중에서 나에겐 workspace 의 유용함이 가장 먼저 다가왔다.  
실제 프로젝트에서 golang 으로 개발하는 경우에 아주 유용한 기능이라고 생각된다.

workspace 는 개발자가 동시에 여러 모듈들을 생성/수정 하면서 개발하는 경우 유용하게 사용될 기능이다.
{: .notice--success}

예를 들자면, 어떤 모듈A 를 수정해야 하는데 , A모듈이 사용 중인 모듈B 도 같이 동시에 수정이 필요한 경우를 가정해보자.

개발 시 분명히 마주치게 될 이런 문제를 1.18 버전 이전에는

A모듈의 go.mod 파일에서 `replace` 지시자를 사용해서, B모듈의 경로를, 지금 수정하는 로컬 위치로 변경 해줘야만 했다.
(이전글 참조 : golang module 작성, 타 모듈에서 로컬 테스트 및 배포 개념 정리) .

로컬 경로는 개발자마다 전부 다를 것이므로, 소스 관리 차원에서도 함부로 `replace` 지시자 가 들어간 go.mod 파일을 소스 repo 에 commit,push 할 수 없었다. 한마디로 매우 성가시고 불편했다.

만약 replace 사용 없이 하려면 각각의 모듈 별로 개별 test 코드를 만들어서 독립적으로 완벽히 테스트 하면서 개발 후, 동시에 repo에 발행한다며, 아마도 어쩌면? 가능 했을지도 모른다.

하지만 개발자는 신이 아니기에, 이런 식으로 개발 할 수는 없을 듯 하다. A 를 수정하다 보면 B 도 수정 필요하고 또 그 반대의 경우도 수시로 발생 될 수 있기 때문이다.

그런데 이제 workspace 를 사용하면 훨씬 편한 개발이 가능해졌다. `go.work` 파일이 존재하면 workspace mode로 동작하게 된다.

그럼 이전 글에서 설명된 예제를 workspace 를 사용하는 것으로 수정을 해본다.

<!-- ### 모듈을 만들고 go 1.18 workspace를 사용해서 타 모듈에서 사용하기 -->
<h3> <span style="color:{{site.span_h3_color}}"> 모듈을 만들고 go 1.18 workspace를 사용해서 타 모듈에서 사용하기 </span> </h3>

<!-- #### workspace 를 위한 디렉토리 생성 -->

<h4> <span style="color:{{site.span_h4_color}}"> workspace 를 위한 디렉토리 생성 </span> </h4>

먼저 workspace 사용을 위해서는 go.work 파일 관리가 용이 하게 끔, 별도의 디렉터리를 하나 만들어서 그 내부에서 개발을 진행 하는 것이 깔끔하다.  
즉 workspace 별 디렉터리를 새로 만들고 그 안에 내가 신규/수정하려는 모듈들을 모아 놓고 작업을 하는 것이다.
(이게 필수는 아니다. 아래에 기술되지만 go.work로부터 의 상대 경로로 모듈을 찾기 때문에 별도 디렉터리 없이도 물론 가능하다. 깔끔한 구조는 안될 것이지만)

그래서 먼저 ~/mydev/workspace_test 라는 디렉토리 를 만든다.

    cd ~/mydev
    mkdir workspace_test
    cd workspace_test

이제 여기에서 신규로 모듈을 만들거나 기존 모듈을 수정하는 작업을 수행하면 된다. 여기서 는 전부 다 새로 모듈을 만드는 것을 가정해보겠다.
기능을 제공하는 모듈과 main package를 가진 모듈까지 전부 신규로 개발하는 초 대형(?) 프로젝트라고 가정해본다. (기존 모듈을 수정하는 경우라면 git clone으로 가져와서 작업하면 될 것이다)
내가 만들려고 하는 모듈이 github.com/jeremyko/my_mod 라고 가정한다.

일단 새로운 모듈을 위한 디렉토리를 만든다. (workspace 디렉토리 내에 새로운 디렉토리 를 만드는 것임)

    mkdir my_mod
    cd my_mod

<!-- #### 신규 작성할 모듈 폴더에서 go mod init 을 수행한다. -->
<h4> <span style="color:{{site.span_h4_color}}">신규 작성할 모듈 폴더에서 `go mod init` 을 수행한다</span> </h4>

    cd ~/mydev/workspace_test/my_mod

    go mod init github.com/jeremyko/my_mod

수행하면 go.mod 이 생성된다. 내용은 다음과 같다.

    module github.com/jeremyko/my_mod
    go 1.18

<!-- #### 모듈 코드를 작성한다 -->
<h4> <span style="color:{{site.span_h4_color}}">모듈 코드를 작성한다</span> </h4>

my_mod.go 파일을 생성하고 다음 내용을 추가한다.

```go
package my_mod

func MyModTest() string {
    return "This is from MyModTest in my_mod"
}
```

<!-- #### my_mod 모듈을 사용하는 코드 (모듈)작성 -->
<h4> <span style="color:{{site.span_h4_color}}">my_mod 모듈을 사용하는 코드 (모듈)작성</span> </h4>

이제, 내가 작성한 my_mod 를 다른 모듈에서 사용하는 측면에서 살펴본다.
그 모듈도 신규로 개발해야 하는 상황이라고 가정 해본다. 신규로 개발해야 할 또 다른 모듈은 main package를 가진 모듈이다.
이 모듈은 github.com/jeremyko/sample 로 하기로 한다.

이를 위해 sample 디렉토리를 생성하고 이동한다.

    cd ~/mydev/workspace_test
    mkdir sample
    cd sample

sample 폴더 안에서 다음을 수행한다.

    go mod init github.com/jeremyko/sample

sample.go 를 새로 만든다.

```go
package main

import (
    "fmt"
    "github.com/jeremyko/my_mod" // 개발중인 신규 모듈을 import
)

func main() {
    message := my_mod.MyModTest()
    fmt.Println(message)
}
```

현재 디렉토리 구조는 다음과 같다.

    └── workspace-test
    ├── my_mod
    │ ├── go.mod
    │ └── my_mod.go
    └── sample
    ├── go.mod
    └── sample.go

이제, my_mod 이 배포되어 공개된 모듈이라면 이 상태에서 바로 sample.go 를 실행하면 되겠지만, 지금은 my_mod 모듈이 아직 공식 repo에 발행(publish) 되기 전 이므로, go 를 사용하여 정식 다운로드를 할수 없다.  
즉 github.com/jeremyko/my_mod 와 같은 경로를 현재 사용 할 수 없다.
그래서 실행 시 다음과 같은 에러가 발생한다.

    ~/mydev/workspace-test/sample# go run sample.go
    sample.go:6:2: no required module provides package github.com/jeremyko/my_mod: go.mod file not found in current directory or any parent directory; see 'go help modules'

<!-- ### 그래서 go 1.18 이전까지는 개발을 위해서는 아래 내용처럼, go.mod 파일 내에서 replace 를 사용해서 로컬 위치로 경로를 조정해 줘야 했지만, 이젠 workspace를 사용하면 된다. -->

<span style="color:{{site.span_emphasis_color}}">그래서 go 1.18 이전까지는 개발을 위해서는 아래 내용처럼, go.mod 파일 내에서 replace 를 사용해서 로컬 위치로 경로를 조정해 줘야 했지만, 이젠 workspace를 사용하면 된다</span>

_~~그러므로 개발 단계에서는 sample.go 를 실행하기 위해서, 로컬 위치에서 my_mod 모듈을 찾을 수 있게 임시로 조정을 해줘야 한다. 이를 위해서는 go mod edit 명령을 사용한다. sample 디렉토리 안에서 다음을 실행~~_

_~~go mod edit -replace github.com/jeremyko/my_mod=../my_mod~~_

_~~이 명령을 실행하면, go.mod 파일은 다음처럼 변경된다.~~_

_~~module github.com/jeremyko/sample~~_
_~~go 1.15~~_
_~~replace github.com/jeremyko/my_mod => ../my_mod~~_

<!-- #### workspace 생성하기 -->

<h4> <span style="color:{{site.span_h4_color}}"> workspace 생성하기 </span> </h4>

workspace 는 go.work 파일 내의 내용으로 정의가 된다.

workspace_test 디렉토리 에서 다음 명령을 실행한다.

    go work init

빈 go.work 파일이 생성된다. 생성된 go.work 파일에는 go directive 만 설정된 상태이다

    ~/mydev/workspace-test# cat go.work
    go 1.18

현재 디렉토리 구조는 다음과 같다.

    └── workspace-test
    ├── go.work
    ├── my_mod
    │ ├── go.mod
    │ └── my_mod.go
    └── sample
    ├── go.mod
    └── sample.go

<!-- #### workspace 에 모듈을 추가 -->
<h4> <span style="color:{{site.span_h4_color}}">workspace 에 모듈을 추가</span> </h4>

이제 my_mod 모듈을 workspace 에 추가해서, 해당 workspace 에 있는 타 모듈이 사용 할 수 있게 한다.

workspace 에 속한 여러 모듈들을 공식 문서에서는 workspace module 이라고 부르고 있다.

my_mod 를 workspace module 로 만들기 위해서, workspace_test 디렉토리 에서 다음 명령을 실행한다.

    go work use ./my_mod

수행하면 go.work 파일의 내용은 다음처럼 변경된다.

    ~/mydev/workspace-test# go work use ./my_mod/
    ~/mydev/workspace-test# cat go.work
    go 1.18

    use ./my_mod

workspace 모듈의 정의는 go.work 파일로부터의 상대 경로로 모듈 위치를 지정하게 된다.

이제 sample 모듈에서는 동일한 workspace 에 있는 my_mod 모듈을 그대로 사용 할 수 있다.

그런데 여기서 주의 할 점은 하위 디렉토리 까지 한번에 모두 적용되는 것은 아니란 것이다. 예를 들어 만약 my_mod 디렉토리 가 또 다른 하위 모듈 디렉토리 (예를 들어 my_sub_mod) 를 가지고 있다면 해당 모듈은 별도로 use 지시자를 사용해서 추가해 줘야 한다.

즉, go.mod 가 있는 모든 모듈은 하위 디렉토리 여부로 판단 하는 게 아니고 모두 개별 추가해 줘야 한다.

    └── workspace-test
    ├── go.work
    ├── my_mod
    │ ├── go.mod
    │ ├── my_mod.go
    │ └── my_sub_mod
    │ ├── go.mod
    │ └── my_sub_mod.go
    └── sample
    ├── go.mod
    └── sample.go

만약 위와 같은 구조라면 sample.go 에서 my_sub_mod 를 사용하기 위해서는 별도로 다음을 한번 더 수행해줘야 한다.

    ~/mydev/workspace-test# go work use ./my_mod/my_sub_mod
    ~/mydev/workspace-test# cat go.work
    go 1.18

    use (
    ./my_mod
    ./my_mod/my_sub_mod
    )

<!-- #### 실행 -->
<h4> <span style="color:{{site.span_h4_color}}">실 행</span> </h4>

sample 폴더에서 다시 실행해서 my_mod 를 제대로 가져와서 정상적으로 실행되는 지 확인한다.

    ~/mydev/workspace-test/sample# go run sample.go
    This is from MyModTest in my_mod

**이처럼 workspace 를 활용하면,**

개발하는 모든 모듈을 하나의 동일한 작업 공간으로 관리 할 수 있다.
{: .notice--info}
모든 모듈들이 동일한 작업 공간에 있으므로 한 모듈을 변경하고 다른 모듈에서 바로 그것을 사용 할 수 있다.
{: .notice--info}
기존에 go.mod 마다 설정해줘야 했던 replace 를 없애고 하나의 go.work 파일을 통해서 여러 모듈을 동시에 개발 할 수 있게 된다.
{: .notice--info}

<!-- #### workspace replace 지시자 -->
<h4> <span style="color:{{site.span_h4_color}}">workspace replace 지시자</span> </h4>

자 그런데 여기서.. workspace 에도 replace 를 사용 할 수도 있다.. 사용법은 기존에 go.mod 내에 사용하던 mod replace 와 동일하다.

나는 이게 좀 헷갈렸는데 , 나름 정리를 해보면, 일단 우선 순위가 높다.

즉, workspace module 내에 go.mod 에 정의된 replace 보다 높은 우선 순위를 갖게 된다. 동일한 모듈에 대해서 go.mod 와 go.work 에서 replace 를 사용했다면 , go.work 에 정의된 것이 적용된다는 개념이다.

아마도 내 생각으로는 replace는 안 쓰고도 개발이 가능 할것 같지만 (그래서 아직도 약간 헷갈리는 게 사실), replace 와 use 지시자를 동시에 사용 할 수도 있으니, 좀 더 유연하게 개발 할 수는 있을 것 같다.

<!-- ### 참고 -->
<h3> <span style="color:{{site.span_h3_color}}">참고</span> </h3>
{: .notice--accent}

[https://go.dev/doc/tutorial/workspaces](https://go.dev/doc/tutorial/workspaces)  
[https://go.dev/ref/mod#workspaces](https://go.dev/ref/mod#workspaces)  
[https://go.dev/blog/go1.18](https://go.dev/blog/go1.18)  
[https://go.googlesource.com/proposal/+/master/design/45713-workspace.md](https://go.googlesource.com/proposal/+/master/design/45713-workspace.md)
