---
layout: post
title: 'golang : interface 로 전달받은 pointer 가 가르키는 type 을 찾기'
date: '2021-08-06T22:59:00.004+09:00'
tags:
    - golang
modified_time: '2021-08-07T11:19:41.153+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-5460075570817372066
blogger_orig_url: https://jeremyko.blogspot.com/2021/08/golang-interface-pointer-type.html
---

reflect.Indirect 를 활용하면, interface 로 전달된 인자가 pointer인 경우, 그 pointer가 가르키는 실제 type 에 대한 정보를 알수 있다. 여러 type을 전달받는 공통 함수 등을 작성할때 유용한 tip

```go
package main

import (
    "fmt"
    "reflect"
)

func testFunc(value interface{}) {
    fmt.Println("---------------------")
    fmt.Println("type = " + reflect.ValueOf(value).Type().String())
    fmt.Println("kind = " + reflect.ValueOf(value).Kind().String())
    if reflect.ValueOf(value).Kind()==reflect.Ptr {
        fmt.Println("--> pointer " )
        pointsToValue := reflect.Indirect( reflect.ValueOf(value))
        fmt.Println("-->",pointsToValue.Kind(),
            " - ",pointsToValue.Type(), " - ", pointsToValue)
        if pointsToValue.Kind()==reflect.Slice {
            fmt.Println("--> slice !! " )
        }
    }
}

func main() {
    var testVar1 string
    var testVar2 []string
    var testVar3 []*string

    testVar2 = append(testVar2,"A")
    testVar2 = append(testVar2,"B")

    testFunc(testVar1)
    testFunc(&testVar1)
    testFunc(testVar2)
    testFunc(&testVar2)
    testFunc(testVar3)
    testFunc(&testVar3)
}
```

[https://play.golang.org/p/1PEOYIPdyFb](https://play.golang.org/p/1PEOYIPdyFb)
