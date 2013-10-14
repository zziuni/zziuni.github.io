---
layout: post
title: "Chrome DevTools Revolutions 2013 in korea"
date: 2013-10-12 23:59
comments: true
categories: [DevTool, Chrome]
---

>이 글은 HTML5Rocks의 [Chrome DevTools Revolutions 2013](http://www.html5rocks.com/en/tutorials/developertools/revolutions2013/#toc-canvas-profiling)을 번역한 글입니다.


## Introduction <a href="" id="introduction">#</a>

웹 애플리케이션의 복잡도와 기능 증가와 함께 Chrome DevTools도 같이 커졌다. Paul Irish이 이와 관련해서 구글 I/O 2013에서 [Chrome DevTools Revolutions 2013](https://www.youtube.com/watch?v=x6qe_kVaBpg)라는 발표를 했다. 웹 앱을 제작하고 테스트하는 방법을 혁신할 수 있는 DevTools의 최신 기능을 볼 수 있다.

<iframe width="560" height="315" src="//www.youtube.com/embed/x6qe_kVaBpg" frameborder="0" allowfullscreen></iframe>

파울의 발표를 못봤다면 위에있는 동영상을 보자.(기다릴테니 보고오삼.) 아니면 다음 기능 목록으로 건너뛰자.

* 개발용 코드 에디터로 DevTools의 [Workspaces](#workspaces)를 사용하자.
* Sass를 사용한다면, DevTools에서 Sass(.scss) 파일을 라이브 에디팅하는 기능에 주목하자. 변경사항이 즉시 페이지에 반영된다.
* 기존에도 안드로이드용 크롬의 페이지를 원격으로 디버깅할 수 있기는 했지만, 이제 [ADB extension](#adb_extension)이 안드로이드 기기와의 연결을 담당한다. 그리고 [Reverse port forwarding](#reverse_port_forwarding)로 안드로이드 장비에서 여러분의 개발 머신의 localhost로 쉽게 연결하자.
* 성능은 항상 웹 앱의 주요 관심사다. 그래서 DevTools은 병목현상 추적을 돕는 몇가지 새로운 기능을 추가했다. CPU 프로파일링을 위한 [Flame Chart](#flame_chart) 시각화, 렌더링 관련 성능 문제와 사용 메모리을 디버깅하기 위한 [몇가지 새로운 도구](#performance_features)를 제공한다.
이 기능들은 크롬 stable 버전 28부터 제공한다.

<!-- more -->


## Workspaces <a href="" id="workspaces">#</a>

로컬 웹서버 리소스를 workspaces를 통해서 디스크의 파일로 매핑할 수 있다. 그래서 DevTools의 Source panel에서 모든 타입의 소스를 수정할 수 있고 그 변경사항은 디스크에 반영된다. 반대로 외부 에디터에서 변경해도 즉시 Source panel에 적용된다.

아래 스크린샷은 workspaces의 예제다. 달력 사이트가 localhost에 떠 있고, Sources panel은 로컬 파일 시스템에 있는 이 사이트의 루트 폴더를 보여준다. 이 폴더 안의 파일을 변경하면 디스트에 저장된다. 스크린샷은 아직 저장되지 않은 변경사항이 Calender.css에 있다. 그래서 asterisk(별표)가 파일명 옆에 붙어있다.

![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/552x327xworkspace-sources-panel.png.pagespeed.ic.A9HdSGEKP_.webp)

Control+S 나 Cmmand+S로 디스크에 저장한다.

그리고 Elements panel에서 문서요소의 스타일을 변경하면, Source panel과 외부 에디터에 반영된다. 하지만 다음 사항에 주의하자.

* DOM은 Elements panel에서 변경해도 저장되지 않는다. 오직 CSS 스타일 변경만 저장된다.
* 별도 CSS 파일로 정의된 CSS 스타일만 변경할 수 있다. element.style 변경이나 인라인 스타일은 디스크에 저장되지 않는다. 인라인 스타일은 Sources panel에서 변경할 수 있다.
* Elements panel에서 CSS 스타일 변경은 바로 저장된다. Control+S나 Command+S를 누를 필요 없다.

![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/620x225xworkspace-elements-panel.png.pagespeed.ic.OoXTC47sJz.webp)



### workspace 폴더 추가하기 <a href="" id="add_workspaces_folder">#</a>

workspaces를 사용하려면 두 단계를 거쳐야 한다. DevTools에서 사용할 로컬 폴더의 콘텐츠를 만들고, [그 폴더를 URL로 매핑](#mapping_workspaces_folder)한다.

새로운 workspace 폴더를 만들기 위해서는

1. DevTools에서 **Settings**를 클릭.
2. **Workspace** 클릭.
3. **Add Folder** 클릭.
4. 프로젝트 소스 파일들이 있는 폴더를 열어서 **Select** 클릭
5. 경고가 뜨면, DevTools가 폴더 접근을 할 수 있도록 **Allow** 클릭

Sources panel에 불러온 소스가 새로운 workspace 폴더로 출력된다. 이제 workspace folder에서 라이브 에디팅을 할 수 있고 그 변경사항은 디스크에 저장된다.

![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/491x198xadded-workspace-folder.png.pagespeed.ic.W_v5Ilwd1n.webp)

### 폴더를 URL과 매핑하기 <a href="" id="mapping_workspaces_folder">#</a>

workspace 폴더를 추가한 후에는 그 폴더를 URL에 매핑할 수 있다. 그러면 크롬이 특정 URL을 불러올 때마다 Source panel에서 URL 콘텐츠에 대한 위치로 workspace 폴더 콘텐츠를 출력한다.

workspace 폴더를 URL로 매핑하려면

1. Source panel에서 workspace 폴더의 파일을 오른쪽 클릭하거나 Control+클릭을 한다.
2. **Map to Newwork Resource**를 선택한다.
![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/415x182xmap-to-resource-menu.png.pagespeed.ic.fFTApZIYmf.webp)
3. 현재 불러온 페이지의 리소스 중에서 일치하는 리소스를 선택하자.
![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/547x280xselect-resource.png.pagespeed.ic.mAuSXJ7N3I.webp)
4. 크롬에서 해당 페이지를 새로 고침 한다.

이렇게 하면, Source panel은 localhost의 소스가 아니라, 여러분의 로컬 workspace 폴더의 콘텐츠를 그냥 바로 보여준다.

![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/462x189xmapped-workspace-folder.png.pagespeed.ic.GQwrmD6ZtB.webp)

>Note: 여전히 localhost 소스를 바라보고 있다면 크롬을 새로 고침 하자.

네트워크 폴더를 workspace 폴더로 연결하는 다른 방법도 있다.

* network reosurce에서 오른쪽 클릭(Control+클릭)을 하고 **Map to File System Resource**를 선택하자.
* DevTools Setting의 Workspace 탭에서 수동으로 매핑을 추가한다.


## Sass debugging (experimental) <a href="" id="sass_debugging">#</a>

이 기능을 이용하면 Sources panel에서 Sass(.scss) 파일을 라이브 에디팅 하고 DevTools을 떠나지않고(새로 고침 없이) 바로 결과를 확인할 수 있다. 여러분이 Sass로 생성된 CSS가 반영된 문서요소를 탐색할 때, Elements penel에는 Sass가 생성한 .css가 아니라 .scss 파일이 출력된다.

![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xsass-debugging.png.pagespeed.ic.jGeiBSbVrl.webp)

링크를 클릭하면 Sources panel에 수정 가능한 SCSS 파일이 뜬다. 맘대로 수정 가능.

![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xsass-sources.png.pagespeed.ic.ugimWXRG6A.webp)

DevTools에서든, 별도 에디터에서든 SCSS 파일을 변경해서 저장하면, Sass 컴파일러가 CSS를 재생성한다. 그리고 DevTools는 그 파일을 다시 로드한다.

>주의: 기술적으로, 이 기능은 소스맵(source map)을 지원하는 모든 CSS 프리프로세서(Less같은)에서 동작해야 하지만 현재는 Sass만 지원한다. 이 기능이 실험단계를 벗어나면, Sass만 지원하는 한계가 없어짐에 따라서 기능명이 바뀔 수 있다.

### Sass debugging 사용하기 <a href="" id="using_sass_debugging">#</a>


이 기능을 쓰려면 [pre-release verson of the Sass compiler](http://sass-lang.com/download.html)이 필요하다. 이 버전은 현재 소스 맵 생성을 지원하는 유일한 버전이다.

    gem install sass -v '>=3.3.0alpha' --pre

DevTools 실험 기능에서 Sass debugging feature를 활성화해야 한다.

1. 크롬에서 **about:flags** 를 연다.
2. **Enable Developer Tools experiments** 를 활성화한다. (한글 버전은 **개발자 도구 실험을 사용합니다**)
3. 크롬 재실행.
4. DevTools Setting을 열고 **Experiments**를 클릭
5. **Support for Sass** 혹은 **Sass stylesheet debugging**을 활성화한다. (브라우저 버전에 따라 살짝 이름이 다르다.)

Sass를 설치하고 나서 Sass 파일 변경을 감지하고 생성된 CSS파일를 위한 소스맵을 생성할 Sass 컴파일러를 실행한다.

    sass --watch --sourcemap sass/style.scss:style.css

Compass를 사용한다면, Sass pre-release 버전이 아직 Compass를 지원하지 않으므로 Sass Debugging과 같이 사용할 수 없다.

### 동작방식  <a href="" id="how_it_works">#</a>


개별 SCSS 소스 파일이 처리되는 동안, Sass 컴파일러는 CSS 파일과 함께 [소스 맵](http://www.html5rocks.com/en/tutorials/developertools/sourcemaps/)파일(.map)을 생성한다. 소스탬 파일은 .scss 파일과 .css 파일의 매핑 정보가 담긴 JSON 파일이다. 각 CSS 파일은 특별한 주석으로 그 안에 자신의 소스맵 파일의 URL을 정의한 힌트를 갖고 있다.

    /*# sourceMappingURL=<url>  */

예를 들어 다음 같은 SCSS 파일이 있다면,

    <!-- styles.scss -->
    $textSize: 26px;
    $fontColor: red;
    $bgColor: whitesmoke;

    h2 {
        font-size: $textSize;
        color: $fontColor;
        background: $bgColor;
    }


Sass는 sourceMappingURL 주석이 달린 CSS 파일과 생성한다.

    <!-- styles.css -->
    h2 {
      font-size: 24px;
      color: orange;
      background-color: darkblue;
    }
    /*# sourceMappingURL=styles.css.map */


다음은 소스맵 파일 샘플이다.

    {
      "version": "3",
      "mappings":"AAKA,EAAG;EACC,SAAS,EANF,IAAI;EAOX,KAAK..."
      "sources": ["sass/styles.scss"],
      "file": "styles.css"
    }

## 안드로이드용 크롬 원격 디버깅을 좀 더 쉽게 하기 <a href="" id="remote_debugging">#</a>

셋업을 통해서 좀 더 쉽게 Android의 Chrome을 원격 디버깅할 수 있는 기능이 두 개 추가됐다. [ADB extension](#adb_extension)과 [revers port worwarding](#reverse_port_forwarding)이다.

ADB extension은 원격 디버깅 설정 과정을 최소화해준다. 장점은 다음과 같다.

* ADB(Android Debug Bridge)가 내장되어있어서 설치할 필요가 없다.
* 컴맨드 라인을 사용하지 않아도 된다.
* ADB 데몬 시작/종료, 연결된 장비 열람이 엄청나게 쉽다.

Reverse port forwarding은 localhost의 서버로 안드로이드용 크롬이 접속하기 쉽게 해준다. 어떤 네트워크 환경은 동일 DNS 트릭이 없으면 접속이 어렵다.

### ADB extension 사용하기 <a href="" id="adb_extension">#</a>


먼저 크롬 웹 스토어에서 [ADB Chrome extension](https://chrome.google.com/webstore/detail/adb/dpngiggdglpdnjdoaefidgiigpemgage)을 설치한다. (**Add a Chrome** 클릭)

> 크롬 웹 스토어의 익스텐션 설치는 Window8에서는 안된다. 설치에 문제가 있다면 대안으로 [Remote Debugging on Android](https://developers.google.com/chrome-developer-tools/docs/remote-debugging)를 보자.

설치 후, 크롬에 회색 처리된 Android 메뉴가 뜬다. ADB를 시작하려면 아이콘을 클릭하고 **Start ADB**를 클릭한다.

![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xadb-menu.png.pagespeed.ic.rCpd-6yJzJ.webp)

ADB가 시작되면, 아이콘이 녹색으로 바뀐다. 그리고 현재 연결된 Android 기기 수가 표시된다.

![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xadb-menu-active.png.pagespeed.ic.QoykBuX1_v.webp)

**View Devices**를 클릭하면 연결된 기기와 그 기기의 Chrome 탭들을 보여주는 **about:inspect** 페이지가 뜬다. DevTools에서 그 탭을 깔려면 "inspect"를 클릭한다.

![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xabout-inspect.png.pagespeed.ic.IB1dutLTXA.webp)

연결기기가 안 보이면, USB연결이 잘 되어있는지, **USB debugging**이 Android 의 Chrome에 설정되어있는지 확인한다. 자세한 소개와 트러블슈팅은 [Remote Debugging on Android](https://developers.google.com/chrome-developer-tools/docs/remote-debugging#enable-usb-debugging)를 참고하자.

### Reverse port forwarding(experimental) <a href="" id="reverse_port_forwarding">#</a>

일반적으로 개발자는 로컬 개발 머신에서 웹 서버를 돌리고, 그 사이트에 Android 장비로 연결하고 싶어한다. 서버와 Android 장비가 동일 네트워크에 있다면 복잡하지 않다. 하지만 사내 네트워크 같은 경우에는 DNS trick을 잘 적용하지 않으면 연결이 불가능한 경우도 있다. 안드로이드용 크롬의 새 기능 중에 이 문제를 간단히 해줄 *reverse port forwarding*라는 기능이 추가되었다. 이 기능은 USB를 통해서 개발 장비의 특정 포트로 트래픽을 보내는 Android 장비의 TCP 포트 리스링을 생성한다.

이 기능을 사용하기 위해서는

* 개발 장비에 크롬 28이상을 설치한다.
* 안드로이드 장비에 안드로이드 용 크롬 베타를 설치한다.
* [Android Debug Bridge](http://developer.android.com/tools/help/adb.html) (ADB Chrome extension 아니면 Android SDK 전체)를 개발 PC에 설치한다.

revers port forwarding를 사용하려면, [Using the ADB extension](#adb_extension)에서 설명한 것 처럼 원격 디버깅을 할 장비가 연결돼야만 한다. 그러고 나서 reverse port forwarding을 활성화하고 여러분의 애플리케이션을 위한 port forwarding 규칙을 추가한다.

먼저 reverse port forwarding을 활성화한다.

1. 개발 PC에서 크롬을 실행한다.
2. **about:flags**에서 **Enable Developer Tools experiments**를 활성화하고 크롬을 재실행한다. (한글 버전은 **개발자 도구 실험을 사용합니다**)
3. **about:inspect**를 연다. 그러면 탭 안에 모바일 장비가 보여야 한다.
4. 사이트 목록 우측에 있는 "inpect"를 클릭한다.
5. DevTools 창이 열린다. 거기서 Setting 패널을 연다.
6. Experiments 를 누르고, **Enable reverse port forwarding**을 클릭한다.
7. DevTools를 닫고, **about:inspect**로 돌아온다.

그러고 나서 port forwarding 규칙을 추가한다.

1. "inspect"를 클릭해서 DevTools를 다시 연다. 그리고 다시 Setting으로 들어간다.
2. **Port Forwarding** 탭을 클릭한다.
3. **Devide port** 필드에 Android 장비에서 크롬이 연결해야 할 포트 번호를 입력한다. (기본값은 8080)
4. **Target** 필드에는 여러분의 개발 PC에서 돌고 있는 웹 앱의 포트 번호를 입력한다.

![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xport-forwarding.png.pagespeed.ic.10P58COCsE.webp)

5. 안드로이드용 크롬에서 **localhost:[device-port-number]**를 열자. [device-port-number]는 **Device port**필드에 입력한 값이다.

그러면 개발 장비에서 가져온 콘텐츠가 보여야 한다.

## 자바스크립트 프로파일링을 위한 Flame chart 시각화 <a href="" id="flame_chart">#</a>


새로운 Flame chart는 시간 흐름에 따른 Javascript 프로세싱의 시각적 표현을 지원한다. Timeline과 Network panel에서 보아온 것과 유사하다.

![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xflame-chart-1.png.pagespeed.ic.BSC62lh_fA.webp)

가로축은 시간 흐름이고 세로축은 콜 스택(call stack)이다. 패널 위쪽에는 전체 레코딩을 보여주는 오버뷰가 있다. 아래 스크린샷에서 보는 것처럼 마우스로 오버뷰 일부를 선택해서 "줌 인"할 수 있다. 하단의 상세 타임라인은 그에 따라 줄어든다.

![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xflame-chart-2.png.pagespeed.ic.1ysnU5qraA.webp)

하단 콜 스택의 상세 뷰는 함수 "불록"의 스택을 표현한다. 위쪽에 쌓인 불록은 그 아래쪽 함수 불록이 호출했다. 불록위에 마우스를 올리면 함수명과 타이밍 데이터가 출력된다.

* **Name** - 함수명
* **Self time** - 이 함수의 해당 호출이 완료되는데 걸린 시간. 내부에서 호출하는 다른 함수들을 제외하고 자기 자신만의 실행시간.
* **Total time** - 이 함수의 해당 호출이 완료되는데 걸린 시간. 내부에서 호출하는 함수들도 포함한다.
* **Aggregated self time** - 레코딩 기간동안 이 함수의 모든 호출에 대한 전체 시간. 이 함수가 호출하는 다른 함수들의 호출 시간은 포함하지 않는다.
* **Aggregated total time** - 이 함수의 모든 호출에 대한 전체 시간. 이 함수가 호출하는 다른 함수들의 호출 시간도 모두 포함한다.


![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xflame-chart-3.png.pagespeed.ic.t48br1oeBT.webp)

함수 불록을 클릭하면 Sources panel에서 그 함수가 있는 자바스크립트 파일이 열린다. 그리고 그 함수가 정의된 라인으로 이동한다.

![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xflame-chart-sources.png.pagespeed.ic.axKNMB9dbW.webp)


Flame chart를 이용하려면

1. DevTools에서 **Profiles** Panel의 **Profiles** 탭을 선택한다.
2. **Record JavaScript CPU profile**를 선택하고 **Start**를 클릭한다.
3. 데이터 수집이 끝나면, **Stop**를 클릭한다.
4. profile 뷰에서 **Flame Chart** 시각화를 선택한다.

![image](http://www.html5rocks.com/en/tutorials/developertools/revolutions2013/flame-chart-menu.png)


## 핵심 성능 측정 도구 다섯가지 <a href="" id="performance_features">#</a>


DevTools에 비약적으로 좋아진 부분은 성능 이슈 조사를 위한 신규 기능들이다.

* 지속적인 페인팅 모드(Continuous painting mode)
* 사각형과 레이어 페인팅 외곽선 보여주기(Showing Paint rectangles and layer borders)
* FPS 계기판(FPS meter)
* 강제로 동기화되는 레이아웃 찾기(Finding forced synchronous layouts, layout thrashing)
* 객체 할당 추적하기(Object allocation tracking)

### 지속적인 페인팅 모드(Continuous painting mode)

Continuous painting mode는 개별 문서요소나 CSS 스타일의 렌더링 비용 식별을 돕는 DevTools Setting의 옵션이다. (Rendering > Enable Continuous page repainting)

일반적으로, 크롬은 레이아웃이나 스타일이 변경될 때만 화면을 페인팅하며, 이마저도 갱신이 필요한 영역만 한다. 하지만 이 옵션을 활성화 하면, 전체 화면이 계속 리페인팅 된다. 그리고 페이지를 페인팅하기 위해서 크롬이 소비한 시간을 페인팅 회수의 범위와 함께 우측 상단에 보여준다. 그래프는 최근 페인팅 횟수의 분포를 보여주고, 히스토그램을 가로지르는 가로선은 16.6ms 를 나타낸다.

![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xpaint-times.png.pagespeed.ic.4GHJaoT9je.webp)

이 기능은 Elements 패널 안에서 DOM 트리를 돌아다니며 개별 문서요소를 숨기거나(현재 선택된 문서요소를 숨기려면 H를 누르면 된다.), 아니면 문서요소의 CSS 스타일을 비활성화할 때 효과를 볼 수 있다. 이런 식으로 한 문서요소나 스타일이 페이지 렌더링에 얼마나 부하를 주는지 볼 수 있다. 만약 렌더링 비용이 있다면, 페이지 페인트 회수가 변경돼서 우측 상단에 출력된다. 단일 문서요소를 숨겼는데, 페인팅 회수를 크게 줄었다면, 여러분은 그 문서요소의 구조나 스타일 개선에 집중해야 한다.

Continuous painting mode를 활성화하려면

1. DevTools Setting을 연다.
2. **General** 탭에서 **Rendering**아래에 **Enable continuous page repainting**을 활성화한다.

>만약 이 옵션이 안 보이면, **about:flags**에 들어가서, **GPU compositing on all pages**를 활성화하고 크롬을 재실행한다.

더 자세한 정보는 [Profiling Long Paint Times with DevTools' Continuous Painting Mode](http://updates.html5rocks.com/2013/02/Profiling-Long-Paint-Times-with-DevTools-Continuous-Painting-Mode)를 참고하자.

### 사각형과 레이어 페인팅 외곽선 보여주기()Showing paint rectangles and layer borders)

화면에서 페인팅 되고 있는 사각형 영역을 시각적으로 보여주는 옵션이다. (Setting > Rendering > Show paint rectangles). 예를 들어, 아래 스크린샷은 CSS 호버 효과가 적용되는 영역을 빨간색 사각형으로 보여주고 있다. 이 정도는 화면대비 작은 영역이라 괜찮다.

![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/paint-rect-1.png.pagespeed.ce.9c4qALXuVJ.png)

개발자는 전체 화면의 리페인팅을 일으키는 디자인이나 개발은 피하기를 원한다. 예를 들어, 다음 스크린샷을 보면, 사용자는 페이지를 스크롤하는 중이다. 페인팅 사각형 하나가 스크롤바를 감싸고 있고, 남은 페이지 영역 전체를 다른 페인팅 사각형이 감싸고 있다. 이 경우 이렇게 되는 원인은 Body의 배경 이미지다. 이미지 위치가 CSS에서 fixed로 지정되어있어서 스크롤마다 전체 페이지를 리페인팅 시킨다.

![image](http://www.html5rocks.com/en/tutorials/developertools/revolutions2013/paint-rect-2.png)

### FPS 계기판(FPS meter)

**FPS meter**는 페이지의 현재 초당 프레임율, 최대/최소 프레임율, 시간당 프레임율을 보여주는 막대 그래프와 프레임율 변화를 보여주는 히스토그램을 출력한다.

![image](http://www.html5rocks.com/en/tutorials/developertools/revolutions2013/fps-meter.png)

FPS 계기판을 보려면

1. DevTools Settings를 연다.
2. **General**를 클릭한다.
3. **Rendering** 아래에, **Force accelerated compositing**과 **Show FPS meter**를 활성화한다.


**about:flag**에서 **FPS counter**를 활성화하고 크롬을 재실행하면, FPS 계기판을 상항 보이게 할 수 있다.

### 강제로 동기화 되는 레이아웃 찾기(Finding forced synchronous layouts(layout thrashing))

렌더링 성능을 최대화하기 위해서, 크롬은 보통 애플리케이션이 요청하는 레이아웃 변경들을 한 번에 적용한다. 그리고 변경 요청을 비동기로 계산하고 렌더링할 레이아웃 전달 스케줄을 잡는다. 하지만 애플리케이션이 레이아웃 관련 프로퍼티를 요청하면(offsetHeight, offsetWidth같은), 크롬은 즉시 강제로 동기화시켜 페이지 레이아웃을 적용한다. 이를 *forced synchronous layouts*이라 부르는데, 이 현상은 중요한 렌더링 성능 저하를 가져올 수 있다. 특히 거대한 DOM 트리가 반복해서 적용될 때, 이를 "layout thrashing"라고 부른다.

Timeline panel에서 녹화를 하면, 관련 녹화물 옆에 노란색 경고 아이콘(![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xwarning-icon.png.pagespeed.ic.kIYn6LmPe4.webp))으로 forced synchronous layouts를 찾아서 알려준다. 그 녹화물에 마우스를 올리면 무효화된 레이아웃(invalidated layout)의 코드와 강요된 레이아웃(forced layout)의 코드에 대한 스택 트레이스가 출력된다.

![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xforced-sync-layout-popup.png.pagespeed.ic.fvGqEI6wkY.webp)


그리고 이 팝업은 레이아웃을 잡는 데 필요한 노드 수(nodes that need layout), 레이아웃을 다시 한 트리의 사이즈(layout tree size), 레이아웃의 유효범위(layout scope), 레이아웃 루트(layout root)도 같이 보여준다.

더 자세한 정보는  [Timeline demo: Diagnosing forced synchronous layouts](https://developers.google.com/chrome-developer-tools/docs/demos/too-much-layout/)를 보자.


### 객체 할당 트랙킹(Object allocation tracking)

Object allocation tracking은 시간별 할당을 보여주는 메모리 프로파일의 새로운 형태다. allocation tracking을 시작하면, DevTools은 시간별로 연속적으로 스냅샷을 쌓는다. allocation 프로파일은 객체가 어디서 생성되고 있는지, 어디서 참조되고 있는지를 보여준다.

![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xallocation-tracker.png.pagespeed.ic.ejjMDH2Qku.webp)

track object allocations를 사용하려면

1. DevTools에서 **Profiles** 탭을 클릭한다.
2. **Record heap allocations**을 선택하고 **Start** 클릭한다.
3. 데이터 수집을 끝냈으면, **Stop recording heap profile**을 클릭한다. (해당 패널 좌측 하단의 빨간 원 아이콘)

## Canvas profiling (experimental)

끝으로 아직 실험적인 기능인 Cavas profile은 canvas 문서요소에 만들어진 WebGL 호출을 녹화하고 재생한다. 개별 WebGL 호출을 관통하며 단계별로 진행해보고 렌더링된 결과를 볼 수 있다. 특별한 호출을 재실행하는 순간을 볼 수 있다.

canvas profile을 사용하려면

1. DevTools setting의 **Experiments**안에 **Canvas profiling**을 활성화한다. (Experiments 탭이 보이지 않으면, **about:flags**에서 **Enable Developer Tools experiments**를 활성화하고 크롬을 재실행한다. 한글 버전은 **개발자 도구 실험을 사용합니다**)
2. **Profiles** panel을 클릭한다.
3. **Capture canvas frame**을 선택하고 **Take snapshot**을 클릭한다.
4. 이제 canvas 프레임을 생성하는데 사용된 호출을 탐색할 수 있다.

![image](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/canvas-profile.png.pagespeed.ce.vmnLpLctAQ.png)












