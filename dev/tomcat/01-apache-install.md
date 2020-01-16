---
title: VS Code. git 셋팅하기
date: 2019-05-09
tags: ["Dev", "tools", "Setting"]
category : ["Dev", "vscode"]
description : vscode에 git 셋팅하는 방법
---

### AWS에 설치
1. yum을 이용하여 설치 \\
   yum을 이용하여 설치합니다. 이 때, 관리자 권한이 필요하므로, sudo를 붙여줘야 합니다.

   ```bash
        sudo yum update
        sudo yum list | grep httpd
         sudo yum install -y httpd24
   ```
참고 : http://progtrend.blogspot.com/2018/06/aws-apacheweb-server-tomcat.html



gcc 설치 
sudo yum install gcc

apxs가 없어서 안 됨. 일단, 이걸 설치 해야 함.
httpd-devel 을 설치하면, apxs가 같이 설치됨.

sudo yum install http-devel
=> httpd24 conflicts with httpd-2.2.34-1.16.amzn1.x86_64 에러 발생.
설치 시 버전 충돌이 아닌가 함.

sudo yum remove httpd24
sudo yum remove httpd24-tools

sudo usermod -a -G apache ec2-user

삭제 후, 

sudo yum install httpd-2.2.34
sudo yum install httpd-devel
which apxs  
==> /usr/sbin/apxs  ==> 이제 나오네...

cd /home/ec2-user/tomcat-connectors-1.2.46-src/native

./configure --with-apxs=/usr/sbin/apxs
sudo make && make install



/usr/sbin/apxs -i mod_jk.la
/usr/lib64/httpd/build/instdso.sh SH_LIBTOOL='/usr/lib64/apr-1/build/libtool' mod_jk.la /usr/lib64/httpd/modules
/usr/lib64/apr-1/build/libtool --mode=install cp mod_jk.la /usr/lib64/httpd/modules/
libtool: install: cp .libs/mod_jk.so /usr/lib64/httpd/modules/mod_jk.so
cp: cannot create regular file '/usr/lib64/httpd/modules/mod_jk.so': Permission denied
apxs:Error: Command failed with rc=65536
.
make[1]: *** [install_dynamic] Error 1
make[1]: Leaving directory `/home/ec2-user/tomcat-connectors-1.2.46-src/native/apache-2.0'
make: *** [install-recursive] Error 1
[ec2-user@ip-172-31-27-142 native]$ make
Making all in common
make[1]: Entering directory `/home/ec2-user/tomcat-connectors-1.2.46-src/native/common'
make[1]: Nothing to be done for `all'.
make[1]: Leaving directory `/home/ec2-user/tomcat-connectors-1.2.46-src/native/common'
Making all in apache-2.0
make[1]: Entering directory `/home/ec2-user/tomcat-connectors-1.2.46-src/native/apache-2.0'
make[1]: Nothing to be done for `all'.
make[1]: Leaving directory `/home/ec2-user/tomcat-connectors-1.2.46-src/native/apache-2.0'
make[1]: Entering directory `/home/ec2-user/tomcat-connectors-1.2.46-src/native'
make[1]: Nothing to be done for `all-am'.
make[1]: Leaving directory `/home/ec2-user/tomcat-connectors-1.2.46-src/native'
target="all"; \
list='common apache-2.0'; \
for i in $list; do \
    echo "Making $target in $i"; \
    if test "$i" != "."; then \
       (cd $i && make $target) || exit 1; \
    fi; \
done;
Making all in common
make[1]: Entering directory `/home/ec2-user/tomcat-connectors-1.2.46-src/native/common'
make[1]: Nothing to be done for `all'.
make[1]: Leaving directory `/home/ec2-user/tomcat-connectors-1.2.46-src/native/common'
Making all in apache-2.0
make[1]: Entering directory `/home/ec2-user/tomcat-connectors-1.2.46-src/native/apache-2.0'
make[1]: Nothing to be done for `all'.
make[1]: Leaving directory `/home/ec2-user/tomcat-connectors-1.2.46-src/native/apache-2.0'

마지막에 에러..
cp 권한 에러네?

    /usr/lib64/httpd/modules 여기에 mod_jk.so 파일은 있는 것으로 파악됨.
    일단 진행 고고~



sudo service httpd start : 확인결과 forwarding 안 되네? ㅡㅡ;;



2. asf \\
    ```bash
        cd /etc/httpd/conf
    ```

3. Settings 파일 내용 중 Extension > git 을 선택합니다.  \\
   설정 내용 중 Path 항목이 있습니다. \\ 
   만약, git을 기본 설정 그대로 설치 안 했다면, 경로가 다르겠죠~ \\ 
   그럴 경우, 밑에 있는 'edit in setting.json' 버튼을 클릭하여,\\
   설정 파일을 엽니다. \\
   해당 내용 안에 "git.path": git경로 \\
   이 때, 경로는 "로 감싸고, 윈도우일 경우에는 \ 를 2번씩 써 주세요. \\ 
   이스케이프 문자이기 때문에~~( 다 아실테지만 혹시나~ )
