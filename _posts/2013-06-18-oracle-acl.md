---
layout: post
title: 'oracle ACL '
date: '2013-06-18T10:13:00.001+09:00'
tags:
    - oracle
    - ACL
modified_time: '2013-06-19T12:35:18.427+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-614107802926524025
blogger_orig_url: https://jeremyko.blogspot.com/2013/06/oracle-acl.html
---

oracle stored procedure 등에서 외부 ip address접근 시도시 ACL오류 발생 하는 경우 다음을 확인해본다. sysdba 권한이 필요하다.

-   먼저 현재 상태 확인

    select `*` from DBA_NETWORK_ACLS ;

-   해당 ip address 없으면 새로운 ACL 생성

    exec dbms_network_acl_admin.create_acl('test_tcp.xml','Network connection permission to HTTP server for TEST', 'TEST', TRUE, 'connect');

    exec dbms_network_acl_admin.add_privilege('test_tcp.xml','DB계정',true,'resolve');

    exec dbms_network_acl_admin.add_privilege('test_tcp.xml','DB계정',true,'connect');

    exec dbms_network_acl_admin.assign_acl('test_tcp.xml','192.168.1.111');
