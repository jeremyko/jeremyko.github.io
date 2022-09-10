---
layout: post
title: 'linux python No such file or directory 발생 시 '
date: '2018-10-21T18:27:00.004+09:00'
tags:
    - python
modified_time: '2021-04-03T15:59:37.594+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-7660103568454859458
blogger_orig_url: https://jeremyko.blogspot.com/2018/10/python-no-such-file-or-directory.html
---

python 스크립트를 다음처럼 `shebang` 을 가진 것으로 작성하고

```sh
#!/usr/bin/env python
....
```

해당 스크립트(test.py 라고 가정)를 Linux 에서 수행시 `'No such file or directory'` 에러가 발생한다.

그러나 python test.py 로 실행 하는 경우에는 문제 없이 동작하는 경우,
해당 file format 이 dos 인지 확인해본다.
윈도우에서 작성해서 리눅스에서 구동하는 경우에 이런 문제가 발생된다.

vi 로 열어서 `:set ff` 로 확인해보고, unix 가 아닌 경우엔
`:set ff=unix` 수행하여 변경한다.
