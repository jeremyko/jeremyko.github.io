---
layout: post
title: 맥에서 telnet으로 linux 접속후 vi 한글 입력
date: '2013-06-19T12:41:00.003+09:00'
tags:
modified_time: '2013-06-19T12:41:46.419+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-1152102777568904620
blogger_orig_url: https://jeremyko.blogspot.com/2013/06/telnet-linux-vi.html
---

mac iterm2 에서 linux 로 telnet을 이용한 접근 시 ,
한글 입력이 안되는 경우, 먼저 터미널 encoding 을 Korean 으로 설정하고,
telnet명령 수행 시 -8 option을 주니 해결되었다.

    telnet -8 -E -l user_id 192.168.1.111

다음은 man page 설명.

    -8      Specifies an 8-bit data path.  This causes an attempt to negotiate the TELNET BINARY option on both input and output.
