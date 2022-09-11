---
layout: post
title: regex basic
date: '2012-09-20T14:14:00.002+09:00'
tags:
    - regular expression
modified_time: '2013-10-31T16:13:38.983+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-7106158565821255188
blogger_orig_url: https://jeremyko.blogspot.com/2012/09/regex-basic.html
---

## regex basic

    [] => [] 안에 존재하는 문자들중 한 문자만을 나타낸다.
        예) [abc] 는 a , b, c 중에 한문자. '-' 는 범위. 즉 [a-f] 는 [abcdf]

    \w => [a-zA-Z0-9_]
    \d => [0-9]
    .  => 모든 문자에 대응되는 한 문자
    +  => 한 문자 이상(하나는 반드시 존재, repeats the preceding regular expression at least once)
    *  => 문자나 집합이 없는 경우 또는 하나 이상 연속하는 경우에 일치한다. 없어도 ok
    ?  => 문자가 없거나 하나와 대응하는 문자   예) https?// - s가 없거나 한번 있는 경우와 대응
        ca?r matches both `car' and `cr', but nothing else.
    \ => 문자그대로 . 를 나타냄 (이스케이프)

    ab{3}" => b가 3개까지 (abbb)
    ab{3,5}" =>  b가 3개에서 5개 까지 (abbb, abbbb, abbbbb )
    ab{2,}" => b의 개수가 2개 이상 (abb, abbbbb ...)
    {3,} => 최소 3번 일치함을 의미한다.

    () => ()안에 있는 글자들을 그룹화, 하위 표현식.
        a(bc)* : a라는 문자를 포함하고 bc라는 문자열이 없거나 계속반복( a, abc abcbc 등)
        a(bc)  : a라는 문자를 포함하고 bc라는 문자열이 있거나 없거나. (a,abc)

        (\d{1,3}\.){3}\d{1,3} => ip addr

    공백 문자 => \n, \r, \t, \f-form feed, \v-vertical tab
    \s => 공백 ,[\f\n\r\t\v],
    \S => 공백아님 , [^\f\n\r\t\v]

    |  => or 연산자, 묶음 안에서 하나를 일치시키고자 할 때
        IP주소를 구성하는 각 숫자 묶음을 유효한 조합으로 정의하는 규칙
        - 모든 한 자리 혹은 두 자리 숫자
        - 1로 시작하는 모든 세자리 숫자
        - 2로 시작하면서 두 번째 자리 숫자가 0부터 4사이의 모든 세 자리 숫자
        - 25로 시작하면서 세 번째 자리 숫자가 0부터 5사이의 모든 세 자리 숫자

        (((\d{1,2})|(1\d{2})|(2[0-4]\d)|(25[0-5]))\.){3}((\d{1,2})|(1\d{2})|(2[0-4]\d)|(25[0-5]))

        주민번호 : \d{2}(0[1-9]|1[0-2])(0[1-9]|[12][0-9]|3[01])-[1-4]\d{6}


    탐욕적 수량자,  게으른 수량자
        *                            *?
        +                            +?
        {n,}                       {n,}?

        예) "<B>111</B> and <B>222</B>" 에서

        <[Bb]>.*</[Bb]>  => 111 and 222
        <[Bb]>.*?</[Bb]> => 111, 222

    egrep 에서 111이후 대문자3자리로 끝나는  프로세스 검색
    ==> -w 옵션 사용

    ps -ef | egrep -w  '111[A-Z]{3}'

---

