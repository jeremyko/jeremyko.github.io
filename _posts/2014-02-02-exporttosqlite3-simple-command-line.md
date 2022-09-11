---
layout: post
title:
    'export_to_sqlite3 : simple command line based utility for exporting oracle
    or mysql to sqlite3'
date: '2014-02-02T16:45:00.002+09:00'
tags:
    - c++
modified_time: '2022-03-26T21:01:53.898+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-2201465302320668236
blogger_orig_url: https://jeremyko.blogspot.com/2014/02/exporttosqlite3-simple-command-line.html
---

<h3> <span style="color:{{site.span_h3_color}}"> 
oracle, mysql 을 sqlite3로 export 하기
</span> </h3>

[https://github.com/jeremyko/export_to_sqlite3](https://github.com/jeremyko/export_to_sqlite3)

This is simple command line based utility for exporting oracle/mysql to sqlite3.

<span style="color:{{site.span_emphasis_color}}">
Usage
</span>

**`export_to_sqlite`** `SQLITE_FILE` `DB_TYPE` `DB_USER` `DB_PASSWD` `IP` `PORT` `DB_NAME`

from oracle

    export_to_sqlite mydb.sqlite ORACLE scott tiger 192.168.1.225 1521 orcl

from Mysql

    export_to_sqlite mydb.sqlite MYSQL user passwd 192.168.1.204 3306 PCN

<span style="color:{{site.span_emphasis_color}}">
auto generated scripts
</span>

-   sqlite_schema_script.sql : sqlite table schema generation script.

-   sqlite_fill_script.sql : sqlite table insert script.
