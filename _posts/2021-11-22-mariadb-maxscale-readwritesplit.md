---
layout: post
title: mariadb maxscale 사용 시 주의점 (readwritesplit)
date: '2021-11-22T17:49:00.003+09:00'
tags:
    - mariadb
    - maxscale
modified_time: '2021-11-22T17:52:31.339+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-7924665233879070002
blogger_orig_url: https://jeremyko.blogspot.com/2021/11/mariadb-maxscale-readwritesplit.html
---

이번에 프로젝트를 하면서 겪었던 상황을 정리해본다.

mariadb [MaxScale](https://mariadb.com/kb/en/mariadb-maxscale-6-about-mariadb-maxscale/) 은 db proxy 로 위치투명성, 고가용성, 로드밸런싱 등의 목적으로 사용된다. 그런데 혹시 코드중에서 insert 후 바로 select 를 수행하는 경우에는, 방금 저장한 내용을 못 가져 오는 경우가 발생할수 있다.

[readwritesplit](https://mariadb.com/kb/en/mariadb-maxscale-6-readwritesplit/) 라는 모듈의 동작으로 발생될수 있는 상황인데, maxscale 은 위에 언급된 여러 기능을 구현하기 위해서 동적으로 로드되는(즉 shared library) 여러개의 모듈들이 존재한다. router 는 이러한 모듈의 한 종류이며, router 유형의 모듈 중에서, 읽기 성능 향상을 위한 모듈이 바로 readwritesplit 이다.

글자 그대로, 이것은 사용자의 query 가 읽기인지 쓰기 인지에 따라 master 혹은 slave 에서 분리 처리되게 해준다. 즉 읽기 query 는 여러개의 slave 중 하나에서 처리되고, 쓰기 query 는 master(single node)에서만 처리된다.

그런데, 이로 인해 문제가 되는 상황은, insert 후 바로 동일 데이터를 select 하는 경우에, 방금 저장한 데이터가 완전하게 slave 로 복제가 안된 상황이 발생될수 있고, 이렇게 되면 조회 결과가 없는 것처럼 처리될수 있다.

그래서 문서를 통해 찾은 해결책은 다음 2 가지가 있다.

-   mariadb 와 maxscale [설정을 변경하거나](https://mariadb.com/docs/architecture/components/maxscale/routers/readwritesplit/ensure-causal-consistency-maxscale-read-write-split-router/) 아니면

-   사용자의 프로그램에서 동일한 트랜잭션으로 묶는다. [문서를 보면](https://mariadb.com/kb/en/mariadb-maxscale-6-readwritesplit/#limitations), 동일한 트랜잭션 안에서는 select 쿼리가 무조건 master로 라우팅되므로 일관성 문제가 해결 된다.

<h3> <span style="color:{{site.span_h3_color}}">참고</span> </h3>

[https://mariadb.com/kb/en/mariadb-maxscale-6-readwritesplit/](https://mariadb.com/kb/en/mariadb-maxscale-6-readwritesplit/)

[https://mariadb.com/kb/en/mariadb-maxscale-6-readwritesplit/#limitations](https://mariadb.com/kb/en/mariadb-maxscale-6-readwritesplit/#limitations)

[https://mariadb.com/docs/architecture/components/maxscale/routers/readwritesplit/ensure-causal-consistency-maxscale-read-write-split-router/](https://mariadb.com/docs/architecture/components/maxscale/routers/readwritesplit/ensure-causal-consistency-maxscale-read-write-split-router/)
