---
title: github 블로그를 위한 jekyll setting
permalink: /docs/blog/blog-setting-exist
excerpt: "blog"
id: /blog
tags: ["blog", "github", "jekyll"]
date: 2008-12-14 10:30:00 +0900
last_modified_at: 2020-01-06T17:00:00+09:00
redirect_from:
  - /theme-setup/
toc: true
toc_label: "2020.01.06"
description : github 블로그 페이지에 jekyll 플러그인 설치
---

### Github에 repository 생성.


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
