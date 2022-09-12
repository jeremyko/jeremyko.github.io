---
layout: post
title: 'MyBatis-Spring : mapper interface 와 annotation을 활용한 게시판 예제'
date: '2012-07-24T16:21:00.002+09:00'
tags:
    - MyBatis
    - spring
    - MyBatis-Spring
    - 게시판
    - java
modified_time: '2012-08-29T13:08:41.462+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-4277483001133499946
blogger_orig_url: https://jeremyko.blogspot.com/2012/07/mybatis-spring-mapper-interface.html
---

### MyBatis-Spring : mapper interface 와 annotation을 활용한 게시판 예제

지난번 MyBatis-Spring 에 대한 소개 에 이어서, 이를 활용한 간단한 게시판 작성 예제를 작성해본다.

XML파일에 맵핑을 정의하지 않고, mapper interface 와 annotation을 이용해서 처리하는것으로 해봤다.

게시판은 전체 목록 출력, 검색기능, 내용보기 및 수정, 삭제등의 아주 기본적인 기능만을 다루고 있다.

이 프로젝트의 소스는 다음 위치에 있으니 참조하시길 바란다.

[https://github.com/jeremyko/SpringMvcBoardMyBatis](https://github.com/jeremyko/SpringMvcBoardMyBatis)

### 우선 기본적인 스프링 MVC 프로젝트를 생성한다

간단하게 설명하기 위해서 스프링 환경 설정 부분은 생략했다. 일단 사용된 툴만 적어 보면 다음과 같다 (환경들은 알아서들 잘 맞추시길..자바는 환경 설정이 제일 어려운것 같다.^^).

-   Eclipse Indigo(3.7)
-   apache-tomcat-7.0.27
-   Maven Integration for Eclipse
-   SpringSource Tool Suite (STS) for Eclipse Indigo(3.7)
-   데이터베이스: oracle 11g XE

사용된 이클립스 플러그인은 help -> Eclipse Marketplace 에서 검색.

이클립스 file -> new -> others -> SpringSource Tool Suite -> Spring Template Project -> Spring MVC Project 로 프로젝트를 생성.

그다음 pom.xml 파일을 열어서 dependency에 mybatis-spring 을 추가한다

```xml
<!-- MyBatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.1.1</version>
</dependency>
```

추가된 dependency를 반영하기 위해서 Run As -> Maven install을 수행한다.

Run As -> Run on Server로 생성된 프로젝트가 정상 작동 하는지 확인하기 바란다.
Hello World 라고 큼지막하게 웹 페이지가 뜨면 성공이다.

### 게시판 테이블 및 seq 생성

이 예제에서는 오라클을 이용하고 있다. XE버전은 무료라서 개인이 사용하기에 좋은것 같다. 단지 단점이라면 리소스를 많이 잡아먹는다 (500 MB 메모리 사용). 이클립스가 사용하는 리소스도 상당한데, Oracle까지 돌리면 기본 1GB 정도는 그냥 두놈이 차지하게 된다. 컴에 램이 4GB 는 되어야 좀 돌릴만하다. 만약 다른 DB를 사용한다면, 아래의 내용을 참조로 수정이 필요할 수도 있겠다.

```sql
CREATE SEQUENCE   "SEQ_ID"  MINVALUE 1 MAXVALUE 99999999999
INCREMENT BY 1 START WITH 1 ;

CREATE TABLE  "SPRING_BOARD"
(
    "ID" NUMBER(10,0) NOT NULL,
    "SUBJECT" VARCHAR2(50),
    "NAME" VARCHAR2(50),
    "CREATED_DATE" DATE,
    "MAIL" VARCHAR2(50),
    "MEMO" VARCHAR2(200),
    "HITS" NUMBER(10,0),
    PRIMARY KEY ("ID") ENABLE
) ;
```

### bean 클래스 생성

데이터를 전달하고 받을때 사용될 bean 클래스를 추가한다.

```java
//BoardBean.java
package kojh.db.beans;

public class BoardBean
{
    int id;
    String name;
    String mail;
    String subject;
    String created_date;
    int hits;
    String memo;
    //이클립스의 자동 setter/getter 생성 기능을 이용하자~
    //개인적으로는 { 를 항상 함수명 다음 라인에 코딩하지만, 분량을 줄이고자 이어 붙였다..
    public int getId()  {
        return id;
    }
    public void setId(int id)  {
        this.id = id;
    }
    public String getName()  {
        return name;
    }
    public void setName(String name)  {
        this.name = name;
    }
    public String getMail()  {
        return mail;
    }
    public void setMail(String mail)  {
        this.mail = mail;
    }
    public String getSubject()  {
        return subject;
    }
    public void setSubject(String subject)  {
        this.subject = subject;
    }
    public int getHits()  {        return hits;
    }
    public void setHits(int hits)  {
        this.hits = hits;
    }
    public String getMemo()  {
        return memo;
    }
    public void setMemo(String memo)  {
        this.memo = memo;
    }
    public String getCreated_date()  {
        return created_date;
    }
    public void setCreated_date(String created_date)  {
        this.created_date = created_date;
    }
}
```

### BoardMapper 생성

Mapper는 앞서 설명된 바와 같이 interface만 가능하다. 실제 구현 클래스로 정의하면 안된다.
또한 annotation을 이용해서 데이터 처리를 하는것으로 하였다.

```java
// BoardMapper.java
package kojh.spring.mappers;
import java.util.ArrayList;
import org.apache.ibatis.annotations.Result;
import org.apache.ibatis.annotations.Results;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Update;
import org.apache.ibatis.annotations.Delete;
import kojh.db.beans.BoardBean;

public interface BoardMapper
{
    final String SELECT_PAGE =
       "SELECT * FROM ( SELECT  ID,SUBJECT,NAME, CREATED_DATE, "+
       "MAIL,MEMO,HITS, ceil( rownum / #{rowsPerPage} ) as page "+
       "FROM SPRING_BOARD  ORDER BY ID DESC ) WHERE page = #{page}";

    final String SELECT_BY_ID =
        "SELECT ID,SUBJECT,NAME,CREATED_DATE,MAIL,MEMO,HITS"+
        "from SPRING_BOARD WHERE ID=#{id}";

    final String SELECT_CNT_BY_SUBJECT =
        "SELECT COUNT(1) FROM SPRING_BOARD WHERE"+
        "SUBJECT LIKE '%'||'${searchThis}'||'%'";

    final String SELECT_ROWS_BY_SUBJECT =
        "SELECT * FROM (SELECT ID,SUBJECT,NAME,"+
        "CREATED_DATE, MAIL,MEMO,HITS, "+
        "ceil( rownum / #{rowsPerPage}) as page FROM SPRING_BOARD "+
        "WHERE SUBJECT LIKE '%'||'${likeThis}'||'%' ORDER BY ID DESC) "+
        "WHERE page = #{page}";

    final String SELECT_CNT_ALL = "SELECT count(1) FROM SPRING_BOARD";

    final String INSERT =
        "INSERT INTO SPRING_BOARD (ID,SUBJECT,NAME,CREATED_DATE,MAIL,"+
        "MEMO) VALUES( SEQ_ID.NEXTVAL,#{subject}, #{name},"+
        "SYSDATE, #{mail}, #{memo})";

    final String UPDATE_BY_ID =
        "UPDATE SPRING_BOARD SET SUBJECT= #{subject},MAIL= #{mail},"+
        "MEMO= #{memo} WHERE ID= #{id}";

    final String DELETE_BY_ID= "DELETE FROM SPRING_BOARD WHERE ID=#{id}";

    //BoardBean 의 속성들과 동일한 이름으로 #{mail} 등을 지정해야한다.
    @Select(SELECT_PAGE)
    @Results(value = {
            @Result(property="id", column="ID"),
            @Result(property="subject", column="SUBJECT"),
            @Result(property="name", column="NAME"),
            @Result(property="created_date", column="CREATED_DATE"),
            @Result(property="mail", column="MAIL"),
            @Result(property="memo", column="MEMO"),
            @Result(property="hits", column="HITS")
        })
    ArrayList<BoardBean> getList(@Param("page") int page, @Param("rowsPerPage") int rowsPerPage);

    @Select(SELECT_BY_ID)
    @Results(value = {
            @Result(property="id", column="ID"),
            @Result(property="subject", column="SUBJECT"),
            @Result(property="name", column="NAME"),
            @Result(property="created_date", column="CREATED_DATE"),
            @Result(property="mail", column="MAIL"),
            @Result(property="memo", column="MEMO"),
            @Result(property="hits", column="HITS")
        })
    BoardBean getSpecificRow(@Param("id") String id);

    // 전체 글 갯수를 조회
    @Select(SELECT_CNT_ALL)
    int getTotalCnt();

    // 해당 주제의 관련글 갯수를 조회
    @Select(SELECT_CNT_BY_SUBJECT)
    int getTotalCntBySubject(@Param("searchThis") String includingThis);

    //해당 주제의 관련글 조회
    @Select(SELECT_ROWS_BY_SUBJECT)
    @Results(value = {
            @Result(property="id", column="ID"),
            @Result(property="subject", column="SUBJECT"),
            @Result(property="name", column="NAME"),
            @Result(property="created_date", column="CREATED_DATE"),
            @Result(property="mail", column="MAIL"),
            @Result(property="memo", column="MEMO"),
            @Result(property="hits", column="HITS")
        })
    public ArrayList<BoardBean> getSearchedList(@Param("page") int page,
                                                @Param("rowsPerPage") int rowsPerPage,
                                                @Param("likeThis") String strSearchThis);

    @Insert(INSERT)
    //@Options(useGeneratedKeys = true, keyProperty = "id")
    void insertBoard (BoardBean boardBean);

    @Update(UPDATE_BY_ID)
    void updateBoard (@Param("id") int id,@Param("subject") String subject, @Param("mail") String mail,@Param("memo") String memo);

    @Delete(DELETE_BY_ID)
    void deleteSpecificRow(@Param("id") int id);
}
```

### BoardService 생성

데이터 처리를 위한 클래스를 생성한다. 원래는 DB처리를 위한 인터페이스를 정의하고 실제 사용은 그 인터페이스를 구현하는 클래스를 정의해서 처리하게 끔 권장된다.

이것은 DB 종류가 변경시 이같은 인터페이스 구현으로 처리하면, 나중에 소스 코드에 영향을 주지 않고 적용이 가능하기 대문이다.

지금은 간단하게 바로 구현 클래스로 처리하였다.

```java
package kojh.spring.board;

import java.util.ArrayList;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import kojh.db.beans.BoardBean;
import kojh.spring.mappers.BoardMapper;

@Component
public class BoardService //직접 구현함~~ 원래는 interface로 정의 하는게 좋아요~
{
    @Autowired
    private BoardMapper boardMapper;

    public ArrayList<BoardBean> getList(int nStartPage, int list_num) {
        return this.boardMapper.getList(nStartPage, list_num);
    }
    public BoardBean getSpecificRow(String id) {
        return this.boardMapper.getSpecificRow(id);
    }

    public int getTotalCnt() {
        int nCnt = 0;
        nCnt= this.boardMapper.getTotalCnt();
        return nCnt;
    }

    public int getTotalCntBySubject(String search) {
        int nCnt = 0;
        nCnt= this.boardMapper.getTotalCntBySubject(search) ;
        return nCnt;
    }

    public void insertBoard (BoardBean boardBean) {
        boardMapper.insertBoard(boardBean);
    }

    public void updateBoard (BoardBean boardBean) {
        boardMapper.updateBoard(boardBean.getId(),
            boardBean.getSubject(), boardBean.getMail(), boardBean.getMemo());
    }

    public void deleteRow(int id)  {
        this.boardMapper.deleteSpecificRow(id);
    }

    public ArrayList<BoardBean> getSearchedList(int nStartPage, int list_num,
                                                                                        String strSearchThis)  {
        return this.boardMapper.getSearchedList(nStartPage, list_num, strSearchThis);
    }
}
```

보다시피 mapper로 호출을 전달하므로 MyBatis에 대한 의존성이 없다.

### context 설정 파일에 추가

자 이제 제일 중요한 스프링 설정 시간이다.

servlet-context.xml 은 변경할 필요 없다.

root-context.xml 을 다음으로 변경한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:util="http://www.springframework.org/schema/util"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"

    xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-3.0.xsd
        http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.0.xsd">

    <!-- Root Context: defines shared resources visible to all other web components -->
    <!-- 데이터 소스 -->
    <bean id="dataSource"
        class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName">
            <value>oracle.jdbc.driver.OracleDriver</value>
        </property>
        <property name="url">
            <value>jdbc:oracle:thin:@localhost:1521:XE</value>
        </property>
        <property name="username">
            <value>study</value>
        </property>
        <property name="password">
            <value>study</value>
        </property>
    </bean>

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
      <property name="dataSource" ref="dataSource" />
      <!-- <property name="configLocation" value="/WEB-INF/spring/mybatis/mybatis-config.xml"/> -->
      <property name="mapperLocations"  value="classpath:kojh/spring/board/mappers/**/*.xml" />
    </bean>

    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
     <constructor-arg index="0" ref="sqlSessionFactory"></constructor-arg>
    </bean>

    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>

    <tx:annotation-driven transaction-manager="transactionManager" />

    <!-- Mapper! -->
    <bean id="boardMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
      <property name="mapperInterface" value="kojh.spring.mappers.BoardMapper" />
      <property name="sqlSessionFactory" ref="sqlSessionFactory" />
    </bean>

    <!-- BoardService -->
    <bean id="boardService" class="kojh.spring.board.BoardService">
    </bean>
</beans>
```

### 새로운 컨트롤러 클래스 생성

원래 Spring MVC Project 로 생성된 프로젝트는 HomeController 가 호출되어, home.jsp 뷰를 호출해서 Hello World 문자열을 찍게 되어 있다.

우리는 이것을 사용하지 않고 새로운 컨트롤러를 생성해서, 처음 게시판 화면 출력 및 기타 게시판 처리시 목록 리스트 출력에 사용할려고 한다.

새로운 컨트롤러 클래스를 생성하자. 이름은 게시판 기능을 전부 담당하는 BbsController 이다.

```java
//BbsController.java
package kojh.spring.board;

import kojh.db.beans.BoardBean;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class BbsController
{
    @Autowired
    BoardService boardService;

    private static final Logger logger = LoggerFactory.getLogger(HomeController.class);

    ////////////////////////////////////////////////////////////////////////////////////////////////////
    @RequestMapping(value = "/show_write_form", method = RequestMethod.GET)
    public String show_write_form( Model model)
    {
        logger.info("show_write_form called!!");
        // 객체를 전달해서 값을 얻어와야 함!!!
        model.addAttribute("boardBeanObjToWrite", new BoardBean());
        return "writeBoard";
    }

    ////////////////////////////////////////////////////////////////////////////////////////////////////
    @RequestMapping(value = "/DoWriteBoard", method = RequestMethod.POST)
    public String DoWriteBoard( BoardBean boardBeanObjToWrite,
                                Model model)
    {
        logger.info("DoWriteBoard called!!");
        logger.info("memo=["+boardBeanObjToWrite.getMemo()+"]");
        boardService.insertBoard(boardBeanObjToWrite);
        model.addAttribute("totalCnt", new Integer(boardService.getTotalCnt()) );
        model.addAttribute("current_page", "1" ); //글을 작성후에는 처음 페이지로 돌아간다
        model.addAttribute("boardList", boardService.getList( 1, 2));
        return "listSpecificPage";
    }

    ////////////////////////////////////////////////////////////////////////////////////////////////////
    //개별 목록 조회
    @RequestMapping(value = "/viewWork", method = RequestMethod.GET)
    public String viewWork    (
                                @RequestParam("memo_id") String memo_id,
                                @RequestParam("current_page") String current_page,
                                @RequestParam("searchStr") String searchStr,
                                Model model
                            )
    {
        logger.info("viewWork called!!");
        logger.info("memo_id=["+ memo_id+"] current_page=["+current_page+
              "] searchStr=["+searchStr+"]");

        model.addAttribute("memo_id", memo_id );
        model.addAttribute("current_page", current_page );
        model.addAttribute("searchStr", searchStr );
        model.addAttribute("boardData", boardService.getSpecificRow(memo_id) );
        return "viewMemo";
    }

    ////////////////////////////////////////////////////////////////////////////////////////////////////
    // 특정 페이지에서 작업중 목록으로 나올경우, 이전 페이지 번호를 참조해서
    // 해당 페이지 출력
    @RequestMapping(value = "/listSpecificPageWork", method = RequestMethod.GET)
    public String listSpecificPageWork    (
                                @RequestParam("current_page") String pageForView,
                                Model model
                            )
    {
        logger.info("listSpecificPageWork called!!");
        logger.info("current_page=["+pageForView+"]");
        model.addAttribute("totalCnt", new Integer(boardService.getTotalCnt()) );
        model.addAttribute("current_page", pageForView );
        model.addAttribute("boardList", boardService.getList( Integer.parseInt(pageForView), 2));
        return "listSpecificPage";
    }

    ////////////////////////////////////////////////////////////////////////////////////////////////////
    // 특정 페이지 수정을 위한 내용 출력
    @RequestMapping(value = "/listSpecificPageWork_to_update", method = RequestMethod.GET)
    public String listSpecificPageWork_to_update    (
                                @RequestParam("memo_id") String memo_id,
                                @RequestParam("current_page") String current_page,
                                Model model
                            )
    {
        logger.info("listSpecificPageWork_to_update called!!");
        logger.info("memo_id=["+ memo_id+"] current_page=["+current_page+"]");
        model.addAttribute("memo_id", memo_id );
        model.addAttribute("current_page", current_page );
        model.addAttribute("boardData", boardService.getSpecificRow(memo_id) );
        return "viewMemoForUpdate";
    }

    ////////////////////////////////////////////////////////////////////////////////////////////////////
    //개별 row 업데이트
    @RequestMapping(value = "/DoUpdateBoard", method = RequestMethod.POST)
    public String DoUpdateBoard(
                                BoardBean boardBeanObjToUpdate,
                                @RequestParam("memo_id") int memo_id, //String,int 둘다 작동됨
                                @RequestParam("current_page") String current_page,
                                Model model)
    {
        logger.info("DoUpdateBoard called!!");
        logger.info("listSpecificPageWork_to_update called!!");
        //boardBeanObjToUpdate.getId() 가 0이다! 값을 설정하지 않았기 때문이다. 대신,memo_id 를 이용하자
        logger.info("memo_id=["+ memo_id+""+"/"+boardBeanObjToUpdate.getId()+"] current_page=["+current_page+"]");
        logger.info("memo=["+boardBeanObjToUpdate.getMemo()+"]");

        boardBeanObjToUpdate.setId(memo_id); // 약간의 꼼수...
        boardService.updateBoard(    boardBeanObjToUpdate    );
        model.addAttribute("totalCnt", new Integer(boardService.getTotalCnt()) );
        model.addAttribute("current_page", current_page );
        model.addAttribute("boardList", boardService.getList( Integer.parseInt(current_page), 2));
        return "listSpecificPage";
    }

    ////////////////////////////////////////////////////////////////////////////////////////////////////
    // 삭제 DeleteSpecificRow
    @RequestMapping(value = "/DeleteSpecificRow", method = RequestMethod.GET)
    public String DeleteSpecificRow(@RequestParam("memo_id") int memo_id,
                                    @RequestParam("current_page") String current_page,
                                    Model model)
    {
        logger.info("DeleteSpecificRow called!!");
        logger.info("memo_id=[" + memo_id + "] current_page=[" + current_page + "]");
        boardService.deleteRow(memo_id);
        //다시 페이지를 조회한다.
        model.addAttribute("totalCnt", new Integer(boardService.getTotalCnt()) );
        model.addAttribute("current_page", current_page);
        model.addAttribute("boardList", boardService.getList( Integer.parseInt(current_page), 2));
        return "listSpecificPage";
    }

    ////////////////////////////////////////////////////////////////////////////////////////////////////
    @RequestMapping(value = "/searchWithSubject", method = RequestMethod.POST)
    public String searchWithSubject    (    @RequestParam("searchStr") String searchStr,
                                        Model model    )
    {
        //redirection...
        return listSearchedSpecificPageWork("1", searchStr, model);//처음에는 1 페이지만 보여줌
    }

    ////////////////////////////////////////////////////////////////////////////////////////////////////
    // 검색된 상태에서 특정 페이지로 이동하기
    @RequestMapping(value = "/listSearchedSpecificPageWork", method = RequestMethod.GET)
    public String listSearchedSpecificPageWork    (    @RequestParam("pageForView") String pageForView,
                                                    @RequestParam("searchStr") String searchStr,
                                                    Model model )
    {
        logger.info("listSearchedSpecificPageWork called!!");
        logger.info("pageForView=["+pageForView+"]");
        logger.info("searchStr=["+searchStr+"]");

        model.addAttribute("totalCnt", new Integer( boardService.getTotalCntBySubject(searchStr) ) );
        model.addAttribute("searchedList",
            boardService.getSearchedList(Integer.parseInt(pageForView),2, searchStr) );
        model.addAttribute("pageForView", Integer.parseInt(pageForView) );
        model.addAttribute("searchStr", searchStr );
        return "listSearchedPage";
    }
}
```

### 유틸 클래스 정의

페이지 설정 로직을 위한 별도 클래스를 정의해 보았다. 지금은 별로 하는일이 없지만, 나중에 기능이 추가되서 게시판 목록 블럭 처리등에 활용할수 있겠다..

```java
//PageNumberingManager.java
package kojh.spring.board;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class PageNumberingManager
{
    private static final PageNumberingManager pageNumberingManager =
            new PageNumberingManager();

    private static final Logger logger =
            LoggerFactory.getLogger(PageNumberingManager.class);

    private PageNumberingManager()    { }

    public  static PageNumberingManager getInstance()
    {
        return pageNumberingManager ;
    }

    public int getTotalPage(int total_cnt, int rowsPerPage)
    {
        logger.info("getTotalPage called!!");
        int total_pages = 0;
        if ((total_cnt % rowsPerPage) == 0)
            total_pages = total_cnt / rowsPerPage;
        else
            total_pages = (total_cnt / rowsPerPage) + 1;
        logger.info("getTotalPage return total_pages="+total_pages);
        return total_pages;
    }

    //게시판의 block처리 추가 필요 (이전/다음 블럭 버튼 처리)
    public int getPageBlock(int curPage, int pagePerBlock)
    {
        int block = 0;
        if ((curPage % pagePerBlock) == 0)
        {
           block = curPage / pagePerBlock;
        }
        else
        {
           block = curPage / pagePerBlock + 1;
        }
        return block;
    }

    // block 에 속한 첫번째 페이지 계산
    public int getFirstPageInBlock(int block, int pagePerBlock)
    {
        return ( ( block - 1 ) * pagePerBlock + 1 ) ;
    }

    // block 에 속한 마지막 페이지 계산
    public int getLastPageInBlock(int block, int pagePerBlock)
    {
        return ( block * pagePerBlock);
    }

    //... 기타 등등...
}
```

### 출력되는 뷰를 구현한다

게시판 화면처리에서는 잘 알다시피 페이지 개념이 있는데, 하나의 페이지에는 여러개의 글들이 포함하게 된다. 특정 페이지를 출력하는 기능을 담당하는 뷰를 정의한다. 간단한 예제 작성이 목적이라는 핑계로 글 목록 블럭 처리기능은 구현되지 않았다...

```jsp
//listSpecificPage.jsp
<%@ page language="java" contentType="text/html; charset=EUC-KR" pageEncoding="EUC-KR"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">

<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="s" uri="http://www.springframework.org/tags"%>
<%@ page session="false" %>
<%@ page import="kojh.db.beans.BoardBean"%>
<%@ page import="java.util.ArrayList"%>
<%@ page import="java.util.Properties"%>
<%@ page import="java.io.IOException"%>
<%@ page import="java.io.FileInputStream"%>
<%@ page import="kojh.spring.board.PageNumberingManager"%>

<html>
<head>
<title>목록</title>
</head>

<c:set var="current_page" value="${current_page}" />
<c:set var="total_cnt" value="${totalCnt}" />
<%
    int c_page = Integer.parseInt( (String)  (pageContext.getAttribute("current_page") ))  ;
    pageContext.setAttribute("c_page",c_page);
%>

<table cellspacing=1 width=700 border=0>
    <tr>
        <td>총 게시물수: <c:out value="${totalCnt}"/></td>
        <td><p align=right> 페이지:<c:out value="${current_page}"/>
        </td>
    </tr>
</table>

<table cellspacing=1 width=700 border=1>
    <tr>
        <td width=50><p align=center>번호</p>
        </td>
        <td width=100><p align=center>이름</p>
        </td>
        <td width=320><p align=center>제목</p>
        </td>
        <td width=100><p align=center>등록일</p>
        </td>
        <td width=100><p align=center>조회수</p>
        </td>
    </tr>

    <c:forEach var="board" items="${boardList}">
        <tr>
        <td width=50><p align=center>${board.getId()}</p></td>
        <td width=100><p align=center>${board.getName()}</p></td>
        <td width=320>
            <p align=center>
                <a href="/SpringMvcBoardMyBatis/viewWork?memo_id=${board.getId()}&current_page=<c:out value="${current_page}"/>&searchStr=None" title="${board.getMemo()}"><c:out value="${board.getSubject()}"/>
            </p>
        </td>
        <td width=100><p align=center><c:out value="${board.getCreated_date()}"/></p></td>
        <td width=100><p align=center><c:out value="${board.getHits()}"/></p></td>
    </tr>
    </c:forEach>
    <%
        int rowsPerPage = 2;
        int total_cnt = ((Integer)(pageContext.getAttribute("total_cnt"))).intValue()  ;

        //전체 페이지
        int total_pages = PageNumberingManager.getInstance().getTotalPage(total_cnt, rowsPerPage) ;
        pageContext.setAttribute("t_pages",total_pages);
    %>
</table>

<table cellspacing=1 width=700 border=1 >
    <tr>
        <td>
        <c:forEach var="i" begin="1" end="${t_pages}">
            <a href="/SpringMvcBoardMyBatis/listSpecificPageWork?current_page=${i}" >
            [
            <c:if test="${i == c_page}" > <b> </c:if>
            ${i}
            <c:if test="${i == c_page}" > </b> </c:if>
            ]
        </c:forEach>
        </td>
    </tr>
</table>

<table width=700>
    <tr>
        <td><input type=button value="글쓰기"  OnClick="window.location='/SpringMvcBoardMyBatis/show_write_form'">    </td>
        <td><form name=searchf method=post action="/SpringMvcBoardMyBatis/searchWithSubject">
            <p align=right><input type=text name=searchStr size=50  maxlength=50>
            <input type=submit value="글찾기"></p>
        </td>
    </tr>
</table>
</html>
```

다음은 리스트에서 선택시, 글 내용을 보여주는 화면이다.

```jsp
// viewMemo.jsp
<%@ page language="java" contentType="text/html; charset=EUC-KR" pageEncoding="EUC-KR"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<c:set var="str_aid" value="${memo_id}" />
<c:set var="str_c_page" value="${current_page}" />
<c:set var="searchString" value="${searchStr}" />

<html>
<head>
<title>글보기</title>
</head>
<%
    String searchString = (String)  (pageContext.getAttribute("searchString"))  ;
%>

<script language="javascript">
    function boardlist()
    {
        var s = "<%=searchString%>";
        if(s=="None")
            location.href = '/SpringMvcBoardMyBatis/listSpecificPageWork?current_page=${current_page}';
        else
            location.href = '/SpringMvcBoardMyBatis/listSearchedSpecificPageWork?pageForView=${current_page}&searchStr=${searchStr}';
    }

    function boardmodify()
    {
        location.href='/SpringMvcBoardMyBatis/listSpecificPageWork_to_update?memo_id=${memo_id}&current_page=${current_page}';
    }

    function boarddelete()
    {
        location.href='/SpringMvcBoardMyBatis/DeleteSpecificRow?memo_id=${memo_id}&current_page=${current_page}';
    }
</script>

<table cellspacing = 0 cellpadding = 5 border = 1 width=500>
    <tr><td><b>조회수</b></td><td> <c:out value="${boardData.getHits()}"/> </td></tr>
    <tr><td><b>이름 </b></td><td> <c:out value="${boardData.getName()}"/> </td></tr>
    <tr><td><b>이메일 </b></td><td> <c:out value="${boardData.getMail()}"/> </td></tr>
    <tr><td><b>제목 </b></td><td> <c:out value="${boardData.getSubject()}"/> </td></tr>
    <tr><td><b>내용 </b></td><td width=350> <c:out value="${boardData.getMemo()}"/> </td></tr>
</table>

<table  cellspacing = 0 cellpadding = 0 border = 0 width=500>
    <tr><td>
        <input type=button value="수정" OnClick="javascript:boardmodify()">
        <input type=button value="목록" OnClick="javascript:boardlist()">
        <input type=button value="삭제" OnClick="javascript:boarddelete()">
    </td></tr>
</table>
</html>
```

다음은 글작성시 사용되는 화면이다.

```jsp
//writeBoard.jsp

<%@ page language="java" import="java.util.*, java.sql.*, javax.servlet.http.*"
contentType="text/html; charset=EUC-KR"  pageEncoding="EUC-KR"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">

<%@ page import="java.io.*, java.text.*" %>
<%@ taglib prefix="sf" uri="http://www.springframework.org/tags/form"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=EUC-KR">
<title>게시판 글쓰기</title>
</head>

<c:url var="insertUrl" value="/DoWriteBoard" />

<sf:form modelAttribute="boardBeanObjToWrite" method="POST" action="${insertUrl}">
    <table width=400 border=1 cellspacing=0 cellpadding=5>
        <tr>
            <td><b>이름</b></td>
            <td><sf:input path="name" size="50" maxlength="50"/><br />
            <sf:errors    path="name" cssClass="error" /></td>
        </tr>
        <tr>
            <td><b>이메일</b></td>
            <td><sf:input path="mail" size="50" maxlength="50"/><br />
            <sf:errors    path="mail" cssClass="error" /></td>
        </tr>
        <tr>
            <td><b>제목</b></td>
            <td><sf:input path="subject" size="50" maxlength="50"/><br />
            <sf:errors    path="subject" cssClass="error" /></td>
        </tr>
        <tr>
            <td><b>내용</b></td>
            <td><sf:textarea  path="memo" size="200" cssStyle="width:350px;height:100px;" maxlength="200"/><br />
            <sf:errors    path="memo" cssClass="error" /></td>
        </tr>

        <tr>
            <td>
            <input type="submit" value="등록" />
            </td>
        </tr>
    </table>
</sf:form>
```

다음은 글 수정시에 사용되는 화면이다.
BoardBean 객체로 내용을 전달하는데, 이때 id와 현재 페이지등의 부가 정보를 넘기기 위해 hidden 항목을 사용했다. 아마 좀더 좋은 방법이 있을거 같다.

```jsp
//viewMemoForUpdate.jsp
<%@ page language="java" contentType="text/html; charset=EUC-KR"  pageEncoding="EUC-KR"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="sf" uri="http://www.springframework.org/tags/form"%>

<html>
<head>
<script language="javascript">
    function writeCheck()
    {
        var form = document.modifyform;
        if( !form.dbsubject.value )
        {
            alert( "제목을 적어주세요" );
            form.dbsubject.focus();
            return;
        }
        if( !form.dbmemo.value )
        {
            alert( "내용을 적어주세요" );
            form.dbmemo.focus();
            return;
        }
        form.submit();
    }

    function boardlist()
    {
        location.href = '/SpringMvcBoardMyBatis/listSpecificPageWork?current_page=${current_page}';
    }
</script>

<title>글보기</title>
</head>

<c:url var="updateUrl" value="/DoUpdateBoard" />
<sf:form modelAttribute="boardData" method="POST" action="${updateUrl}">
    <table width=400 border=1 cellspacing=0 cellpadding=5>
        <%-- modelAttribute 로 데이터를 넘기는데, 추가적인 정보를 넘기기 위해서 hidden field를 사용함 --%>
        <input type=hidden name="memo_id"  value="${memo_id}">
        <input type=hidden name="current_page"  value="${current_page}">

        <tr>
            <td><b>이름</b></td>
            <%-- 이름은 read only 처리--%>
            <td><sf:input readonly="true" path="name" size="50" maxlength="50" /><br />
            <sf:errors  path="name" cssClass="error" /></td>
        </tr>
        <tr>
            <td><b>이메일</b></td>
            <td><sf:input path="mail" size="50" maxlength="50"/><br />
            <sf:errors  path="mail" cssClass="error" /></td>
        </tr>
        <tr>
            <td><b>제목</b></td>
            <td><sf:input path="subject" size="50" maxlength="50"/><br />
            <sf:errors  path="subject" cssClass="error" /></td>
        </tr>
        <tr>
            <td><b>내용</b></td>
            <td><sf:textarea  path="memo" size="200" cssStyle="width:350px;height:100px;" maxlength="200"/><br />
            <sf:errors  path="memo" cssClass="error" /></td>
        </tr>
    </table>

    <table  cellspacing = 0 cellpadding = 0 border = 0 width=500>
        <tr>
            <td> <input type="submit" value="재등록" />
            <input type=button value="목록" OnClick="javascript:boardlist()">
            </td>
        </tr>
    </table>
</sf:form>
</html>
```

마지막으로 검색시에 리스트를 출력하는 화면이다.

```jsp
//listSearchedPage.jsp

<%@ page language="java" contentType="text/html; charset=EUC-KR" pageEncoding="EUC-KR"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">

<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="s" uri="http://www.springframework.org/tags"%>
<%@ page session="false" %>
<%@ page import="kojh.db.beans.BoardBean"%>
<%@ page import="java.util.ArrayList"%>
<%@ page import="java.util.Properties"%>
<%@ page import="java.io.IOException"%>
<%@ page import="java.io.FileInputStream"%>
<%@ page import="kojh.spring.board.PageNumberingManager"%>

<html>
<head>
<title>목록</title>
</head>


<c:set var="total_cnt" value="${totalCnt}" />
<c:set var="searchString" value="${searchStr}" />
<c:set var="pageForView" value="${pageForView}" />
<%
    int total_cnt = ((Integer)(pageContext.getAttribute("total_cnt"))).intValue()  ;
    String searchStr =(String) pageContext.getAttribute("searchString");
    int rowsPerPage = 2;
    int total_pages = PageNumberingManager.getInstance().getTotalPage(total_cnt, rowsPerPage) ;
    pageContext.setAttribute("t_pages",total_pages);
%>

<table cellspacing=1 width=700 border=0>
    <tr>
        <td>총 게시물수: <c:out value="${totalCnt}"/></td>
        <td><p align=right> 페이지:<c:out value="${t_pages}"/>
        </td>
    </tr>
</table>

<table cellspacing=1 width=700 border=1>
    <tr>
        <td width=50><p align=center>번호</p>
        </td>
        <td width=100><p align=center>이름</p>
        </td>
        <td width=320><p align=center>제목</p>
        </td>
        <td width=100><p align=center>등록일</p>
        </td>
        <td width=100><p align=center>조회수</p>
        </td>
    </tr>
    <c:forEach var="board" items="${searchedList}">
        <tr>
        <td width=50><p align=center>${board.getId()}</p></td>
        <td width=100><p align=center>${board.getName()}</p></td>
        <td width=320>
            <p align=center>
                <a href="/SpringMvcBoardMyBatis/viewWork?memo_id=${board.getId()}&current_page=${pageForView}&searchStr=${searchStr}" title="${board.getMemo()}"><c:out value="${board.getSubject()}"/>
            </p>
        </td>
        <td width=100><p align=center><c:out value="${board.getCreated_date()}"/></p></td>
        <td width=100><p align=center><c:out value="${board.getHits()}"/></p></td>
    </tr>
    </c:forEach>
</table>

<table cellspacing=1 width=700 border=1 >
    <tr>
        <td>
        <c:forEach var="i" begin="1" end="${t_pages}">
            <a href="/SpringMvcBoardMyBatis/listSearchedSpecificPageWork?pageForView=${i}&searchStr=<c:out value="${searchStr}"/>" >
            [
            <c:if test="${i == pageForView}" > <b> </c:if>
            ${i}
            <c:if test="${i == pageForView}" > </b> </c:if>
            ]
        </c:forEach>
        </td>
    </tr>
</table>
<table width=700>
    <tr>
        <td><input type=button value="전체 목록으로 돌아가기"  OnClick="window.location='/SpringMvcBoardMyBatis/'">    </td>
    </tr>
</table>
</html>
```

### 이 프로젝트의 소스

[https://github.com/jeremyko/SpringMvcBoardMyBatis](https://github.com/jeremyko/SpringMvcBoardMyBatis)
