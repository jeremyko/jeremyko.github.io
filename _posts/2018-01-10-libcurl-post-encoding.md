---
layout: post
title: libcurl POST 한글 encoding
date: '2018-01-10T22:53:00.004+09:00'
tags:
    - c++
    - curl
    - libcurl
modified_time: '2022-03-26T20:57:19.558+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-7625330704236198543
blogger_orig_url: https://jeremyko.blogspot.com/2018/01/libcurl-post-encoding.html
---

curl 프로그램을 사용하여 POST 전송 시, 한글 데이터를 endcoding 해서 전송하는 방법은 다음과 같다.

    curl -d "NAME1=test data" --data-urlencode "NAME2=한글데이터"  http://xxx.yyy.zzz

`-d` 옵션으로 여러 번 지정을 해도 curl이 merge해줘서 전달되는 데이터는 `"NAME1=test data&NAME2=한글데이터"` 이 된다. URL-encoding 이 필요한 값은 `-d` 대신 `--data-urlencode` 로 지정 해 준다.

한편 c++ 에서 libcurl을 사용하여 동일한 작업을 수행하려면 다음처럼 해 주면 된다.

```cpp
curl_easy_setopt(curl, CURLOPT_URL, "http://postit.example.com/moo.cgi");

//encoding할 한글데이터만 분리해서 처리 후
char temp_buffer[1024];
int len=snprintf(temp_buffer, sizeof(temp_buffer),"%s","한글데이터");
char\* encoded = curl_easy_escape(curl, temp_buffer, len);

//하나의 문자열로 조합해야한다.
char post_field [1024];
snprintf(post_field, sizeof(post_field),"%s&NAME2=%s","NAME1=test data", encoded);
curl_easy_setopt(curl, CURLOPT_POSTFIELDS, post_field);

curl_easy_perform(curl);
curl_free(encoded);
```

<span style="color:{{site.span_emphasis_color}}"> 
그런데 curl 프로그램으로 전송할때와 달리 libcurl 사용 시에는 `CURLOPT_POSTFIELDS` 를 한번 만 사용해야 한다. 
</span>
이를 위해서는 encoding될 한글 데이터만을 별도로 분리하여 `curl_easy_escape` 함수를 호출해야 한다. 즉 아래처럼 사용하는 경우에는 에러가 발생된다.

```cpp
//error 발생!
char temp_buffer[1024];
int len=snprintf(temp_buffer, sizeof(temp_buffer),"%s","NAME1=test data&NAME2=한글데이터");
char\* encoded = curl_easy_escape(curl, temp_buffer, len);
curl_easy_setopt(curl, CURLOPT_POSTFIELDS, encoded);
```

이 경우에는 데이터가 모두 encoding 되면서 `'='` 문자도 함께 되버리기 때문에, 받는 쪽에서 에러를 뱉게 된다.
`curl_easy_escape` 함수 문서를 보면 다음과 같이 설명되어 있다.

    All input characters that are not a-z, A-Z, 0-9, '-', '.', '*' or '~' are converted to their "URL escaped" version (%NN where NN is a two-digit hexadecimal number).

처음에 나온 curl 프로그램을 사용하는 경우에는

    curl -d "NAME1=test data" --data-urlencode "NAME2=한글데이터" http://xxx.yyy.zzz

`--data-urlencode "NAME2=한글데이터"` 이 부분에서 curl 이 `"="` 이후 부분 만 알아서 인코딩 해주기 때문에 수신 측에서도 문제가 없다. 하지만 libcurl 은 이런식으로 동작하지 않으므로 주의가 필요하다.
