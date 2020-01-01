---
title: 자아~ 백업받고 RDS를 일단 닫자~
date: 2019-12-03
last_modified_at: 2019-12-03T13:00:00+09:00
tags: ["diary", "glogen", "dev"]
category : ["diary", "glogen", "dev"]
description : 하으... 사람이 안 들어오네...
toc: true
toc_label: ""
toc_icon: "cog"  # corresponding Font Awesome icon name (without fa prefix)
---

### RDS를 없애야 할 듯해요.
음... 비용 나오는 걸 보면 RDS가 꽤나 금액이 나오네요. \\
그래서 꼭 RDS를 사용해야 하나 해서 검색을 해 보니... 그렇지 않네요. \\
EC2 인스턴스를 늘리지 않는 한 과금은 동일한거 같고, \\
그렇다면, 현재 상황에서는 EC2 하나에 DB도 설치해서 그냥 같이 사용하는 것이 \\
더 효율적으로 판단되었습니다. (( 차피 들어오는 사람도 없는데 머... OTL ))

https://niceman.tistory.com/60 \\
https://puttico.tistory.com/150 \\
https://sarc.io/index.php/mariadb/346-innodb-myisam


### RDS DB 백업
먼저 RDS DB를 백업받았습니다. \\
RDS 콘솔에서 백업을 받을 수 있나 했더니, 그건 없는 것으로 보였습니다. 검색으로도 안 나오네요. \\
그래서, EC2에서 접근이 되도록 설정이 되어 있으니, \\
EC2에서 접근..... 하려다가 현재 RDS에 public으로 접근 할 수 있도록 해 둔 것이 기억이 나서 \\
현재 개발 서버로 하나 있는 centos 서버에서 처리하기로 했습니다. \\
일단 설치가 되어 있는지 확인을 해 보니 없어서, 먼저 설치를 합니다. \\
그래야 백업 명령어를 실행하든 머하든 할 수 있으니까요.

```bash
    $ yum list | grep mariadb
    mariadb-libs.x86_64                       1:5.5.64-1.el7                 @base  
    mariadb.x86_64                            1:5.5.64-1.el7                 base   
    mariadb-bench.x86_64                      1:5.5.64-1.el7                 base   
    mariadb-devel.i686                        1:5.5.64-1.el7                 base   
    mariadb-devel.x86_64                      1:5.5.64-1.el7                 base   
    mariadb-embedded.i686                     1:5.5.64-1.el7                 base   
    mariadb-embedded.x86_64                   1:5.5.64-1.el7                 base   
    mariadb-embedded-devel.i686               1:5.5.64-1.el7                 base   
    mariadb-embedded-devel.x86_64             1:5.5.64-1.el7                 base   
    mariadb-libs.i686                         1:5.5.64-1.el7                 base   
    mariadb-server.x86_64                     1:5.5.64-1.el7                 base   
    mariadb-test.x86_64                       1:5.5.64-1.el7                 base  

    $ sudo yum install mariadb
```

이후 백업을 실행합니다.
```bash
    $ mysqldump -u[유저id] -p -A -h [rds주소] > glossary_db_backup.sql
```

### EC2에 mariadb 설치 및 복원
파일을 EC2에 업로드 합니다. \\
EC2에 mariaDB를 설치합니다. \\
amazon linux가 어느 계열인지 정확하게 찾을 수가 없네요. \\
mariadb 설치를 하려면, repo를 등록해야 하고, \\
그러기 위해선, 운영체제를 선택해야 하거든요. \\
일단 타블로거들을 보면, centos를 기준으로 하는 듯합니다. \\
그렇게 진행을 해 보죠. \\
하지만, aws 어디에서든 그 근거를 좀 찾았으면 좋겠네요. \\

인데, 그 전에 타임존을 변경해야 합니다. \\
안 할경우. 는 어디서 봤는데... \\
타임존 설정 파일은 /etc/localtime 입니다. \\
그러니, 이 파일을 삭제하거나, 수정하거나 해야 합니다. \\
사실 모든 타임존 설정 파일이 /usr/share/zoneinfo 폴더 안에 존재합니다. \\
서울은 그 안에 Asia/Seoul 로 파일이 존재하죠. \\
이를 복사해서 localtime 파일명으로 만들어도 되겠지만, \\
그냥 링크 설정으로 간편?하게 설정하겠습니다.

```bash
    $ date
    $ cd /etc
    $ sudo mv localtime localtime--         // 이건 그냥 백업한 것 뿐입니다. 삭제해도 무방
    $ sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
    $ date
```

release 버전을 확인합니다. \\

```bash
    $ cat /proc/version
    Linux version 4.14.146-93.123.amzn1.x86_64 (mockbuild@koji-pdx-corp-builder-60003) (gcc version 7.2.1 20170915 (Red Hat 7.2.1-2) (GCC)) #1 SMP Tue Sep 24 00:45:23 UTC 2019
```

ami이므로 centos > x86_64 > 10.2 로 선택합니다. \\
선택을 하게 되면, repo를 등록하라고 하는데, 파일을 작성해 줍니다.


```bash
    $ sudo vi /etc/yum.repos.d/mariadb.repo
        - 내용작성 - 
```

확인 해 보았습니다.

```bash
    $ sudo yum list | grep MariaDB
```

MariaDB로 적혀있는데 꽤나 많네요. \\
아까 MariaDB repo 설정하는 페이지에서 보면 yum으로 install 하는 방법 링크를 따라가보면 \\
https://mariadb.com/kb/en/library/yum/ \\
package로 설치하는 방법 등이 있는데, client 등은 필요없을 것으로 판단됩니다. \\
mariadb-server만 설치합니다. 


