---
layout: post
title: Ruby on Rails 간단 게시판 다시 만들어보기
date: '2012-09-03T16:04:00.004+09:00'
tags:
    - Ruby on Rails
    - 게시판
modified_time: '2012-09-21T11:46:13.644+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-32924054166468967
blogger_orig_url: https://jeremyko.blogspot.com/2012/09/ruby-on-rails-rails.html
---

예제 소스: https://github.com/jeremyko/RailsBoardSample/tree/moreRailsWay
참고한 문서 : http://ruby.railstutorial.org/ruby-on-rails-tutorial-book

앞서 예제를 좀더 수정해서 약간 더 개선시킨 소스이다. 추가된 부분은 다음과 같다.

-   RESTful routing
-   form_for , 모델을 폼과 연결
-   Partial 사용, 중복 제거
-   Validation check
-   bootstrap-sass

RESTful 제공

rails가 제공하는 resources 메서드 를 호출해서 RESTful한 route를 사용할수 있다.
route.rb 파일을 다음처럼 변경한다.

```ruby
RailsBoard::Application.routes.draw do
    root :to => 'my_rails_board_rows#index'

    #REST-style URI
    resources :my_rails_board_rows

    #matchs...
    match '/listSpecificPageWork',  to: 'my_rails_board_rows#listSpecificPageWork'
    match '/searchWithSubject',  to: 'my_rails_board_rows#searchWithSubject'
    match '/EditViaPostReq',  to: 'my_rails_board_rows#EditViaPostReq'
end
```

이렇게 한줄 추가함으로서 rails에서 처리가능한 route는 다음처럼 구성되게 된다.

< Table 1 RESTful URI >

| request | URI                         | Action  | Purpose                             |
| ------- | --------------------------- | ------- | ----------------------------------- |
| GET     | /my_rails_board_rows        | index   | 모든 게시물들 조회                  |
| GET     | /my_rails_board_rows/1      | show    | id 1의 게시물 조회                  |
| GET     | /my_rails_board_rows/new    | new     | 새로운 게시물 작성을 위한 화면 보임 |
| POST    | /my_rails_board_rows        | create  | 새로운 게시물을 생성 처리           |
| GET     | /my_rails_board_rows/1/edit | edit    | id 1 게시물 수정을 위한 화면 보임   |
| PUT     | /my_rails_board_rows/1      | update  | id 1 게시물 수정 작업 처리          |
| DELETE  | /my_rails_board_rows/1      | destroy | id 1 게시물 삭제 처리               |

게시물 CRUD 처리를 RESTful하게 처리하기 때문에, 이전 예제와 비교시 action이 간단해졌다. 또한 Rails는 url을 간단하게 표시할수 있게끔 helper를 같이 생성해주는데, resources :my_rails_board_rows 의 경우 다음과 같은 helper들이 생성된다.

| helper                            | 호출시 return 되는 값         |
| --------------------------------- | ----------------------------- |
| my_rails_board_rows_path          | /my_rails_board_rows          |
| new_my_rails_board_row_path       | /my_rails_board_rows/new      |
| edit_my_rails_board_row_path(:id) | /my_rails_board_rows/:id/edit |
| my_rails_board_row_path(:id)      | /my_rails_board_rows/:id      |

RESTful 구현을 위해 컨트롤러를 작성해보자

resources 호출로 RSESTful한 접근을 설정한 경우, Rails 는 위에서 표시된 것처럼(예를 들어, my_rails_board_rows/new), 모델명을 기준으로 컨트롤러를 찾게 된다. 즉, my_rails_board_rows_controller.rb를 찾는데, 현재 이 파일이 없으므로 생성해줘야 한다.

rails g controller my_rails_board_rows 을 수행하거나, 직접 파일을 생성해도 된다.
그리고 다음처럼 코딩한다.

-   my_rails_board_rows_controller.rb