```java
// regexTest.java

import java.util.regex.*;

class regexTest
{
    public static void runRegExTest()
    {
        String[] data1 = {   "bat", "baby", "bonus", "c", "cA",
                            "ca", "co", "c.", "c0", "c#","1","9",
                            "car","combat","count", "date", "disc"
        };
        String[] pattern1 = { "(\\d+)","\\d+",".",".*","c[a-z]*","c[a-z]",
                            "c[a-zA-Z]", "c[a-zA-Z0-9]",
                            "c.","c.*","c\\.","c\\w","c\\d","c.*t", "[b|c].*",
                            ".*a.*", ".*a.+", "[b|c].{2}"
        };

        String[] data2 = { "1","9","01","012","22.133.244.111",
                            "a","b","c","abc","abcbc","0","00",
                            " ", "  ","1 a","1 ","2  ",
                            "2  a", "1aaaaaaaa1","1aaaaaaaa21","22cccc22dd33",
                            "cd","cde","NTRY50BA","22","55","123","233","444",
                            "ab12a0","abcd2a044", "abab12a0b1",
                             "6511","621217-1812222","681109-1652222",
                            "쭌안아빠","20/09/2012","20-09-2012","가","하",
                            "01","0412","12","14","11","17"
        };

        String[] pattern2 = { "(\\d+)","\\d+","\\d","\\d*","(\\d{1,3}\\.){3}\\d{1,3}",
                              "a(bc)","(0)","0", "\\s", "\\s\\s","\\S",
                              "\\s.", "\\s*", "\\d\\s+", "\\d\\s+\\w","\\d[a-zA-Z]*\\d+",
                              "\\d[a-zA-Z]*\\d+?",
                              "((?:[a-z][a-z]+))","(?:[a-z][a-z]+)","\\d{1,2}",
                              "((?:[a-z][a-z]*[0-9]+[a-z0-9]*))",
                              "[a-z][a-z]*[0-9]+[a-z0-9]*",
                              "\\d{2}(0[1-9]|1[0-2])",
                              "\\d{2}(0[1-9]|1[0-2])-[1-4]\\d{6}",
                              "(0[1-9]|[12][0-9]|3[01])",
                              "\\d{2}(0[1-9]|1[0-2])(0[1-9]|[12][0-9]|3[01])-[1-4]\\d{6}",
                              "[가-힣]","[가-힣]*","\\d{2}","0[1-9]|1[0-2]",
                              "\\d{1,2}[-\\/]\\d{1,2}[-\\/]\\d{2,4}"
        };

        for(int x=0; x < pattern2.length; x++)
        {
            Pattern p = Pattern.compile(pattern2[x]);
            System.out.print( String.format("%-40s", " " + pattern2[x] ) );
            System.out.print( " =>  " );

            for(int i=0; i < data2.length; i++)
            {
                Matcher m = p.matcher(data2[i]);

                if(m.matches())
                {
                    System.out.print(data2[i] +" ,");
                }
            }

            System.out.println("");
        }

        System.out.println("\n\n");

        for(int x=0; x < pattern1.length; x++)
        {
            Pattern p = Pattern.compile(pattern1[x]);
            System.out.print( String.format("%-40s", " " + pattern1[x] ) );
            System.out.print( " =>  " );

            for(int i=0; i < data1.length; i++)
            {
                Matcher m = p.matcher(data1[i]);

                if(m.matches())
                {
                    System.out.print(data1[i] +" ,");
                }
            }
            System.out.println("");
        }
    }

    public static void main(String[] args)
    {
        runRegExTest();
    }
}
```

