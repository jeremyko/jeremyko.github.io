---
layout: post
title: oracle PCC-S-02322, found undefined identifier 에러 발생의 어이없는 이유
date: '2017-12-24T13:35:00.001+09:00'
tags:
    - 삽질기록
    - oracle
modified_time: '2018-10-12T13:04:23.814+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-297329018388343033
blogger_orig_url: https://jeremyko.blogspot.com/2017/12/oracle-pcc-s-02322-found-undefined.html
---

지금 하고있는 프로젝트에서 오라클을 사용하는 부분이 있어서, 그 옛날의 proc 를 사용해서 개발하는 부분이 있는데, 이런 오류가 발생했다.

    PCC-S-02322, found undefined identifier

그런데 소스 변경은 하나도 하지 않고, 단지 서버 장비만 변경된 경우이다.
이것 저것 확인하여 알아낸 사실은, 소스 코드내에서의 문제가 아니고 oracle NLS_LANG 설정이 기존과 다르면 발생할수 있다.

이 설정이 기존에는 `AMERICAN_AMRRICA.UTF8` 인데 변경된 장비에서는 `AMERICAN_ARERICA.KO16KSC5601` 로 되어 있었다.
(그리고 pc 소스파일은 utf-8로 인코딩 된 상태)
이설정을 원래대로 하고 컴파일 하니 이상없게 된다.

그런데 왜 난데없이 undefined identifier 라는 에러가 튀어나오는지는 정말 어이없고 마음에 안든다.
