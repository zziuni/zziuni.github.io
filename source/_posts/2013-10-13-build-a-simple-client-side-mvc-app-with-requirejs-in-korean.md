---
layout: post
title: "Build a simple client-side MVC app with RequireJS in korean"
date: 2013-10-12 21:48
comments: true
categories: [javascript, RequireJS, AMD]
---

> 이 글은 @verekia 씨가 작성한 [Build a simple client-side MVC app with RequireJS](http://verekia.com/requirejs/build-simple-client-side-mvc-app-require-js)를 허락을 받아 번역한 글입니다.

웹 개발자라면 흔히 파일 하나로 자바스크립트 코드를 짜기 시작합니다. 그리고 그 코드가 점점 커져서 나중에는 수정하기가 정말 어려워지죠. 이런 문제의 해결하기 위해서 코드를 여러 파일로 쪼갤 수 있습니다.  하지만 그러면 script 태그가 많아지고 다른 파일에서 정의한 함수를 조회하기 위한 글로벌 변수가 많아집니다. 그래서 글로벌 네임스페이스는 지저분해지고, 추가한 js 파일들의 HTTP 요청이  네트워크 대역폭을 차지해서 정작 해당 페이지는 로딩이 느려집니다.

이런 일을 겪었다면 프런트 앤드 코드를 뭔가 다른 방법으로 관리해야 겠다는 필요성을 느끼게 됩니다. 특히 자바스크립트가 수천 라인이 넘는 대형 사이즈의 웹 앱을 제작해야 한다면 더욱 그렇습니다. 유지보수를 쉽게 할 수 있도록 이 모든 문제를 해결할 새로운 방법이 필요합니다. **스크립트 로더**가 바로 이를 위한 새로운 기법입니다. 스크립트 로더들은 웹에서 쉽게 찾을 수 있지만 여기서는 그중에서도 **RequireJS**라는 라이브러리를 보겠습니다.

여러분은 단계별로 따라 하는 튜토리얼을 통해서 RequireJS 기반의 간단한  MVC(Model - View - Controller) 앱 제작법을 배울 겁니다. 스크립트 로더에 대한 사전 지식은 필요 없습니다. 이제 기초부터 살펴 봅시다.

<!-- more -->

## 개 요

### RequestJS란 무엇이고  왜 좋은가

RequireJS는 [AMD](https://github.com/amdjs/amdjs-api/wiki/AMD)(Asynchronous Module Definition)의 구현체입니다. AMD란 모듈을 정의하는 방법과 모듈이 필요할 때 비동기로 로딩하는 방법을 정의한 API 입니다. [제임스 버크(James Burke)](https://twitter.com/#!/jrburke)씨가 개발했는데, 2년간 개발해서 겨우 버전 1.0을 찍었습니다. 여러분은 RequireJS로 Javascript코드를 모듈화 할 수 있고  비동기로 관리하면서 여러파일을 병렬로 다운로드 할 수 있습니다. 스크립트 파일이 필요할 때만 병렬로 로딩되기 때문에, 페이지 로딩 속도는 빨라집니다. 그래서 이게 대단한거죠!

### 프런트 앤드를 위한 MVC?

MVC는 서버 사이트 코드를 구조화하고 모듈로 만들며 유지보수가 용의하도록 돕는 매우 잘 알려진 **디자인 패턴**입니다. 그러면 프런트 앤드에서 MVC를 사용하려면 어떻게 해야 할까요? 자바스크립트에서 이 디자인패턴을 적용할 수 있을 까요? 만약 여러분이 자바스크립트를 단지 애니메이션과 폼 유효성 검사, 100라인이 넘지 않는 간단한 처리를 위해서 사용한다면 MVC를 사용해서 여러분의 스크립트 파일을 구조화할 필요 없습니다. RequireJS도 사용할 필요 없을 겁니다. 하지만, 뷰(view)가 많은 리치 웹 앱을 제작 중이라면 반드시 필요합니다!

### 우리가 만들어 볼 앱

RequireJS를 사용해서 MVC 코드로 구조화하는 감을 잡기 위해서 뷰가 딱 2개인 정말 간단한 앱을 만들어 보겠습니다.

* 사용자 목록을 보여주는 뷰. (name 속성으로 나타낸)
* 사용자를 추가할 수 있는 뷰.

다음은 완성된 모습입니다.

![] (http://farm8.staticflickr.com/7092/7350164136_ff1c1375a3.jpg)

비즈니스 로직이 정~말 간단하기 때문에 여러분은 다른건 신경쓰지 않고 코드 구조화를 이해하는데만  집중 할 수 있습니다. 또한 이 튜토리얼을 읽으면서 같이 짜보기를 강력히 추천합니다. 진짜 간단하거든요. 오래걸리지 않습니다. 여러분이 모듈화 프로그래밍을 해본적이 없거나 RequireJS를 사용해본 적이 없다면,  프로그래밍 실력을 늘리는데 도움이 될겁니다. 후회 안 할테니 꼭 해보세요.

### HTML과 CSS 파일

예제에서 사용할 HTML 마크업입니다.

    <!doctype html>
    <html>
    <head>
        <meta charset="utf-8">
        <title>A simple MVC structure</title>
        <link rel="stylesheet" href="css/style.css">
    </head>
    <body>
        <div id="container">
            <h1>My users</h1>
            <nav><a href="#list">List</a> - <a href="#add">Add</a></nav>
            <div id="app"></div>
        </div>
        <script data-main="js/main" src="js/require.js"></script>
    </body>
    </html>

`nav` 메뉴의 링크는 모든 페이지에서 유지할 앱의 네이게이션이고 `#app` div에서  MVC 애플리케이션의 마술이 일어납니다. `body` 끝에는 RequireJS를 추가했습니다. `script` 태그에 `data-main="js/main"` 이라는 특별한 속성을 추가했는데, 이 속성의 값은 RequireJS가 애플리케이션 전체의 경로 기준점으로 사용합니다.

기본적인 스타일쉬트도 추가해 봅시다.

    #container{
        font-family:Calibri, Helvetica, serif;
        color:#444;
        width:200px;
        margin:100px auto 0;
        padding:30px;
        border:1px solid #ddd;
        background:#f6f6f6;
        -webkit-border-radius:4px;
           -moz-border-radius:4px;
                border-radius:4px;
    }
     
    h1, nav{
        text-align:center;
        margin:0 0 20px;
    }

### OOP 돌아보기. 모듈(module)이란?

자바스크립트 객체 지향 프로프래밍에는 모듈 패턴(Module Pattern)이라는 정말 많이 사용하는 디자인 패턴이 있습니다. 이 패턴은 글로벌 네임스페이스를 지저분하게 하지 않고 객체 안에 메서드와 속성을 캡슐화하기 위해서 사용합니다.(그래서 **모듈**이라고 합니다.) 그리고 자바나 PHP 같은 다른 OOP 언어의 클래스를 흉내내기 위해서 사용하기도 합니다. 다음은 `main.js` 파일에 정의한 `MyMath`라는 간단한 모듈입니다.

    var MyMath = (function(){
     
        // Put your private variables and functions here
     
        return { // Here are the public methods
            add:function(a, b){
                return a + b;
            }
        };
    })();
     
    console.log(MyMath.add(1, 2));

공개 메서드를 리터럴 객체 형태로 정의했지만 불편합니다. 대신에 리빌링 모듈 패턴(Revealing Module Pattern)을 사용해서 비공개 속성과 메서드를 반환 합니다.

    var MyMath = (function(){
     
        // With this pattern you can use the usual function notation:
     
        function add(a, b){
            return a + b;
        }
     
        return {
            add:add // But don't forget to declare it in the returned object!
        };
    })();
     
    console.log(MyMath.add(1, 2));

이 튜토리얼에서는 리빌링 모듈 패턴을 사용할 겁니다.

## RequireJS

### RequireJS로 모듈 정의하기

앞 절에서 우리는 변수 MyMath에 호출할 모듈을 정의했습니다. 모듈을 꼭 이렇게 정의해야 하는건 아닙니다. 이번에는 RequireJS를 사용해 보겠습니다. RequireJS는 유지보수가 용의하도록 자바스크립트 파일을 쪼개는 역할을 합니다. 그래서 `main.js`와 같은 폴더에 `MyMath` 모듈을 정의할 `MyMath.js` 파일을 생성하겠습니다.

    define(function(){
     
        function add(a, b){
            return a + b;
        }
     
        return {
            add:add
        };
    });

MyMath 변수는 사라졌고 `define` 함수의 파라미터로 모듈을 집어넣었습니다. `define`함수는 RequireJS가 제공하는 함수로 인데, 외부에서 모듈에 접근할 수 있도록 합니다.

### main 파일에서 모듈 호출하기

그럼 다시 `main.js`파일로 돌아가겠습니다. RequireJS는 우리가 만든 `MyMath` 모듈을 호출하는데 사용할 `require`라는 함수도 제공합니다. `main.js`는 이렇게 바뀝니다.

    require(['MyMath'], function(MyMath){
     
        console.log(MyMath.add(1, 2)); 
     
    });

`MyMath`를 호출하는 부분은 다음 두 파라미터를 가진 `require` 함수안에 있습니다.

* 첫 파라미터는 로딩할 모듈의 배열입니다.  모듈의 경로는 경로 기준점(HTML파일의 data-main 속성이 생각나나요?) 기준으로 정의됩니다.  `.js` 확장자는 생략합니다.
* 두번째 파라미터는 모듈이 로드될 때 호출할 함수입니다. 모듈은 이 함수의 파라미터로 전달됩니다. 그래서 그냥 모듈명을 파라미터 명으로 하면 됩니다.

자. 그럼 이제 페이지를 리로드 해봅시다... 와우! 축하해요! 여러분은 다른 파일의 모듈을 호출했어요! 그렇습니다. 이건 정말 쉽습니다. 그러면 이제 여러분은 그 무섭다는 MVC 아키텍처를 할 준비가 된겁니다. MVC는 여러분이 정의한 모듈과 정말 비슷하게 동작합니다. 그러므로 당연히 할 수 있습니다.

## MVC 패턴의 구조

> **잠깐**! : 이 튜토리얼에서는 서버 사이드 MVC처럼 컨트롤러 하나에 뷰 하나를 연결합니다. 하지만 프런트 앤드 개발에서는 한 컨트롤러에 여러 뷰를 연결하는건 정말 일반적이며 이런 경우 뷰는 버튼이나 입력필드 같은 시각적인 컨포넌트가 됩니다. Backbone 같은 자바스크립트 MVC 프레임워크가 이같은 다른 접근을 사용합니다만 그건 이 글이 목적하는 바와는 다릅니다. 여기서 제 목표는 실제 사용하는 MVC 프레임워크 전부를 만들자는게 아니라 여러분이 이미 잘 알고 있는 구조들을 통해서 RequireJS가 어떻게 동작하는지를 그려보자는 겁니다.

일단 간단하게 우리 프로젝트의 파일과 폴더부터 생성합시다. 데이터를 표현하는 모델을 사용하려 합니다. 비즈니스 로직은 컨트롤러에서 다루고 그 컨트롤러는 페이지를 렌더링할 특정 뷰를 호출할 겁니다. 그럼 어떻게 될까요? 폴더는 Models, Controllers, Views가 필요하고 두 개의 컨트롤러와 두 개의 뷰, 하나의 모델이 필요합니다.

자바스크립트 폴더구조는 다음과 같습니다.

    * Controllers
        * AddController.js
        * ListController.js
    * Models
        * User.js
    * Views
        * AddView.js
        * ListView.js
    * main.js
    * require.js

전체 구조가 준비되었나요? 좋습니다! 가장 간단한 모델 부터 구현합시다.

## 모델: User.js

이 예제에서  `User`는 `name` 속성 하나를 가진 간단한 클래스입니다.

    define(function(){
     
        function User(name){
            this.name = name || 'Default name';
        }
     
        return User;
    });

`main.js` 파일로 돌아와서, `require`  메서드로 `User`를 사용하게 정의할 수 있습니다. 그리고 예제 목적에 부합하게 사용자 목록을 생성해 보겠습니다.

    require(['Models/User'], function(User){
     
        var users = [new User('Barney'),
                     new User('Cartman'),
                     new User('Sheldon')];
     
        for (var i = 0, len = users.length; i < len; i++){
            console.log(users[i].name);
        }
     
        localStorage.users = JSON.stringify(users);
    });

사용자 배열을 JOSN에서 시리얼라이즈 한 후에 데이터베이스처럼 접근하기 위해서 HTML5 로컬 스토리지에 저장합니다.

![](http://farm8.staticflickr.com/7238/7164952261_79b56e2412.jpg)

> **잠깐** : JSON을 시리얼라이즈하는 stringify와 디시리얼라이즈하는 parse를 IE7에서 사용하려면 폴리필(polyfill)이 필요합니다. 더글라스 클락포트씨의 [Github 레파지토리](https://github.com/douglascrockford/JSON-js)에 보면`json2.js`라고 있습니다.

## 사용자 목록 출력

이제 이렇게 저장한 사용자를 출력할 차례입니다. 이 일을 할 `ListController.js`와 `ListView.js`를 만들겠습니다. 두 컴포넌트는 관련이 있습니다. 그래서 어떤 식으로든 연결이 되야 합니다. 많은 방법이 있습니다만 예제를 단순하게 하기 위해서 여기선 이렇게 하겠습니다. `ListView`는 `render`란 메서드를 갖고, `ListController`는 로컬 스토리지에서 가져온 사용자를 파라미터로 해서 `ListView의 render` 메서드를 호출합니다.  그렇습니다. `ListController`는 `ListView`에 의존합니다.

RequireJS에서는 `require` 메서드처럼 `define` 메서드에서도 의존성 있는 관련 모듈을 배열로 넘길 수 있습니다. 모듈안에서 해당 컨트롤러의 메인 동작으로 놓을 `start` 메서드를 만듭시다. (뭐, `run`이나 `main`같은 이름도 좋습니다.) `ListController.js` 를 봅시다.

    define(['Views/ListView'], function(ListView){
     
        function start(){
            var users = JSON.parse(localStorage.users);
            ListView.render({users:users});
        }
     
        return {
            start:start
        };
    });

여기서는 로컬스토리지의 users를  디시리얼라이징 합니다. 그리고 `render`메서드에 객체로 넘깁니다. 이제 `ListView.js`의 `render` 메서드만 구현하면 끝이군요.

    define(function(){
     
        function render(parameters){
            var appDiv = document.getElementById('app');
     
            var users = parameters.users;
     
            var html = '<ul>';
            for (var i = 0, len = users.length; i < len; i++){
                html += '<li>' + users[i].name + '</li>';
            }
            html += '</ul>';
     
            appDiv.innerHTML = html;
        }
     
        return {
            render:render
        };
    });

`render` 메서드는 `#app` 문서요소에 집어넣을 HTML 문자열을 만들기 위해서 users를 순회작업 하는게 전부입니다.

> **잠깐** : 이런 식으로 자바스크립트 파일에서 HTML을 사용하는건 좋은 생각이 아닙니다. 나중에 수정할 때 너무 힘들거든요. 템플릿 사용을 고려해야 합니다. 템플릿은 HTML 마크업에 데이터를 넣는 훌륭한 방법입니다. 좋은 템플릿 시스템이 정말 많이 있습니다. 예를 들면 [jQuery-tmpl](https://github.com/jquery/jquery-tmpl)이나 [Mustache.js](https://github.com/janl/mustache.js) 가 있습니다. 하지만 이 글의 범위를 벗어나는 주제이고 지금의 구조를 복잡하게 만들겁니다. 여기선 단순하게 갑시다.

이제 `ListController` 모듈을 **실행**할 차례입니다. `main.js` 파일에서 `require` 메서드로 `ListController`를 선언하고 `ListController.start()`를 호출합시다.

    require(['Models/User', 'Controllers/ListController'], function(User, ListController){
     
        var users = [new User('Barney'),
                     new User('Cartman'),
                     new User('Sheldon')];
     
        localStorage.users = JSON.stringify(users);
     
        ListController.start();
    });

페이지를 리로드 하면 멋진 사용자 목록을 볼 수 있습니다.

![](http://farm8.staticflickr.com/7094/7350164284_74cb98faa9.jpg)

와~~! 이겁니다! 여러분도 같이 코딩했다면 축하합니다!

> **잠깐** : 지금은 라우팅 시스템이 없기때문에 실행하고 싶은 컨트롤러는 수동으로만 정의할 수 있습니다. 하지만 곧 정말 간단히 생성할 수 있게 됩니다. 기다리세요. ㅋ

## 사용자 등록하기

이제 목록에 사용자를 등록할수 있으면 좋겠군요. 간단한 텍스트 입력 필드와 버튼를 보여주고 버튼을 클릭하면 로컬 스토리지에 사용자를 추가하는 이벤트 핸들러를 등록할 예정입니다. 앞에서 했던 작업과 비슷한 `AddController`에서 시작합시다. 이 파일은 뷰로 파라미터를 전혀 넘기지 않기때문에 무척 간단합니다.  `AddContoller.js`를 보시죠.

    define(['Views/AddView'], function(AddView){
     
        function start(){
            AddView.render();
        }
     
        return {
            start:start
        };
    });

이어서 뷰 파일 `AddView.js`입니다.

    define(function(){
     
        function render(parameters){
            var appDiv = document.getElementById('app');
            appDiv.innerHTML = '<input id="user-name" /><button id="add">Add this user</button>';
        }
     
        return {
            render:render
        };
    });

이제 여러분의 `main.js`파일에서 `AddController`를 정의해서 다음 뷰를 무사히 가져올 `start` 메서드를 호출할 수 있습니다.

![](http://farm8.staticflickr.com/7218/7164952349_2179e0f982.jpg)

하지만 아직 버튼에 이벤트 연결을 하지 않았으므로 지금의 뷰는 별 의미는 없습니다. 이제 그 작업을 하기전에 한가지 질문을 해봅니다. 클릭 이벤트에 대한 이벤트 로직은 어디에 놓아야 할까요? 뷰일까요? 컨트롤러일까요? 이벤트 리스너 위치가 뷰라고 생각하고 이벤트 비즈니스 로직을 놓는건 상당히 안 좋은 습관입니다. 비록 컨트롤러에 뷰에 있는 div의 ID가 없는게 더 좋으니까 완벽하다 할 수는 없지만 컨트롤러에 이벤트 로직를 놓는게 더 좋습니다.

> **잠깐** : 가장 좋은 방법은 뷰에는 이벤트 리스너 함수가 있고, 그 함수가 컨트롤러나 이벤트 처리 전용 모듈에 있는 비즈니스 로직 메서드를 호출하는 겁니다. 이 방식이 어려운건 아니지만 이 때문에 예제가 복잡해져서 여러분이 포기할길 원치는 않습니다. 연습할 때 시도해도 됩니다.

말한 것 처럼, 이벤트 로직을 컨트롤러에 짜봅시다. `AddController`에 `bindEvents` 함수를 만듭니다. 그리고 뷰가 HTML 렌더링을 끝내면 이 함수를 호출합니다.

    define(['Views/AddView', 'Models/User'], function(AddView, User){
     
        function start(){
            AddView.render();
            bindEvents();
        }
     
        function bindEvents(){
            document.getElementById('add').addEventListener('click', function(){
                var users = JSON.parse(localStorage.users);
                var userName = document.getElementById('user-name').value;
                users.push(new User(userName));
                localStorage.users = JSON.stringify(users);
                require(['Controllers/ListController'], function(ListController){
                    ListController.start();
                });
            }, false);
        }
     
        return {
            start:start
        };
    });

`bindEvents`에서는 `#add` 버튼의 클릭 이벤트에 이벤트 리스너를 추가합니다. (IE의 `attachEvent`를 위해서 만들어둔 함수가 있다면 그걸 써도 되고 jQuery를 써도 됩니다. ) 버튼을 클릭하면, 로컬 스토리지에서 사용자를 users 변수에 문자열을 가져와서 배열로 디시리얼라이즈하고 `#user-name` 입력 필드에 있는 이름으로 새로운 user를 넣습니다. 그리고 갱신된 users 배열을 로컬 스토리지에 다시 저장합니다. `require`로 `ListController`를 가져와서 `start` 메서드를 실행합니다. 그러면 다음 화면을 볼 수 있습니다.

![](http://farm8.staticflickr.com/7218/7164952397_3190c81f3a.jpg)

멋지군요! 여기까지 예제를 같이 짰다면 이제 잠시 쉬어도 됩니다. 커피 한잔 타오고 계속하죠.

## 라우터로 뷰 화면간 이동하기

자 다시 합시다. 우리가 만든 앱은 상당히 멋져보이긴 하지만 실제로는 사용자를 한 명 더 추가하기 위해서  등록 뷰로 이동할 수 없으므로 영 별로입니다. 라우팅(*routing*) 시스템을 빼먹었습니다. 서버 사이드 MVC 프레임워크로 작업해본 적이 있다면 아마도 라우팅이 익숙할 겁니다. URL은 각자 다른 뷰를 불러옵니다. 하지만 우리는 클라이언트 사이드 작업을 하고 있고 서버 사이드와는 확실히 다릅니다. 이와 같은 자바스크립트 단일 페이지 인터페이스에서 해당 앱의 다른 부분으로 이동하려면 URL 해시(hash)를 사용합니다. 우리 경우에는 다음 URL을 입력할 때 두 개의 다른 뷰를 찾을 수 있어야 합니다.

* http://yourlocalhostpath/#list
* http://yourlocalhostpath/#add

이렇게해서 각 페이지를 쉽게 접근할 수 있고 북마크도 가능해 집니다.

> **잠깐** : 파이어폭스, 크롬, 오페라는 HTML5 히스토리 관리(pushState, popState, replaceState) 지원 기능이 있어서 해시를 다루지 않아도 됩니다.


### 브라우저 호환성과 응용

구형 브라우저를 지원해야 한다면 히스토리와 해시 이동 관리가 쉽지 않습니다. 지원하기로 정한 브라우저와 관련해서 여러분이 결정할 수 있는 몇가지 해법이 여기 있습니다.

* [가장 앞선 브라우저](http://caniuse.com/#search=history) : HTML5 history management.
* [대부분의 최신 브라우저](http://caniuse.com/#search=hashchange) : HTML5 hashchange event.
* 구형 브라우저 : hash 변경 감시를 수동으로 해야 합니다.
* 정말 오래된 브라우저 : 수동 감시 + iframe 핵

이중에 구현이 간단한 수동 감시를 구현할 겁니다. n 밀리초 마다 hash가 변경되었는 지를 확인하는게 전부입니다. 변경이 확인되면 필요한 함수를 호출합니다.

> **잠깐** : 이 작업을 해주는 [jQuery 플러그인](https://github.com/cowboy/jquery-hashchange)도 있습니다.

### 라우터와 메인 라우터 순회

`main.js` 다음에는 라우팅 로직을 관리할 `Router.js` 파일을 만듭시다. `Router.js`에서는 경로를 정의하고 URL에 정의된게 없을 때 사용할 기본값을 정의해야 합니다. 우리는 인스턴스에 **해시(hash)**와 관련 **컨트롤러(controller)**를 가진 객체의 단순 배열을 사용할 수 있습니다. URL에 해시가 없을 때 사용할 **기본값(defaultRoute)**도 필요합니다.

    define(function(){
     
        var routes = [{hash:'#list', controller:'ListController'},
                      {hash:'#add',  controller:'AddController'}];
        var defaultRoute = '#list';
        var currentHash = '';
     
        function startRouting(){
            window.location.hash = window.location.hash || defaultRoute;
            setInterval(hashCheck, 100);
        }
     
        return {
            startRouting:startRouting
        };
    });

`startRouting`이 호출되면, URL에 기본 해시 값을 지정합니다. 그리고 `hashCheck`를 반복해서 호출하기 시작합니다. `hashCheck`는 아직 구현하지 않은 함수입니다. `currentHash` 변수는 해시 변경이 감지되면 해시의 현재 값을 저장할 때 사용합니다.

### 해시 변화 확인

이 함수가 `hashCheck`입니다. 100 밀리초마다 호출됩니다.

    function hashCheck(){
        if (window.location.hash != currentHash){
            for (var i = 0, currentRoute; currentRoute = routes[i++];){
                if (window.location.hash == currentRoute.hash)
                    loadController(currentRoute.controller);
            }
            currentHash = window.location.hash;
        }
    }

`hashCheck`는 `currentHash`와 값 비교를 해서 해시가 변경됬는지 확인합니다. 그리고 라우터중 하나와 일치하면 관련 컨트롤러 명으로 `loadController`를 호출합니다.

### 적절한 컨트롤러 로딩

이제, `loadController`는 컨트롤러의 모듈을 로드할 `require` 를 호출해서 해당 모듈의 `start` 함수를 실행합니다.

    function loadController(controllerName){
        require(['Controllers/' + controllerName], function(controller){
            controller.start();
        });
    }

`Router.js`는 결국 이렇게 완성됩니다.

    define(function(){
     
        var routes = [{hash:'#list', controller:'ListController'},
                      {hash:'#add',  controller:'AddController'}];
        var defaultRoute = '#list';
        var currentHash = '';
     
        function startRouting(){
            window.location.hash = window.location.hash || defaultRoute;
            setInterval(hashCheck, 100);
        }
     
        function hashCheck(){
            if (window.location.hash != currentHash){
                for (var i = 0, currentRoute; currentRoute = routes[i++];){
                    if (window.location.hash == currentRoute.hash)
                        loadController(currentRoute.controller);
                }
                currentHash = window.location.hash;
            }
        }
     
        function loadController(controllerName){
            require(['Controllers/' + controllerName], function(controller){
                controller.start();
            });
        }
     
        return {
            startRouting:startRouting
        };
    });

### 새로 만든 라우팅 시스템 적용하기

이제 남은 일은 `main.js` 파일에서 라우터 모듈을 `require`로 요청해서 `startRouting` 함수를 호출하는 일 뿐입니다.

    require(['Models/User', 'Router'], function(User, Router){
     
        var users = [new User('Barney'),
                     new User('Cartman'),
                     new User('Sheldon')];
     
        localStorage.users = JSON.stringify(users);
     
        Router.startRouting();
    });

우리가 만드는 앱의 한 컨트롤러에서 다른 곳으로 이동하려면, 새 컨트롤러의 해시 경로로  현재 `window.hash`를 교체하면 됩니다. 이런 경우 새로운 라우팅 시스템 대신에 아직 수동으로 `AddController`의 `ListController`를 로딩하고 있습니다.

    require(['Controllers/ListController'], function(ListController){
        ListController.start();
    });

이 3줄을 해시 업데이트로 바꿔 버립시다.

    window.location.hash = '#list';

자! 이겁니다. 우리 앱은 이제 응용 가능한 라우팅 시스템을 갖췄습니다. 한 뷰에서 다른 뷰로 이동할 수 있고, 반대로  URL에 원하는 해시를 넣을 수 있습니다. 그러면 정의된 라우터에 일치하는 해시를 찾아서 적절한 컨트롤러를 로딩합니다. 멋지죠?

[동작하는 데모](http://verekia.com/demo/require-js/)는 여기서 보세요.

## 결론

여러분은 프레임워크 없이 완전한 MVC 앱을 만들었으므로 자부심을 가져도 됩니다. 모듈 생성에 필요한 필수 요소만 있는 파일들을 연결하기 위해서 RequireJS를 사용했을 뿐이죠. 그럼 다음은 뭘 해야 할까요? 이 튜토리얼을 보고 여러분이 직접한 이 작은 작업이 맘에 든다면 앱에 기능을 추가해서 우리가 만든 작은 프레임워크를 확장할 수 있습니다. 그러려면 필요한 새로운 기법들이 있습니다. 시도해볼 만한 다음 단계에 대해 몇가지 아이디어가 있습니다.

* 템플릿 시스템을 통합합니다.
* 라우팅 시스템처럼 앱과 직접 관련 없으면서 다른 프로젝트에서 재사용 가능한 부분을 분리해서 작은 확장 라이브러리를 생성합니다.
* 그 라이브러리에서 모델, 뷰, 컨트롤러를 객체로 정의합니다.
* 다양한 소스의 데이터(RESTfull APS, localStorage, IndexedDB 등)를 다룰 새로운 추상 계층을 생성합니다.

이런 DIY 접근은 학습용으로는 훌륭하지만, 현시점에서 실 프로젝트에 적용하기에는 현재 프레임워크 상태가 그리 적합하지는 않습니다. 앞에 나열한 기능을 구현할만큼 부지런하지 않다면, 이미 존재하는 MVC 프레임워크 사용법을 배워 볼 수 있습니다. 가장 인기 있는 MVC 프레임워크입니다.

* [Backbon](http://documentcloud.github.com/backbone/)
* [JavaScriptMVC](http://javascriptmvc.com/)
* [Sproutcore](http://www.sproutcore.com/)
* [ExtJS](http://www.sencha.com/products/extjs/)
* [Knockout](http://knockoutjs.com/)

개인적으로는 최소화 버전이 5kb가 안되서 Backbone를 좋아합니다. 그래서 다음 튜토리얼은 RequireJS와 Backbone를 정말 멋지게 엮어서 사용하는 법에 대해 다룰 예정입니다. 이 내용을 알고 싶으면 [@verekia](http://twitter.com/#!/verekia)를 팔로우 하세요. 그리고 **애디 오스마니(Addy Osmani)**씨의 [대형 jQuery 애플리케이션](http://addyosmani.com/blog/large-scale-jquery/)과 [자바스크립트 모듈](http://addyosmani.com/writing-modular-js/)에 대한 튜토리얼을 읽고 그를 팔로우 하고 RequireJS를 만든 **제임스 퍼크(James Burke)**씨도 팔로우하기를 추천합니다. 두 사람 모두 모듈 기반 자바스크립트 앱의 정보통입니다. 애디 오스마니씨는 [TodoMVC](http://addyosmani.github.com/todomvc/)라는 프로젝트도 시작했습니다. TodoMVC는  간단한 동일 웹 앱을 각기 다른 MVC 프레임워크를 사용해서 만드는 방법을 비교하는 프로젝트입니다. 여러분이 적절한 프레임워크를 선택하는데 도움이 될 겁니다.

오늘은 여기까집니다. 읽어주셔서 감사합니다.


