---
layout: post
title: MyBatis-Spring
date: '2012-07-17T16:07:00.001+09:00'
tags:
    - MyBatis
    - spring
    - MyBatis-Spring
    - java
modified_time: '2012-08-08T11:21:13.785+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-8066922142380525463
blogger_orig_url: https://jeremyko.blogspot.com/2012/07/mybatis-spring.html
---

[MyBatis-Spring](http://www.mybatis.org/spring/index.html)

## Getting Started

### 설치

MyBatis-Spring 을 이용하기 위해서는 mybatis-spring-x.x.x.jar 과 그 dependency들을 클래스 패스에 추가해야 한다. Maven을 사용중이라면 pom.xml 에 다음 dependency 를 추가한다.

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis-spring</artifactId>
  <version>x.x.x</version>
</dependency>
```

### Quick Setup

스프링 어플리케에션 컨텍스트에는 기본적으로 다음 2가지가 필요하다.

-   **SqlSessionFactory**
-   **최소 하나이상의 mapper interface**

SqlSessionFactory를 생성하기 위해 SqlSessionFactoryBean 이 사용된다. 즉 다음처럼 스프링 설정 파일에 선언해야 한다.

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
</bean>
```

데이터 소스가 필요한것을 잊지 말것. 그 다음 Mapper Interface 가 필요하다.

```java
public interface UserMapper {
    // mapper XML 파일 대신 annotation 사용가능!
    @Select("SELECT \* FROM users WHERE id = #{userId}")
    User getUser(@Param("userId") String userId);
}
```

이 인터페이스는 다음처럼 MapperFactoryBean 를 사용해서 스프링에 추가된다.

```xml
<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
  <property name="mapperInterface" value="org.mybatis.spring.sample.mapper.UserMapper" />
  <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```

맵퍼 클래스는 반드시 인터페이스여야 한다. 실제 구현 파일이면 안된다.
{: .notice--warning}

위처럼 annotation 을 이용해서 SQL을 지정할수도 있지만, MyBatis mapper XML 파일또한 사용가능하다.

일단 설정이 완료되면, 기존 스프링 빈과 동일하게 비즈니스/서비스 객체들에게 mapper들을 inject할수 있다.

MapperFactoryBean 은 SqlSession을 생성하고, 닫는 역활을 수행한다. 또한 스프링 트랜잭션이 진행 중 이라면 그것이 완료될때, 세션 또한 커밋과 롤백 처리된다.

그리고 모든 예외들을 스프링의 DataAccessException 로 전환시키는 역활도 수행한다.

이제 MyBatis 데이터 메서드들은 코드 한줄 이면 된다.

```java
public class FooServiceImpl implements FooService
{
private UserMapper userMapper;

    public void setUserMapper(UserMapper userMapper)
    {
        this.userMapper = userMapper;
    }

    public User doSomeBusinessStuff(String userId)
    {
        return this.userMapper.getUser(userId);
    }
}
```

다음부터는 앞서의 내용에 대한 좀더 상세한 설명들이다.

## SqlSessionFactoryBean

기본적으로 MyBatis에서 SqlSessionFactory는 SqlSessionFactoryBuilder를 이용해서 만들어진다. MyBatis-Spring 에서는 대신, SqlSessionFactoryBean 가 사용된다.

### 설정

팩토리빈을 생성하기 위한 스프링 설정 파일 내용은 다음과 같다.

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
</bean>
```

SqlSessionFactoryBean 은 스프링의 FactoryBean interface 를 구현하고 있다는것을 알 필요가 있다 (section 3.8 of the Spring documentation 참조).

이 말은 스프링이 생성하는 빈은 SqlSessionFactoryBean 그자체가 아니며, 팩토리가 리턴해주는 객체는 그 팩토리의 getObject()를 호출한 결과라는 뜻이다.

이 경우 스프링은 어플리케이션 시작시에 SqlSessionFactory 를 생성하고 그이름은 sqlSessionFactory 으로 저장하게 된다. 자바 코드로 나타내면 다음과 같을것이다.

```java
SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
SqlSessionFactory sessionFactory = factoryBean.getObject();
//팩토리의 getObject()를 호출한 결과가 빈으로 생성된다.
```

일반적인 MyBatis-Spring 사용시에는 SqlSessionFactoryBean 혹은 상응하는 SqlSessionFactory 를 직접 사용할일은 없을것이다. 대신 세션 팩토리는 MapperFactoryBeans나 다른 DAO들(SqlSessionDaoSupport을 상속받은)에게 인젝트 될것이다.

### 속성들

`SqlSessionFactory` 는 필요로 하는 하나의 속성을 가지는데, 바로 `JDBC DataSource` 이다. 이것은 다른 스프링 데이터 연결과 마찬가지로 어떤것이든 설정될 수 있다.

또하나 일반적인 속성이 있는데 MyBatis XML 설정 파일의 위치를 설정하는 configLocation 이다.

이것이 필요한 이유는 MyBatis 설정이 변경될 필요가 있는 경우이다. 또한 만약 동일한 클래스 패스상에 맵퍼 XML파일과 맵퍼 클래스가 존재하지 않는 경우에도 이 설정파일이 필요하다.

이런 경우라면 2가지의 옵션이 존재하는데 첫째는 `<mapper>` 섹션지정으로 수동으로 MyBatis XML 맵퍼 파일들을 지정하는것이고, 두번째 옵션은 팩토리 빈의 속성인 `mapperLocations` 을 사용하는것이다.

`mapperLocations` 는 리소스 위치들의 리스트를 필요로 한다. 이 속성은 MyBatis XML 맵퍼 파일의 위치를 지정하기 위해 사용될수 있다.

값은 디렉토리의 모든 파일을 로드하거나, 기본 위치로부터 재귀적으로 모든 경로를 찾게끔 Ant스타일의 패턴을 포함할수 있는데, 예를 들면 다음과 같다.

```java
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
  <property name="mapperLocations" value="classpath*:sample/config/mappers/**/*.xml" />
</bean>
```

이것은 클래스 패스로부터 `sample.config.mappers` 패키지와 그 서브 패키지들에 있는 모든 MyBatis 맵퍼 XML 파일들을 로드 할 것이다.

## 트랜잭션

<span style="color:{{site.span_emphasis_color}}">
**MyBatis-Spring 을 사용하는 주된 이유중의 하나는 MyBatis를 스프링 트랜잭션에 참여하게 해주기 때문이다.**
</span>

MyBatis를 위한 특별한 트랜잭션 매니져를 만들기 보다는, MyBatis-Spring는 기존 스프링의 `DataSourceTransactionManager` 를 활용하고 있다.

스프링 트랜잭션 매니져가 설정이 되면, 일반적으로 하던것처럼 스프링에서의 트랜잭션을 설정할수 있다.

`@Transactional` 어노테이션들과 AOP 스타일의 설정들 모두 지원된다.

트랜잭션 기간동안 하나의 SqlSession 객체가 생성되어 사용된다.

이 세션은 트랜잭션이 완료될때 적절하게, 커밋 혹은 롤백처리 될것이다.

MyBatis-Spring 은 일단 설정이 되면 트랜잭션들을 투명하게 관리할것이다. 사용자의 DAO 클래스들에게 추가적인 코드는 불필요하다.

### Standard Configuration

스프링 트랜잭션 처리를 활성화 시키기 위해서는, 간단하게 `DataSourceTransactionManager` 를 스프링 설정 파일에 생성하면 된다.

```java
<bean id="transactionManager"  class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <property name="dataSource" ref="dataSource" />
</bean>
```

트랜잭션 매니져에 설정된 DataSource는 SqlSessionFactoryBean 생성 시에 설정했던 것과 동일해야만 한다. 만약 틀리다면 트랜잭션 관리는 작동하지 않을것이다.

### 컨테이너 관리 트랜잭션들 (Container Managed Transactions)

만약 JEE container 를 사용중이고 스프링이 컨테이너 관리 트랜잭션들(CMT) 에 참가하게 하려면, `JtaTransactionManager` 혹은 특정 컨테이너에 대한 그것의 서브 클래스중의 하나로 스프링 설정을 해줘야 한다. 이것을 하기 위한 가장 쉬운 방법은 스프링 트랜잭션 네임스페이스를 사용하는 것이다.

```xml
<tx:jta-transaction-manager />
```

이 설정으로, MyBatis 는 CMT로 설정된 여타 다른 스프링 트랜잭션 처리되는 리소스들과 같이 동작할것이다.

스프링은 자동으로, 존재하는 컨테이너 트랜잭션을 사용할것이고 `SqlSession` 을 그것에 덧붙일것이다.

만약 트랜잭션이 시작 안된 상태에서 트랜잭션 설정을 기본으로 그것(트랜잭션)이 필요한 경우면, 스프링은 새로운 컨테이너 관리 트랜잭션을 시작할것이다.

만약 CMT를 원하고 스프링 트랜잭션 관리를 원하지 않는다면, 어떠한 스프링 트랜잭션 설정도 해서는 안된다.

또한 기본 MyBatis 관리 트랜잭션 팩토리를 사용하기 위해 `SqlSessionFactoryBean` 을 설정해야한다.

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
  <property name="transactionFactory">
    <bean class="org.apache.ibatis.transaction.managed.ManagedTransactionFactory" />
  </property>
</bean>
```

### 코드를 통한 트랜잭션 관리

MyBatis SqlSession 은 특정 메서드를 통해서 트랜잭션들을 프로그래밍적으로 다룰수 있다. 하지만 MyBatis-Spring를 사용할때는 당신의 bean들에게는 스프링이 관리하는 SqlSession이나 스프링이 관리하는 맵퍼가 인젝트 될것이다.

즉, 스프링이 언제나 당신의 트랜잭션을 다룬다는 의미이다. 이러한 스프링 관리하의 SqlSession에서는 `SqlSession.commit()`, `SqlSession.rollback()` 혹은 `SqlSession.close()` 을 호출할수 없다.

만약 호출한다면 `UnsupportedOperationException` 예외가 발생 할 것이다.

이러한 메서드들은 인젝트된 맵퍼 클래스안에서 노출이 안된다는것을 알기 바란다.

당신의 JDBC connection's autocommit 설정에 상관없이, SqlSession 데이터 메서드 실행 혹은 스프링 트랜잭션 외부에서의 mapper 메서드 호출은 모두 자동적으로 커밋 될 것이다.

만약 당신이 트랜잭션을 프로그램적으로 제어하길 원한다면, 스프링 레퍼런스 매뉴얼 10.6 장을 참고하기 바란다.

다음 코드는 10.6.2 section에 기술된 부분인데, `PlatformTransactionManager` 를 사용해서 수동으로 트랜잭션을 다루고 있다.

```java
DefaultTransactionDefinition def = new DefaultTransactionDefinition();
def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
TransactionStatus status = txManager.getTransaction(def);

try {
    userMapper.insertUser(user);
}
catch (MyException ex) {
    txManager.rollback(status);
    throw ex;
}
txManager.commit(status);
```

이코드는 mapper를 사용하고 있지만, SqlSession을 이용해서도 동작할것이다.

## SqlSession 사용

MyBatis 에서는 (기본적인 MyBatis를 사용한다면, 즉 MyBatis-Spring을 사용하는것이 아님!) SqlSession 을 생성하기 위해 `SqlSessionFactory` 를 사용한다. 그리고 일단 세션을 얻게되면, 그것을 이용해서 당신의 맵핑된 구문들을 실행하고 연결들을 커밋하고 롤백하며, 더이상 필요하지 않다면 최종적으로 세션을 닫게 된다.

MyBatis-Spring에서는 `SqlSessionFactory` 를 직접 사용할 필요가 없다. 스프링의 트랜잭션 설정에 근거해서 자동으로 커밋, 롤백, 세션종료가 이루어지며 쓰레드에도 안전한 SqlSession 이 당신의 빈들에게 인젝트 될수있기 때문이다.

또한, 일반적으로 SqlSession 을 직접 사용할 필요가 없다는것도 알아둘 필요가 있다. 대부분의 경우, 단지 필요한 것은 당신의 빈들에게 맵퍼들을 인젝트 할 `MapperFactoryBean` 일 것이다(이것은 다음 장에 설명된다).

### SqlSessionTemplate

이것은 MyBatis-Spring의 심장이다. 이 클래스는 MyBatis SQL method들을 호출하면서 예외들을 전환하면서 MyBatis SqlSession 들을 관리하는 책임이 있다. `SqlSessionTemplate` 은 쓰레드 안전하고 여러 DAO들 사이에서 공유 가능하다.

`getMapper()` 호출로 얻은 맵퍼의 SQL 메서드들을 호출할때, `SqlSessionTemplate` 은 SqlSession 이 현재 스프링 트랜잭션에 연관된 것임을 보장할것이다. 또한, 필요에 따라 세션의 종료, 커밋, 롤벡을 포함하는 세션 생명주기를 관리한다.

SqlSessionTemplate 은 SqlSession을 구현하고 있는데, 코드에 존재하는 기존 SqlSession 사용에 대해 간단히 교체 된다.

기본 MyBatis 구현인 DefaultSqlSession 대신 `SqlSessionTemplate` 이 항상 사용되어져야 하는데 스프링 트랜잭션에 템플릿이 참가 가능하고 여러개의 인젝트 되어진 맵퍼 클래스에 의해 사용될때 쓰레드 안전하기 때문이다.

동일 어플리케이션에서 이 두가지 클래스(`DefaultSqlSession` 과 `SqlSessionTemplate`)를 혼용하는것은 데이터 무결성 문제를 야기할수 있다.

`SqlSessionTemplate` 은 생성자 인자로 `SqlSessionFactory` 를 사용해서 생성될수 있다.

```xml
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
  <constructor-arg index="0" ref="sqlSessionFactory" />
</bean>
```

이 bean은 이제 당신의 DAO 빈들에게 직접 인젝트될수 있다. 당신의 bean에 SqlSession 프러퍼티가 다음처럼 필요하다.

<span style="color:{{site.span_emphasis_color}}">
**(다음 코드는 직접 SqlSession를 사용하는 경우이다. Mapper를 이용 한다면 달라질것이다)**
</span>

```java
public class UserDaoImpl implements UserDao
{
private SqlSession sqlSession;

    public void setSqlSession(SqlSession sqlSession)
    {
        this.sqlSession = sqlSession;
    }

    public User getUser(String userId)
    {
        return
        (User) sqlSession.selectOne("org.mybatis.spring.sample.mapper.UserMapper.getUser",
                                                                userId);
    }
}
```

그리고 다음처럼 `SqlSessionTemplate` 을 인젝트 한다.

```xml
<bean id="userDao" class="org.mybatis.spring.sample.dao.UserDaoImpl">
  <property name="sqlSession" ref="sqlSession" />
</bean>
```

`SqlSessionTemplate` 은 또한 ExecutorType을 입력인자로 가지는 생성자를 가지고 있다. 다음처럼 batch SqlSession 을 생성할수 있게 해준다.

```xml
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
  <constructor-arg index="0" ref="sqlSessionFactory" />
  <constructor-arg index="1" value="BATCH" />
</bean>
```

이제 모든 당신의 구문들은 배치로 동작할것이고 DAO에서 다음처럼 코딩될수 있다.

```java
public void insertUsers(User[] users) {
    for (User user : users) {
        sqlSession.insert("org.mybatis.spring.sample.mapper.UserMapper.insertUser", user);
    }
}
```

이런 설정 스타일은 `SqlSessionFactory` 를 위한 기본 설정과 상이한 실행 메서드가 요구되는 경우만 필요 하다는 것을 알 필요가 있다.

이러한 방식에서 잊지 말아야 할것은
<span style="color:{{site.span_emphasis_color}}">
**이 메서드가 호출될때 서로다른 ExecutorType에 진행중인 기존 트랜잭션이 존재하면 안된다** </span>는 점이다.

별도 트랜잭션에서 동작하는 다른 ExecutorType으로 SqlSessionTemplate들을 호출 하던지 (즉, PROPAGATION_REQUIRES_NEW 사용으로) 아니면 완전히 트랜잭션 밖에서 동작하게 해야 한다.

### SqlSessionDaoSupport

SqlSession을 제공해주는 추상 클래스이다. 다음 예 처럼 getSqlSession() 을 호출해서 `SqlSessionTemplate` 을 얻을수 있고, SQL 메서드들을 수행할수 있다.

```java
public class UserDaoImpl extends SqlSessionDaoSupport implements UserDao {
    public User getUser(String userId) {
        return (User) getSqlSession().
        selectOne("org.mybatis.spring.sample.mapper.UserMapper.getUser", userId);
    }
}
```

일반적으로 이 클래스보다 `MapperFactoryBean` 이 선호된다(**추가 코드가 불필요하기 때문**).

하지만, 당신의 DAO 에서 다른 비-MyBatis 작업을 수행할 필요가 있을때나, concrete 클래스들이 필요할때 유용하다.

`SqlSessionDaoSupport` 는 `sqlSessionFactory` 와 `sqlSessionTemplate` 속성중 하나만 필요로 한다. Spring에 의해 autowired 되거나, 명시적으로 설정 가능하다. 만약 두 속성이 모두 설정이 된 경우라면, `sqlSessionFactory` 는 무시된다.

다음 예에서 UserDaoImpl 가 만약 `SqlSessionDaoSupport` 의 자식 클래스라고 하면, 스프링에서 다음처럼 설정 되어질 수 있다.

```xml
<bean id="userMapper" class="org.mybatis.spring.sample.mapper.UserDaoImpl">
    <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```

## Mappers 인젝팅

`SqlSessionDaoSupport` 나 `SqlSessionTemplate` 사용해서 data access objects (DAOs)를 수동으로 코딩하는 것보다, Mybatis-Spring 는 `MapperFactoryBean` 이라는 프록시 팩토리를 제공하고 있다.

이클래스는 data mapper 인터페이스들을 다른 빈들에게 직접 인젝트 할수 있게 해준다.

맵퍼들을 이용할때는 DAO들을 호출했던것과 동일하게 호출하지만, Mybatis-Spring이 프록시를 생성할것이기 때문에, DAO 구현을 위해서 어떠한 코딩도 필요하지 않다.

인젝트되어진 맵퍼들 인해 당신의 코드는 MyBatis, 스프링 혹은 MyBatis-Spring과 직접적인 의존관계를 갖지 않게 될것이다.

`MapperFactoryBean` 가 생성하는 프록시는 세션을 열고 닫는 것을 다루며 또한 예외들을 스프링의 `DataAccessException` 로 변환한다.

더 나아가 만약 필요한경우 그 프록시는 새로운 Spring 트랜잭션을 시작하거나, 트랜잭션이 활성화 되어 있는 경우, 기존 트랜잭션에 참가 할것이다.

### MapperFactoryBean

데이터 맵퍼는 스프링에 다음과 같이 추가된다.

```xml
<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
  <property name="mapperInterface" value="org.mybatis.spring.sample.mapper.UserMapper" />
  <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```

`MapperFactoryBean` 는 UserMapper 를 구현하는 프록시 클래스를 생성하고, 그것을 어플리케이션에 인젝트 한다.

프록시는 런타임시에 생성되므로, 지정된 맵퍼는 실제 구현 클래스가 아닌, 인터페이스여야만 한다.

만약 UserMapper 가 상응하는 MyBatis XML mapper 파일을 가지고 있고, mapper interface 와 동일한 클래스 패스에 존재한다면, 자동적으로 `MapperFactoryBean` 에 의해 해석될것이다.

만약 mapper XML 파일이 다른 클래스 패스 위치상에 있는 경우만 아니면 MyBatis 설정 파일에 맵퍼를 지정할 필요는 없다.( SqlSessionFactoryBean 의 configLocation 속성 참조)

`MapperFactoryBean` 는 `SqlSessionFactory` 나 `SqlSessionTemplate` 를 필요로 하는것에 유의하기 바란다.

이것들은 각각의 `sqlSessionFactory` 나 `sqlSessionTemplate` 속성이나 스프링에 의해 autowired 에 의해 설정되어질수 있다.

만약 둘다 설정이 된다면 `SqlSessionFactory` 가 무시된다. `SqlSessionTemplate` 가 session factory 설정을 필요로 하므로, 그 팩토리는 `MapperFactoryBean` 에 의해 사용된다.

이제 맵퍼들을 당신의 비즈니스/서비스 객체들에게 직접 인젝트 가능하다.

```xml
<bean id="fooService" class="org.mybatis.spring.sample.mapper.FooServiceImpl">
<property name="userMapper" ref="userMapper" />
</bean>
```

이 bean은 어플리케이션 로직에서 직접 사용가능하다.

```java
public class FooServiceImpl implements FooService
{
private UserMapper userMapper;

    public void setUserMapper(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    public User doSomeBusinessStuff(String userId) {
        return this.userMapper.getUser(userId);
    }
}
```

이 코드에는 어떠한 SqlSession 나 MyBatis 참조도 없다는것을 주목하기 바란다.

또한 세션을 생성하고, 열고 닫을 필요도 없다. MyBatis-Spring가 그 일들을 해줄 것이다.

### MapperScannerConfigurer

모든 맵퍼들을 스프링 XML파일에 등록할 필요없이, `MapperScannerConfigurer` 를 이용해서 MapperFactoryBean 처럼 당신의 맵퍼들을 위해 클래스 패스를 검색하고 자동적으로 그것들을 설정하게 할 수 있다. 다음처럼 스프링 설정에 추가하면 된다.

```xml
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  <property name="basePackage" value="org.mybatis.spring.sample.mapper" />
</bean>
```

basePackage 속성은 맵퍼 인터페이스 파일들의 기본 패키지를 설정하는것이다. 세미콜론이나 콤마 구분자를 이용해서 하나이상의 패키지를 설정할수 있다. 그러면 지정된 패키지부터 시작해서 재귀적으로 mapper가 검색된다.

SqlSessionFactory 나 SqlSessionTemplate 를 지정할 필요가 없는것에 주목하기 바란다. 왜냐하면 MapperScannerConfigurer 가 autowired될수 있는 MapperFactoryBean들을 생성할 것이기 때문이다.

그런데 만약 하나 이상의 데이터 소스를 사용중이라면, autowire 는 작동하지 않을 수 있다. 이 경우에는 sqlSessionFactoryBeanName 나 sqlSessionTemplateBeanName 속성들을 사용해서 사용할 올바른 bean 이름을 지정 할 수 있다.

주의할것은 bean의 참조가 아니라 이름이 필요하다는 것이다. 그 때문에 일반적인 ref 대신 value 어트리뷰트가 사용된다.

```xml
<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
```

알아둘것 : `sqlSessionFactoryBean` 과 `sqlSessionTemplateBean` 속성은 MyBatis-Spring 1.0.2 버전까지 가능했던 그냥 옵션일 뿐이었다. 하지만 `MapperScannerConfigurer` 가 기동 프로세스의 초기에 구동된다고 할때 `PropertyPlaceholderConfigurer` 에 빈번한 에러가 발생했다. 그래서 이 속성들은 deprecated 되었고 새로운 속성, 즉 `sqlSessionFactoryBeanName` 과 `sqlSessionTemplateBeanName` 사용이 권장된다.
{: .notice--info}

MapperScannerConfigurer 는 marker interface 지정 혹은 annotation 에 의해 생성된 맵퍼들을 필터링하는것을 지원한다.

annotationClass 속성은 찾고자 하는 annotation 을 지정한다. markerInterface 속성은 찾고자 하는 부모 인터페이스 를 지정한다.

만약 두 속성이 모두 지정되면 한쪽 조건에 일치하는 멥퍼들이 추가된다. 디폴트로 이속성들은 null이다. 따라서 기본 패키지에 있는 모든 인터페이스들이 맵퍼들로 로드 될것이다.

발견된 맵퍼들은 스프링의 자동 감지된 컴퍼넌트들을 위한 기본 네이밍 전략에 의해 이름이 정해진다(section 3.14.4 of the Spring manual).

즉, 어노테이션이 발견 안된 경우, 소문자로 구성된 맵퍼의 클래스(non-qualified 클래스명,즉 패키지 제외한 단순 클래스명만)를 이름으로 사용할것이다.

하지만 @Component 혹은 JSR-330 @Named 어노테이션이 발견된다면 그것으부터 이름이 결정된다.

`annotationClass` 속성에 `org.springframework.stereotype.Component`, `javax.inject.Named` (JSE 6 경우만) 혹은 사용자 고유의 annotation 을 설정할수 도 있는데 이러면 annotation 이 마커와 네임 프로바이더로 모두 동작하게 된다.

## MyBatis API 사용하기

MyBatis-Spring 에서 MyBatis API를 직접 사용하는 방법이다. 스프링에서 `SqlSessionFactoryBean` 을 이용해서 `SqlSessionFactory` 를 생성하고, 코드상에서 factory 를 사용한다.

```java
public class UserMapperSqlSessionImpl implements UserMapper {
// SqlSessionFactory would normally be set by SqlSessionDaoSupport
private SqlSessionFactory sqlSessionFactory;

    public void setSqlSessionFactory(SqlSessionFactory sqlSessionFactory) {
        this.sqlSessionFactory = sqlSessionFactory;
    }

    public User getUser(String userId) {
        // note standard MyBatis API usage - opening and closing the session manually
        SqlSession session = sqlSessionFactory.openSession();

        try {
            return
            (User) session.selectOne("org.mybatis.spring.sample.mapper.UserMapper.getUser",
                                    userId);
        } finally {
            session.close();
        }
    }
}
```

<span style="color:{{site.span_emphasis_color}}">
**이 방법을 사용할때는 주의를 기울여야 한다.** </span>잘못된 사용으로 인해 런타임 에러가 발생하거나, 심하면 데이터 무결성에 문제를 일으킬수도 있다.

직접 API를 사용하기 위해서는 다음을 명심해야 한다.

-   이것은 어떠한 스프링 트랜잭션들에게도 참가하지 않는다

-   만약 SqlSession 사용중인 DataSource 를 Spring transaction manager 도 사용중이고,
    현재 진행중인 트랜잭션이 존재한다면 이 코드는 예외를 던질것이다.

-   MyBatis의 DefaultSqlSession 은 쓰레드에 안전하지 않다. 만약 그것을 당신의 bean들에게
    인젝트 한다면 에러가 발생할것이다.

-   DefaultSqlSession 를 사용해서 만들어진 mapper들도 쓰레드에 안전하지 않다.
    만약 그것을 당신의 bean들에게 인젝트 한다면 에러가 발생할것이다.

-   SqlSessions 이 finally 블럭에서 항상 닫혀지는지 확인해야 한다.

## 간단한 게시판 예제를 참고하기 바란다.

[MyBatis-Spring : mapper interface 와 annotation을 활용한 게시판 예제]({% post_url 2012-07-24-mybatis-spring-mapper-interface %})
