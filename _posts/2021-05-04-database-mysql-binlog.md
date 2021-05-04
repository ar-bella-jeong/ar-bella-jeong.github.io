---
title: "Binary Log"
date: 2021-05-04 10:46:28 -0400
categories: database
tags:
  - database
  - mysql
  - mariadb
  - binlog
comments: true
---

database에 대한 모든 변경 사항(data 및 구조)과 각 명령문 실행 시간이 기록되어 있다. **InnoDB 같은 Storage Engine에서 기록하는 Redo Log와는 다른 별도의 로그**이다.
binary log는 일련의 binary log file 집합과 index file로 구성된다.
CREATE, ALTER, INSERT, UPDATE 및 DELETE와 같은 statement는 기록되지만, SELECT 및 SHOW와 같이 data에 영향을 미치지 않는 statement는 기록되지 않는다. (성능 상 비용은 더 들지만) 이를 기록하려면 [general query log](https://mariadb.com/kb/en/general-query-log/)를 사용해야 한다.
(row-based logging이 아닌 statement-based logging이 default인 경우) 아무런 row를 return 하지 않는 UPDATE or DELETE또한 logging 된다.

<U>binary log의 목적은 백업 및 복구 작업을 지원할 뿐만 아니라, 하나 이상의 master에서 하나 이상의 slave server로 data가 전송되어 복제하도록 하기 위한 것</U>이다.
아카이브 된 data가 있고 그 뒤에 event를 기록한 binary log가 있으면, 원하는 시점으로 data를 복구할 수 있다.


binary log가 활성화 된 Maria DB server는 약간 느리게 실행될 수 있다. binary log는 암호를 포함한 민감한 정보를 포함할 수 있으므로 보호해야한다. 이를 위해 binary log는 일반 text가 아닌 이진으로 저장되므로 일반 편집기로 볼수 없다.  하여 MariaDB에는 bin log을 일반 text로 변환하여 볼 수 있도록 [mysqlbinlog](https://mariadb.com/kb/en/mysqlbinlog/) 가 포함되어 있다.

```bash
mysqlbinlog mariadb-bin.000006
```

bin log file name에는 prefix 외에 6자리의 일련번호가 포함되고 이 번호는 한계치만큼 기록되면 자동으로 증가하여 새로운 file이 생성된다. 자리수도 자동으로 늘어난다. (내부적으로는 unsigned long으로 선언되어 거의 무제한으로 증가해도 문제 없음)


## Binary log 활성화
기본적으로 bin log는 기록되지 않는다.
server 기동 시 --log-bin[=name] option을 주면 binary logging 이 활성화된다. (또는 설정파일에서 설정 가능. log_bin=/var/log/mysql/mysql-bin.log)
name option(absolute path)를 주지 않으면 다음 중의 한 이름으로 file이 생성된다.
- datadir/log-basename-bin
- datadir/mysql-bin
- datadir/mariadb-bin

datadir는 system variable 설정에 따른다.
log-basename-bin의 log-basename은 --log-basename option으로 주어진 이름이다.
 만약 --log-bin[=name] 에서 name 이 주어지지 않고, --log-basename 옵션도 주어지지 않을 경우, 파일 이름은 mysql-bin 또는 mariadb-bin 으로 결정된다. 이는 server version 에 따라 다르다.
 되도록이면 외부 환경에 의해 파일 이름이 결정되지 않도록 사용자가 이름을 결정하는 편이 좋다.

 binary log 파일이 저장되는 디렉토리에는 자동으로 binary log index 파일도 생성된다. binary log index 는 모든 binary log 들의 리스트를 순서대로 저장하고 있다. 기본적으로 binary log 파일의 이름과 동일한 이름에 .index 확장자가 붙어서 생성되며, 이름을 변경하고 싶은 경우 --log-bin-index[=filename] 옵션을 사용한다.

```bash
1.  [root@dpleevbox mysql]# cat mariadb-bin.index
2.  /var/log/mysql/mariadb-bin.000001
3.  /var/log/mysql/mariadb-bin.000002
4.  /var/log/mysql/mariadb-bin.000003
5.  /var/log/mysql/mariadb-bin.000004
6.  /var/log/mysql/mariadb-bin.000005
7.  /var/log/mysql/mariadb-bin.000006
8.  /var/log/mysql/mariadb-bin.000007
```

확장자는 자동으로 증가하는 숫자를 이용하며 다음의 경우에 새로운 file이 생성된다.
- server가 기동될 때
- log가 flush 될 때
- max_binlog_size로 설정된 max size에 도달하였을 때

SUPER 권한을 가진 client 의 경우 다음과 같이 [sql_log_bin](https://mariadb.com/kb/en/library/replication-and-binary-log-system-variables/#sql_log_bin) 변수 값을 변경하여 현재 세션에 대해 binary log 를 활성화 또는 비활성화할 수 있다
```bash
SET sql_log_bin = 0;
SET sql_log_bin = 1;
```


## Binary log format
변경 event를 bin log에 기록할 때 다음의 3가지 format을 지원한다.
-   **Statement-based Logging**
-   **Row-based Logging**
-   **Mixed Logging**

아마도 다음과 같은 상황일 경우 binary log 의 format 을 변경하는게 좋을 수 있다.

-   하나의 Update 문이 다수의 row 를 변경할 경우 Statement-based Logging 이 Row-based Logging 보다 slave 에서 download 시 효율적일 것이다.
-   다수의 statement 가 row 변경을 거의 발생시키지 않을 경우, Row-based Logging 이 Statement-based Logging 보다 slave 에서 download 시 효율적일 것이다.
-   Long-run Query 임에도 불구하고 실제 변경하는 row 는 극히 제한적일 경우 Row-Based Logging 이 slave 에게는 효율적일 것이다.

**MariaDB 10.2.3 버전까지는 Statement-based Logging 이 Default 였으나, 10.2.4 버전부터는 Mixed Logging 이 Default** 이다. format 은 다음과 같이 binlog_format system variable 를 통해 변경할 수 있다. 물론 server 기동 전에 .cnf 파일에 binlog_format=ROW 와 같이 설정할 수도 있다.

```bash
MariaDB  (none)> SET GLOBAL binlog_format = 'STATEMENT';
MariaDB  (none)> SET GLOBAL binlog_format = 'ROW';
MariaDB  (none)> SET GLOBAL binlog_format = 'MIXED';
```

STATEMENT format 을 사용할 경우 주의할 점이 있는데, Order by 없는 LIMIT 절과 같이 결과를 예측할 수 없는 [non-deterministic 구문](https://mariadb.com/kb/en/library/unsafe-statements-for-statement-based-replication/)의 경우 replication 을 사용할 때 안전하지 않을 수 있다. 이 때는 **Row-based Logging 이나 Mix Logging 을 사용**한다. **Mixed Logging 을 사용할 경우 replication 에서 안전하지 않은(non-deterministic) statement 는 자동으로 Row-based Logging 을 통해 기록된다.**

**replication 을 사용할 경우에는 binlog_format 값을 변경할 때 주의를 해야 한다. binlog_format 에 대한 변경은 해당 서버(master)에만 적용되며 이는 replication 의 데이터 정합성에 악영향을 주거나 fail 을 발생시킬 수 있다.**

parallel replication 을 사용하고 있는 경우 binlog_format 값을 동적으로 변경할 때도 주의해야 한다. worker thread 가 동작 중인 경우 해당 세션에 변경 값이 적용되지 않을 수도 있기 때문이다. 다음과 같은 작업 과정을 통해 이 문제를 피할 수 있다.

```bash
STOP SLAVE;
SET GLOBAL slave_parallel_threads=0;
SET GLOBAL binlog_format='ROW';
SET GLOBAL slave_parallel_threads=4;
START SLAVE
```


## 관련 Command

-   [SHOW BINARY LOGS](https://mariadb.com/kb/en/library/show-binary-logs/)  
    해당 서버에 있는 binary log 파일의 리스트를 보여준다. PURGE BINARY LOGS 명령은 이 리스트를 바탕으로 수행된다.
-   [PURGE BINARY LOGS](https://mariadb.com/kb/en/library/purge-binary-logs/)  
    log index 파일에 기록된 모든 파일을 삭제하는데 사용된다. datetime 옵션을 통해 원하는 시간 이전의 파일만을 삭제할 수도 있다. 만약 slave 가 읽고 있는 파일이라면 삭제되지 않는다. RESET MASTER 도 모든 binary log 파일을 삭제하지만, 이는 master 초기 셋업 시 강제 초기화 개념이기 때문에 replication 을 시작한 이후로는 사용하지 않는 것이 좋다.
-   [SHOW BINLOG EVENTS](https://mariadb.com/kb/en/library/show-binlog-events/)  
    특정 binary log 파일에 기록된 event 들을 보기 위해 사용한다.
-   [SHOW MASTER STATUS](https://mariadb.com/kb/en/library/show-master-status/)  
    replication master 에 있는 binary log 의 상태를 보여주는데 사용된다.

> 출처: http://cloudrain21.com/mariadb-binary-log
