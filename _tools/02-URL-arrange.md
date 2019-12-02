---
title: "URL-arrange"
permalink: /docs/URL-arrange/
excerpt: "URL Summary"
last_modified_at: 2019-06-13T12:00:00+09:00
redirect_from:
  - /theme-setup/
toc: true
---

### DEV 관련
fabric@localhost:~/go/src/github.com/ldt-hf/tools/network: 
$ git remote -v 
origin  https://github.com/simon0210/hyperledger-study02.git (fetch)
origin  https://github.com/simon0210/hyperledger-study02.git (push)

fabric@localhost:~/go/src/github.com/ldt-hf/tools/network: 
$ git remote add ldt_fabric https://github.com/bumsgy/ldt_fabric.git

fabric@localhost:~/go/src/github.com/ldt-hf/tools/network: 
$ git remote -v 
ldt_fabric      https://github.com/bumsgy/ldt_fabric.git (fetch)
ldt_fabric      https://github.com/bumsgy/ldt_fabric.git (push)
origin  https://github.com/simon0210/hyperledger-study02.git (fetch)
origin  https://github.com/simon0210/hyperledger-study02.git (push)

fabric@localhost:~/go/src/github.com/ldt-hf/tools/network: 
$ git fetch ldt_fabric   
:: 이거 중요. add 한 upstream 에서 브랜치의 head를 가져온다.
https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EB%A6%AC%EB%AA%A8%ED%8A%B8-%EB%B8%8C%EB%9E%9C%EC%B9%98


fabric@localhost:~/go/src/github.com/ldt-hf/tools/network: 
$ git branch -r
  ldt_fabric/master
  origin/HEAD -> origin/master
  origin/master
::: ldt_fabric 뒤에 /master가 붙었음.

fabric@localhost:~/go/src/github.com/ldt-hf/tools/network: 
$ git branch --set-upstream-to=ldt_fabric/master master
'master' 브랜치가 리모트의 'master' 브랜치를 ('ldt_fabric'에서) 따라가도록 설정되었습니다.
:: --set-upstream-to=<remote명>   ==> remote명을 안 쓰면. origin/master를 따라가지 않을까?
https://jw910911.tistory.com/16

: 이후 push하면 내 저장소로 push~~~~!!!!

fabric@localhost:~/go/src/github.com/ldt-hf/tools/network: 
$ git checkout master





// remote 삭제
fabric@localhost:~/go/src/github.com/ldt-hf/tools/network: 
$ git remote rm ldt_fabric

: https://mylko72.gitbooks.io/git/content/remote/remove.html

// branch 삭제
https://jw910911.tistory.com/16


$ git commit -m "mychange"
[HEAD 분리됨 f22de87] mychange
 Committer: fabric <fabric@localhost.localdomain>


$ git reflog
: git rebase, git reset 등으로 commit이 삭제될 수 있음.
: 하지만, 이력은 존재하는데. 이를 보기 위함
https://suwoni-codelab.com/git/2018/04/07/Git-reflog/

$ git branch <쓸 branch 명> reflogId

$ git merge ~
merge는 root가 될 녀석 branch에서 







https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EB%B8%8C%EB%9E%9C%EC%B9%98%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80