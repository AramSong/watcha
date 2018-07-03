```ruby
~/workspace $ gem install rails -v 5.0.7
~/workspace $ cd ..
~ $ rails _5.0.7_ new watcha_app
~/watcha_app $ rails g scaffold movies
```

`config/routes.rb`

```ruby
Rails.application.routes.draw do
  root 'movies#index'
  resources :movies
  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
end
```

`Gemfile`

```ruby
#beautify
gem 'bootstrap','~>4.1.1'
#authentication
gem 'devise'
#file upload
gem 'carrierwave'

group :development do
    gem 'rails_db' #추가
end
```

* 오류가 생기기 쉽기 때문에 주석처리

```ruby
#Turbolinks makes navigating your web application faster. Read more: https://github.com/turbolinks/turbolinks

#gem 'turbolinks', '~> 5'

```

`app/assets/javasctips/application.js` : 아래 항목 수정 

```ruby
#삭제
//= require turbolinks
//= require tree . 
#추가
//= require popper
//= require bootstrap
```

`app/assets/stylesheets/application.scss`

```ruby
@import 'bootstrap';
```

`views/layouts/application.html.erb`

```erb
<head>
    <%=csrf_meta_tags%>
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
</head>   
.
.
.
<%= stylesheet_link_tag    'application', media: 'all' %>
    <%= javascript_include_tag 'application' %>
```



## Watcha(영화 정보 저장)

* scaffold
* user authenticate(devise) : user관리를 devise gem으로

```ruby
~/watcha_app $ rails g devise:install
```

1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

     In production, :host should be set to the actual host of your application.

     `config/enviroments/devlopment.rb`

     ```ruby
       # Don't care if the mailer can't send.
       config.action_mailer.raise_delivery_errors = false
     
       config.action_mailer.perform_caching = false
       
       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
     ```

       3. Ensure you have flash messages in app/views/layouts/application.html.erb.
          For example:

            <p class="notice"><%= notice %></p>
            <p class="alert"><%= alert %></p>

     ```ruby
       <body>
         <% flash.each do |k,v| %>
           <div class="alert alert-<%= k %>"><%= v %> </div>
         <% end %>
         <div class="container">
         <%= yield %>
         </div>
       </body>
     ```

     ```ruby
     ~/watcha_app $ rails g devise users
     ```

     `db/migrate/create_movies.rb`

     ```ruby
     class CreateMovies < ActiveRecord::Migration[5.0]
       def change
         create_table :movies do |t|
           t.string   :title
           t.string   :genre
           t.string   :director
           t.string   :actor
           t.string   :image_path
           #사용자가 영화정보 등록
           t.integer  :user_id     # =>t.references :user_id
           
           
           t.text     :description
      
           t.timestamps
         end
       end
     ```

     `model/movie.rb`

     ```ruby
     class Movie < ApplicationRecord
         belongs_to :user
     end
     ```

     `model/user.rb`

     ```ruby
       has_many :movies
     ```

     ```ruby
     ~/watcha_app $ rake db:migrate
     ```

* 로그인

`movies/index.html.erb` : 로그인창 추가 

```erb
<hr>
<%if user_signed_in? %>
  <!--로그인 된 상태 -->
  <%=current_user.email%>
  <%= link_to "로그아웃",destroy_user_sessio_path,method: "delete" %>
<% else %>
  <%= link_to "로그인",new_user_session_path %>
<% end %>
```

```erb
<% @movies.each do |movie| %>
<div class="card" style="width: 18rem;">
  <img class="card-img-top" src="<%=movie.image_path.thumb.url%>" alt="<%=movie.title%>">
  <div class="card-body">
    <h5 class="card-title"><%= movie.title%></h5>
    <p class="card-text">장르:<%=movie.genre%></p>
    <p class="card-text">주연배우:<%=movie.actor%></p>
    <p class="card-text">감독:<%=movie.director%></p>
    <a href="#" class="btn btn-primary">Go somewhere</a>
  </div>
</div>
<% end %>
```



`movies_controller.rb`

```ruby
class MoviesController < ApplicationController
  #로그인된 유저만 볼 수 있도록. 로그인을 안했을 경우, index랑 show만
  before_action :authenticate_user!, except: [:index,:show]
  before_action :set_movie, only: [:show, :edit, :update, :destroy]
```

