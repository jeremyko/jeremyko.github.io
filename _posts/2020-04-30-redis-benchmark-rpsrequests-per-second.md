---
layout: post
title: redis-benchmark 에서의 RPS(requests per second) 의미
date: '2020-04-30T11:47:00.003+09:00'
tags:
    - redis-benchmark
    - redis
modified_time: '2021-04-03T15:56:38.359+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-4109560623283999847
blogger_orig_url: https://jeremyko.blogspot.com/2020/04/redis-benchmark-rpsrequests-per-second.html
---

redis-benchmark 를 수행했을때 출력되는 requests per second(RPS) 의미를 client 개수와 연관해보면 다음과 같다.

    RPS = 전체 요청 명령 수 / client 가 나눠서 모두 처리한 시간

즉, 다음과 같이 백만건의 set 명령을 client 1 개가 수행하게 실행한 경우

    redis-benchmark -d 70 -r 1000000 -n 1000000 -c 1 -t set

client 1 개가 백만번 set 을 수행해서 걸린 시간에 대한 rps 의미이며, 만약 client 2 개로 테스트 했다면

    redis-benchmark -d 70 -r 1000000 -n 1000000 -c 2 -t set

client 2 개가 백만번의 set 명령을 나눠 수행하는데 걸린 시간에 대한 rps 의미이다 (이경우, client 별로 명령 처리가 정확하게 등분되지는 않기 때문에 정확하게 client당 50만개씩 수행되는것은 아니다).

즉 요청한 전체 수행 건수를 client 들이 나눠서 모두 처리하는데 걸린 시간으로 rps 를 계산한다 (개인적으로 명확하게 알지 못했던 부분).

    $ redis-benchmark -d 70 -r 1000000 -n 1000000 -c 1 -t set
    ====== SET ======
    1000000 requests completed in 62.08 seconds
    1 parallel clients
    70 bytes payload
    keep alive: 1

    100.00% <= 1 milliseconds
    100.00% <= 2 milliseconds
    100.00% <= 2 milliseconds
    16107.73 requests per second


    $ redis-benchmark -d 70 -r 1000000 -n 1000000 -c 2 -t set
    ====== SET ======
    1000000 requests completed in 26.71 seconds -> 동일한 건수를 client가 나눠서 더 빨리 처리함
    2 parallel clients
    70 bytes payload
    keep alive: 1

    100.00% <= 1 milliseconds
    100.00% <= 1 milliseconds
    37443.37 requests per second -> RPS도 증가

초당 처리 해야 할 명령 개수 목표치가 있다면, 이런 테스트를 통해서 client 가 대략 얼마나 필요할지 예상할수 있을 것이다.

<h3> <span style="color:{{site.span_h3_color}}"> 
참고 
</span> </h3>

redis-5.0.8/src/redis-benchmark.c
