---
layout: post
title:
    'tomcat maven plugin : 이클립스-maven 플러그인을 이용해서 생성된 웹프로젝트를 Tomcat7.x 에 자동 deploy후
    실행시키기'
date: '2012-07-13T16:12:00.000+09:00'
tags:
    - tomcat maven plugin
    - java
modified_time: '2012-07-13T16:55:39.801+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-556772249782575132
blogger_orig_url: https://jeremyko.blogspot.com/2012/07/tomcat-maven-plugin-maven-tomcat7x.html
---

간만에 자바 공부 좀 하려니 역시.. 아주 오래전 느꼈던 감정을 다시 느끼게 된다.

자바 == 짜증나게 많은 라이브러리, 툴, 트릭 및 팁의 연속... - - ;;

이클립스-maven 플러그인을 이용해서 생성된 웹프로젝트를 Tomcat7.x 서버에서 실행해보기 위해서는 , 직접 사용자가 톰캣 서버의 manager로 들어가서
war파일을 deploy하는 방법도 있지만, 번거롭다.

이클립스 상에서, 간단하게 마우스 조작만으로 톰캣서버에 자동으로 deploy 시킬수 있게 해주는 것이 tomcat maven plugin 이다.

### 리소스 다운로드

-   Maven Download

-   이클립스-maven 플러그인 다운로드: help -> eclipse Marketplaces -> "Maven Integration for Eclipse"

-   톰캣 WAS 설치

### 이클립스에서 톰캣서버로 직접 deploy하기 위한, 권한이 필요하므로, 톰캣서버 사용자를 등록한다

(이미 기존 사용자가 존재하면 불필요)

```xml
<tomcat-users>
  <role rolename="manager-script"/>
  <role rolename="manager-gui"/>
  <role rolename="admin"/>
  <role rolename="manager"/>
  <user username="admin" password="1111" roles="admin,manager,manager-gui,manager-script"/>
</tomcat-users>
```

### 이클립스-maven 플러그인을 사용해서 Maven 프로젝트 생성.

    New -> Maven Project

### 해당 maven 프로젝트, pom.xml 파일에 tomcat maven plugin 을 정의한다.

```xml
<build>
  <plugins>
    <plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>tomcat-maven-plugin</artifactId>
    <version>1.1</version>
    <configuration>
    <charset>UTF-8</charset>
    <mode>war</mode>
    <url>http://localhost:8080/manager/text</url>
    <path>/spring-security-test</path>
    <username>admin</username>
    <password>1111</password>
    </configuration>
    </plugin>
  </plugins>
</build>
```

### 프로젝트 Run 설정

Goal 에 tomcat:redeploy 를 설정한다.

### 실행

-   톰캣서버는 이미 기동된 상태여야 한다.

-   톰캣서버에 deploy시킨다: 프로젝트 -> 오른쪽 버튼 -> Run As -> Maven Build

-   그다음, 웹 브라우져를 열어서
    http://localhost:8080/spring-security-test/ 입력.
