---
layout: post
title: '(windows 10) IntelliJ IDEA  + IdeaVim 사용시 벨 소리 제거'
date: '2021-03-14T15:08:00.010+09:00'
tags:
    - IdeaVim
    - IntelliJ IDEA
modified_time: '2021-04-03T15:55:22.305+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-7717579987287003809
blogger_orig_url: https://jeremyko.blogspot.com/2021/03/intellij-idea-ideavim.html
---

IdeaVim 으로 편집기를 사용하는 경우 땡땡 벨소리 안나게 하는 팁

windows 사용자 홈 디렉토리 밑에 다음 파일을 생성한다

    .ideavimrc

다음 내용을 저장한다

    set visualbell

intellij 를 재기동한다.
