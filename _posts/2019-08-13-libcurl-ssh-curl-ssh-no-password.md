---
layout: post
title: libcurl + ssh 비번없이 사용하는 경우 (curl + ssh no password)
date: '2019-08-13T15:43:00.001+09:00'
tags:
    - c++
modified_time: '2021-04-03T15:59:16.945+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-8816530190688508182
blogger_orig_url: https://jeremyko.blogspot.com/2019/08/libcurl-ssh-curl-ssh-no-password.html
---

<h4> <span style="color:{{site.span_h4_color}}"> 
참고 :이 글은 curl 버전 7.30 기준으로 작성 되었음. 그 이후 버전에는 해당 없을 수 있음.
</span> </h4>

ssh 접속시 비밀번호를 안물어보게 하기 위해 `authorized_keys` 에 공개키를 추가하는 작업을 수행하게 된다. 작업 후에 그냥 ssh, sftp 로 명령을 수행하면 비밀번호 입력 없이 바로 접속이 되는데 curl 혹은 libcurl(c++) 을 이용하여 접속할때 인증이 실패하는 경우, 다음처럼 시도해본다.

    $ curl --list-only "sftp://172.25.180.88//some/remote/path/" -u "userid:"
    .....
    curl: (67) Authentication failure

인증이 실패한다. verbose 옵션을 주고 재시도 해본다.

    $ curl -vvv --list-only "sftp://172.25.180.88//some/remote/path/" -u "userid:"

        * About to connect() to 172.25.180.88 port 22 (#0)
        *   Trying 172.25.180.88...
        ...중략...
        * SSH authentication methods available: publickey,gssapi-keyex,gssapi-with-mic,password
        * Using ssh public key file /UPMS/.ssh/id_dsa.pub
        * Using ssh private key file /UPMS/.ssh/id_dsa
        * SSH public key authentication failed: Unable to open public key file
        * Failure connecting to agent
        * Authentication failure
        * Closing connection 0
        curl: (67) Authentication failure

공개키, 비밀키의 이름이 다르다. 알고보니 현재 curl 이 기본값으로 dsa 방식을 가정하고 있었다 (최신 curl 버전은 rsa 를 먼저 찾게 되어 있음. 이글의 curl 버전은 7.3 입니다).

그런데 현재 local 장비의 `.ssh` 폴더에는 rsa 방식의 키가 존재하기 때문에 인증 오류가 발생한다.그래서 알아낸 방식은 키 이름을 명시적으로 지정하는 것이다.

    $ curl -vvv --list-only --key /UPMS/.ssh/id_rsa --pubkey /UPMS/.ssh/id_rsa.pub "sftp://172.25.180.88//some/remote/path/" -u "userid:"

        * About to connect() to 172.25.180.88 port 22 (#0)
        *   Trying 172.25.180.88...
        ...중략...
        * SSH host check: 0, key: AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBGKMQquxJoaMZUPPACwzHChPWYJcqvoNz2zNOHfFooJ121VfHOBXMBklf5CT9coSmkNAypj66aQnrhc5U/ueXik=
        * SSH authentication methods available: publickey,gssapi-keyex,gssapi-with-mic,password
        * Using ssh public key file /UPMS/.ssh/id_rsa.pub
        * Using ssh private key file /UPMS/.ssh/id_rsa
        * Initialized SSH public key authentication
        * Authentication complete
        .
        ..
        ipdb.txt.1
        ipdb-v1
        ipdb.data.txt
        ....
        * Connection #0 to host 172.25.180.88 left intact

성공적으로 수행되었다.

만약 libcurl 을 이용해서 c++ 에서 개발을 하는 경우에도 마찬가지로 키를 지정하면 된다.
이렇때는 다음처럼 추가로 호출해주면 된다.

```cpp
#define PUBLIC_KEYFILE "/UPMS/.ssh/id_rsa.pub"
#define PRIVATE_KEYFILE "/UPMS/.ssh/id_rsa"

.... curl 초기화 작업 수행 후 ....
//다음을 추가 호출.
curl_easy_setopt(curl, CURLOPT_SSH_PUBLIC_KEYFILE, PUBLIC_KEYFILE );
curl_easy_setopt(curl, CURLOPT_SSH_PRIVATE_KEYFILE, PRIVATE_KEYFILE);
```

공개키, 개인키를 찾는 방식이 사용하는 curl버전에 따라 틀리다. 지금 사용중인 curl 버전은 7.30 이고, 최신 curl 소스(7.65) 를 확인해보니, rsa 를 기본으로 찾고 있다 ( - -; ).
확인을 위해서는 curl 소스 `lib/libssh2.c` 의 `libssh2_userauth_publickey_fromfile_ex` 호출 부분 근처를 확인 해보면 알수 있다.

    $ curl --version
    curl 7.30.0 (x86_64-unknown-linux-gnu) libcurl/7.30.0 OpenSSL/1.0.2k zlib/1.2.7 c-ares/1.15.0 libssh2/1.9.0
    Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp scp sftp smtp smtps telnet tftp
    Features: AsynchDNS IPv6 Largefile NTLM NTLM_WB SSL libz

<h3> <span style="color:{{site.span_h3_color}}"> 
참고 
</span> </h3>
[https://curl.haxx.se/libcurl/c/CURLOPT_SSH_PRIVATE_KEYFILE.html](https://curl.haxx.se/libcurl/c/CURLOPT_SSH_PRIVATE_KEYFILE.html)
