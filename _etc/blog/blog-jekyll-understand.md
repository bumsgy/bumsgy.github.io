---
title: jekyll 블로그 폴더에 대한 이해
permalink: /docs/blog/jekyll-understand
excerpt: "blog"
id: /blog
tags: ["blog", "github", "jekyll"]
date: 2008-12-14 10:30:00 +0900
last_modified_at: 2020-01-06T17:00:00+09:00
redirect_from:
  - /theme-setup/
toc: true
toc_label: "2020.01.16"
description : jekyll 플러그인의 폴더 구조 이해
---

### 폴더구조
1. _includes 
  페이지 구성 시 include되어지는 파일들이 존재합니다.
2. _layouts
  각 페이지의 layouts을 구성하는 파일들이 존재합니다. \\
  이 폴더 안에 있는 파일들을 열어보면 위 includes 의 파일들을 \\
  include 하고 있는 것을 볼 수 있습니다.
3. _posts
  날짜가 있는 글들이 위치하는 폴더입니다. \\
  실제로 블로그에 글을 적을 때 여기에 파일을 생성하게 됩니디ㅏ. \\
  위에 적었듯이, 이 글들은 날짜와 관련이 있는 글들이 해당 됩니다.
4. _sites
  작성한 파일들이 build 되어 생성되는 폴더입니다. \\
  여기에 있는 파일들을 직접 생성, 수정 등을 하는 것이 아니고, \\
  _posts 등의 폴더에 있는 파일들을 생성, 수정하게 되면 \\
  build등을 통해, 이 폴더 안에 적용되게 되며, \\
  실제 웹 페이지에서는 이 폴더 하위의 내용을 보여주게 됩니다. 
5. _data
  메뉴 설정하는 파일( navigation )과 ui에 대한 설정 파일이 존재합니다. \\
  자신이 원하는 블로그를 만들기 위해서 수정이 이루어지기도 합니다.
6. 



### ruby 설치
jekyll은 ruby를 이용하여 설치되기 때문에, ruby를 먼저 설치해야 합니다. \\
이 부분은 jekyll의 페이지를 참조하여 설치합니다. \\
[jekyll installation](https://jekyllrb-ko.github.io/docs/installation/) ([새창](https://jekyllrb-ko.github.io/docs/installation/){: target="_blank"})

설치 이후, MSYS2를 설치 할 것인지 물어보는데, 설치를 진행해 줍니다. \\
1, 2, 3 모두 진행하는데, 아직 무엇인지 정확하게는 확인을 못 했네요.. ;ㅁ; \\
번호를 입력하지 않으면, 종료됩니다. 

설치가 잘 되어 있는지, cmd창을 열어서, 확인 해 봅니다.
```bash
  ruby -v
```

### jekyll 설치
이제 ruby를 이용하여 jekyll을 설치합니다. \\
이 때, bundler도 같이 설치 해 줍니다. \\
아무래도, jekyll 실행 시에 모듈 의존성 관리를 위해 사용하는 듯합니다. \\

```bash
  gem install bundler jekyll
```

### bundler 설치
자신의 블로그 소스 경로로 들어가 보면, Gemfile이 있습니다. \\
이를 이용하여 bundler install을 진행해야 합니다. \\
하지 않으면, jekyll 로컬 서버가 동작하지 않습니다.

```bash
  bundler install
```

### jekyll 로컬서버 실행
로컬서버를 실행합니다. \\
정상적으로 실행되면, URL과 포트를 알려주니, \\
이를 이용하여 접속하여 자신이 올린 글을 확인합니다.

```bash
  jekyll serve
```

굳이 로컬 서버를 기동하지 않고, 바로 github에 push해도 되겠지만, \\
이 경우, 문법 오류 등으로 글이 올라가지 않을 때에도 빠르게 확인이 어렵습니다. \\
github에 올린 후, github의 setting 화면으로 가서, \\
제대로 publishing 되었는지 봐야하니까요~


### TODO
기존에 소스를 가지고 있을 경우인데, 신규에 대해서도 정리를 해야 겠네요.
