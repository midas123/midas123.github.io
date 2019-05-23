---
  layout: single
  title: 리눅스 mariaDB 세팅
---

리눅스(데비안) 환경에서 마리아DB 세팅 방법을 정리했습니다. [mariaDB](https://mariadb.com/kb/ko/mariadb-korean-mariadb/)는 mysql 엔지니어가 만든 오픈 소스 데이터베이스 입니다. ([GNU General Public License, version 2](https://www.olis.or.kr/license/Detailselect.do?lId=1004&mapCode=010004).)

mysql과 호환될 뿐만 아니라 더 많은 기능과 빠른 성능을 가지고 있다고 합니다. [특징들](https://mariadb.com/kb/en/library/mariadb-vs-mysql-features/)



<br>

## mariaDB 인스톨

**서버/클라이언트 설치**

```
sudo apt-get install mariadb-server mariadb-client
```

※dirmngr 미설치로 인한 에러시 아래코드로 설치

```
sudo apt-get install dirmngr --install-recommends
```

**마리아DB 버전 확인**

```
mysql --version
```

<br>

<br>

## mariaDB 시작

systemd는 리눅스 시스템 부팅시 작동할 프로그램을 관리하는 서비스 매니져이다. 본인 리눅스 버전의 지원 여부확인해야 한다. MariaDB 10.1.8 버전부터 [systemd]()가 사용되고 있다.(그 이전엔 [sysVnit](https://mariadb.com/kb/en/library/sysvinit/) )

<br>

**리눅스 root 계정 로그인**

```
su root //루트 계정 로그인(비밀번호 입력)

sudo passwd root //최초 설치 후 루트 계정 비밀번호 설정
```

**시작 설정**

```
sudo systemctl status mariadb //실행 상태 확인

systemctl start mariadb

systemctl restart mariadb
```

**자동 시작 설정**

```
systemctl enable mariadb
```

**참고**

https://mariadb.com/kb/en/library/systemd/#interacting-with-the-mariadb-server-process

<br>

## DB 계정 생성

마리아DB에 root 유저로 접속

```
mysql -u root -p
```

새로운 계정으로 사용할 데이터 베이스 생성

```
CREATE DATABASE 'DB명';

SHOW DATABASES //DB 목록
USE 'DB명'; //DB 선택
```

root 계정으로 외부에서 접속하는 것은 보안상 좋지 않으므로 새로 계정을 생성한다. 아래는 예시는 계정명이 abc123, 접속대역은 제한없음(%)이고 비밀번호는 1234이다. [CREATE USER 문법](https://mariadb.com/kb/en/library/create-user/)

```
CREATE USER 'abc123'@'%' IDENTIFIED BY '1234';

SELECT user,host FROM mysql.user; //mysql데이터베이스의 유저 테이블 조회
DROP USER 'abc123'@'%'; //계정 삭제
```

생성한 계정에 권한 부여.  eft 데이터베이스의 모든 테이블(*)에 대한 모든 권한을 abc123에게 부여한다.

```
GRANT all privileges ON efg.* TO 'abc123';

REVOKE DELETE ON efg.* FROM abc123@'%'; //권한 제거
```

※INSERT, UPDATE, DELETE 등 DML로 GRANT 테이블을 직접 조작하여 권한을 부여하면 db서버를 재시작 하기전까지 변경사항이 적용되지 않는다. [링크](https://dev.mysql.com/doc/refman/5.7/en/privilege-changes.html) 

위에서 처럼 **GRANT(DCL)로 권한을 부여하지 않은 경우** 아래 명령어로 권한 변경을 갱신해줘야 한다.

```
FLUSH PRIVILEGES;
```



<br>

## DB 포트 및 listen IP 설정

my.cnf 파일을 오픈 후 포트 수정 및 IP 바인딩이 가능하다.

```
vi /etc/mysql/my.cnf
```

로컬이 아닌 구글 클라우드 인스턴스에 설치했으므로 아래와 같이 **bind-address**을 변경했다.

```
bind-address = 127.0.0.1     ->      0.0.0.0
```

마리아DB 재시작 후 아래 명령어로 **bind-address**가 변경 되었는지 확인한다.

```
sudo lsof -i -P -n | grep 3306
```

**port 번호**는 그대로 사용한다. (기본값은 3306) 참고로 필자는 구글 클라우드 인스턴스에 mariaDB를 설치했으므로 구글 클라우드 인스턴스의 방화벽 설정에서 port를 열어줘야 외부에서 접속할 수 있다.



<br>

*-구글 클라우드 방화벽 설정 결과*

![]({{ site.url }}{{ site.baseurl }}\assets\images\post\0428\gc-config.jpg){: .center-image }

<br>

## DB 접속 테스트

기존에 사용 중이던 sqldeveloper로 접속 테스트를 진행했다.  mysql/mariaDB에 접속하려면 [Connector/J](Connector/J)를 별도로 설치해야 한다. db-connection

**connector/J 설치방법**

 1.zip 파일로 다운 받은 후 적당한 곳에 압축을 푼 후 sqldeveloper를 실행한다.

 2.도구 탭 -> 환경설정 -> 데이터베이스 -> 타사 JDBC 드라이버 -> 항목추가 -> 압축 푼 폴더에서 *mysql-connector-java-x.x.xx-bin.jar* 파일 선택 후 sqldeveloper 재시작

<br>

*-sqldeveloper 연결화면*

![]({{ site.url }}{{ site.baseurl }}\assets\images\post\0428\db-connection.jpg){: .center-image }

------

<br>

## DB 데이터 한글  깨지는 현상

sqldeveloper로 mariaDB에 한글 데이터를 넣었는데 DB에 저장된 데이터가 깨졌다. 인코딩에는 문제가 없었고 DB에서 디코딩이 잘못되어 한글이 깨진걸로 보였다.

아래 코드를 /etc/mysql/my.cnf 입력하고 mariaDB를 재시작한다.

```
[mysql]
default-character-set=utf8

[client]
default-character-set=utf8

[mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8
```

charset 변경 전에 만들어진 테이블은 직접 수정해야 한다.

```
ALTER TABLE 테이블명 convert to charset utf8;
```

<br>

참고:

[설치방법](https://alvinbunk.wordpress.com/2017/06/29/using-oracle-sql-developer-to-connect-to-mysqlmariadb-databases/)

[프로젝트에 설치방법](https://m.blog.naver.com/CommentList.nhn?blogId=50after&logNo=220912861796)

[구글 클라우드 인스턴스에서 mysql 설정하는 방법](https://cloud.google.com/solutions/setup-mysql?hl=ko)