`views/movies/_form.html.erb`

```erb
  <div class="form-group">
    <%= f.label :title %>
    <%= f.text_field :title, class: "form-control" %>
  </div>
  <div class="form-group">
    <%= f.label :genre %>
    <%= f.text_field :genre, class: "form-control" %>
  </div>
  <div class="form-group">
    <%= f.label :director %>
    <%= f.text_field :director, class: "form-control" %>
  </div>
    <div class="form-group">
    <%= f.label :actor %>
    <%= f.text_field :actor, class: "form-control" %>
  </div>
  <div class="form-group">
    <%= f.label :description %>
    <%= f.text_area :description, class: "form-control" %>
  </div>
    
  <div class="form-group">
    <%= f.label :image_path %>
    <%= f.text_area :image_path, class: "form-control" %>
  </div>
  
```

`movies_cotroller.rb`

```ruby
def create
    @movie = Movie.new(movie_params)
    @movie.user_id = current_user.id		#추가
    respond_to do |format|
      if @movie.save
        format.html { redirect_to @movie, notice: 'Movie was successfully created.' }
        format.json { render :show, status: :created, location: @movie }
      else
        format.html { render :new }
        format.json { render json: @movie.errors, status: :unprocessable_entity }
      end
    end
end

def movie_params          params.require(:movie).permit(:title,:genre,:director,:actor,:description,:image_path)
end
```

에러확인 : `p @instance.errors`

* comment model
* image upload(local)

`Gemfile` ~> carrierwave  : 파일 업로드를 위해 필요.

```ruby
~/watcha_app $ rails g uploader image
```

`uploaders/image_uploader.rb` 생성.

`movie.rb`

```ruby
mount_uploader :image_path,ImageUploader
```

* 썸네일용 이미지 

```ruby
~/watcha_app $ sudo apt-get update
~/watcha_app $ sudo apt-get install imagemagick
```

`Gemfile`

```ruby
gem 'mini_magick'
```

`image_uploader.rb` : 주석 해제 

```ruby
   include CarrierWave::MiniMagick

  # Create different versions of your uploaded files:
  version :thumb do
    process resize_to_fit: [50, 50]
  end
```



-resize to fit

-resize to fill

* 기본적인 js 설명(front)
  * HTML.CSS Selector
  * 선택자 : 
    * getElementsByClassName
    * getElementById
    * getElementsByTagName
    * querySelector	

```ruby
class 	->  .class_name
id		-> #id
name	->  [ ]
<input>	-> input
```

document.getElementById("title");

```html
<h1 id="title">Movies</h1>
```

document.getElementsByTagName("h5")

```html
HTMLCollection(3) [h5.card-title, h5.card-title, h5.card-title]
0
:
h5.card-title
1
:
h5.card-title
2
:
h5.card-title
length
:
3
```

***id를 찾을 때는 element . 즉, 먼저 발견된 하나의 id만 나온다. elements는 발견된 모든 결과를 반환하지만 element는 먼저 발견된 하나의 값만 나온다.* **

document.querySelector(".btn")

```html
<a class="btn btn-primary" href="/movies/1">영화 정보보기</a>
```

=>먼저 발견된 하나의 결과값만 반환. 

document.querySelectorAll(".btn")

```html
NodeList(3) [a.btn.btn-primary, a.btn.btn-primary, a.btn.btn-primary]
```

=>전부 반환

```javascript
var bnt = document.getElementsByClassName("btn");
undefined
bnt
HTMLCollection(3) [a.btn.btn-primary, a.btn.btn-primary, a.btn.btn-primary]
```

console.log()

console.error()

console.dir()

* 이벤트

이벤트 감지 (이벤트 리스너) => 행동(행위)발생(이벤트핸들러)

이벤트 등록 => `요소.onClick = 어떤 행동`

ex ) btns[0].onmouseover  = alert("안녕!");

이벤트 등록하는 방법

​		*  요소 . on이벤트이름 = function(매개변수){}

​		*                  이벤트리스너            이벤트핸들러

* 기본적인 js 설명(front)

