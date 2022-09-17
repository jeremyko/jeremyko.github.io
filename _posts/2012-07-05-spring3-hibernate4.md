---
layout: post
title: spring3 + Hibernate4 예제
date: '2012-07-05T16:07:00.001+09:00'
tags:
    - hibernate4
    - spring
    - java
modified_time: '2012-07-25T10:17:35.195+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-4689006753718484969
blogger_orig_url: https://jeremyko.blogspot.com/2012/07/spring3-hibernate4.html
---

## hibernate4 예제

최근의 자바 개발 환경에 약간이라도 익숙해져 볼 맘으로 작성해 보았다. 그런데 보다시피 이 간단한 예제(테이블에 insert) 하나를 수행하기 위해서도 설정할것, 알아야 할것이 많다. 10년전쯤 자바 개발 하다가 이런 설정과 환경에 질려버렸던것 같은데 지금도 별로 좋아진것 없는것 같다. 수많은 설정...설정....

간단한 고객 데이터를 테이블에 저장하기 위한 예제를 작성해본다.사용된 개발환경은 다음과 같다

-   hibernate-release-4.1.3.Final
-   데이터베이스: oracle 11g XE

### 데이터베이스 설정

고객 테이블을 만든다.

```sql
CREATE TABLE  "CUSTOMER" (
    "SEQ" NUMBER NOT NULL ENABLE,
    "ID" VARCHAR2(8) NOT NULL ENABLE,
    "PWD" VARCHAR2(8),
    "NAME" VARCHAR2(20),
    CONSTRAINT "CUSTOMER_PK" PRIMARY KEY ("ID") ENABLE
) ;
```

### 고객 정보를 억세스 하기 위한 클래스를 작성한다.

```java
// Customer.java

package app;
public class Customer
{
    private int seq;
    private String id;
    private String password;
    private String name;

    public Customer()
    {
    }
    public int getSeq()
    {
        return seq;
    }
    // 자동으로 생성되는 번호이므로 set 금지
    private void setSeq(int seq)
    {
        this.seq = seq;
    }
    public String getId()
    {
        return id;
    }
    public void setId(String id)
    {
        this.id = id;
    }
    public String getPassword()
    {
        return password;
    }
    public void setPassword(String password)
    {
        this.password = password;
    }
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        this.name = name;
    }
}
```

### Customer.hbm.xml 맵핑 작성한다(Customer 클래스와 맵핑을 정의)

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
                                    "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="app.Customer" table="CUSTOMER">

    <!-- 기본키가 되는 필드 -->
    <id column="ID" name="id"> </id>

    <property column="SEQ" name="seq" type="int"/>
    <property column="PWD" generated="never" lazy="false" name="password" type="string"/>
    <property column="NAME" generated="never" lazy="false" name="name" type="string"/>
    </class>
</hibernate-mapping>
```

### 하이버네이트 세션을 가져오기 위해, 유틸클래스를 정의해서 사용한다.

```java
// HibernateUtil.java

package util;
import org.hibernate.HibernateException;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;
import org.hibernate.service.ServiceRegistry;
import org.hibernate.service.ServiceRegistryBuilder;

public class HibernateUtil
{
    public static final SessionFactory sessionFactory;
    private static ServiceRegistry serviceRegistry;

    static
    {
        try
        {
            // Create the SessionFactory from hibernate.cfg.xml
            Configuration configuration = new Configuration();
            configuration.configure();

            serviceRegistry =
            new ServiceRegistryBuilder().applySettings(configuration.getProperties()).buildServiceRegistry();
            sessionFactory = configuration.buildSessionFactory(serviceRegistry);

            // sessionFactory = new
            // Configuration().configure().buildSessionFactory(); //deprecated
        }
        catch (Throwable ex)
        {
            // Make sure you log the exception, as it might be swallowed
            System.err.println("Initial SessionFactory creation failed." + ex);
            throw new ExceptionInInitializerError(ex);
        }
    }

    public static final ThreadLocal<Session> session = new ThreadLocal<Session>();

    public static Session getCurrentSession() throws HibernateException
    {
        Session s = session.get();
        // Open a new Session, if this thread has none yet
        if (s == null)
        {
            s = sessionFactory.openSession();
            // Store it in the ThreadLocal variable
            session.set(s);
        }
        return s;
    }

    public static void closeSession() throws HibernateException
    {
        Session s = (Session) session.get();
        if (s != null)
            s.close();
        session.set(null);
    }
}
```

### 데이터 저장,수정, 삭제를 위한 helper 클래스를 하나 만들자.

```java
// CustomerManager.java

package app;
import org.hibernate.Session;
import util.HibernateUtil;

public class CustomerManager
{
    public Customer insertCustomer(String id, String pass, String name)
    {
        Session session = HibernateUtil.getCurrentSession();
        Customer customer = new Customer();
        customer.setId(id);
        customer.setPassword(pass);
        customer.setName(name);
        //session.save(customer);
        session.persist(customer);
        return customer;
    }

    public Customer updateCustomer(String Id, String newPass, String newName)
    {
        Customer customer = getCustomer(Id);
        customer.setId(Id);
        customer.setPassword(newPass);
        customer.setName(newName);
        return customer;
    }