class MyRailsBoardRowsController < ApplicationController
include ApplicationHelper

    def index
        @searchStr = 'None'
        @totalCnt = MyRailsBoardRow.all.count
        @totalPageList = getTotalPageList( @totalCnt, rowsPerPage)
        @boardList =
                MyRailsBoardRow.find_by_sql ["select ID,SUBJECT,NAME, CREATED_AT, MAIL,MEMO,HITS \
                    from MY_RAILS_BOARD_ROWS ORDER BY id desc limit %s offset 0", rowsPerPage ]

        render 'listSpecificPageWork'
    end

    def show
        # 내용보기 구현
        @id = params[:id]
        @current_page = params[:current_page]
        @searchStr= params[:searchStr]
        # hits increase
        MyRailsBoardRow.increment_counter(:hits, @id  )
        @rowData = MyRailsBoardRow.find(params[:id])
        render 'viewWork'
    end

    def new
        # 새로운 게시물 생성 구현
        @rowData = MyRailsBoardRow.new
        render 'show_write_form'
    end

    def create
        # 글쓰기 등록 구현
        @rowData = MyRailsBoardRow.new(params[:my_rails_board_row])

        if @rowData.save
          redirect_to '/'
        else
          render 'show_write_form'
        end
    end

    def update
        # 변경내용을 저장 구현
        @id = params[:id]
        @current_page = params[:current_page]
        @searchStr = params[:searchStr]

        # Update DataBase
        @rowData = MyRailsBoardRow.find(params[:id])

        if @rowData.update_attributes(params[:my_rails_board_row])
            # Display Page => POST 요청은 redirection!
            url = '/listSpecificPageWork?current_page=' + @current_page+'&searchStr='+@searchStr
            redirect_to url
        else
            render 'update'
        end
    end

    def destroy
        # 게시물 삭제 구현
        MyRailsBoardRow.find(params[:id]).destroy

        redirect_to my_rails_board_rows_path
    end

    def EditAdditionalParams
        # 내용 수정을 위해 내용 출력 구현
        @id = params[:id]
        @current_page = params[:current_page]
        @searchStr = params[:searchStr]
        @rowData = MyRailsBoardRow.find(params[:id])

        render 'update'
    end

    def searchWithSubject
        @searchStr = params[:searchStr]
        url = '/listSpecificPageWork?searchStr=' + @searchStr +'&current_page=1'
        Rails.logger.debug "***** url:" + url

        uri = URI.encode(url.strip) # 한글로 검색하는 경우를 위해, encode해준다.
        Rails.logger.debug uri

        redirect_to uri
    end

    def listSpecificPageWork

        @searchStr = URI.decode(params[:searchStr]) # 한글로 검색하는 경우를 위해, decode해준다.
        Rails.logger.debug "listSpecificPageWork: @searchStr=" + @searchStr
        @current_page = params[:current_page]
        Rails.logger.debug "!!! searchStr: #{@searchStr}"

        if @searchStr == 'None'
            @totalCnt = MyRailsBoardRow.all.count
            @totalPageList = getTotalPageList( @totalCnt, rowsPerPage)

            #sqlite3
            @boardList =
                MyRailsBoardRow.find_by_sql ["select ID,SUBJECT,NAME, CREATED_AT, MAIL,MEMO,HITS \
                    from MY_RAILS_BOARD_ROWS ORDER BY id desc limit %s offset %s",
                    rowsPerPage, @current_page.to_i ==1 ? 0 : 2*(@current_page.to_i-1) ]
        else
            # 검색처리
            @totalCnt = MyRailsBoardRow.where("subject LIKE ?","%#{@searchStr}%").count()
            @totalPageList = getTotalPageList( @totalCnt, rowsPerPage)
            @boardList =
                MyRailsBoardRow.find_by_sql [
            "select ID,SUBJECT,NAME, CREATED_AT, MAIL,MEMO,HITS from MY_RAILS_BOARD_ROWS where subject like '%%%s%%' ORDER BY id desc
             limit %s offset %s", @searchStr,rowsPerPage, @current_page.to_i ==1 ? 0 : 2*(@current_page.to_i-1) ]
         end
    end

end

이전 예제와 비교해서, RESTful 처리로 인해 이름이 변경되거나 삭제된 method 는 다음과 같다.

show_write_form => new
DoWriteBoard => create
viewWork => show
listSpecificPageWork_to_update => EditViaPostReq
updateBoard => update
DeleteSpecificRow => destroy
listSearchedSpecificPageWork => 삭제됨,listSpecificPageWork로 통합

다소 중복되었던 기능 통합을 위해 listSearchedSpecificPageWork 메서드는 listSpecificPageWork 로 통합되었다. 이전 예제에서는 컨트롤러의 action 명에 맞는 view template 를 자동으로 찾아갈수 있게끔 처리했지만(rails 규칙), 이제 action명이 RESTful 한 것으로 변경되었으므로, render 를 이용해서 명시적으로 뷰를 지정해 준다. 그리고 이전 예제와 비교시 중요한 차이점은 새로운 게시물을 생성할때 모델을 생성해서 넘긴다는 점이다. 즉, @rowData = MyRailsBoardRow.new 이부분이다. 이전 예제에서는 신규 작성 화면에서 넘겨준 내용을 컨트롤러에서 받아서 새로운 모델을 생성해서 저장했었다. 이 부분을 모델을 먼저 생성해서 뷰에 전달하고 돌려받는 식으로 처리할수 있게 변경한 것이다.

그럼 새로운 글을 작성하기 위한 뷰를 정의해보자.
그전에 먼저 listSpecificPageWork.html.erb 파일 내용을 다음처럼 변경한다. 중복되던 소스를 통합한 소스이다.

-   listSpecificPageWork.html.erb

<table cellspacing=1 width=700 border=0>
    <tr>
        <td>총 게시물수: <%=@totalCnt%></td>
        <td><p align=right> 페이지:<%=@current_page%></td>        
    </tr>
</table>

<table cellspacing=1 width=700 border=1>
    <tr>
        <td width=50><p align=center>번호</p></td>
        <td width=100><p align=center>이름</p></td>
        <td width=320><p align=center>제목</p></td>
        <td width=100><p align=center>등록일</p></td>
        <td width=100><p align=center>조회수</p></td>
    </tr>
    
    <% if @boardList.any? %>
        <ul>            
            <%  @boardList.each do |boardRow| %>
            <tr>
            <td width=50><p align=center><%=boardRow.id%></p></td>
            <td width=100><p align=center><%=boardRow.name%></p></td>                
            <td width=320>
                <p align=center>
                    <a href="/my_rails_board_rows/<%=boardRow.id%>?current_page=<%=@current_page%>&searchStr=<%=@searchStr%>" title="<%=boardRow.memo%>"><%=boardRow.subject %>                      
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
            <a href="/listSpecificPageWork?current_page=<%=page%>&searchStr=<%=@searchStr%>" >
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

<% if @searchStr == 'None' %>

<form name="searchf" method="post" action="/searchWithSubject/">
<table width=700>
<tbody>
<tr>
<td valign="center">
<p align="left"><input class="btn btn-large btn-primary" value="글쓰기" onclick="window.location='/my_rails_board_rows/new'"  type="button"></p>
</td>
<td valign="right">  
 <p align="right"><input name="searchStr" size="30" maxlength="50" type="text"></p>  
 </td>
<td valign="center">
<p align="right"><input class="btn btn-large btn-primary" value="글찾기" type="submit"></p>
</td>
</tr>
</tbody>
</table>
</form>
<% else %>
<table width=700>
<tr>
<td><input type=button class="btn btn-large btn-primary" value="전체 목록으로 돌아가기"  OnClick="window.location='/'" >  
 </tr>
</table>
<% end %>

<a href="/my_rails_board_rows/<%=boardRow.id%>?...대신 link_to 를 사용해서 다음 처럼 써도 된다.

<%= link_to boardRow.subject, my_rails_board_row_path(:id => boardRow.id, :current_page=>@current_page, :searchStr=>@searchStr), :method => :get %>

별로 중요한 내용은 없고, 이전 예제에서 중복 소스를 통합한 것뿐이다. 소스상에서 글작성을 위한 Action이 /my_rails_board_rows/new 인것을 확인할수 있을 것이다. 컨트롤러에서는 new액션이 호출되고, show_write_form 뷰가 호출되게 된다. 이제 글작성을 위해 뷰를 수정해보자. show_write_form.html.erb 파일이 이미 존재할것이다. 이 내용을 다음처럼 변경한다.

-   show_write_form.html.erb

<% provide(:title, '게시판 글쓰기') %>

<table cellspacing = 0 cellpadding = 5 border = 1 width=500>

    <%= form_for(@rowData) do |f| %>
        <!-- 여기서부터: 공통적으로 사용이 될법한 부분 -->
        <tr>
            <td><b><%= f.label :name %></b></td>
            <td><%= f.text_field :name %><br /></td>
        </tr>
        <tr>
            <td><b><%= f.label :mail %></b></td>
            <td><%= f.text_field :mail %><br /></td>
        </tr>
        <tr>
            <td><b><%= f.label :subject %></b></td>
            <td><%= f.text_field :subject %><br /></td>
        </tr>
        <tr>
            <td><b><%= f.label :memo %></b></td>
            <td><%= f.text_area :memo %><br /></td>
        </tr>
        <!-- 여기까지: 공통적으로 사용이 될법한 부분 -->

        <table width="150">
            <tr>
                <td>
                    <%= f.submit "등록", class: "btn btn-large btn-primary" %>
                </td>
            </tr>
        </table>
    <% end %>

</table>

2. form_for

이전 예제에서는 직접 form을 구성하는 부분이 있었지만 이제는, 모델(Active Record)의 속성을 가지고 자동적으로 입력폼을 만들어주는 form_for helper method를 사용했다. 즉 다음의 코드는 실제 html에서는 다음으로 변환된다.

<%= f.label :name %>
<%= f.text_field :name %>

------>

<label for="my_rails_board_row_name">Name</label>
<input id="my_rails_board_row_name" name="my_rails_board_row[name]" size="30" type="text" />

또한 form_for는 새로운 form을 생성하게 되는데, 실제 생성되는 html 소스는 다음과 같다.

<form accept-charset="UTF-8" action="/my_rails_board_rows" class="new_my_rails_board_row" id="new_my_rails_board_row" method="post">

흥미로운 점은 Rails가 판단해서 새로운 글 작성시 필요한 URI, method 를 설정해 줬다는 점이다. 즉, 현재 이 뷰가 호출되게 만든 actin은 GET방식- /my_rails_board_rows/new action 이었다. 그리고 글작성 뷰에서 form 이 처리하는 action은 Rails에 의해 POST 방식- /my_rails_board_rows 액션으로 설정 되었다. 이 결과로 RESTful하게 호출되는것은 create action이 될것이다 ( Table 1 RESTful URI 에서처럼). RESTful하게 처리되게끔 Rails 가 자동으로 생성을 해었다.

그럼 다시 글작성 뷰 소스로 돌아가 보자. 곰곰 생각해보니 글 작성과 조회, 수정시에 공통적으로 사용이 될만한 부분이 보인다. 바로 이부분들이다.

<tr>
    <td><b><%= f.label :name %></b></td>       
    <td><%= f.text_field :name %><br /></td>       
</tr>
...

3. partial

rails에서는 중복되는 뷰를 공통적으로 사용할수 있게끔 partial 을 사용할수 있다.
위 소스를 다음처럼 수정한다.

<% provide(:title, '게시판 글쓰기') %>

<table cellspacing = 0 cellpadding = 5 border = 1 width=500>

    <%= form_for(@rowData) do |f| %>
        <%= render 'shared/error_messages' %>
        <% @readOnlyFlag = 0 %>
        <%= render :partial => "rowDataInput", :locals => { :f => f , :readOnlyFlag => @readOnlyFlag } %>

        <table width="150">
            <tr>
                <td>
                    <%= f.submit "등록", class: "btn btn-large btn-primary" %>
                </td>
            </tr>
        </table>
    <% end %>

</table>

rowDataInput 이라는 partial을 사용해서 중복없이 소스를 관리할수 있다. 이를 위해서는 '\_' 로 시작되는 partial 파일을 생성해야 한다. 즉, \_rowDataInput.html.erb 파일이다.

-   \RailsBoard\app\views\my_rails_board_rows_rowDataInput.html.erb

      <tr>
          <td><b><%= f.label :name %></b></td>
          <% if @readOnlyFlag == 1 %>
              <td><%= f.text_field :name,:readonly => true %><br /></td>
          <% else %>
              <td><%= f.text_field :name %><br /></td>            
          <% end %>
      </tr>
      <tr>
          <td><b><%= f.label :mail %></b></td>
          <% if @readOnlyFlag == 1 %>
              <td><%= f.text_field :mail,:readonly => true %><br /></td>
          <% else %>
              <td><%= f.text_field :mail %><br /></td>            
          <% end %>        
      </tr>
      <tr>
          <td><b><%= f.label :subject %></b></td>
          <% if @readOnlyFlag == 1 %>
              <td><%= f.text_field :subject,:readonly => true %><br /></td>
          <% else %>
              <td><%= f.text_field :subject %><br /></td>            
          <% end %>        
      </tr>
      <tr>
          <td><b><%= f.label :memo %></b></td>
          <% if @readOnlyFlag == 1 %>
              <td><%= f.text_area :memo,:readonly => true %><br /></td>
          <% else %>
              <td><%= f.text_area :memo %><br /></td>            
          <% end %>        
      </tr>

약간 수정된 부분이 있다. readOnlyFlag 을 partial에 전달하는 목적은 글작성, 조회, 변경시에 수정가능 여부를 조정하기 위한 목적이다.
그런데 <%= render 'shared/error_messages' %> 이부분은 무었인가?

4. Validation

이것은 입력 유효성 체크를 위해서 사용되었다. rails에서 입력 유효성을 체크하기 위해서는 모델에 validates 호출을 추가해야한다.
\Ruby_Dev\RailsBoard\app\models\my_rails_board_row.rb 파일을 다음처럼 수정한다.

class MyRailsBoardRow < ActiveRecord::Base
attr_accessible :hits, :mail, :memo, :name, :subject

    # 유효성 체크
    before_save { |rowData| rowData.mail = mail.downcase }
    validates :name,  presence: true, length: { maximum: 50 }
    validates :subject,  presence: true, length: { maximum: 50 }
    VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
    validates :mail, presence: true, format: { with: VALID_EMAIL_REGEX }
    validates :memo,  presence: true, length: { maximum: 200 }

end

무엇을 의미하는지는 바로 알수 있을것 같다.
presence: true -> 설정해서 반드시 입력해야하는 값임을 설정하는것이다. 이메일 입력의 경우 유효한 이메일 주소인지 판단시 정규식을 이용할수 있다. 이제 error_messages partial 을 생성한다.

\RailsBoard\app\views\shared_error_messages.html.erb

<% if @rowData.errors.any? %>

  <div id="error_explanation">
    <div class="alert alert-error">
      The form contains <%= pluralize(@rowData.errors.count, "error") %>.
    </div>
    <ul>
    <% @rowData.errors.full_messages.each do |msg| %>
      <li>* <%= msg %></li>
    <% end %>
    </ul>
  </div>
<% end %>
이 partial이 하는 일은 모델에 에러가 있는 경우 모든 에러에 대한 상세 메시지를 화면에 출력해 주는 것이다.

5. bootstrap-sass

이전 예제에서 아무런 CSS처리없이 화면 처리가 되었는데, 이제 약간 화면 디자인을 고려해서 CSS를 적용해보자. bootstrap-sass gem을 설치해서 사용해 보려 한다. Gemfile에 gem 'bootstrap-sass', '2.0.4' 을 추가한다.

source 'https://rubygems.org'

gem 'rails', '3.2.8'
gem 'sqlite3', '1.3.5'
gem 'bootstrap-sass', '2.0.4'

# gem 'ruby-oci8'

# gem 'activerecord-oracle_enhanced-adapter'

group :assets do
gem 'sass-rails', '~> 3.2.3'
gem 'coffee-rails', '~> 3.2.1'  
 gem 'uglifier', '>= 1.0.3'  
end
gem 'jquery-rails'

그리고 bundle install 을 수행해서 설치한다.
그다음 해야 할일은 app/assets/stylesheets/custom.css.scss 파일을 생성하는것이다.

-   custom.css.scss

@import "bootstrap";

$grayMediumLight: #eaeaea;

html {
overflow-y: scroll;
}

body {
padding-top: 60px;
background-color:$grayMediumLight;  
}

section {
overflow: auto;
}

.center {
text-align: center;
}

.center h1 {
margin-bottom: 10px;
}

@mixin box_sizing {
-moz-box-sizing: border-box;
-webkit-box-sizing: border-box;
box-sizing: border-box;
}

/_ forms _/
textarea
{
border: 1px solid #bbb;
width: 100%;
padding: 5px;
height: 100px;
margin-bottom: 5px;
@include box_sizing;

resize: vertical;  
}

input
{
border: 1px solid #bbb;
width: 100%;
padding: 5px;
height: auto;
margin-bottom: 5px;
@include box_sizing;  
}

#error_explanation
{
color: #f00;
ul {
list-style: none;
margin: 0 0 5px 0;
}
}

.field_with_errors
{
@extend .control-group;
@extend .error;  
}

여기까지 작업후, 테스트를 해본다. 서버를 기동시킨후, http://localhost:3000 로 가보면 화면이 약간 이쁘게 변한것을 볼수 있을것이다.

아까 만든 유효성 체크를 위해 글작성 화면으로 가서, 아무 내용없이 등록 버튼을 눌러 보자.

다음 처럼 상세 에러메시지가 보이게 된다.

정상적으로 입력후 등록 버튼을 누르면 저장이 될것이다.

이제 글 조회를 위한 기능을 처리해보자. 메인 뷰에서 글 제목을 선택시, 소스 코드상에는 다음 link로 이동하게 처리하고 있다.

<a href="/my_rails_board_rows/<%=boardRow.id%>?current_page=<%=@current_page%>&searchStr=<%=@searchStr%>" title="<%=boardRow.memo%>"><%=boardRow.subject %>
혹은 다음처럼 작성해도 무방.
<%= link_to boardRow.subject, my_rails_board_row_path(:id => boardRow.id, :current_page=>@current_page, :searchStr=>@searchStr), :method => :get %>

GET방식, /my_rails_board_rows/글id 로 처리되므로, RESTful 처리에 의해, show 액션이 호출될것이다.
게시물 보기를 위한 뷰, viewWork 를 작성해보자.

-   viewWork.html.erb
    <% provide(:title, '글보기'+@rowData.name) %>

<table cellspacing = 0 cellpadding = 5 border = 1 width=500>
    <%= form_for(@rowData) do |f| %>                    
        <% @readOnlyFlag = 1 %>       
        <%= render :partial => "rowDataInput", :locals => { :f => f , :readOnlyFlag => @readOnlyFlag } %>
    <% end %>    
</table>

<table width="150">
    <tr>            
        <td><%= button_to '수정' , {:action => "EditAdditionalParams", :id => @id, :current_page => @current_page, :searchStr => @searchStr} ,class: "btn btn-large btn-primary" %></td>

        <td><%= button_to  "삭제", my_rails_board_row_path(:current_page => @current_page, :searchStr => @searchStr),class: "btn btn-large btn-primary", :confirm => 'Are you sure?', :method=> :delete %></td>

        <td><%= button_to '목록' , {:action => "listSpecificPageWork", :current_page => @current_page, :searchStr => @searchStr} ,class: "btn btn-large btn-primary" %></td>
    </tr>

</table>

글 수정을 위해서 button_to 를 사용해서 EditAdditionalParams 액션을 호출하고 있다. 그런데 RESTful 구현을 위해서는 edit action을 호출해 줘야 할것 같은데 왜 EditAdditionalParams action을 사용하고 있는가? 이것은 edit 처리시, id 뿐 아니라, current_page, searchStr의 정보도 같이 전달해줘야 하는데, RESTful edit를 처리시에는 이러한 추가적인 parameter들을 전달할 방법을 찾을 수 없어서 이렇게 구현을 하였다. 즉 만약, 위에 수정 부분을 다음과 같이 변경한다면,

<td><%= button_to  "수정", edit_my_rails_board_row_path(:id => @id), class: "btn btn-large btn-primary", :method=> :get  %>

이결과로 만들어진 html소스를 보면 다음과 같다.

<td><form action="/my_rails_board_rows/8/edit" class="button_to" method="get"><div><input class="btn btn-large btn-primary" type="submit" value="수정" /></div></form>

RESTful edit가 호출은 되는데 추가로 전달해줘야할 parameter 정보가 누락되게 된다. 그래서 혹시나 하고 다음처럼 해보면,

<td><%= button_to  "수정", edit_my_rails_board_row_path(:id => @id, :current_page=> @current_page, :searchStr => @searchStr), class: "btn btn-large btn-primary" %>

html소스는 다음처럼 만들어지는데,

<td><form action="/my_rails_board_rows/41/edit?current_page=1&amp;searchStr=None" class="button_to" method="post"><div><input class="btn btn-large btn-primary" type="submit" value="수정" /><input name="authenticity_token" type="hidden" value="bczaTatv1RFdtg8bsyrQ5RUELi1bobZhD7ZZAWjj6Dk=" /></div></form>
화면에서 클릭시에는 다음과 같은 오류가 발생한다.

No route matches [POST] "/my_rails_board_rows/41/edit"

즉, RESTful 한 edit를 위해서는 Post방식이 아니라 Get방식을 사용해야 함을 나타낸다. button_to 는 기본적으로 post 를 사용하기 때문이다. 그래서 다음처럼 method를 get으로 설정하고 다시 호출해보면,

<td><%= button_to  "수정", edit_my_rails_board_row_path(:id => @id, :current_page=> @current_page, :searchStr => @searchStr), class: "btn btn-large btn-primary",:method=> :get %>

html소스는 다음처럼 만들어지는데,

<td><form action="/my_rails_board_rows/41/edit?current_page=1&amp;searchStr=None" class="button_to" method="get"><div><input class="btn btn-large btn-primary" type="submit" value="수정3" /></div></form>

보다시피 form action이 RESTful edit 경로에 추가적인 정보를 덧붙인 것으로 만들어져 버리기 때문에, 나머지 current_page, searchStr 정보는 제대로 전달이 되지 않았다.
button_to 사용을 안하고 직접 form 을 만들어서 hidden field 로 처리하면 가능할것이지만 그냥 RESTful routing은 사용안하는것으로 하고, 새로운 EditAdditionalParams action을 만들어서 button_to를 사용해서 GET방식으로 호출하였다. button_to 를 사용해서 RESTful edit path를 호출하면서 id 이외의 인자들을 전달하는 방법은 어떻게 하면 될지..? 아마 내가 알지 못하는 방법이 있을지도 모르겠다.

정리 해서 말하자면...:

-   button_to 를 사용하면 결국 form 이 만들어지기 때문에 id 이외의 추가 정보를 전송하려면 Post방식을 사용해야만 한다.
-   그런데 RESTful edit 처리는 Get방식으로만 처리해야 한다.
-   button_to 와 Get방식을 사용하면 RESTful edit URI로 추가적인 인자를 전송할수 없다.
-   만약 form 을 생성하지 않는 link_to 를 사용한다면 RESTful edit URI 를 사용 할수 있다.

<td><%= link_to "수정", {:action => 'edit',:id => @id, :current_page => @current_page, :searchStr => @searchStr }, class: "btn btn-large btn-primary" %></td>

그럼 글수정을 위한 뷰를 살펴보도록 하자. update.html.erb 파일을 생성하고 다음처럼 수정한다.

-   update.html.erb

<% provide(:title, '글 수정'+@rowData.name) %>

<table cellspacing = 0 cellpadding = 5 border = 1 width=500>

    <%= form_for(@rowData) do |f| %>
        <input type=hidden name=id  value="<%=@id%>">
        <input type=hidden name=current_page  value="<%=@current_page%>">
        <input type=hidden name=searchStr  value="<%=@searchStr%>">

        <%= render 'shared/error_messages' %>
        <% @readOnlyFlag = 0 %>
        <%= render :partial => "rowDataInput", :locals => { :f => f , :readOnlyFlag => @readOnlyFlag } %>

        <table width="150">
            <tr>
                <td>
                    <%= f.submit "재등록", class: "btn btn-large btn-primary" %>
                </td>

                <td>
                    <%= button_to  "목록", {:action => "listSpecificPageWork", :current_page => @current_page, :searchStr => @searchStr} ,class: "btn btn-large btn-primary" %>
                </td>
            </tr>
        </table>

    <% end %>

</table>

글 수정을 위한 RESTful uri 는 다음처럼 되어야 함을 앞서의 표에서 확인할수 있을 것이다.

PUT /my_rails_board_rows/1 update id 1 게시물 수정 작업 처리

그럼 rails form_for를 사용했을때, 실제 html로 변환된 부분에서 이것을 확인해 보도록 하자.

<form accept-charset="UTF-8" action="/my_rails_board_rows/43" class="edit_my_rails_board_row" id="edit_my_rails_board_row_43" method="post"><div style="margin:0;padding:0;display:inline"><input name="utf8" type="hidden" value="&#x2713;" /><input name="_method" type="hidden" value="put" />...

앞서의 delete와 마찬가지로 PUT 요청을 바로 처리할수 없기 때문에 폼의 hidden field를 만들어서 전달하고 있음을 알수 있다. 또한 글 작성 때와 마찬가지로 form 처리시 rails에 의해 적합한 action으로 처리되고 있음도 알수 있다.

이제 기본적인 글작성, 수정, 삭제, 조회, 목록 조회기능이 구현되었다.
