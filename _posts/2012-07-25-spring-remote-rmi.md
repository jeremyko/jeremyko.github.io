---
layout: post
title: Spring Remote (RMI) 예제
date: '2012-07-25T16:39:00.000+09:00'
tags:
    - spring remote
    - RMI
    - spring
    - java
modified_time: '2012-08-08T11:20:34.069+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-3178278464245613105
blogger_orig_url: https://jeremyko.blogspot.com/2012/07/spring-remote-rmi.html
---

오늘 살펴볼 스프링 Remote 중에서 RMI 지원은 단순한 POJO 작성만으로도 RMI 서비스를 간편하게 발행할 수 있게 해준다.

간단한 예제를 작성해보자.

일단 프로그램은 java application 으로 하였다. RMI를 통한 통신이므로 서버 및 클라이언트 2개 프로젝트를 만들어야 한다.
먼저 서버를 만든다.

### 서버 작성

소스코드는 다음을 참조한다.
[https://github.com/jeremyko/MySpringRMI](https://github.com/jeremyko/MySpringRMI)

**Eclipse 에서 신규 프로젝트를 생성한다**

File -> New -> java project -> "MySpringRMI" 프로젝트 생성.  
그리고 스프링 관련된 라이브러리들을 프로젝트 속성 -> Java Build Path -> Libraries 에 모두 추가한다.

**스프링 설정파일을 다음과 같이 설정한다**

spring_conf.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xmlns:p="http://www.springframework.org/schema/p"
 xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

  <bean class="org.springframework.remoting.rmi.RmiServiceExporter"
    p:service-ref="helloService"
    p:serviceName="HelloService"
    p:serviceInterface="kojh.spring.rmi.HelloService" />

  <bean id="helloService" class="kojh.spring.rmi.HelloServiceImpl"/>
</beans>
```

service-ref 에는 서비스를 담당할 bean 즉, helloService로 설정한다.

serviceName 속성을 통해서 RMI 서비스명을 HelloService 로 설정한다, 나중에 보게 되겠지만 이것을 참조해서 client가 이 서비스를 호출하게 된다.

serviceInterface 는 해당 서비스가 구현하고 있는 인터페이스를 설정한다.

참고로, 기본적으로 RMI가 사용하는 포트는 1099 이다. 이값을 변경하기 위한 속성은 registryPort 이다.

**서비스 인터페이스를 정의한다**

```java
// HelloService.java
package kojh.spring.rmi;

public interface HelloService
{
    String sayHello(String name);
}
```

**이 인터페이스를 구현하는 클래스를 정의한다**

```java
package kojh.spring.rmi;
public class HelloServiceImpl implements HelloService
{
    public String sayHello(String name)
    {
        return "Hello " + name;
    }
}
```

**마지막으로 RMI 서비스를 발행하기 위한 Main 이 필요하다**

```java
// DrivingMain.java
package kojh.spring.test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class DrivingMain
{
    public static void main(String[] args)
    {
        ApplicationContext context = new ClassPathXmlApplicationContext("spring_conf.xml");
    }
}
```

이것으로 RMI 서버가 완성 되었다. RMI 기능을 위한 코딩이라고 느낄만한 부분은 코드상에 어디에도 보이지 않는다는것이 Spring RMI사용의 장점이다. 모든 마법은 스프링 환경 설정 파일에서 이루어지고 있다.

---

### 클라이언트 작성

소스코드는 다음을 참조한다.

[https://github.com/jeremyko/MySpringRMI_Client](https://github.com/jeremyko/MySpringRMI_Client)

**Eclipse 에서 신규 프로젝트를 생성한다**

File -> New -> java project -> "MySpringRMI_Client" 프로젝트 생성. 그리고 서버 작성시와 동일하게 스프링 관련된 라이브러리들을 프로젝트 속성 -> Java Build Path -> Libraries 에 모두 추가한다.

**스프링 설정파일을 다음과 같이 작성한다**

```xml
// spring_conf.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation=
        "http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd">

  <bean id="helloServiceOfClient" class="org.springframework.remoting.rmi.RmiProxyFactoryBean"
    p:serviceUrl="rmi://localhost/HelloService"
    p:serviceInterface="kojh.rmi.interfaces.HelloService" />

  <context:component-scan base-package="kojh.rmi.client"/>
</beans>
```

serviceUrl 속성은 RMI 서비스 접근 URL 정보를 나타낸다. `"rmi://" + 서버 ip + ":" + port 번호 + "/" + 서비스 명 (ex.rmi://localhost:1011/SomeService)` 형식이다.

**서버에서 사용된 서비스의 인터페이와 동일한 인터페이스가 필요하다**

```java
// HelloService.java
package kojh.rmi.interfaces;

public interface HelloService
{
    String sayHello(String name);
}
```

**서비스를 호출하는 작업을 담당할클래스를 정의한다**

```java
// RmiClient.java
package kojh.rmi.client;

import kojh.rmi.interfaces.HelloService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class RmiClient
{
    // static 이어야만 동작.
    //@Autowired: 흠..이상하게도 여기서 Autowired를 사용시에 와이어링이 되지 않는다
    private static HelloService helloServiceOfClient;

    @Autowired // 흠..여기서 Autowired 를 사용해서 Setter를 호출하게 해야 동작한다.
    public void setHelloServiceOfClient(HelloService helloService)
    {
        helloServiceOfClient = helloService;
    }

    public void invokeRmi(String name)
    {
        System.out.println("invoking RMI with name ["+ name +"]");

        String rslt = helloServiceOfClient.sayHello(name);

        System.out.println("----> Result: ["+ rslt +"]");
    }
}
```

**클라이언트의 동작을 확인하기 위한 Main 클래스를 작성한다**

```java
// RmiClientDrivingMain.java

package kojh.rmi.client;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class RMIClientDrivingMain
{
    public static void main(String[] args)
    {
        ApplicationContext context = new ClassPathXmlApplicationContext("spring_conf.xml");
        RmiClient rmiClient = new RmiClient();
        rmiClient.invokeRmi("Tom");
        rmiClient.invokeRmi("Jane");
    }
}
```

### 구동 결과

    invoking RMI with name [Tom]
    ----> Result: [Hello Tom]
    invoking RMI with name [Jane]
    ----> Result: [Hello Jane]