    public void deleteArticle(String id)
    {
        Session session = HibernateUtil.getCurrentSession();
        Customer customer = getCustomer(id);
        session.delete(customer);
    }

    public Customer getCustomer(String id)
    {
        Session session = HibernateUtil.getCurrentSession();
        return (Customer) session.get(Customer.class, id);
    }
}
```

### 제일 중요한~ 하이버네이트 설정 파일을 작성한다.

```java
// hibernate.cfg.xml

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <property name="hibernate.connection.driver_class">oracle.jdbc.driver.OracleDriver</property>
        <property name="hibernate.connection.password">study</property>
        <property name="hibernate.connection.url">jdbc:oracle:thin:@localhost:1521/XE</property>
        <property name="hibernate.connection.username">study</property>
        <property name="hibernate.dialect">org.hibernate.dialect.Oracle10gDialect</property>

        <!-- Echo all executed SQL to stdout -->
        <property name="show_sql">true</property>

        <!-- Mapping -->
        <mapping resource="app/Customer.hbm.xml"/>

    </session-factory>
</hibernate-configuration>
```

### 이것을 테스트 해보기 위한 main class

보다시피, 직접 세션을 얻고, 트랜잭션을 시작한다. 나중에 스프링과의 결합에서는 이런 작업들은 프레임워크 뒷단으로 사라지게 된다.

```java
// HibernateTest.java

package app;
import org.hibernate.HibernateException;
import org.hibernate.Session;
import org.hibernate.Transaction;

import org.apache.log4j.Logger;
import util.HibernateUtil;

public class HibernateTest
{
    public static Logger log = Logger.getLogger(HibernateTest.class);

    public static void logThis(Customer customer)
    {
        String id = customer.getId();
        String pass = customer.getPassword();
        String name = customer.getName();

        log.info("id   [" +id+"]");
        log.info("pass [" +pass+"]");
        log.info("name [" +name+"]");
    }

    public static void main(String[] args)
    {
        Session sess = null;
        Transaction tx = null;

        try
        {
            // 세션 열기
            sess = HibernateUtil.getCurrentSession();

            // 트랜잭션 시작
            tx = sess.beginTransaction();

            CustomerManager custManager = new CustomerManager();

            //Insert
            Customer customer1 = custManager.insertCustomer("id1", "pass1", "name1");
            Customer customer2 = custManager.insertCustomer("id2", "pass2", "name2");
            logThis(customer1);
            logThis(customer2);

            //update
            Customer customer3 = custManager.updateCustomer("id2", "pass3", "name3");
            logThis(customer3);

            //delete
            custManager.deleteArticle("id1");
            custManager.deleteArticle("id2");

            tx.commit(); // 커밋
        }
        catch (HibernateException e)
        {
            tx.rollback(); // 롤백
            e.printStackTrace();
        }
        finally
        {
            // 세션 닫기
            HibernateUtil.closeSession();
        }
    }
}
```

## spring_hibernate4 예제

<span style="color:{{site.span_emphasis_color}}">
**이번에는 동일한 예제를 스프링3+하이버네이트 로 작성해본다.**
</span>
{: .notice--info}

사용된 개발환경은 다음과 같다

-   hibernate-release-4.1.3.Final
-   데이터베이스: oracle 11g XE
-   spring-framework-3.1.1.RELEASE

그밖에 다음 jar 파일이 필요하다 (트랜잭션을 위해 AOP 설정 필요).

    com.springsource.org.aopalliance-1.0.0.jar

### 데이터베이스 설정 : 고객 테이블은 앞서의 예제와 동일

### 고객 정보를 억세스 하기 위한 클래스(POJO) 작성.

```java
//Customer.java
package app;

import java.math.BigDecimal;

public class Customer
{
    private String id;
    private BigDecimal seq;
    private String pwd;
    private String name;

