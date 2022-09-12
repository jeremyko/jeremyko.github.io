---
layout: post
title: 'AngularJS : Binding Primitive, Reference Type'
date: '2014-12-23T18:17:00.003+09:00'
tags:
    - javascript
    - AngularJS
modified_time: '2015-09-20T12:26:24.951+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-2171127438747442789
blogger_orig_url: https://jeremyko.blogspot.com/2014/12/angularjs-binding-primitive-reference.html
---

<span style="color:{{site.span_emphasis_color}}"> 
AngularJS의 2 way-data binding 사용에 시행착오를 겪은 부분이 있어서 글로 남겨본다.
</span>

`directive` , `expression` 등을 사용하여 binding 하는 경우 `$watch` 가 생성되어 model의 변경을 감시하게 된다. 다음 예제를 한번 살펴보자.

<h3> <span style="color:{{site.span_h3_color}}"> 
예제1) 정상적으로 동작하지 않는 경우
</span> </h3>

버튼을 눌러봐도 값이 변경되지 않는다.

[https://jsbin.com/sonaye/16/edit?html,js,output](https://jsbin.com/sonaye/16/edit?html,js,output)

이 문제는 바인딩된 model(즉 `$watch` 가 감시하는 대상)이 javascript가 제공하는 primitive type(즉, string, number, boolean) 변수인 경우에는 그 변경 여부를 $watch를 통해 알수 없기 때문에 발생한다.

이에 대한 자세하고 훌륭한 설명들은 이미 많이 있지만, 다시 한번 정리하자면 다음과 같다.

```js
var strVal = 'string A';
var v1 = strVal;
strVal = 'string B';
console.log(v1); //string A
```

위 코드를 수행시키면 결과는 'string A' 이다. primitive type은 변경불가(immutable)하므로 한번 바인딩되면 거기서 끝이다.

즉 strVal 변수의 값을 변경할수 있지만 'string A' 자체를 변경할 수는 없다.

마찬가지로 위 controller 코드에서 setData()를 통해서 값을 변경해도 그변경을 $watch를 통해서 추적할수 없다. 그러므로 화면에서의 갱신도 일어나지 않는다.

이를 수정하려면 `$watch` 가 변경 가능한 그 무언가를 감시하게 하면 된다.

이를 위해서 위 예제를 다음처럼 변경했다.

<h3> <span style="color:{{site.span_h3_color}}"> 
예제2) 정상적으로 동작하는 경우
</span> </h3>

<span style="color:{{site.span_emphasis_color}}"> 
primitive type을 객체로 포장
</span>

[https://jsbin.com/nixupu/11/edit?html,css,js,output](https://jsbin.com/nixupu/11/edit?html,css,js,output)
{: .notice--info}

그럼 이제 Primitive Type만 피해가면 더이상의 문제는 없을까?

다음은 약간 주의가 필요할지도 모르는 (시행착오를 겪은 소감) 경우다.

<h3> <span style="color:{{site.span_h3_color}}"> 
예제3) Reference type, 정상적으로 동작하지 않는 경우
</span> </h3>

[https://jsbin.com/togame/17/edit?html,css,js,output](https://jsbin.com/togame/17/edit?html,css,js,output)
{: .notice--info}

<span style="color:{{site.span_emphasis_color}}"> 
기존 바인딩된 배열을 새로운 배열로 교체하는 경우이다.
</span>

`$http` 등을 통해 데이터를 받아서 기존 바인딩된 내용을 변경하려고 하는 경우가 해당될것이다(이 예제는 하단에 보여짐). 그런데 아주 기본적인 javascript 문법 사항이지만, (나처럼) 틀리기 쉬운것이..

```js
varArray = newArray;
```

이런식으로 처리한 경우다.  
당연히 이렇게 처리하면 안된다. 그리고 그 이유는 앞서와 마찬기지로 `$watch` 입장에서는 변경 사항이 없기 때문이다. 좀더 자세히 설명하면,

```js
var varArray = [{ val: 'default 1' }, { val: 'default 2' }];
$scope.myArrayData = varArray;
```

이를 통해 varArray와 myArrayData는 동일한 객체, 즉 배열에 대한 reference를 가지게 된다.  
만약 해당 배열을 직접 변경(push등) 한다면 당연히 그 변경이 화면에 반영된다. 하지만

```js
varArray = newArray; //새로운 배열로 교체가 필요해서...
```

처럼 사용한 경우 varArray는 새로운 newArray에 대한 새로운 reference를 갖게 되지만, 기존 myArrayData는 여전히 동일한 reference 을 가르키고 있을 뿐이다.

즉 `$watch` 가 보기엔 변경 사항이 없으므로 화면 갱신도 일어나지 않는다.

이를 수정하려면 기존 바인딩된 배열에 대한 reference를 유지하면서, 새로운 배열의 내용 전부를 기존 배열에 복사 (deep copy)를 수행해야 한다.

reference를 유지한다는 의미는 다음과 같은 코드는 안된다는 뜻이다.

```js
varArray = []; //배열 reset? 이때 기존 reference에 대한 연결을 잃는다.
varArray.push(...)
```

deep copy를 구현하는 방법은 기존 배열을 다 지우고 새로push하는 방법 등등 여러가지 있겠지만 , 간단하게 `angular.copy()` 를 이용할 수도 있다.

<h3> <span style="color:{{site.span_h3_color}}"> 
예제4) Reference type, 정상적으로 동작하는 경우
</span> </h3>

[https://jsbin.com/cinipa/5/edit?html,js,output](https://jsbin.com/cinipa/5/edit?html,js,output)
{: .notice--info}

<h3> <span style="color:{{site.span_h3_color}}"> 
예제5) 공유데이터 서비스를 2개의 컨트롤러가 이용하는 예제
</span> </h3>
예: $http로 json 객체 배열을 받은 경우를 등등

[https://jsbin.com/dujibi/6/edit?html,js,output](https://jsbin.com/dujibi/6/edit?html,js,output)
{: .notice--info}