출력

    (\d+)                                   =>  1 ,9 ,01 ,012 ,0 ,00 ,22 ,55 ,123 ,233 ,444 ,6511 ,01 ,0412 ,12 ,14 ,11 ,17 ,
    \d+                                     =>  1 ,9 ,01 ,012 ,0 ,00 ,22 ,55 ,123 ,233 ,444 ,6511 ,01 ,0412 ,12 ,14 ,11 ,17 ,
    \d                                      =>  1 ,9 ,0 ,
    \d*                                     =>  1 ,9 ,01 ,012 ,0 ,00 ,22 ,55 ,123 ,233 ,444 ,6511 ,01 ,0412 ,12 ,14 ,11 ,17 ,
    (\d{1,3}\.){3}\d{1,3}                   =>  22.133.244.111 ,
    a(bc)                                   =>  abc ,
    (0)                                     =>  0 ,
    0                                       =>  0 ,
    \s                                      =>    ,
    \s\s                                    =>     ,
    \S                                      =>  1 ,9 ,a ,b ,c ,0 ,가 ,하 ,
    \s.                                     =>     ,
    \s*                                     =>    ,   ,
    \d\s+                                   =>  1  ,2   ,
    \d\s+\w                                 =>  1 a ,2  a ,
    \d[a-zA-Z]*\d+                          =>  01 ,012 ,00 ,1aaaaaaaa1 ,1aaaaaaaa21 ,22 ,55 ,123 ,233 ,444 ,6511 ,01 ,0412 ,12 ,14 ,
    11 ,17 ,
    \d[a-zA-Z]*\d+?                         =>  01 ,012 ,00 ,1aaaaaaaa1 ,1aaaaaaaa21 ,22 ,55 ,123 ,233 ,444 ,6511 ,01 ,0412 ,12 ,14 ,
    11 ,17 ,
    ((?:[a-z][a-z]+))                       =>  abc ,abcbc ,cd ,cde ,
    (?:[a-z][a-z]+)                         =>  abc ,abcbc ,cd ,cde ,
    \d{1,2}                                 =>  1 ,9 ,01 ,0 ,00 ,22 ,55 ,01 ,12 ,14 ,11 ,17 ,
    ((?:[a-z][a-z]*[0-9]+[a-z0-9]*))        =>  ab12a0 ,abcd2a044 ,abab12a0b1 ,
    [a-z][a-z]*[0-9]+[a-z0-9]*              =>  ab12a0 ,abcd2a044 ,abab12a0b1 ,
    \d{2}(0[1-9]|1[0-2])                    =>  6511 ,0412 ,
    \d{2}(0[1-9]|1[0-2])-[1-4]\d{6}         =>
    (0[1-9]|[12][0-9]|3[01])                =>  01 ,22 ,01 ,12 ,14 ,11 ,17 ,
    \d{2}(0[1-9]|1[0-2])(0[1-9]|[12][0-9]|3[01])-[1-4]\d{6} =>  621217-1812222 ,681109-1652222 ,
    [가-힣]                                 =>  가 ,하 ,
    [가-힣]*                                =>  쭌안아빠 ,가 ,하 ,
    \d{2}                                   =>  01 ,00 ,22 ,55 ,01 ,12 ,14 ,11 ,17 ,
    0[1-9]|1[0-2]                           =>  01 ,01 ,12 ,11 ,
    \d{1,2}[-\/]\d{1,2}[-\/]\d{2,4}         =>  20/09/2012 ,20-09-2012 ,


    (\d+)                                   =>  1 ,9 ,
    \d+                                     =>  1 ,9 ,
    .                                       =>  c ,1 ,9 ,
    .*                                      =>  bat ,baby ,bonus ,c ,cA ,ca ,co ,c. ,c0 ,c# ,1 ,9 ,car ,combat ,count ,date ,disc ,  disc ,
    c[a-z]*                                 =>  c ,ca ,co ,car ,combat ,count ,
    c[a-z]                                  =>  ca ,co ,
    c[a-zA-Z]                               =>  cA ,ca ,co ,
    c[a-zA-Z0-9]                            =>  cA ,ca ,co ,c0 ,
    c.                                      =>  cA ,ca ,co ,c. ,c0 ,c# ,
    c.*                                     =>  c ,cA ,ca ,co ,c. ,c0 ,c# ,car ,combat ,count ,
    c\.                                     =>  c. ,
    c\w                                     =>  cA ,ca ,co ,c0 ,
    c\d                                     =>  c0 ,
    c.*t                                    =>  combat ,count ,
    [b|c].*                                 =>  bat ,baby ,bonus ,c ,cA ,ca ,co ,c. ,c0 ,c# ,car ,combat ,count ,
    .*a.*                                   =>  bat ,baby ,ca ,car ,combat ,date ,
    .*a.+                                   =>  bat ,baby ,car ,combat ,date ,
    [b|c].{2}                               =>  bat ,car ,
