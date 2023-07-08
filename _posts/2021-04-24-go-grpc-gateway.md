---
layout: post
title: go 에서 gRPC-Gateway 사용하기
date: '2021-04-24T12:03:00.008+09:00'
tags:
    - golang
    - proto buffer
    - gRPC-Gateway
    - gRPC
modified_time: '2021-04-25T13:09:22.085+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-6672523820203146098
blogger_orig_url: https://jeremyko.blogspot.com/2021/04/go-grpc-gateway.html
---

이번에는 go 에서 gRPC-Gateway 를 사용하는 방법에 대해 알아보려 한다.

최종 코드는 다음에서 확인 가능 : [https://github.com/jeremyko/grpc-gateway-sample](https://github.com/jeremyko/grpc-gateway-sample)

앞서 살펴본 go 에서 proto buffer 사용하기 와 거의 비슷한 절차이나 gRPC-Gateway 사용을 위해 추가되는 절차가 있다.  
다음 내용을 기초로 작성되었다 (그대로 따라 했더니 에러가 발생되어, 최신 go 버전에 맞게 내용이 추가된 부분이 있다. go 1.16 버전 기준).

[https://grpc-ecosystem.github.io/grpc-gateway/docs/tutorials/introduction/](https://grpc-ecosystem.github.io/grpc-gateway/docs/tutorials/introduction/)

<h3> <span style="color:{{site.span_h3_color}}"> 
필요한 패키지 다운로드
</span> </h3>

    go get google.golang.org/grpc
    go get github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway
    go get google.golang.org/protobuf/cmd/protoc-gen-go
    go get google.golang.org/grpc/cmd/protoc-gen-go-grpc

<h3> <span style="color:{{site.span_h3_color}}"> 
테스트 grpc 모듈 생성
</span> </h3>

임의의 위치에 my_grpc_module 디렉토리 생성. 테스트 편의를 위해 main package 를 포함한 모듈을 생성한다.  
모듈명은 github.com/jeremyko/my_grpc_module 으로 한다.

    root: ~/mydev# mkdir my_grpc_module
    root: ~/mydev# cd my_grpc_module

**go mod init 수행한다**

    root: ~/mydev/my_grpc_module# go mod init github.com/jeremyko/my_grpc_module

**생성된 go.mod 파일 내용 확인**

    root: ~/mydev/my_grpc_module# cat go.mod
    module github.com/jeremyko/my_grpc_module
    go 1.16

**간단한 테스트용 hello world 서비스를 작성한다.**

protocol buffer를 사용하여 gRPC service 를 정의해야 한다. 현위치에서 proto/helloworld/hello_world.proto 파일을 생성한다.

    mkdir -p proto/helloworld/
    cd proto/helloworld/
    touch hello_world.proto

**hello_world.proto 파일을 다음처럼 작성한다**

    syntax = "proto3";
    package helloworld;

    // The greeting service definition
    service Greeter {
        // Sends a greeting
        rpc SayHello (HelloRequest) returns (HelloReply) {}
    }
    // The request message containing the user's name
    message HelloRequest {
        string name = 1;
    }
    // The response message containing the greetings
    message HelloReply {
        string message = 1;
    }

**protoc 혹은 buf 를 사용해서 컴파일 한다.**

    cd ~/mydev/my_grpc_module

    protoc -I ./proto --go_out ./proto --go_opt paths=source_relative --go-grpc_out ./proto --go-grpc_opt paths=source_relative ./proto/helloworld/hello_world.proto

_튜토리얼 그대로 하면 에러가 발생한다.--> go_package 를 정의해야 함_

    protoc-gen-go: unable to determine Go import path for "helloworld/hello_world.proto"
    Please specify either:
    • a "go_package" option in the .proto source file, or
    • a "M" argument on the command line.
    See https://developers.google.com/protocol-buffers/docs/reference/go-generated#package for more information.
    --go_out: protoc-gen-go: Plugin failed with status code 1.

다시 hello_world.proto 파일을 수정한다(package 경로를 모듈명에 덧붙인 형식으로)

    syntax = "proto3";
    package helloworld;
    option go_package = "github.com/jeremyko/my_grpc_module/proto/helloworld" ;

    // The greeting service definition
    service Greeter {
        // Sends a greeting
        rpc SayHello (HelloRequest) returns (HelloReply) {}
    }
    // The request message containing the user's name
    message HelloRequest {
        string name = 1;
    }
    // The response message containing the greetings
    message HelloReply {
        string message = 1;
    }

다시 컴파일 하면 정상적으로 파일들이 생성된다.

    protoc -I ./proto --go_out ./proto --go_opt paths=source_relative --go-grpc_out ./proto --go-grpc_opt paths=source_relative ./proto/helloworld/hello_world.proto

    root: ~/mydev/my_grpc_module/proto/helloworld# ll
    total 16
    -rw-r--r-- 1 root root 3445 Apr 23 16:51 hello_world_grpc.pb.go
    -rw-r--r-- 1 root root 7260 Apr 23 16:51 hello_world.pb.go
    -rw-r--r-- 1 root root 432 Apr 23 16:51 hello_world.proto

gRPC server 코드를 작성한다.

/root/mydev/my_grpc_module 에 main.go 를 생성한다

```go
package main

import (
    "context"
    "log"
    "net"
    "google.golang.org/grpc"
    helloworldpb "github.com/jeremyko/my_grpc_module/proto/helloworld" ;
)

type server struct{}

func NewServer() *server {
    return &server{}
}

func (s *server) SayHello(ctx context.Context, in *helloworldpb.HelloRequest)
    (*helloworldpb.HelloReply, error) {
    return &helloworldpb.HelloReply{Message: in.Name + " world"}, nil
}

func main() {
    // Create a listener on TCP port
    lis, err := net.Listen("tcp", ":8080")
    if err != nil {
    log.Fatalln("Failed to listen:", err)
    }

    // Create a gRPC server object
    s := grpc.NewServer()
    // Attach the Greeter service to the server
    helloworldpb.RegisterGreeterServer(s, &server{})
    // Serve gRPC Server
    log.Println("Serving gRPC on 0.0.0.0:8080")
    log.Fatal(s.Serve(lis))
}
```

<h3> <span style="color:{{site.span_h3_color}}"> 
gRPC-Gateway 기능 추가
</span> </h3>

<span style="color:{{site.span_emphasis_color}}">
자..여기까지가 Go gRPC server 를 작업한것이고 ;-), 이제 추가로 gRPC-Gateway 를 위한 처리가 필요하다.  
</span>

다시 proto 파일에 다음처럼 변경을 해줘야 한다.

import "google/api/annotations.proto"; 를 추가한다

HTTP->gRPC mapping 을 추가한다

    syntax = "proto3";
    package helloworld;
    import "google/api/annotations.proto";
    option go_package = "github.com/jeremyko/my_grpc_module/proto/helloworld" ;

    // The greeting service definition
    service Greeter {
        // Sends a greeting
        rpc SayHello (HelloRequest) returns (HelloReply) {
            option (google.api.http) = {
                post: "/v1/example/echo"
                body: "*"
            };
        }
    }
    // The request message containing the user's name
    message HelloRequest {
        string name = 1;
    }
    // The response message containing the greetings
    message HelloReply {
        string message = 1;
    }

이제 stub 코드를 생성한다.

protoc 혹은 buf 를 사용한다.
protoc 를 사용하는 경우에는 googleapis 의존 파일을 다음 폴더 구조를 만들어서 복사해야 한다.

    proto
    ├── google
    │ └── api
    │   ├── annotations.proto
    │   └── http.proto
    └── helloworld
      └── hello_world.proto

my_grpc_module/proto/ 에 google/api 폴더 생성
mkdir -p google/api

사용할 googleapis 파일을 가져와야 한다. 임의의 위치에서 git clone 을 수행한다.

    git clone https://github.com/googleapis/googleapis.git

그리고 파일을 복사한다.

    cd googleapis/google/api
    cp annotations.proto /root/mydev/my_grpc_module/proto/google/api/.
    cp http.proto /root/mydev/my_grpc_module/proto/google/api/.

gRPC-Gateway generator 추가를 위해 protoc 를 실행한다.

    cd ~/mydev/my_grpc_module/

    root: ~/mydev/my_grpc_module# protoc -I ./proto --go_out ./proto --go_opt paths=source_relative --go-grpc_out ./proto --go-grpc_opt paths=source_relative --grpc-gateway_out ./proto --grpc-gateway_opt paths=source_relative ./proto/helloworld/hello_world.proto

실행하면 hello_world.pb.gw.go 파일이 생성된다.

    root: ~/mydev/my_grpc_module/proto/helloworld# ll
    total 24
    -rw-r--r-- 1 root root 3445 Apr 23 17:20 hello_world_grpc.pb.go
    -rw-r--r-- 1 root root 7664 Apr 23 17:20 hello_world.pb.go
    -rw-r--r-- 1 root root 6377 Apr 23 17:20 hello_world.pb.gw.go
    -rw-r--r-- 1 root root 586 Apr 23 17:10 hello_world.proto

main.go 를 수정한다

```go
package main

import (
    "context"
    "log"
    "net"
    "net/http" //추가
    "github.com/grpc-ecosystem/grpc-gateway/v2/runtime" //추가
     "google.golang.org/grpc"
     helloworldpb "github.com/jeremyko/my_grpc_module/proto/helloworld" ;
)

type server struct{
    helloworldpb.UnimplementedGreeterServer //추가
}

func NewServer() *server {
    return &server{}
}

func (s *server) SayHello(ctx context.Context, in *helloworldpb.HelloRequest)
    (*helloworldpb.HelloReply, error) {
    return &helloworldpb.HelloReply{Message: in.Name + " world"}, nil
}

func main() {
    // Create a listener on TCP port
    lis, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatalln("Failed to listen:", err)
    }

    // Create a gRPC server object
    s := grpc.NewServer()
    // Attach the Greeter service to the server
    helloworldpb.RegisterGreeterServer(s, &server{})
    // Serve gRPC Server
    log.Println("Serving gRPC on 0.0.0.0:8080")
    //log.Fatal(s.Serve(lis)) //막고 다음을 추가
    //------------------------------------------ START
    go func() {
        log.Fatalln(s.Serve(lis))
    }()

    // Create a client connection to the gRPC server we just started
    // This is where the gRPC-Gateway proxies the requests
    conn, err := grpc.DialContext(
        context.Background(),
        "0.0.0.0:8080",
        grpc.WithBlock(),
        grpc.WithInsecure(),
    )
    if err != nil {
        log.Fatalln("Failed to dial server:", err)
    }

    gwmux := runtime.NewServeMux()
    // Register Greeter
    err = helloworldpb.RegisterGreeterHandler(context.Background(), gwmux, conn)
    if err != nil {
        log.Fatalln("Failed to register gateway:", err)
    }

    gwServer := &http.Server{
        Addr:    ":8090",
        Handler: gwmux,
    }

    log.Println("Serving gRPC-Gateway on http://0.0.0.0:8090")
    log.Fatalln(gwServer.ListenAndServe())
    //------------------------------------------ END
}
```

go mod tidy 실행

    jeremyko: ~/mydev/go_dev/my_grpc_module# go mod tidy
    go: finding module for package google.golang.org/grpc/codes
    go: finding module for package google.golang.org/grpc/grpclog
    go: finding module for package google.golang.org/genproto/googleapis/api/annotations
    go: finding module for package github.com/grpc-ecosystem/grpc-gateway/v2/utilities
    go: finding module for package github.com/grpc-ecosystem/grpc-gateway/v2/runtime
    go: finding module for package google.golang.org/grpc/metadata
    go: finding module for package google.golang.org/grpc
    ... 중략 ...
    go: downloading github.com/golang/protobuf v1.5.2
    go: downloading golang.org/x/net v0.0.0-20210316092652-d523dce5a7f4
    go: downloading golang.org/x/sys v0.0.0-20210320140829-1e4c9ba3b0c4

<h3> <span style="color:{{site.span_h3_color}}"> 
서버 실행
</span> </h3>

자체에서 main package 로 테스트 하는 경우이므로 이제 바로 서버를 실행할수 있다.

    jeremyko: ~/mydev/go_dev/my_grpc_module# go run main.go
    2021/04/24 11:28:47 Serving gRPC on 0.0.0.0:8080
    2021/04/24 11:28:47 Serving gRPC-Gateway on http://0.0.0.0:8090

서버로 http post 요청을 보내본다

    curl -X POST -k http://localhost:8090/v1/example/echo -d '{"name": " hello"}'

    jeremyko: ~# curl -X POST -k http://localhost:8090/v1/example/echo -d '{"name": " hello"}'
    {"message":" hello world"}jeremyko: ~#

서버 응답이 오는것을 확인할수 있다.

<h3> <span style="color:{{site.span_h3_color}}"> 
느낀점
</span> </h3>

해줘야 할게 많고 복잡 ㅠ
