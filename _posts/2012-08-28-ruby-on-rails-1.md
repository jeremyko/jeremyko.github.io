---
layout: post
title: 'Ruby on Rails 간단 게시판 '
date: '2012-08-28T16:59:00.000+09:00'
tags:
    - Ruby on Rails
    - 게시판
modified_time: '2012-09-04T18:05:55.897+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-6780179456494406193
blogger_orig_url: https://jeremyko.blogspot.com/2012/08/ruby-on-rails-1.html
---

요즘 취미로 Ruby on Rails 에 대해 공부하고 있는데, 간단한 게시판 예제를 작성 해보자.

이 예제의 소스는 여기서 받을수 있다.

[https://github.com/jeremyko/RailsBoardSample](https://github.com/jeremyko/RailsBoardSample)

여기서 만들 게시판은 앞서 spring, django 에서 예제로 만든것 과 동일한 기능의 게시판이다.

간단한 기능을 가진 게시판이라서, Rails가 제공해주는 scaffolding 을 사용하면 10분 이내로도 작성이 가능할 수 있지만, 이 예제에서는 scaffolding 을 사용하지 않고 처음부터 작성하는것으로 한다.

그리고 Rails 에서 가능한 여러 기능들 즉, RSpec을 사용한 BDD, REST지원, validation check, coffee script, 뷰에서의 partial 등은 일단 간단 게시판을 먼저 작성한후, 향후 포스팅을 통해서 점차 적용시켜보도록 할것이다.

rails와 ruby 개발 환경은 완료된 상태라고 가정한다.

이 예제에서의 사용된 개발환경은 다음과 같다.

-   ruby 1.9.3p194 (2012-04-20) [i386-mingw32]

-   Rails 3.2.8

그리고 사용할 DB는 기본 Sqlite 로 하기로 한다.

만약, Oracle 사용시에는 그에 따른 약간의 소스 코드 수정이 필요한데, 그것은 주석으로 표시하였다.

### rails application 생성

적정한 폴더상에서 다음 명령을 수행해서 새로운 rails application을 생성한다.

    rails new RailsBoard

rails는 디렉토리 구조를 생성하고, 자동적으로 bundle install 명령을 수행하여
필요한 gem들을 설치한다.

### Gemfile 및 database.yml

이 예제에서는 그냥 sqlite3 을 사용할것이므로 특별히 수정할 필요가 없다.

그런데, 만약 oracle을 사용하고 싶다면, [다음을 참고]({% post_url 2012-08-14-windows-ruby-on-rails-ror-oracle-xe-11g %})해서 두 파일의 내용을 변경하기 바란다.

### 컨트롤러 및 뷰 작성

Rails의 MVC pattern의 흐름을 간단하게 살펴보면,

1. 브라우져의 요청은 rails에 의해 특정 컨트롤러의 메서드로 route 되어진다.

2. 해당 컨트롤러에서는 요청을 모델로 전달한다.

3. 해당 모델은 요청에 따라 데이터베이스에 대한 작업(조회,생성,변경,삭제등)을 수행하고,
   필요한 경우 데이터를 리턴한다.( 예를 들어서 조회)

4. 컨트롤러는 모델로부터 데이터를 받아서 .html.erb의 확장자를 가지는 뷰로 전달한다.

5. 해당 뷰에서는 erb(embedded ruby)를 이용해서 html 형식으로 렌더링한후, 컨트롤러로 리턴한다.

6. 컨트롤러는 이 html결과를 클라이언트(브라우져)에게 돌려주게 된다.

만약 컨트롤러의 메서드에서 아무런 작업을 하지 않는 경우라면 어떻게 처리가 될까?
그렇더라도 연관된 해당 뷰는 항상 호출이 된다 (즉 이런 경우는 정적인 페이지 처리가 된다).

간단한 예제를 빨리 만들어보기 위해서, 이 예제에서는 여러개의 컨트롤러를 사용하지는 않을 것이다. 게시판 기능을 위한 BoardController 하나를 작성하고 그 내부에는 게시판 기능에 대한 요청을 처리할 메서드를 정의할것이다.(이 메서드를 action 이라고도 한다).

게시판 기능을 위한 컨트롤러를 board라고 하고, 처음 화면을 위한 action은 index 라고 한다면, rails g (generate) 명령을 사용하여, 컨트롤러 및 액션, 뷰까지 한번에 생성 가능하다.

    rails g controller board index

board 컨트롤러를 생성하면서, index Action(메서드)도 같이 생성했다.
그럼 동시에, 이에 대한 뷰도 자동으로 생성이 되는데, 바로 RailsBoard\app\views\board\index.html.erb 이다.
뷰의 이름은 action의 이름과 동일하게 생성이 된다. erb 는 embedded ruby 를 나타낸다.

### route 지정

`RailsBoard\config\routes.rb` 에 방금 추가한 컨트롤러,action을 url에 mapping 해줘야 한다.

먼저 route.rb 에서 get "board/index" 는 주석 처리한다.

    # get "board/index"

그리고 root :to =>... 부분을 다음처럼 수정한다.

    root :to => 'board#index'

이제 사이트의 root가 board 컨트롤러의 index action으로 설정이 되었다.

### Model 작성

다음에 해야할일은 데이터베이스 작업을 위한 모델을 정의하는 것이다.

게시판 데이터를 저장하기 위해서 먼저 생각해본 스키마는 다음과 같다. (이번에도, 간단한 예제작성을 위해 테이블 하나만 사용하기로 한다)

    "SUBJECT" : 제목 (text)
    "NAME" : 이름 (text)
    "MAIL" : 메일 (text)
    "MEMO" : 게시물 내용 (text)
    "HITS" : 조회수 (integer)

이 스키마를 기초로, 모델을 생성하기 위해 다음을 수행한다.

    rails g model MyRailsBoardRow subject:string name:string mail:string memo:string hits:integer

성공적으로 수행이 되면, 그 결과로 `\RailsBoard\db\migrate` 폴더 및 폴더 내부에는 migration 파일이 생성된다. (실제 DB 생성/변경이 발생하는것은 아니다)

```ruby
class CreateMyRailsBoardRows < ActiveRecord::Migration
    def change
        create_table :my_rails_board_rows do |t|
            t.string :subject
            t.string :name
            t.string :mail
            t.string :memo
            t.integer :hits
            t.timestamps
        end
    end
end
```

이처럼, DB구조에 변경이 발생할때마다 migration파일을 먼저 생성해야 한다.

이를 통해 우리는 DB를 점진적으로 변경할수 있게 된다. 아울러 변경 이력도 관리 가능하다.

흥미로운 것은 Rails가 만들어준 이름이다.

모델 이름을 단수(MyRailsBoardRow)로 정의했는데, Rails에서는 이것을 복수(CreateMyRailsBoardRows)로 자동으로 변환해서 테이블 생성 메서드를 정의해 준다.

곰곰 생각해보면 이러한 명명법이 말이 된다. 데이터베이스는 집합이므로 즉 복수.. 내가 생성한 모델은 그 집합을 이루는 각각의 요소 하나, 즉 단수..

이제, 이 마이그레이션을 실제 DB에 반영하기 위해서는 다음 명령을 수행한다.

    bundle exec rake db:migrate

성공적으로 수행이 되었다면, 실제 생성된 테이블은 다음과 같다.

#### SQLITE3 경우

```sql
CREATE TABLE "my_rails_board_rows"
("id" INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
"subject" varchar(255),
"name" varchar(255),
"mail" varchar(255),
"memo" varchar(255),
"hits" integer,
"created_at" datetime NOT NULL,
"updated_at" datetime NOT NULL);
```

#### 오라클 경우

```sql
CREATE TABLE "MY_RAILS_BOARD_ROWS"
( "ID" NUMBER(38,0) NOT NULL ENABLE,
"SUBJECT" VARCHAR2(255 CHAR),
"NAME" VARCHAR2(255 CHAR),
"MAIL" VARCHAR2(255 CHAR),
"MEMO" VARCHAR2(255 CHAR),
"HITS" NUMBER(38,0),
"CREATED_AT" DATE NOT NULL ENABLE,
"UPDATED_AT" DATE NOT NULL ENABLE,
PRIMARY KEY ("ID") ENABLE ) ;
```

실제로 테이블이 생성되었는지 확인해 본다.

### 컨트롤러 구현

이제 생성된 컨트롤러에 action을 구현해야 할 차례이다.

먼저 좀 해줘야 할 것이 있는데,컨트롤러들에서 사용할 rowsPerPage 전역 변수를 위해 `\RailsBoard\app\controllers\application_controller.rb` 파일에 다음처럼 추가한다.

```ruby
class ApplicationController < ActionController::Base
    protect_from_forgery
    # rowsPerPage 한 페이지당 표시될 게시물 수
    # 모든 컨트롤러에서 사용가능하게 여기에 정의한다.
    def rowsPerPage
        @rowsPerPage ||= 2
    end
end
```

그리고, 게시물의 페이지처리라던지, 웹페이지의 타이틀 변경같은 몇몇 공통기능 구현을 위해서 ApplicationHelper module에 정의할것이 있다.

`\RailsBoard\app\helpers\application_helper.rb` 파일을 다음 처럼 수정해준다.

```ruby
module ApplicationHelper
    #--------------------------------------------------------------------------# # title 표시
    def full_title(page_title)
        base_title = "RailsBoard"
        if page_title.empty?
            base_title
        else
            "#{base_title} | #{page_title}"
        end
    end

    #--------------------------------------------------------------------------#
    # Paging helper
    def getTotalPageList( total_cnt, rowsPerPage )
        if ((total_cnt % rowsPerPage) == 0)
            total_pages = total_cnt / rowsPerPage;
        else
            total_pages = (total_cnt / rowsPerPage) + 1;
        end

        totalPageList = (1..total_pages).to_a
        #totalPageList = Array (1..total_pages)
    end
end
```

이제 컨트롤러 action을 구현해본다.

`\RailsBoard\app\controllers\board_controller.rb` 파일에 index action을 추가한다.

이 action이 최초 루트 페이지인, 게시판 전체 목록을 보여주는 화면을 처리한다.

```ruby
class BoardController < ApplicationController
    # 모든 helper는 view에서는 사용가능하지만 controller 에서는 명시적으로
    # include 해줘야 한다.
    include ApplicationHelper

    #--------------------------------------------------------------------------#
    def index
        @boardList = MyRailsBoardRow.find(:all, :limit =>rowsPerPage, :order=>'created_at desc')  #(1)
        @totalCnt = MyRailsBoardRow.all.count   #(2)
        @current_page = 1        #(3) 최초 화면이므로 1
        @totalPageList = getTotalPageList(@totalCnt, rowsPerPage) #(4)
    end
end
```

이코드를 살펴보면,

-   모델(MyRailsBoardRow)을 사용해서 rowsPerPage 개만 조회해서 리스트를 돌려준다.
-   전체 게시물 갯수를 구하고
-   현재 페이지 변수를 설정하고 (루트 페이지이므로 1을 설정)
-   ApplicationHelper mixin 의 getTotalPageList를 호출해서 전체 페이지 목록을 구한다.

그리고 이러한 @로 시작되는 instance 변수들은 자동적으로 뷰(`RailsBoard\app\views\board\index.html.erb`) 에서도 사용가능하다.

### 뷰 작성

컨트롤러가 모델을 사용해서, 데이터를 조회한후, 그 리스트를 돌려주었다.

이제 이 리스트를 화면에 출력하기 위해 뷰를 작성해야 한다.

이 화면이 최초 보여지는 전체 게시물 출력 화면이다.

`RailsBoard\app\views\board\index.html.erb` 를 다음처럼 수정한다.

```html
<table cellspacing=1 width=700 border=0>
    <tr>
        <td>총 게시물수: <%= @totalCnt %></td>
        <td><p align=right> 페이지:<%= @current_page %>
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

    <% if @boardList.count > 0 %>
        <ul>
            <%  @boardList.each do |boardRow| %>
            <tr>
            <td width=50><p align=center><%= boardRow.id %></p></td>
            <td width=100><p align=center><%= boardRow.name %></p></td>
            <td width=320>
                <p align=center>
                    <a href="/viewWork?id=<%=boardRow.id%>&current_page=<%= @current_page %>&searchStr=None" title="<%= boardRow.memo%>"><%= boardRow.subject %>
                </p>
            </td>
            <td width=100><p align=center><%= boardRow.created_at %></p></td>
            <td width=100><p align=center><%= boardRow.hits %></p></td>
            </tr>
            <% end %>
        </ul>
    <% else %>
        <p>No Data.</p>
    <% end %>

</table>

<table cellspacing=1 width=700 border=1 >
    <tr>
        <td>
        <%  @totalPageList.each do |page| %>
            <a href="/listSpecificPageWork?current_page=<%=page%>" >
            [
            <% if page == @current_page.to_i %>
                <b>
            <% end %>

            <%=page%>

            <% if page == @current_page.to_i %>
                </b>
            <% end %>
            ]
         <% end %>
        </td>
    </tr>

</table>

<table width=700>
    <tr>
        <td><input type=button value="글쓰기"  OnClick="window.location='/show_write_form'" >  </td>
        <td><form name=searchf method=post action="/searchWithSubject/">
            <p align=right><input type=text name=searchStr size=50  maxlength=50>
            <input type=submit value="글찾기"></p>
        </td>
    </tr>
</table>
```

뷰에서 컨트롤러의 정보를 참조하는 방법을 살펴보면 된다.

### Test

이 뷰가 출력되기 위해서는 한가지 꼭 해줄것이 있는데, **다음 파일을 delete** 해주는 것이다.

    public/index.html

**rails가 기본적으로 출력하는 이것을 삭제해야, 내가 정의한 뷰가 보여진다.**

이제 작성한 컨트롤러와 뷰가 제대로 동작하는지 확인하기 위해서, 어플리케이션 폴더상에서 다음을 실행해서 서브를 띄운다.

    rails s

그리고 http://localhost:3000/ 으로 확인해본다.

![blog-image](/assets/img/20120828-1.jpg)

### 게시판 기능 구현

### 글 쓰기 구현

이제 글쓰기 기능을 구현할 차례이다.

새로운 기능들을 추가하기위해서는 다음과 같은 일정한 절차를 반복해주면 된다.

-   url을 컨트롤러의 action에 mapping

-   action을 구현 3.해당 뷰를 구현.

그럼, 글쓰기를 위한 url, `/show_write_form` 을 컨트롤러의 action에 mapping 해보자.

`RailsBoard\config\routes.rb` 에 다음처럼 mapping 해준다.

```ruby
RailsBoard::Application.routes.draw do
    root :to => 'board#index' # 컨트롤러와 action을 mapping
    match '/show_write_form', to: 'board#show_write_form'
end
```

그리고, action을 구현한다.

`\RailsBoard\app\controllers\board_controller.rb` 파일에 `show_write_form action` 을 추가한다.

```ruby
class BoardController < ApplicationController
    include ApplicationHelper
    def index
        @boardList = MyRailsBoardRow.find(:all, :limit => rowsPerPage, :order=> 'created_at desc')
        @totalCnt = MyRailsBoardRow.all.count
        @current_page = 1
        @totalPageList = getTotalPageList(@totalCnt, rowsPerPage)
    end

    def show_write_form
     #입력폼만을 풀력할것이므로 여기서는 처리할것이 없다.
     # 그러나 만약 RESTful을 적용한다면, 여기서 model객체를 생성해서 전달하고, 데이터를 담아올것이다.
    end

end
```

이제, 뷰를 생성해야 한다.

`\RailsBoard\app\views\board\` 에 새로운 파일, `show_write_form.html.erb` 를 생성하고 내용은 다음처럼 작성해준다.

```html
<% provide(:title, '게시판 글쓰기') %>

<script language="javascript">
    function writeCheck() {
        var form = document.writeform;

        if (!form.name.value) {
            alert('이름을 적어주세요');
            form.name.focus();
            return;
        }
        if (!form.subject.value) {
            alert('제목을 적어주세요');
            form.subject.focus();
            return;
        }
        if (!form.memo.value) {
            alert('내용을 적어주세요');
            form.memo.focus();
            return;
        }

        form.submit();
    }
</script>

<table width="700" border="1" cellspacing="0" cellpadding="5">
    <form name="writeform" method="post" action="/DoWriteBoard">
        <tr>
            <td><b>이름</b></td>
            <td><input type="text" name="name" size="50" maxlength="50" /></td>
        </tr>
        <tr>
            <td><b>이메일</b></td>
            <td><input type="text" name="mail" size="50" maxlength="50" /></td>
        </tr>
        <tr>
            <td><b>제목</b></td>
            <td>
                <input type="text" name="subject" size="50" maxlength="50" />
            </td>
        </tr>
        <tr>
            <td><b>내용</b></td>
            <td><textarea name="memo" cols="50" rows="10"></textarea></td>
        </tr>
    </form>
</table>

<table width="700" border="1" cellspacing="0" cellpadding="0">
    <tr>
        <td>
            <input
                type="button"
                value="등록"
                OnClick="javascript:writeCheck();"
            />
        </td>
    </tr>
</table>
```

만약, rails가 지원하는 RESTful 을 적용하고 form_for 등을 이용하면 위 와는 다른 코딩을 할수도 있다. 이건 추후에 다시 정리해보기로 하고 일단 pass.

이제, 게시물 작성 폼까지 완성이 되었고, 등록 버튼을 클릭시 `/DoWriteBoard` url 처리가 필요하다.

앞서의 기능구현과 동일하게 url mapping및 컨트롤러에서의 구현, 그리고 뷰작성으로 계속 진행될것이다.

`RailsBoard\config\routes.rb` 에 다음처럼 mapping 해준다.

```ruby
RailsBoard::Application.routes.draw do
    root :to => 'board#index'

    # 컨트롤러와 action을 mapping
    match '/show_write_form',  to: 'board#show_write_form'
    match '/DoWriteBoard',  to: 'board#DoWriteBoard'
end
```

그리고, action을 구현한다.

`\RailsBoard\app\controllers\board_controller.rb` 파일에 `DoWriteBoard` action을 추가한다.

```ruby
class BoardController < ApplicationController
    include ApplicationHelper
    def index
    ....생략
    end

    def show_write_form
    end

    def DoWriteBoard
        @rowData = MyRailsBoardRow.new( name: params[:name], mail: params[:mail],
            subject: params[:subject], memo: params[:memo], hits:0)

        @rowData.save

        redirect_to '/'
    end
end
```

DoWriteBoard action에서는 폼이 전달한 인자들을 얻어서, 새로운 모델 객체를 생성한후, 실제 DB에 저장한다. 그리고 루트 화면 으로 전환한다.

잘 처리되는지 확인하기 위해서 웹페이지에서 글쓰기를 몇번 시도해보자.

![blog-image](/assets/img/20120828-2.jpg)

### 페이지 처리 구현

여기서, 페이지 처리를 위한 action 을 정의하려 한다.

사용자가 page 1,2,3 등을 클릭시에 그에 맞는 처리를 해줘야하기 때문이다.

index.html.erb에서 이처리를 위해서 `\listSpecificPageWork` url로 처리하게 해주었다. 이를 구현한다.

`match '/listSpecificPageWork', to: 'board#listSpecificPageWork'` 를 route.rb에 추가한다.

```ruby
RailsBoard::Application.routes.draw do
    root :to => 'board#index' # 컨트롤러와 action을 mapping
    match '/show_write_form', to: 'board#show_write_form'
    match '/DoWriteBoard', to: 'board#DoWriteBoard'
    match '/listSpecificPageWork', to: 'board#listSpecificPageWork'
end
```

`RailsBoard\app\controllers\board_controller.rb` 파일에 `listSpecificPageWork` action을 추가한다.

```ruby
class BoardController < ApplicationController
    include ApplicationHelper

    def index
        ....생략
    end

    def show_write_form
    end

    def DoWriteBoard
        .....생략
    end

    def listSpecificPageWork
        @current_page = params[:current_page]
        @totalCnt = MyRailsBoardRow.all.count
        @totalPageList = getTotalPageList( @totalCnt, rowsPerPage)

        # 페이지를 가지고 범위 데이터를 조회한다 => raw SQL 사용
        #Oracle 사용시
        # @boardList = MyRailsBoardRow.find_by_sql ["SELECT Z.* FROM(SELECT X.*, ceil( rownum / %s ) \
        #     as page FROM ( SELECT ID,SUBJECT,NAME, CREATED_AT, MAIL,MEMO,HITS \
        #         FROM MY_RAILS_BOARD_ROWS  ORDER BY ID DESC ) X ) Z WHERE page = %s", rowsPerPage, @current_page]

        #sqlite3 사용시
        @boardList = MyRailsBoardRow.find_by_sql ["select ID,SUBJECT,NAME, CREATED_AT, MAIL,MEMO,HITS \
                from MY_RAILS_BOARD_ROWS ORDER BY id desc limit %s offset %s",
                rowsPerPage, @current_page.to_i ==1 ? 0 : 2*(@current_page.to_i-1) ]
    end

end
```

listSpecificPageWork action 은 페이지를 전달받아서 해당 페이지 게시물만을 목록에 출력한다.

Oracle에서는 `rownum`을 이용했고, sqlite 에서는 이를 대체하기 위해 `limit ~ offset ~` 을 사용했다.

이제 뷰를 구현한다.

`\RailsBoard\app\views\board\` 에 새로운 파일, `listSpecificPageWork.html.erb` 를 생성하고 내용은 다음처럼 작성해준다.

```html
<table cellspacing=1 width=700 border=0>
    <tr>
        <td>총 게시물수: <%=@totalCnt%></td>
        <td><p align=right> 페이지:<%=@current_page%>
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

    <% if @boardList.any? %>
        <ul>
            <%  @boardList.each do |boardRow| %>
            <tr>
            <td width=50><p align=center><%=boardRow.id%></p></td>
            <td width=100><p align=center><%=boardRow.name%></p></td>
            <td width=320>
                <p align=center>
                    <a href="/viewWork?id=<%=boardRow.id%>&current_page=<%=@current_page%>&searchStr=None" title="<%=boardRow.memo%>"><%=boardRow.subject %>
                </p>
            </td>
            <td width=100><p align=center><%=boardRow.created_at%></p></td>
            <td width=100><p align=center><%=boardRow.hits%></p></td>
            </tr>
            <% end %>
        </ul>
    <% else %>
        <p>No Data.</p>
    <% end %>

</table>

<table cellspacing=1 width=700 border=1 >
    <tr>
        <td>
        <%  @totalPageList.each do |page| %>
            <a href="/listSpecificPageWork?current_page=<%=page%>" >
            [
            <!-- Fixnum 과 String 을 비교하기 위해서 변환 -->
            <% if page == @current_page.to_i %>
                <b>
            <% end %>

            <%=page%>

            <% if page == @current_page.to_i %>
                </b>
            <% end %>
            ]
         <% end %>
        </td>
    </tr>

</table>

<table width=700>
    <tr>
        <td><input type=button value="글쓰기"  OnClick="window.location='/show_write_form'">    </td>
        <td><form name=searchf method=post action="/searchWithSubject/">
            <p align=right><input type=text name=searchStr size=50  maxlength=50>
            <input type=submit value="글찾기"></p>
        </td>
    </tr>
</table>
```

그런데, 이 뷰의 내용은 이미 작성한 `index.html.erb` 와 동일하다.

그래서 기존 작성된 내용에 대해서 약간 수정을 해보기로 한다. index action은 다음처럼 변경 가능하다.

```ruby
def index
    #@boardList = MyRailsBoardRow.find(:all, :limit => rowsPerPage, :order=> 'created_at desc')
    #@totalCnt = MyRailsBoardRow.all.count
    #@current_page = 1
    #@totalPageList = getTotalPageList(@totalCnt, rowsPerPage)

    # listSpecificPageWork 로 redirect 한다.
    url = '/listSpecificPageWork?current_page=1'
    redirect_to url
end
```

그리고 기존의 index.html.erb 은 중복되므로 삭제해버리자.

테스트를 위해서 메인 페이지가 제대로 표시되는지 여부 및 페이지 전환이 정상인지 확인한다.

### 내용보기 구현

그럼 이제 게시물 내용보기 기능을 구현한다.

`match '/viewWork', to: 'board#viewWork'` 를 `route.rb` 에 추가한다.

```ruby
RailsBoard::Application.routes.draw do
    root :to => 'board#index' # 컨트롤러와 action을 mapping
    match '/show_write_form', to: 'board#show_write_form'
    match '/DoWriteBoard', to: 'board#DoWriteBoard'
    match '/listSpecificPageWork', to: 'board#listSpecificPageWork'
    match '/viewWork', to: 'board#viewWork'
end
```

그리고, action을 구현한다.

`\RailsBoard\app\controllers\board_controller.rb` 파일에 `viewWork` action을 추가한다.

```ruby
class BoardController < ApplicationController
    include ApplicationHelper
    def index
    ....생략
    end

    def show_write_form
    end

    def DoWriteBoard
        ....생략
    end

    def listSpecificPageWork
    ....생략
    end

    def viewWork
        @id = params[:id]
        @current_page = params[:current_page]
        @searchStr= params[:searchStr]
        MyRailsBoardRow.increment_counter(:hits, @id ) # hits increase
        @rowData = MyRailsBoardRow.find(params[:id])
    end
end
```

조회수를 증가하기 위해서 모델의 increment_counter 메서드를 이용했다.

그리고, 내용보기 화면에서 사용할 모델 객체 `@rowData` 를 모델의 find class 메서드를 이용해서 구한다.

그럼 다음은, view를 추가해야한다. `\RailsBoard\app\views\board\` 에 새로운 파일, `viewWork.html.erb` 를 생성하고 내용은 다음처럼 작성해준다.

```html
<% provide(:title, '글보기'+@rowData.name) %>

<script language="javascript">
    function boardlist() {
        var s = '<%=@searchStr%>';

        if (s == 'None')
            location.href =
                '/listSpecificPageWork?current_page=<%=@current_page%>';
        else
            location.href =
                '/listSearchedSpecificPageWork?pageForView=<%=@current_page%>&searchStr=<%=@searchStr%>';
    }
    function boardmodify() {
        location.href =
            '/listSpecificPageWork_to_update?id=<%=@id%>&current_page=<%=@current_page%>&searchStr=<%=@searchStr%>';
    }
    function boarddelete() {
        location.href =
            '/DeleteSpecificRow?id=<%=@id%>&current_page=<%=@current_page%>';
    }
</script>

<table cellspacing="0" cellpadding="5" border="1" width="500">
    <tr>
        <td><b>조회수</b></td>
        <td><%=@rowData.hits%></td>
    </tr>
    <tr>
        <td><b>이름 </b></td>
        <td><%=@rowData.name%></td>
    </tr>
    <tr>
        <td><b>이메일 </b></td>
        <td><%=@rowData.mail%></td>
    </tr>
    <tr>
        <td><b>제목 </b></td>
        <td><%=@rowData.subject%></td>
    </tr>
    <tr>
        <td><b>내용 </b></td>
        <td width="350"><%=@rowData.memo%></td>
    </tr>
</table>

<table cellspacing="0" cellpadding="0" border="0" width="500">
    <tr>
        <td>
            <input
                type="button"
                value="수정"
                OnClick="javascript:boardmodify()"
            />
            <input
                type="button"
                value="목록"
                OnClick="javascript:boardlist()"
            />
            <input
                type="button"
                value="삭제"
                OnClick="javascript:boarddelete()"
            />
        </td>
    </tr>
</table>
```

여기까지 완료후 테스트를 해보자,. 게시물중 하나를 선택하면 내용이 보일것이다.

이제 남은 기능은 게시물 수정, 삭제, 검색, 목록으로 돌아가기 등인데, 여기까지 코딩하다보면, 현재까지 매우 루틴한 작업이 반복되고 있음을 알수 있다.

그래서 남은 기능구현은 소스코드를 참고해도 충분할것 같아 여기서 이만 마무리 할까 한다.

향후에는 동일한 게시판 만들기를 주제로, Rails 가 제공하는 여러 기능들을 잘 활용해서 다시 작성해보기로 하자.

### 2012-09-04

다시 작성된 게시판 예제는 [다음 글]({% post_url 2012-09-03-ruby-on-rails-rails %})을 참조바람.
