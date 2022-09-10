---
layout: post
title: 'go 에서 proto buffer 사용하기 '
date: '2021-04-22T21:18:00.010+09:00'
tags:
    - golang
    - proto buffer
modified_time: '2021-05-21T11:17:30.567+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-1699552116637081071
blogger_orig_url: https://jeremyko.blogspot.com/2021/04/go-proto-buffer.html
---

<h3> <span style="color:{{site.span_h3_color}}"> 
proto buffer 를 정의 -> go 로 변환하여 모듈을 만들고 -> 이 모듈을 로컬에서 호출해서 사용 하는 간단한 예제를 정리해 본다 (go 1.16 버전을 기준)
</span> </h3>

최종 코드는 다음을 참고 :
[https://github.com/jeremyko/go-proto-buffer-sample](https://github.com/jeremyko/go-proto-buffer-sample)

protoc, go protocol buffers plugin 설치.

    apt install -y protobuf-compiler
    go install google.golang.org/protobuf/cmd/protoc-gen-go

    주의점 : protoc-gen-go 는 go version 1.16 이상에서 설치 가능함.
    이 조건이 안된다면 go get 으로 바이너리를 직접 받는다.

        go get google.golang.org/protobuf/cmd/protoc-gen-go

먼저 모듈(github.com/jeremyko/my_proto 라고 정한다)을 만들기 위한 디렉토리를 만든다.

    mkdir my_proto
    cd my_proto

my_proto.proto 파일을 생성한다.

    syntax = "proto3";

    //package 명은 go_package 에 지정한 경로의 제일 마지막 것으로 해야 한다.
    package my_proto ;
    option go_package = "github.com/jeremyko/my_proto";
    //위 경로는 실제 배포시 적용될 것으로 해준다.

    message MyProto {
        string msg = 1;
        int32 number = 2;
    }

proto 파일이 있는 그 위치에 protoc 컴파일한다.

    ~/mydev/my_proto#  protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative my_proto.proto

    ~/my_dev/my_proto# ll
    total 12
    -rw-r--r-- 1 root root 4568 Apr 22 14:03 my_proto.pb.go
    -rw-r--r-- 1 root root 249 Apr 22 13:37 my_proto.proto

go mod init , go mod tidy 실행 한다.

    jeremyko: ~/mydev/my_proto# go mod init github.com/jeremyko/my_proto
    go: creating new go.mod: module github.com/jeremyko/my_proto
    go: to add module requirements and sums:
    go mod tidy
    jeremyko: ~/mydev/my_proto# go mod tidy
    go: finding module for package google.golang.org/protobuf/runtime/protoimpl
    go: finding module for package google.golang.org/protobuf/reflect/protoreflect
    ... 생략
    jeremyko: ~/mydev/my_proto# ll
    total 20
    -rw-r--r-- 1 jeremyko jeremyko 89 Apr 24 12:41 go.mod
    -rw-r--r-- 1 jeremyko jeremyko 739 Apr 24 12:41 go.sum
    -rw-r--r-- 1 jeremyko jeremyko 4534 Apr 24 12:40 my_proto.pb.go
    -rw-r--r-- 1 jeremyko jeremyko 310 Apr 24 12:39 my_proto.proto
    jeremyko: ~/mydev/my_proto# cat go.mod
    module github.com/jeremyko/my_proto

    go 1.16

    require google.golang.org/protobuf v1.26.0

이제 이 my_proto 모듈을 사용하기 위한 모듈(github.com/jeremyko/proto_test_main)을 별도로 만든다.  
새로 만드는 모듈은 main package 를 가지고 단지 my_proto 모듈 호출 테스트 용도 이다.  
(물론 my_proto 모듈 안에서 main package 를 만들어도 상관없다)

임의의 위치 (여기서는 ~/proto_test_main 폴더를 새로 만들었다) 에서 mod init 을 수행한다.

    jeremyko: ~/mydev/proto_test_main# go mod init github.com/jeremyko/proto_test_main
    go: creating new go.mod: module github.com/jeremyko/proto_test_main
    jeremyko: ~/mydev/proto_test_main# cat go.mod
    module github.com/jeremyko/proto_test_main

    go 1.16

~/my_dev/proto_test_main 에 main.go 를 만든다.

```go
package main

import (
    "log"
    pb "github.com/jeremyko/my_proto"
)

func main() {
    myProto := &pb.MyProto{msg:"some text", number:999}
    log.Printf("msg: %s", myProto.Getmsg())
    log.Printf("number: %d", myProto.Getnumber())
}
```

로컬에 작성된 my_proto 모듈을 사용하기 위해 일단 mod edit -replace 를 해준다.

    go mod edit -replace github.com/jeremyko/my_proto=../my_proto

이 내용은 다음링크를 참조

http://jeremyko.blogspot.com/2021/03/golang-module.html

    jeremyko: ~/mydev/proto_test_main# go mod edit -replace  github.com/jeremyko/my_proto=../my_proto
    jeremyko: ~/mydev/proto_test_main# cat go.mod
    module github.com/jeremyko/proto_test_main

    go 1.16

    replace github.com/jeremyko/my_proto => ../my_proto

mod tidy 를 수행한다

    jeremyko: ~/mydev/proto_test_main# go mod tidy
    go: finding module for package github.com/golang/protobuf/proto
    go: found github.com/golang/protobuf/proto in github.com/golang/protobuf v1.5.2
    ... 생략
    jeremyko: ~/mydev/proto_test_main# cat go.mod
    module github.com/jeremyko/proto_test_main

    go 1.16

    replace github.com/jeremyko/my_proto => ../my_proto

    require (
        github.com/golang/protobuf v1.5.2
        github.com/jeremyko/my_proto v0.0.0-00010101000000-000000000000
    )

main.go 를 실행하여 정상여부 확인

    go run main.go

    2021/04/22 14:25:11 msg: some text
    2021/04/22 14:25:11 number: 999

<h3> <span style="color:{{site.span_h3_color}}"> 
참고
</span> </h3>

[https://developers.google.com/protocol-buffers/docs/gotutorial](https://developers.google.com/protocol-buffers/docs/gotutorial)