> 1. 마우스를 오버하면 '건드리지마'라는 alert()창을 띄운다.
> 2. 3번 되면 '건드리지말라고'라는 alert()창을 띄운다.

    ```
btn[2].onmouseover = function(){
	alert("건드리지마");
	if(btn[2].onmouseover)
	{
		count = count+1;
		if(count==3){
			alert("건드리지말라고!!!");
		}
	}
}
    ```

`movie/index.html.erb`

```erb
<script>
  var btn = documnet.getElementsByClassNmae("btn")[0];
  var count = 0;
  var msg = "나 좀 가만히 놔둘래? 혼자있고 싶어 ㅠㅠ";
  btn.onclick = function(){
    count++;
    if(count > 3){
      msg = ".......오오오오오오오오오집에갈래";
    }
    alert(msg);
  }
</script>
```

* 이벤트 등록하는 방법

* 함수 만들기(function())

  1.  변수에 바로 저장: 함수표현식. 선언되기 이전에 사용할 수 없다.

     ```ruby
     var func = function() {
     	alert("안녕");
     }
     ```

  2. 함수 이름 지정 : 함수선언식.선언되기 이전에도 사용 가능.

  ``` ruby
  function func1(){
  	alert("안녕");
  }
  ```

  ```javascript
  <script>
    func1() //선언되기 이전에 사용할 수 없음. 오류
    func2() //사용가능
  
    //함수표현식
    var func1 = function(){
      alert("안녕 1 ~~");
    }
   //함수선언식
    function func2(){
      alert("안녕 2~~");
    }
  
   var btn = document.getElementsByClassName("btn")[0];
   btn.addEventListener("mouseover",func2()); 
  //함수를 실행할 때, 함수를 열고 닫으면 된다.
  //우리가 원하는건 함수를 불러오는 것.
  //함수 이름만 적어서 사용한다.
  //함수이름() <- 이 형태는 함수의 실행을 의미한다.
  btn.addEventListener("mouseover",func2);
  
  var btn2 = document.getElementsByClassName("btn")[1];
  btn2.onmouseover = func1;
    
  //버튼(요소)에 마우스를 오버(이벤트)했더니 (이벤트리스너)
  //그 위에 글자(요소)들이 갑자기 이상한 글자로 변해버린다(이벤트핸들러).
    var btn = document.getElementsByClassName("btn")[0];
    btn.addEventListener("mouseover",function(){
      var title = document.getElementsByClassName("card-title")[0];
      console.dir(title);
         var str = "";
         str = title.innerText;
      title.innerText = "Don't Touch me....";
    });
   btn.addEventListener("mouseout",function(){
      var title = document.getElementsByClassName("card-title")[0];
      title.innerText = str;
    });
    
  </script>
  ```

  3. 익명함수 

  ```javascript
  btn.addEventListener("mouseover",function(){})
  ```

  *** innerText : 태그 다 떼고, innerHtml: 태그 안 떼고 *** 

  ```html
  <div class="card" data-id="1">여는 태그에는 속성을 줄 수 있다.
       속성명 속성값 
  </div>닫는 태그
  ```

  `document.getElementsByTagName("H1")[0].setAttribute("속성명", "속성값"); `

  ``` javascript
  //버튼에 마우스를 올리면 
    //해당 버튼의 class가 btn btn-danger로 변함
    var btn = document.getElementsByClassName("btn")[0];
    btn.addEventListener("mouseover",function(){
      btn.setAttribute("class","btn btn-danger");
    })
    btn.addEventListener("mouseout",function(){
      btn.setAttribute("class","btn btn-primary");
    }) 
  ```

  

* 이벤트 동작시키기 + jQuery + ajax =>바닐라 스크립트

* 댓글달기 + 수정  +  삭제

* 좋아요 + 별점

* infinity scroll





<금요일>

* template 수업

* 사용자 기능정의 ( 사용자가 사용할만한 기술들)

  * 슬라이드 한 장에 사용자 기능 명세 

  (사용자 입장에서 어떤 기능이 필요한지)

  * 회원가입
  * 메인페이지
  * 게시판 목차
  * 게시판 입력창
  * 게시판 수정창(입력창과 같은 경우 필요 x)
  * 게시글 보기창
  * ...