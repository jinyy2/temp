# 리눅스 APM 구성 (Apache, PHP, MySQL)
## APM 이란? 
: Apache, PHP, Mysql의 앞 글자를 따서 온 단어
`apache` : 웹 서버
`PHP` : 프로그래밍 언어
` MySQL` : 데이터베이스 서버 
+ Linux => LAMP

## 리눅스 미들웨어 표준 구성 이해
## 리눅스 기반 WEB/WAS 개념 및 종류
### WEB
: 웹 브라우저와 같은 클라이언트로부터 HTTP 요청을 받아들이고, HTML 문서와 같은 웹 페이지를 반환하는 컴퓨터 프로그램
- html, css, javascript와 같은 정적인 페이지를 담당함
- Apache
- IHS(IBM Http Server)
- WebToBE(티맥스소프트)
- Lena Web Server(LG CNS)
- Nginx

### WAS
: 웹 애플리케이션 서버(Web Application Server, 약자 WAS)는 웹 애플리케이션과 서버 환경을 만들어 동작시키는 기능을 제공하는 소프트웨어 프레임워크
- php, jsp, asp와 같은 언어들을 사용해 동적인 페이지를 생성할 수 있는 서버
- DB 접속 및 조회, 비지니스 로직과 같은 동적인 페이지를 담당함
- 웹 서버 + 웹 컨테이너 (컨테이너: jsp, servlet을 실행시킬 수 있는 소프트웨어)
- Tomcat
- WebSphere(IBM)
- Jeus(티맥스소프트) 
- Lena Web Application Server(LG CNS)

## 리눅스 WEB/WAS 환경 설정
### Apache & Tomcat 연동(mod_jk module)
Virtual Machine
환경 : Cent OS 7
Apache : Apache/2.4.6 (CentOS)
Tomcat : apache-tomcat-8.5.69
JDK : openjdk version "1.8.0_292"


1. jdk 설치
```
$ yum list java*jdk-devel
$ yum install -y java-1.8.0-openjdk.devel.x86_64
$ java -version
openjdk version "1.8.0_292"
```
2. Apache 설치
```
$ yum install httpd httpd-devel -y
```
시작 : systemctl start httpd
종료 : systemctl stop httpd
상태 : systemctl status httpd

```
$ vi /etc/profile
	export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.292.b10.el7_9.x86_64
$ source /etc/profile
```
3. Tomcat 설치
```
$ wget http://mirror.apache-kr.org/apache/tomcat/tomcat-8/v8.5.69/bin/apache-tomcat-8.5.69.tar.gz
$ tar zxvf apache-tomcat-8.5.69.tar.gz
```
cd /usr/local/src/apache-tomcat-8.5.69/bin
시작 : ./startup
종료 : ./catalina.sh stop
상태 : ps -ef | grep tomcat

4. Connector 설치
mod_jk.so :  Linux용 Tomcat Connector 

```
$ wget http://mirror.navercorp.com/apache/tomcat/tomcat-connectors/jk/tomcat-connectors-1.2.48-src.tar.gz
$ tar zxvf tomcat-connectors-1.2.48-src.tar.gz
$ yum install gcc autoconf libtool -y
$ cd tomcat-connectors-1.2.48-src
$ ./buildconf.sh
$ ./configure -with-apxs=/usr/bin/apxs # which apxs로 위치 확인
$ make && make install
```

**/etc/httpd/conf/httpd.conf**
```
LoadModule jk_module modules/mod_jk.so
JkWorkersFile “conf/workers.properties”
JkLogFile logs/mod_jk.log
JkLogLevel info
JKLogStampFormat “[%y %m %d %H:%M:%S]”
JkRequestLogFormat "%w %V %T"

JkMount /*.do jinyong
JkMount /*.jsp jinyong
JkMount /servlet/* jinyong

serverName localhost
```

**/etc/httpd/conf/uriworkermap.properties**
```
/*.do=ajp13
/*.jsp=ajp13
```

**/etc/httpd/conf/workers.properties**
```
workers.tomcat_home="/usr/local/src/apache-tomcat-8.5.69"
workers.java_home="$JAVA_HOME"

worker.list=jinyong
worker.jinyong.port=8009
worker.jinyong.host=localhost
worker.jinyong.type=ajp13
```

**/usr/local/src/apache-tomcat-8.5.69/conf/server.xml**
```
<Connector protocol=”AJP/1.3″
port=”8009″
secretRequired=”false”
redirectPort=”8443″ />

<Host name=”localhost” appBase=”/home/jinyong/public_html/”
unpackWARs=”true” autoDeploy=”true” xmlValidation=”false” xmlNamespaceAware=”false”>
<Context path=”” docBase=”.” reloadable=”true”/>
```

```
vi /home/jinyong/public_html/index.jsp
<%@ page language="java" %>
<html>
<head><title>JSP Test</title></head>
<body>
	<h1>Apache Tomcat Connection</h1>	
</body>
</html>

## DataSource 
- DataSource는 서버에서 관리하기 때문에 DB나 JDBC 드라이버가 변경이 수월
- Connection, Statement 객체를 pooling 할 수 있으며, 분산 트랜잭션을 다룰 수 있음 


### Apache 설치

```
$ yum install -y httpd 
$ sudo systemctl start httpd.service
$ http://localhost/
$ systemctl enable httpd #서비스 활성화(부팅시 자동 구동)
$ httpd -v #Apache 설치 확인
```
### MariaDB 설치

```
$ sudo yum install mariadb-server mariadb
$ sudo systemctl start mariadb
$ sudo systemctl enable mariadb.service
```
### PHP 7.3 설치 
```
$ sudo vi /var/www/html/info.php
<?php phpinfo(); ?>
```

server-ip/info.php 접속 화면 확인

> https://eine.tistory.com/entry/CentOS-7%EC%97%90-LAMP-%EC%8A%A4%ED%83%9DLinux-Apache-MySQL-PHP-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0

