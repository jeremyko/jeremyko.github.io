---
layout: post
title: WSL 1, 2  has no numa support
date: '2020-06-11T11:53:00.007+09:00'
tags:
    - numa
    - wsl
modified_time: '2021-04-03T15:56:03.275+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-3483691293311629420
blogger_orig_url: https://jeremyko.blogspot.com/2020/06/wsl-1-virtualbox-and-numa.html
---

<h3> <span style="color:{{site.span_h3_color}}"> 
wsl 와 linux 는 완전히 동일 한건 아니다
</span> </h3>

When I run a linux program that uses c++ numa lib , I get the following error and core dump.

    request to allocate mask for invalid number: Invalid argument

But, It works normally on linux installed in virtualbox.  
프로그램이 wsl 에서 동작하지 않는 경우를 처음 겪었음.

<h4> <span style="color:{{site.span_h4_color}}"> 
virtualbox
</span> </h4>

    $ numactl -s
    policy: default
    preferred node: current
    physcpubind: 0 1 2 3
    cpubind: 0
    nodebind: 0
    membind: 0

<h4> <span style="color:{{site.span_h4_color}}"> 
wsl ubuntu
</span> </h4>

    $ numactl -s
    physcpubind: 0 1 2 3 4 5 6 7
    No NUMA support available on this system.

<h3> <span style="color:{{site.span_h3_color}}"> 
2020-07-22 update 
</span> </h3>
wsl 2 에서도 마찬가지. custom kernel build 가 필요하다고 :-(
