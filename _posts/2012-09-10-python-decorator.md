---
layout: post
title: python decorator
date: '2012-09-10T22:57:00.001+09:00'
tags:
    - python
    - decorator
modified_time: '2012-09-10T22:57:22.597+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-6182511303626000019
blogger_orig_url: https://jeremyko.blogspot.com/2012/09/python-decorator.html
---

```python
#!/usr/bin/env python
# -*- encoding:utf-8-*-
import time

def elapsed_time(functor):
    def decorated(*args, **kwargs):
        start = time.time()
        functor()
        end = time.time()
        print "Elapsed time: %f" % (end-start)
    return decorated

@elapsed_time
def hello():
    print 'hello'
```

위의 코드는 다음과 동일.

```python
def hello(*args, **kwargs):
    print 'hello'
```

를 정의후 다음을 호출하는것과 같음

```python
hello = elapsed_time(hello)
```
