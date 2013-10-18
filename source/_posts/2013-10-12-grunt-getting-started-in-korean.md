---
layout: post
title: "Grunt:Getting Started in korean"
date: 2013-10-13 00:01
comments: true
categories: [Grunt, Javascript, Translate]
banner: false
---


>[Grunt](http://gruntjs.com)은 Javascript Task Runner 입니다. 이 문서는 Grunt [Getting Started](http://gruntjs.com/getting-started) 문서의 번역입니다.

![](http://gruntjs.com/img/grunt-logo.svg)
# Getting started

Grunt와 Grunt 플러그인은 [npm]()으로 설치하고 관리된다. npm은 [Node.js]()의 페키지 메니저 도구다.

Grunt -.4.x는 Node.js 버전이 `>=0.8.0` 이여야 한다.

## Installing the CLI

**Grunt 0.3에서 업그레이드 하는 거라면, [Grunt 0.3 Notes](http://gruntjs.com/getting-started#grunt-0.3-notes)를 봐라.**

Grunt를 사용하려면 먼저 Grunt's Command line interface (CLI)를 설치해야 한다. OSX나 nix, BSD에서는 sudo가, 윈도우즈에서는 administrator 권한이 필요할 수도 있다.

    npm install -g grunt-cli

이 명령어는 여러분의 시스템 경로에 `grunt` 명령어를 설정해서 어느 디렉토리에서나 사용할 수 있게 만든다.

<!-- more -->

주의할 점은 `grunt-cli`는 Grunt task runner를 설치하지 않는다는 것이다. Grunt CLI의 역활은 간단하다. `Gruntflie`이 있는 곳에 설치된 버전의 Grunt를 실행하는 것이다. 즉, 같은 장비에서 여러 버전의 Grunt를 설치할 수 있다.

## How the CLI works
`grunt`실행할 때 마다 node의 `require()`를 사용해서 프로젝트 로컬의 grunt를 실행한다. 그러므로 프로텍트 하위 폴더 어디서든 `grunt`를 실행할 수 있다.

로컬에 인스톨된 Grunt를 찾으면 CLI는 Grunt 라이브러리의 로컬 인스톨본을 불러온다. 이때 `Gruntfile`의 환경설정을 적용하고, 동작시키려고 설정한 task들을 실행한다.

무슨일이 일어나는지 궁금하면 [코드](https://github.com/gruntjs/grunt-cli/blob/master/bin/grunt)를 읽어보자. 겁나 짧다.

## Preparing a new grunt project

특별히 설정을 하지 않은 셋업은 프로젝트에 두 개의 파일을 추가한다. `package.json`과 `Grunfile`이다.

**package.json**: 이 파일은 [npm](https://npmjs.org/)이 npm 모듈로 해당 프로젝트를 출판할 때 사용할 메타데이터 저장을 위해 사용된다. 이 파일의 [devDependencies](https://npmjs.org/doc/json.html#devDependencies)에 여러분의 프로젝트에 필요한 grunt와 Grunt 플러그인들을 나열할 수 있다.

**Gruntfile**: 이 파일의 이름은 `Grunffile.js`이거나 `Grunffile.coffee`이다. task를 설정하거나 정의하고 Grunt 플러그인을 불러오는데 사용한다.

### package.json

`package.json`파일은 `Gruntfile`과 함께 프로젝트 루트 디렉토리에 있어야 하고, 프로텍트 소스에 속해야 한다. `package.json`이 있는 폴더에서 `npm install`를 실행하면 파일안에 dependency 목록의 최신 버전을 인스톨한다.

프로젝트에 `package.json`를 추가하는 방법은 여러가지가 있다.

* 대부분의 [grunt-init](http://gruntjs.com/project-scaffolding#h5o-9) 템플릿은 자동으로 프로젝트 전용  `package.json`파일을 생성한다.
* [npm init] 컴맨드 명령어는 기본 `package.json`을 생성한다.
* 다음 예제를 참고해서 필요한 부분은 [specification](https://npmjs.org/doc/json.html)을 참고해서 확장한다.

```javascript
    {
      "name": "my-project-name",
      "version": "0.1.0",
      "devDependencies": {
        "grunt": "~0.4.1",
        "grunt-contrib-jshint": "~0.6.3",
        "grunt-contrib-nodeunit": "~0.2.0",
        "grunt-contrib-uglify": "~0.2.2"
      }
    }
```


#### Installing Grunt and gruntplugins

Grunt와 플러그인을 `package.json`와 연동해서 설치하는 가장 쉬운 방법은 `npm install <module> --save-dev` 컴맨드를 사용하는 것이다. `<module>`만 설치하지 않고, 자동으로 `package.json`의 [devDependencies](https://npmjs.org/doc/json.html#devDependencies) 항목에 추가된다. 버전은 [tiled version range](https://npmjs.org/doc/json.html#version)를 사용한다.

예를 들면, 다음은 프로젝트에 Grunt 최신버전을 설치하고 devDependencies 항목에 추가한다.

    npm install grunt --save-dev

grunt 프러그인과 다른 node 모듈도 마찬가지다. 이렇게 설치하면 프로젝트의 `package.json`이 갱신된다.

### The Gruntfile

`Gruntfile.js`나 `Grunffile.coffee`파일은 프로젝트 루트 폴더에 있어야 하는 자바스크립트 혹은, 커피스크립트 파일이다. 프로젝트 소스의 일부어야 한다.

`Gruntfile`은 다음 부분들로 구성되어 있다.

* "wrapper" 함수.
* 프로젝트와 task의 환경설정.
* grunt plugin과 task 로딩.
* 사용자 정의 task


#### An sample Gruntfile

다음 `Grunffile`은 `package.json` 파일에서 프로젝트의 메타데이터를 Grunt config로 인포트 하고, [grunt-contrib-uglify]()플러그인의 `uglify` 타스크를 소스코드를 minify하기 위해서 설정했다. 그리고 메타데이터를 이용해서 동적인 베너 주석을 생성한다. 컴맨드 라인에 `grunt` 명령어를 실행하면 기본값으로 `uglify` 타스크가 실행된다.

```javascript
    module.exports = function(grunt) {

      // Project configuration.
      grunt.initConfig({
        pkg: grunt.file.readJSON('package.json'),
        uglify: {
          options: {
            banner: '/*! <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
          },
          build: {
            src: 'src/<%= pkg.name %>.js',
            dest: 'build/<%= pkg.name %>.min.js'
          }
        }
      });

      // Load the plugin that provides the "uglify" task.
      grunt.loadNpmTasks('grunt-contrib-uglify');

      // Default task(s).
      grunt.registerTask('default', ['uglify']);

    };
```

이제 여러분은 `Grunfile`의 전체 모습을 보았다. 이제 컴포넌트 파트를 보자.

#### The "wrapper" function

모든 `Gruntfile`(그리고 플러그인)은 래퍼(wrapper) 함수를 기본 형태로 사용한다. 모든 Grunt 코드는 이 함수 안쪽에 있어야 한다.

```javascript
    module.exports = function(grunt){
        // Do grunt-related things in here
    }
```

#### Project and task configuration

대부분의 Grunt task는 [grunt.initConfig](http://gruntjs.com/grunt#grunt.initconfig) 메서드에서 인자로 전달된 객체의 configuraton 데이터를 참조한다.

위의 예제에서, `grunt.file.readJSON('package.json')` 코드는 grunt config로 `package.json`에 저장된 JOSN 메타데이터를 인포트한다. 그러면 `<% %>` 템플릿 문사열이 config의 모든 프로퍼티를 참조할 수 있다. 파일패스나 파일 목록 같은 configuration 데이터는 반복을 줄이기 위해서 이 방법으로 지정한다.

configuraton 객체안에 임의의 객체를 저장한다면, 여러분의 task가 요청하는 프로퍼티와 충돌하지 않는이상, 무시된다. 그리고 이 파일은 단순 JSON이 아니라 자바스크립트 파일이므로, 유효한 JS코드는 모두 사용가능하다. 필요하면 configuration을 프로그래밍적으로 생성할 수도 있다. ㅌ

다른 task도 마찬가지지만, [grunt-contrib-uglify](http://github.com/gruntjs/grunt-contrib-uglify) 플러그인의 `uglify` task는 configuration을 통해서 같은 이름의 프로퍼티에 설정할 값이 들어오길 기대한다. 여기서는 `banner` 옵션을 지정했다. 그리고 소스파일(src) 하나를 타겟파일(dest) 하나로 미니파이(minify)하는 어그리파이(uglify) 타켓명인 `build`를 지정했다.

```javascript
    // Project configuration.
    grunt.initConfig({
      pkg: grunt.file.readJSON('package.json'),
      uglify: {
        options: {
          banner: '/*! <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
        },
        build: {
          src: 'src/<%= pkg.name %>.js',
          dest: 'build/<%= pkg.name %>.min.js'
        }
      }
    });
```

#### Loading grunt plugins and tasks

자주 사용하는 [concatenation](), [minification](), [linting]()같은 task들은 [grunt plugin]()처럼 사용할 수 있다. 즉, `package.json`의 dependency에 플러그인을 설정하기만 하면, `npm install`을 통해서 이미 설치되었고, 간단한 명령어로 `Gurntfile`에서 사용가능하다.

```javascript
    // Load the plugin that provides the "uglify" task.
    grunt.loadNpmTasks('grunt-contrib-uglify');
```

**주의사항**: `grunt --help` 명령어를 사용하면 사용가능한 taks 목록을 볼 수 있다.

#### Custom tasks

`default` task를 설정하면 기본으로 실행할 task를 하나이상 지정할 수 있다. 예를 들면, 컴맨드 라인에 특정 task를 지정하지 않고 `grunt`를 입력하면 `uglify` task가 실행된다. 이는 실제로는 `grunt uglify`나 `grunt default`와 같다. 배열안에는 여러 task를 지정할 수 있다. (아규먼트는 있을 수도 있고 없을 수도 있다.)

```javascript
    // Default task(s)
    grunt.registerTask('default', ['uglify']);
```

만약 프로젝트에 Grunt 프러그인이 지원하지 않는 task가 필요하다면, `Gruntfile`에 직접 custom task를 만들 수 있다. 예를 들면, 다음 `Gruntfile`은 task configuration을 전혀 활용하지 않는 custom `default` task를 정의했다.

```javascript
    module.exports = function(grunt){

          // A very basic default task.
          grunt.registorTask('default', 'Long some stuff', function() {
              grunt.log.write('Logging some stuff...').ok();
          });
    };
```

프로젝트 전용 task라면 `Gruntfile`안에 정의할 필요없이. 별도의 `.js` 파일로 정의하고 [grunt.loadTask](http://gruntjs.com/grunt#grunt.loadtasks) 메서드로 불러오면 된다.

### Further Reading

* [Installing grunt](http://gruntjs.com/installing-grunt/) 가이드는 인스톨 스펙, 프로덕션, 개발, Grunt와 grunt-cli의 버전에 대한 상세한 정보를 제공한다.
* [Configuring Tasks](http://gruntjs.com/configuring-tasks/) 가이드는 `Gruntfile`에서 task, target, option, file을 설정하는 방법을 상세하게 설명한다. tempaltes, globbing pattern, importing external data도 설명한다.
* [Creating Tasks](http://gruntjs.com/creating-tasks/) 가이드는 Grunt task의 타입간의 차이점 목록을 제공하고, task와 configuration의 샘플을 제공한다.
* custom task나 Grunt 플러그인에 대한 더 많은 정보는 [developer documentation](http://gruntjs.com/grunt)을 확인하자.


### Grunt 0.3 Notes

Grunt 0.3에서 업그레이드 할거라면 전역 `grunt`를 제거해야 한다.

```javascript
    npm install -g grunt
```

*이 안내서는 Grunt 0.4.x를 위해 작성되었다. 하지만 Grunt 0.3.x에서도 유효하다. 다만, "The Gruntfile"절의 플러그인 명과 task configuration 옵션은 다를 수 있다. *

