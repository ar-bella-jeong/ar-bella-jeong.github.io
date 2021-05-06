---
title: "Maria DB Relay log"
date: 2021-05-06 09:45:28 -0400
categories: database
tags:
  - database
  - mysql
  - mariadb
comments: true
---

Slave는 I/O thread를 통해서 받은 event를 local에 있는 file에 저장하는데, 이를 Relay log라고 부른다.
The relay log is set of log files created by a replica during replication.
It's the same format at the binary log, containing a record of events that affect the data or structure; thus, [mysqlbinlog](https://mariadb.com/kb/en/mysqlbinlog/) can be used to display its contents. It consists of a set of relay log files and an index file containing a list of all relay log files.

Events are read from the primary's binary log and written to the replica's relay log. They are then performed on the replica. Old relay log files are automatically removed once they are no longer needed.

## Creating Relay Log Files
-   when the IO thread starts
-   when the logs are flushed, with  [FLUSH LOGS](https://mariadb.com/kb/en/flush/)  or  [mysqladmin flush-logs](https://mariadb.com/kb/en/mysqladmin/).
-   when the maximum size, determined by the  [max_relay_log_size](https://mariadb.com/kb/en/replication-and-binary-log-server-system-variables/#max_relay_log_size)  system variable, has been reached

## Relay Log Names
By default, the relay log will be given a name `host_name-relay-bin.nnnnnn`, with `host_name` referring to the server's host name, and #nnnnnn `the sequence number.`

 This will cause problems if the replica's host name changes, returning the error `Failed to open the relay log` and `Could not find target log during relay log initialization`. To prevent this, you can specify the relay log file name by setting the [relay_log](https://mariadb.com/kb/en/replication-and-binary-log-server-system-variables/#relay_log) and [relay_log_index](https://mariadb.com/kb/en/replication-and-binary-log-server-system-variables/#relay_log_index) system variables.

## Viewing Relay Logs
The [SHOW RELAYLOG EVENTS](https://mariadb.com/kb/en/show-relaylog-events/) shows events in the relay log, and, since relay log files are the same format as binary log files, they can be read with the [mysqlbinlog](https://mariadb.com/kb/en/mysqlbinlog/) utility.

## Removing Old Relay Logs
Old relay logs are automatically removed once all events have been implemented on the replica, and the relay log file is no longer needed. This behavior can be changed by adjusting the [relay_log_purge](https://mariadb.com/kb/en/replication-and-binary-log-server-system-variables/#relay_log_purge) system variable from its default of `1` to `0`, in which case the relay logs will be left on the server.

보통 SQL Thread가 event를 읽고 나면 지운다. 하지만 SQL Thread가 멈추어 있으면 relay log는 계속해서 커지게 된다. 하여 , I/O thread는 자동으로 새 relay log file을 만들어 file이 너무 커지는 것을 막는다.

If the relay logs are taking up too much space on the replica, the [relay_log_space_limit](https://mariadb.com/kb/en/replication-and-binary-log-server-system-variables/#relay_log_space_limit) system variable can be set to limit the size. The IO thread will stop until the SQL thread has cleared the backlog. By default there is no limit.

> 출처: https://mariadb.com/kb/en/relay-log/
