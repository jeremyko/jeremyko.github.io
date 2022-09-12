---
layout: post
title: "AngularJS : Best practice, 'dot rule' "
date: '2014-12-26T22:56:00.001+09:00'
tags:
    - javascript
    - AngularJS
modified_time: '2017-03-02T14:46:50.333+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-3431927377824024751
blogger_orig_url: https://jeremyko.blogspot.com/2014/12/angularjs-best-practice-dot-rule.html
---

AngularJS 에서 model을 UI 에 바인딩 시키는 경우, 이른바 **'dot rule'** 이라고 하는 idiom 이 있다.

<h4> <span style="color:{{site.span_h4_color}}"> "ng-model을 사용 시 dot(.) 이 존재해야 한다. 그렇지 않다면 잘못하고 있는것이다"  
"Whenever you have ng-model there’s gotta be a dot in there somewhere. If you don’t have a dot, you’re doing it wrong."
</span> </h4>

[AngularJS Best Practices](http://www.youtube.com/watch?v=ZhfUv0spHCY) 에 나오는 내용이고 간단한 예제를 보이면서 설명하고 있다.
이 동영상에 나오는 예제와 동일한 맥락의 다음 예제를 살펴보자.
(scope 구분을 쉽게 하기 위해 붉은색 박스로 처리했다)

[에러가 발생되는 예제](https://jsbin.com/henuxa/2/edit?html,js,output)
{: .notice--info}

![blog-image](/assets/img/20141226-notworking-capture.JPG)

```html
<div ng-controller="ParentCtrl">
    This is ParentCtrl :
    <input type="text" ng-model="stringPrimitive" />
    <div ng-controller="ChildCtrl">
        This is ChildCtrl :
        <input type="text" ng-model="stringPrimitive" />
    </div>
</div>
```

ng-controller 안에 또다른 ng-controller directive를 사용하여, scope가 중첩된 경우이다. ParentCtrl 이 생성한 scope 안에 ChildCtrl scope가 포함된 것을 볼수 있다.
다음처럼 테스트 해본다.

-   먼저 ParentCtrl 의 input에 입력해본다. ChildCtrl 의 input도 동시에 업데이트 되는것을 볼수있다.

-   이번엔 ChildCtrl input에 입력해본다.

-   다시 ParentCtrl 의 input에 입력해본다. 이제 ChildCtrl input은 더이상 업데이트가 되지 않는다.

AngularJS 에서 `$scope` 객체는 상속 관계를 가진다. 스코프가 중첩되는 경우 하위 scope는 상위 scope를 prototypal inheritance (원형 상속) 하게 된다.

**위 예제의 AngularJS scope 상속관계를 평범한 javascript 에서의 상속 관계로 표현 하자면 다음과 같다.**

[https://jsbin.com/niwoco/2/edit?html,js,output](https://jsbin.com/niwoco/2/edit?html,js,output)
{: .notice--info}

```js
var parentScope = { property: '' };
var childScope = Object.create(parentScope);

console.log('************************* Set A to parentScope.property');
parentScope.property = 'A';
console.log('parentScope.text=' + parentScope.property);
console.log('childScope.text=' + childScope.property);

console.log('************************* Set B to childScope.property');
childScope.property = 'B';
console.log('parentScope.text=' + parentScope.property);
console.log('childScope.text=' + childScope.property);

console.log('************************* Set X to parentScope.property');
parentScope.property = 'X';
console.log('parentScope.text=' + parentScope.property);
console.log('childScope.text=' + childScope.property);
```

console 출력

    ************************* Set A to parentScope.property
    parentScope.text=A
    childScope.text=A
    ************************* Set B to childScope.property
    parentScope.text=A
    childScope.text=B
    ************************* Set X to parentScope.property
    parentScope.text=X
    childScope.text=B

<span style="color:{{site.span_emphasis_color}}">
결국 **dot rule idiom** 이란, javascript의 상속에 있어서의 child가 parent의 property를 숨기는 (hide/shadow) 문제를 방지하기 위한 idiom이라고 할수 있다.
</span>

그럼, 다시 처음의 예제로 돌아가 어떤일이 생긴건지 알아보도록 하자.

-   ParentCtrl 의 input에 입력
    ChildCtrl 의 scope는 ParentCtrl 의 scope를 상속 받는다. child scope의 stringPrimitive 는 prototype chain에 의해 parent scope의 stringPrimitive 를 찾게 될것이다.
    parent scope의 변경 사항이 그대로 child scope에 반영되므로 양쪽 모두 화면이 갱신된다.

-   ChildCtrl input에 입력
    이때 일이 틀어지기 시작한다. 이제 child scope는 자신만의 property, stringPrimitive를 갖게 된다. 즉, parent/child 연결이 끊긴다.
    이제 더이상 prototype chain에 의해 parent scope의 stringPrimitive 를 찾지 않게 될것이다. 때문에 ParentCtrl input에는 더 이상 갱신되지 않는다.

-   다시 ParentCtrl 의 input에 입력
    마찬가지로 child scope에 고유한 stringPrimitive property가 존재하므로, parent scope의 변경이 child scope에 아무런 영향을 미치지 못한다.
    즉, parent/child 연결이 끊긴 상태다.

<span style="color:{{site.span_emphasis_color}}">
이처럼 ng-model을 사용할 때, primitive type (객체가 아닌)을 직접 바인딩하게 되는 경우, 
</span> 중첩된 scope에서 이러한 부작용이 발생하게 되므로, 항상 dot(.)을 사용하는것이 좋다 (물론 중첩되지 않는 경우에도 dot을 사용해야 한다.

http://jeremyko.blogspot.kr/2014/12/angularjs-binding-primitive-reference.html" target="\_blank">이전글 참조</a>).

<h3> <span style="color:{{site.span_h3_color}}"> 
dot rule 적용 </span> </h3>

예제에서는 objStringPrimitive 객체를 사용하는 것을 의미한다.

[정상 동작하게 수정한 예제](https://jsbin.com/wadumi/4/edit?html,js,output)
{: .notice--info}

![blog-image](/assets/img/20141226-working-capture.JPG)

이제 ChildCtrl input에 입력을 하면 child scope에는 객체가 존재하지 않으므로, prototype chain을 따라 parent scope의 objStringPrimitive 에 반영된다.
child scope가 자신만의 property를 생성하는 것이 아니다. parent/child 연결이 이제 유지된다.

<h3> <span style="color:{{site.span_h3_color}}">
결론
</span> </h3>

scope가 중첩되는 경우, $scope객체는 일반적인 javascript의 prototypal inheritance 을 따르므로, child의 parent shadowing 문제가 발생 가능하다. 이를 방지하려면 primitive type을 직접 바인딩하는것을 피하고 객체를 참조하게 하여 (dot rule idiom), prototype chain 이 동작하게 한다.

<h3> <span style="color:{{site.span_h3_color}}"> 참고 </span> </h3>

[AngularJS Best Practices](http://www.youtube.com/watch?v=ZhfUv0spHCY)  
[https://github.com/angular/angular.js/wiki/Understanding-Scopes](https://github.com/angular/angular.js/wiki/Understanding-Scopes)  
[http://nathanleclaire.com/blog/2014/04/19/5-angularjs-antipatterns-and-pitfalls/](http://nathanleclaire.com/blog/2014/04/19/5-angularjs-antipatterns-and-pitfalls/)  
[http://jimhoskins.com/2012/12/14/nested-scopes-in-angularjs.html](http://jimhoskins.com/2012/12/14/nested-scopes-in-angularjs.html)