```bash 
    $ sudo yum install -y MariaDB-server
```

나중 확인해야 할 것  \\
왜 모두 리스트로 나오는건까요? MariaDB-server가 의존성을 가지고 있어서 모두 설치가 되는 것인지... \\
이건 한 번 실험 해 봐야 겠네요. \\
```bash
    $ sudo yum list installed | grep MariaDB
    MariaDB-client.x86_64                 10.2.29-1.el6                @mariadb     
    MariaDB-common.x86_64                 10.2.29-1.el6                @mariadb     
    MariaDB-compat.x86_64                 10.2.29-1.el6                @mariadb     
    MariaDB-server.x86_64                 10.2.29-1.el6                @mariadb
```

```bash
    $ sudo service mysql start

    Starting MariaDB.191204 12:32:54 mysqld_safe Logging to '/var/lib/mysql/ip-172-31-27-142.err'.
    191204 12:32:54 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
    SUCCESS! 
```

최초 설정을 해 준다. \\
예전에는 mysqladmin 명령어를 이용해서 비밀번호를 설정 해 주었는데, 그럴 필요가 없네요.

```bash
    $ sudo mysql_secure_installation
```

root 비밀번호 \\
익명 유저 삭제 여부 \\
test 테이블 삭제 여부 \\ 
등등을 설정하도록 되어 있습니다. \\
보안을 위해, 익명 유저등은 모두 삭제하는 설정 해줍니다. \\


이제 복원을 해 봅시다. \\

```bash
    $ mysql -u gloroot -p < glossary_db_backup.sql 
    Enter password: 
    ERROR 1044 (42000) at line 164: Access denied for user 'gloroot'@'localhost' to database 'innodb'
```
흠... 에러가 나네요. innodb....가 왜 나오지? 내 DB는 그게 아닌데... \\
싶어서 sql 파일을 확인 해 보니, innodb...가 있네요. \\
백업 시에 -A 옵션. 그러니까 모든 DB를 백업 받아서 그런 듯 합니다. \\
다음 번 백업시에는 DB명을 명시 해 주어서 필요한 DB만 백업받는다면 복원시에 이런 일도 발생하지 않을 듯합니다. \\
(( 그런데, 복원 시에 유저를 먼저 만들어줘야하나...는 여전히 궁금증으로 남아있네요. 또한 유저명을 다르게 한다면? 도요. ))

이제 서버쪽에서 EC2에 있는 mariadb를 보도록 설정 해 준 다음, \\
서버를 다시 확인 해 보니, \\
에러가 나네요? \\
아무래도 대소문자 구분을 하는 듯합니다. \\
기존 RDS의 경우, 제가 대문자로 생성을 했음에도 불구하고, 모두 소문자 테이블로 만들더군요. \\
/usr/my.conf.d 밑에 있는 설정 파일들을 수정해 줍니다. \\
대소문자를 구분하지 않도록요. (( 맘 같아선 구분하도록 해 주고 테이블명을 다 바꾸고 싶지만... )) 

```bash
    $ sudo vi mysql-clients.cnf 

    #
    # These groups are read by MariaDB command-line tools
    # Use it for options that affect only one utility
    #

    [mysql]

    [mysqld]
    lower-case-table_names=1

    [mysql_upgrade]

    [mysqladmin]

    [mysqlbinlog]

    [mysqlcheck]

    [mysqldump]

    [mysqlimport]

    [mysqlshow]

    [mysqlslap]
```

하는 김에, autocommit되지 않도록 설정을 추가해 줍니다. 

```bash
    $ sudo vi server.cnf 

    #
    # These groups are read by MariaDB server.
    # Use it for options that only the server (but not clients) should see
    #
    # See the examples of server my.cnf files in /usr/share/mysql/
    #

    # this is read by the standalone daemon and embedded servers
    [server]

    # this is only for the mysqld standalone daemon
    [mysqld]
    autocommit=0
    --- 하략  ---
```

톰캣에서 정상적으로 동작하네요. \\
이로써, RDS에서 EC2로 DB 옮기기 완료~ \\







여기서부터는 참고 자료인데, 정리 예정입니다. 

https://downloads.mariadb.org/mariadb/repositories/#distro=CentOS&distro_release=centos6-amd64--centos6&mirror=guzel&version=10.2






 
aws에서는 이 정도만 있네요. \\ 
[Amazon Linux](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/amazon-linux-ami-basics.html)( [새 창](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/amazon-linux-ami-basics.html){: target="_blank"})

대충 비슷한 검색 내용인데, \\
centos로 시작했다가, 이제는 독립적으로 구현되고 있다는 듯합니다. \\
(( 아니 그럼, mariadb랑은 뭘 선택하냐고... ㅡㅡ )) \\ 
https://serverfault.com/questions/798427/what-linux-distribution-is-the-amazon-linux-ami-based-on \\


https://aws.amazon.com/ko/premiumsupport/knowledge-center/ec2-enable-epel/


https://mariadb.org/ DB 내용은 따로 해야지~



amazon ec2 입문 : https://blog.leedoing.com/13

ami2에 설치... https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ec2-lamp-amazon-linux-2.html \\
ami에는 mysql. ami2에는 mariadb... 왜지?


-- fedora
# MariaDB 10.4 Fedora repository list - created 2019-12-03 08:04 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.4/fedora30-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1

-- redhat
# MariaDB 10.4 RedHat repository list - created 2019-12-03 08:06 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.4/rhel8-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1