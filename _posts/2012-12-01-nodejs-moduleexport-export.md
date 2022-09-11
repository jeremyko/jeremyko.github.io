---
layout: post
title: node.js 에서 module.export, export차이는?
date: '2012-12-01T14:00:00.000+09:00'
tags:
    - node.js
modified_time: '2013-01-01T20:16:37.562+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-6952953763913209611
blogger_orig_url: https://jeremyko.blogspot.com/2012/12/nodejs-moduleexport-export.html
---

사용자의 스크립트에서 사용되는 전역객체, module의 속성 exports 와 그냥 exports의 사용시 의 차이점을 알아본다.

module.js 파일에서 사용자의 스크립트를 실행 시에는

    '(function (exports, require, module, __filename, __dirname) {
    ... 요청한 js의 코드 내용 ...
    });'

이렇게 사용자 모듈을 감싸서 함수 정의로 변경이 된다. 그리고 실제 함수 실행 시 전달되는 인자들은 다음과 같다.

```js
...

var args = [self.exports, require, self, filename, dirname];
return compiledWrapper.apply(self.exports, args);

...
```

즉, 사용자의 스크립트에서 사용되는 `exports` = `self.exports` , `module` = `self` 이다.

그러므로, 사용자 모듈 입장에서는 `exports` 는 `module.exports와` 동일하다.

이 객체에 뭔가를 추가 하고자 하는 경우, 두 가지를 혼용 사용해도 문제는 없겠지만, 주의할 점이 존재하는데, `module.exports` 에 새로운 객체를 생성해서 대입하는 경우다.

이경우, call by reference 로 전달된 module 객체의 exports 속성에 새로운 객체를 대입하는 것이며, 결과적으로 현재의 exports를 변경시켜 버린다.

한편 사용자 모듈에서 전달된 export 객체에 새로운 객체를 생성해서 대입하려 한다면, 아무 영향을 미치지 않는다. 전달된 객체(call by reference)는 객체 자체가 아니라 객체의 값만을 변경할수 있기에 그러한 시도는 무시된다.

```js
//driver.js

var mod = require('./mymodule.js');
mod.name();

//mymodule.js

exports.name = function () {
    console.log('My name is kojh');
};

/* 이것도 OK. 
module.exports.name = function() {
console.log('My name is June!!');
*/

//하지만, 여기서 다음처럼 했다면, mod.name(); 호출은 실패.
module.exports = { name: 'hehe~' }; //FAIL!!
```