    public Customer(String id, BigDecimal seq, String pwd, String name) {
        this.id = id;
        this.seq = seq;
        this.pwd = pwd;
        this.name = name;
    }
    public Customer() {
    }
    public String getId() {
        return this.id;
    }
    public void setId(String id) {
        this.id = id;
    }
    public BigDecimal getSeq() {
        return this.seq;
    }
    public void setSeq(BigDecimal seq) {
        this.seq = seq;
    }
    public String getPwd() {
        return this.pwd;
    }
    public void setPwd(String pwd) {
        this.pwd = pwd;
    }
    public String getName() {
        return this.name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

### 맵핑 파일을 작성한다. 이것은 테이블당 1개씩 필요하다. (앞서와 동일함)

Customer.hbm.xml

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd" >

<hibernate-mapping>
<class    name="app.Customer"    table="CUSTOMER"    lazy="false">
    <id name="id"  type="java.lang.String"   column="ID"  >
        <generator class="assigned" />
    </id>
    <property  name="seq"  type="java.math.BigDecimal" column="SEQ"  not-null="true" />
    <property  name="pwd"  type="java.lang.String"     column="PWD"  length="8"      />
    <property  name="name" type="java.lang.String"     column="NAME" length="20"     />
</class>
</hibernate-mapping>
```

### DAO 클래스를 작성한다.

앞서의 하이버네이트 예제에서는 HibernateUtil 클래스를 선언하고, 이를 이용해서, 직접 세션을 구해서 사용했다.

아울러 트랜잭션 처리도 직접 수행했다.

스프링과 결합되는 경우에는, 설정에 의해서 세션 객체가, 스프링으로부터 주어지고, 기존 트랜잭션처리는 스프링 트랜잭션(하이버네이트와 결합된)을 이용한다.

CustomerDAO.java

```java
package app;

import org.hibernate.SessionFactory;
import org.hibernate.Session;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.annotation.Propagation;
import java.util.*;

@Repository
@Transactional(propagation=Propagation.REQUIRED)
public class CustomerDAO
{
    //다음 프로퍼티는 생성자 호출시에 스프링이 전달해준다.
    //이를 위해서 생성자에 @Autowired 사용이 필요하다.
    private SessionFactory sessionFactory;

    public CustomerDAO()
    {
        //반드시 필요!!!
    }

    @Autowired
    public CustomerDAO(SessionFactory sessionFactory)
    {
        this.sessionFactory = sessionFactory;
    }

    public Session currentSession()
    {
        return  sessionFactory.getCurrentSession();
    }

    public void addCustomer(Customer customer)
    {
        currentSession().save(customer);
    }

    public void deleteCustomer(String id)
    {
        currentSession().delete(getCustomerById(id));
    }

    public Customer getCustomerById(String id)
    {
        return (Customer) currentSession().get(Customer.class, id);
    }

    public void saveCustomer(Customer customer)
    {
        currentSession().update(customer);
    }
}
```

### 제일 중요한 스프링 설정 파일을 작성한다.

우리는 스프링이 제공하는 DI 를 이용해서, 하이버네이트의 세션 팩토리를 설정하게 할것이다.

또한 트랜잭션 처리를 위해, 하이버네이트 트랜잭션 관리 빈을 설정하고 그것의 세션 팩토리에, 우리가 정의한 하이버네이트 세션 팩토리를 전달할것이다.

DAO객체 생성자에 하이버네이트 세션 팩토리를 전달하기 위해, @Autowired 어노테이션을 사용한다. 와이어링 설정에 따라서, 어플리케이션 실행시, 스프링이 생성자를 호출하면서 하이버네이트 세션팩토리 객체를 넘겨줄것이다 (즉, 우리가 세션을 직접 생성할 필요 없다).

트랜잭션을 위해서, AOP 설정이 필요하며, 이를 위해서 `com.springsource.org.aopalliance-1.0.0.jar` 이 꼭 필요하다.

HibernateSpringTest1.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:util="http://www.springframework.org/schema/util"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation=
    "http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.0.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/util
        http://www.springframework.org/schema/util/spring-util.xsd" >

    <!-- autowiring 사용위함 -->
    <context:annotation-config />

    <!-- 데이터 소스, oracle 11g XE 를 사용 -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource" >
    <property name="driverClassName"><value>oracle.jdbc.driver.OracleDriver</value></property>
    <property name="url"><value>jdbc:oracle:thin:@localhost:1521:XE</value></property>
    <property name="username"><value>study</value></property>
    <property name="password"><value>study</value></property>
    </bean>

    <!-- Hibernate Session Factory 설정 -->
    <bean id="sessionFactory" class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
    <property name="dataSource"><ref bean="dataSource"/></property>
    <property name="mappingResources">
        <list>
        <value>app/Customer.hbm.xml</value>
        <value>app/CustomerOrder.hbm.xml</value>
        </list>
    </property>

    <property name="hibernateProperties">
        <props>
        <prop key="hibernate.dialect">org.hibernate.dialect.Oracle10gDialect</prop>
        <prop key="hibernate.show_sql">true</prop>
        </props>
    </property>
    </bean>

    <!-- 트랜잭션 -->
    <tx:annotation-driven transaction-manager="transactionManager" />
    <bean id="transactionManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>

    <!-- CustomerDAO autowiring되었으므로 property지정은 불필요~ -->
    <bean id="cutomerDao" class="app.CustomerDAO"> </bean>
</beans>
```

### 실제 사용을 위한 테스트 main이다.

DrivingMain.java

```java
package app;
import java.math.BigDecimal;
import org.hibernate.HibernateException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class DrivingMain
{
    public static void main(String[] args)
    {
        ApplicationContext context = new ClassPathXmlApplicationContext("app/HibernateSpringTest1.xml");

        CustomerDAO customerDAO = (CustomerDAO) context.getBean("cutomerDao");

        try
        {
            //customer
            BigDecimal seq1 = new BigDecimal(1);
            Customer customer = new Customer ("id_1", seq1, "passwd1", "name_jeremyko");

            System.out.println("deleteCustomer");
            customerDAO.deleteCustomer("id_1");

            System.out.println("addCustomer");
            customerDAO.addCustomer( customer); //고객 테이블에 insert

        }
        catch (HibernateException e)
        {
            e.printStackTrace();
        }
        finally
        {
        }
    }
}
```
