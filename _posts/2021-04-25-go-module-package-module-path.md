---
layout: post
title: 'go module과 package 관계, 그리고 module path 개념 정리 '
date: '2021-04-25T16:06:00.010+09:00'
tags:
    - golang
    - module path
modified_time: '2022-03-25T18:56:05.120+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-7542288710681271258
blogger_orig_url: https://jeremyko.blogspot.com/2021/04/go-module-package-module-path.html
---

[https://github.com/jeremyko/go-module-package-path-test](https://github.com/jeremyko/go-module-package-path-test)
{: .notice--info}

go 의 기본적인 개념 중에서 모듈, package 그리고 모듈 경로에 대해 알아본다.

지금 우리가 go 로 어떤 프로젝트를 수행한다고 가정해 보면, 다음처럼 정리 할 수 있겠다.

-   **이 프로젝트는 여러개의 모듈을 가질수 있다.**

-   **각 모듈에는 여러개의 package들이 존재할수 있다.**

-   **각 package 들은 1개 이상의 go 소스파일로 구성된다.**

-   **모듈내의 각 package 들은 자신의 package 폴더를 go mod init 으로 지정한 모듈 경로 + 폴더로 만들면 된다.**

-   **패키지명은 (반드시) 경로의 마지막 문자열로 해준다.**

-   **개발중인 모듈은 반드시 원격 저장소에 배포될 필요가 없다.**

-   **로컬 개발환경에서 동시에 여러 모듈을 개발중이고, A모듈이 B모듈을 사용해야 한다면, B모듈의 경로를 개발 중인 로컬 경로로 변경하는 작업이 필요하다.**(update: <span style="color:{{site.span_emphasis_color}}">go 1.18 부터는 그냥 [workspace]({% post_url 2022-03-24-golang-118-workspace-mode %}) 를 사용하면 됨</span>)

위 내용들을 하나씩 살펴보자. (go 1.16 기준으로 작성됨)

먼저 테스트를 위해 프로젝트 디렉토리를 임의로 하나 만든다.

    mkdir go_test

<h3> <span style="color:{{site.span_h3_color}}">
프로젝트는 여러개의 모듈을 가질수 있다.
</span> </h3>

go_test 프로젝트에 2개의 모듈이 필요하다고 임의로 가정하자.

<span style="color:{{site.span_emphasis_color}}"> common_mod </span> 모듈은 공통 기능을 위한 것  
<span style="color:{{site.span_emphasis_color}}"> main_mod </span> 모듈은 메인 package가 있는 실제 실행되는 모듈이라고 가정한다.

이 2개 폴더를 생성해준다.

    jeremyko: ~/mydev/go_test# mkdir common_mod
    jeremyko: ~/mydev/go_test# mkdir main_mod

자신이 작성하는 모듈이 github 같은 원격 저장소에 저장, 발행되지도 않고  
순수하게 로컬에서 동작하는 경우라면 (당연히) github... 등으로 시작되는  
모듈경로를 사용할 필요가 없다.

프로젝트 내에서 의미있는 분류로 모듈경로를 설정하면 된다.

먼저 common_mod 에서 go mod init 을 실행한다. 모듈 경로는 그냥 common_mod 로 한다.

    jeremyko: ~/mydev/go_test/common_mod# go mod init common_mod
    go: creating new go.mod: module common_mod
    jeremyko: ~/mydev/go_test/common_mod#cat go.mod
    module common_mod

    go 1.16

그리고 main_mod 에서도 go mod init 을 실행한다. 모듈 경로는 그냥 main_mod 로 한다.

    jeremyko: ~/mydev/go_test/main_mod# go mod init main_mod
    go: creating new go.mod: module main_mod
    jeremyko: ~/mydev/go_test/main_mod#cat go.mod
    module main_mod

    go 1.16

2개의 모듈을 만들었다.

<h3> <span style="color:{{site.span_h3_color}}">각 모듈에는 여러개의 package들이 존재할수 있다</span> </h3>

common_mod 모듈 디렉토리내에 패키지 폴더 test_pkg1 를 만들고, 그 안에 test_pkg1.go 파일을 생성한다.

    jeremyko: ~/mydev/go_test/common_mod# mkdir test_pkg1
    jeremyko: ~/mydev/go_test/common_mod# cd test_pkg1/
    jeremyko: ~/mydev/go_test/common_mod/test_pkg1# touch test_pkg1.go

그리고 아래 내용을 저장한다. 간단한 문자열을 리턴하는 기능을 정의했다.

```go
package test_pkg1

func MyPkgFunc1() string {
    return "This is from test_pkg1.common_mod"
}
```

또다른 패키지를 만들어 본다. common_mod 모듈 디렉토리내에 패키지 폴더 test_pkg2 를 만들고,  
그 안에 test_pkg2_1st.go 파일을 생성한다.

    jeremyko: ~/mydev/go_test/common_mod# mkdir test_pkg2
    jeremyko: ~/mydev/go_test/common_mod# cd test_pkg2
    jeremyko: ~/mydev/go_test/common_mod/test_pkg2# touch test_pkg2_1st.go

그리고 아래 내용을 저장한다. 간단한 문자열을 리턴하는 기능을 정의했다.

```go
package test_pkg2

func MyPkgFunc1st() string {
    return "This is from 1st.test_pkg2.common_mod"
}
```

이제 common_mod 모듈은 test_pkg1, test_pkg2 패키지를 가진다.

<h3> <span style="color:{{site.span_h3_color}}">각 package 들은 1개 이상의 go 소스파일로 구성된다.</span> </h3>

test_pkg2 에 test_pkg2_2nd.go 파일을 생성한다.

    jeremyko: ~/mydev/go_test/common_mod/test_pkg2# cp test_pkg2_1st.go test_pkg2_2nd.go

test_pkg2_2nd.go 파일 내용을 다음처럼 변경한다.

```go
package test_pkg2

func MyPkgFunc2nd() string {
    return "This is from 2nd.test_pkg2.common_mod"
}
```

test_pkg2 는 test_pkg2_1st.go, test_pkg2_2nd.go 2개의 go 소스 파일을 가진다.

<h3> <span style="color:{{site.span_h3_color}}">
모듈내의 각 package 들은 자신의 package 폴더를 go mod init 으로 지정한 모듈 경로 + 폴더로 만들면 된다.
</span> </h3>

package 폴더를 common_mod/test_pkg1, common_mod/test_pkg2 로 생성했고,  
실제 go 파일에서는 test_pkg1, test_pkg2 로 package 명을 사용했다.

<h3> <span style="color:{{site.span_h3_color}}">
패키지명은 (반드시) 경로의 마지막 문자열로 해준다.
</span> </h3>

_반드시 안 해줘도 동작하게 할 수는 있다. 하지만 매우 혼란스러울 것이다._

만약 common_mod/test_pkg1 폴더 내의 test_pkg1.go 파일을  
다음처럼 작성했다면 어떻게 될까.

```go
package test_pkg1_hahaha //XXX

func MyPkgFunc1() string {
    return "This is from test_pkg1.common_mod"
}
```

이런식으로 폴더명과 패키지명이 일치하지 않는다면, 이 패키지를 사용하는 입장에서는 다음처럼 사용해야 할것이다.  
이렇게 해도 동작은 된다.

```go
import "common_mod/test_pkg1"
.....
fmt.Println(test_pkg1_hahaha.MyPkgFunc1())
```

하지만 사용하는 측면에서 매우 혼란스럽기 때문에 폴더명과 패키지명을 일치시켜야 한다.

<h3> <span style="color:{{site.span_h3_color}}">
개발중인 모듈은 반드시 원격 저장소에 배포될 필요 없다.
</span> </h3>

이제 main_mod 모듈내에 새로 패키지를 작성해본다.  
여기에는 test_pkg3 이라고 가정한다.  
test_pkg3 폴더를 생성하고 그안에 test_pkg3.go 파일을 다음처럼 작성한다.

    jeremyko: ~/mydev/go_test/main_mod# mkdir test_pkg3
    jeremyko: ~/mydev/go_test/main_mod# cd test_pkg3
    jeremyko: ~/mydev/go_test/main_mod/test_pkg3# touch test_pkg3.go

```go
package test_pkg3

func MyPkgFunc() string {
    return "This is from test_pkg3.main_mod"
}
```

main_mod 모듈은 main 함수를 실행하는 모듈이기 때문에 main package 를 정의해준다.  
main_mod 폴더 안에 main.go 를 생성하고 다음처럼 작성한다.

    jeremyko: ~/mydev/go_test/main_mod# touch main.go

```go
package main

import (
    "fmt"
    "main_mod/test_pkg3"
)

func main() {
    fmt.Println(test_pkg3.MyPkgFunc())
}
```

개발중인 main_mod **모듈 내에 정의된** test_pkg3 패키지는 바로 사용할수 있다.  
이처럼 개발중인 모듈은 원격 저장소에 저장될 필요가 없이 바로 main.go 를 실행할수 있다.

    jeremyko: ~/mydev/go_test/main_mod# go run main.go
    This is from test_pkg3.main_mod

디렉토리 경로와 패키지명 이 올바르다면 , 바로 로컬에서 모듈을 실행해 볼수 있다.

지금까지 작업한 폴더 구조는 다음과 같다.

    jeremyko: ~/mydev/go_test# tree
    .
    ├── common_mod
    │ ├── go.mod
    │ ├── test_pkg1
    │ │ └── test_pkg.go
    │ └── test_pkg2
    │   ├── test_pkg2_1st.go
    │   └── test_pkg2_2nd.go
    └── main_mod
      ├── go.mod
      ├── main.go
      └── test_pkg3
        └── test_pkg3.go

<h3> <span style="color:{{site.span_h3_color}}">
여러 모듈을 개발중이며 다른 모듈의 기능을 사용하는 경우
</span> </h3>

자 이제 개발중인 main_mod 모듈에서 또 다른 개발중인 common_mod 모듈의 기능을 사용하는 경우를 살펴본다.  
내가 로컬에서 수정/개발중인 타 모듈을 사용해야 하는 경우이다.  
(수정할 필요 없는 타모듈은 당연히 그냥 import 해서 사용할면 될것이다)  
main_mod 의 main.go 를 다음처럼 수정한다.

```go
package main

import (
    "fmt"
    "main_mod/test_pkg3"
    "common_mod/test_pkg1"
    "common_mod/test_pkg2"
)

func main() {
    fmt.Println(test_pkg3.MyPkgFunc())
    fmt.Println(test_pkg1.MyPkgFunc1())
    fmt.Println(test_pkg2.MyPkgFunc1st())
    fmt.Println(test_pkg2.MyPkgFunc2nd())
}
```

이 상태에서 go run main.go 를 수행하면 당연히 common_mod 모듈의 패키지들을 찾을수 없다는 에러가 발생할것이다.  
(로컬에서 개발중인 모듈을 go 는 찾을수 없으므로)

    jeremyko: ~/mydev/go_test/main_mod# go run main.go
    main.go:7:5: package common_mod/test_pkg1 is not in GOROOT (... 생략
    main.go:8:5: package common_mod/test_pkg2 is not in GOROOT (... 생략

이것을 해결하려면 앞서 살펴본golang module 작성, 타 모듈에서 로컬 테스트 및 배포 개념 정리 내용중에  
**go mod edit -replace** 명령으로 타 모듈의 경로를, 개발 중인 로컬 경로로 변경하는 작업이 필요하다.

main_mod 폴더에서 다음을 수행해서 로컬경로로 대치한다.

    go mod edit -replace common_mod=../common_mod
    jeremyko: ~/mydev/go_test/main_mod# go mod edit -replace common_mod=../common_mod
    jeremyko: ~/mydev/go_test/main_mod# cat go.mod
    module main_mod

    go 1.16

    replace common_mod => ../common_mod

그리고 go mod tidy 로 의존성 해결을 해준다.

    jeremyko: ~/mydev/go_test/main_mod# go mod tidy
    go: found common_mod/test_pkg1 in common_mod v0.0.0-00010101000000-000000000000
    go: found common_mod/test_pkg2 in common_mod v0.0.0-00010101000000-000000000000
    go.mod 를 확인해본다.

    jeremyko: ~/mydev/go_test/main_mod# cat go.mod
    module main_mod

    go 1.16

    replace common_mod => ../common_mod

    require common_mod v0.0.0-00010101000000-000000000000

이제 main_mod 모듈을 실행할수 있다.

    jeremyko: ~/mydev/go_test/main_mod# go run main.go
    This is from test_pkg3.main_mod
    This is from test_pkg1.common_mod
    This is from 1st.test_pkg2.common_mod
    This is from 2nd.test_pkg2.common_mod

<h3> <span style="color:{{site.span_h3_color}}">
2022-03-24 update
</span> </h3>

[go 1.18부터 도입된 workspace 기능]({% post_url 2022-03-24-golang-118-workspace-mode %})
을 사용하면 mod replace 를 대체 가능하다
