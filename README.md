# zabbix-server
ProLinux 8.3 기준 Zabbix 5.0 LTS 서버 설치 가이드

## Prerequisites

1. 공용망 사용 
2. ProLinux 8.3 설치
## 구축 가이드

1. Zabbix 저장소 추가

```
# rpm -ivh http://repo.zabbix.com/zabbix/5.0/rhel/8/x86_64/zabbix-release-5.0-1.el8.noarch.rpm
```

2. Zabbix, DB(MariaDB), WEB(Apache), PHP 설치 진행

```
# dnf -y install zabbix-server-mysql zabbix-web-mysql mariadb-server zabbix-apache-conf zabbix-agent
```

3. MariaDB 구동

```
# systemctl start mariadb
# systemctl enable mariadb
# ps -ef | grep mysql

mysql      9817      1  2 22:55?        00:00:00 /usr/libexec/mysqld --basedir=/usr
```

4. MariaDB 기본 설정

```
# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): [패스워드가 없기 때문에 엔터]

OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] Y    [DB ROOT 패스워드 설정]
New password: 패스워드 입력
Re-enter new password: 패스워드 재입력
Password updated successfully!
Reloading privilege tables..
 ... Success!

 

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y    [익명의 접근을 막을 것인지? 보안을 위해 Y 엔터]

 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y    [DB ROOT 원격을 막을 것인지? 보안을 위해 Y 엔터]

 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y  [Test 용으로 생성된 데이터베이스를 삭제할 것인가? Y 엔터]

 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y  [현재 설정한 값을 적용할 것인지? 당연히 Y 엔터]

 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB! [완료]


```

5. MariaDB 접속 및 Zabbix 데이터베이스 생성

```
# mysql -u root -p
Enter password: 패스워드 입력
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 16
Server version: 10.3.17-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

 

* Zabbix 데이터 베이스 생성[ ※ 중요! UTF8 생성]
MariaDB [(none)]> create database zabbix character set utf8 collate utf8_bin;

Query OK, 1 row affected (0.001 sec)

* 권한 부여

MariaDB [(none)]> grant all privileges on zabbix.* to zabbix@localhost identified by 'test123';
Query OK, 0 rows affected (0.001 sec)

 

* 적용 후 종료

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.001 sec)

 

MariaDB [(none)]> exit
Bye
```

6. Zabbix 초기 테이블 값 정보 데이터베이스에 적용

```
* 경로 이동
# cd /usr/share/doc/zabbix-server-mysql/


* 압축해제
# gunzip create.sql.gz
# ls

AUTHORS  COPYING  ChangeLog  NEWS  README  create.sql (파일 확인)  double.sql

* Zabbix 데이터베이스에 복원
# mysql -u root -p zabbix < create.sql
Enter password: 패스워드 입력
```

7. Zabbix Config 설정[수정 후 저장]

```
# vi /etc/zabbix/zabbix_server.conf

38 LogFile=/var/log/zabbix/zabbix_server.log      [기본 설정]
72 PidFile=/var/run/zabbix/zabbix_server.pid      [기본 설정]
82 SocketDir=/var/run/zabbix                      [기본 설정]
91 DBHost=localhost                               [주석(#) 제거]
100 DBName=zabbix                                 [DB 생성 이름과 동일하게 설정]
116 DBUser=zabbix                                 [DB 유저 생성 이름과 동일하게 설정]
124 DBPassword=test123                            [주석(#) 제거 후 DB 유저 패스워드와 동일하게 설정]

:wq
```

8. Zabbix PHP-FPM Config 설정[수정 후 저장]


```
# vi /etc/zabbix/zabbix_server.conf

38 LogFile=/var/log/zabbix/zabbix_server.log      [기본 설정]
72 PidFile=/var/run/zabbix/zabbix_server.pid      [기본 설정]
82 SocketDir=/var/run/zabbix                      [기본 설정]
91 DBHost=localhost                               [주석(#) 제거]
100 DBName=zabbix                                 [DB 생성 이름과 동일하게 설정]
116 DBUser=zabbix                                 [DB 유저 생성 이름과 동일하게 설정]
124 DBPassword=test123                            [주석(#) 제거 후 DB 유저 패스워드와 동일하게 설정]

:wq
```

9. Zabbix PHP-FPM Config 설정[수정 후 저장]

```
* 기본 셋팅되어 있으므로 하나의 라인만 수정
# vi /etc/php-fpm.d/zabbix.conf

 [zabbix]
user = apache
group = apache

listen = /run/php-fpm/zabbix.sock
listen.acl_users = apache,nginx
listen.allowed_clients = 127.0.0.1

pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35

php_value[session.save_handler] = files
php_value[session.save_path]    = /var/lib/php/session

php_value[max_execution_time] = 300
php_value[memory_limit] = 128M
php_value[post_max_size] = 16M
php_value[upload_max_filesize] = 2M
php_value[max_input_time] = 300
php_value[max_input_vars] = 10000
php_value[date.timezone] = Asia/Seoul     [주석(;) 제거 후 한국 시간으로 수정]

:wq
```

9. IPTABLES 방화벽 포트 허용 및 재시작

```
* 수정 후 저장
# vi /etc/sysconfig/iptables

# sample configuration for iptables service
# you can edit this manually or use system-config-firewall
# please do not ask us to add additional ports/services to this default configuration
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 8888 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT             [HTTP 웹포트 (추가)]
-A INPUT -p tcp -m state --state NEW -m tcp --dport 10050 -j ACCEPT          [Zabbix Agent 포트 (추가)]
-A INPUT -p tcp -m state --state NEW -m tcp --dport 10051 -j ACCEPT          [Zabbix Server 포트 (추가)]
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT

* 재시작
# systemctl restart iptables

* 적용 확인[허용된 포트 확인]
# iptables -nL

ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:80
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:10050
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:10051
```

10. Zabbix Server, Httpd, PHP-FPM 서비스 구동

```
* 전체 구동
# systemctl start zabbix-server httpd php-fpm

* 부팅 시 자동 활성화
# systemctl enable zabbix-server httpd php-fpm

* 프로세스 구동 확인[Zabbix의 경우는 스크린샷 참고]
# ps -ef | egrep "httpd|zabbix"

zabbix    10162      1  0 01:06 ?        00:00:00 /usr/sbin/zabbix_server -c /etc/zabbix/zabbix_server.conf
apache    10477  10470  0 01:11 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache    10478  10471  0 01:11 ?        00:00:00 php-fpm: pool zabbix
```

11. Zabbix Web 설치 페이지 접속 [http://서버 IP 주소 또는 호스트 네임/zabbix]

```
Configure DB connection
Database name : zabbix
User : zabbix
Password : test123(위에서 설정 했던 비밀번호)

Zabbix 로그인
Username : Admin
Password : zabbix
```

