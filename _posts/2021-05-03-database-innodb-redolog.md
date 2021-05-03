---
title: "InnoDB의 Redo Logging"
date: 2021-05-03 14:53:28 -0400
categories: database
tags:
  - database
  - mysql
  - mariadb
  - innodb
comments: true
---

## Redo logging process
MariaDB도 Oracle과 같이 data 안전을 보장하고 성능을 향상시키기 위해 commit된 transaction 내용을 먼저 log file로 기록하고, 실제 data file(disk)의 변경은 나중에 모아서 batch 형태로 처리한다. => WAL (Write Ahead Logging)
![](https://sarc.io/images/innodb-transactions.png)

-  data 변경 시 해당 되는 page를 수정 후 Dirty 마크를 표시 (innodb_buffer_pool)
- 관련 Redo log record를 memory 상의 내부적인 log 관련 memory 에 저장함
- Rego log record를 innodb_log_buffer로 이동함
- Redo log record를 redo log file에 flush함
- buffer pool 내의 변경 된 dirty page를 checkpoint를 수행하여 Disk에 저장함

## 관련 Parameter
- innodb_flush_log_at_trx_commit (default: 1)
	- transaction commit 명령이 실행 될 경우 변경된 내용이 disk에 언제 반영될 지 설정 
![](https://sarc.io/images/innodb_flush_log_at_commit.png)
	- 해당 값에 따라 순간적인 장애 시 transaction을 잃을 수 있음
		-	0: 1s마다 log buffer의 내용을 log file로 내려 씀. MySQL이나 OS가 갑자기 crash 되면 최대 1s 동안의 transaction을 잃을 수 있음
		-	1: log file로 바로 내려 쓰므로 안전하나 성능이 느림
		-	2: OS buffer/cache로 내려쓰고 1s마다 log file로 내려씀.  OS가 갑자기 crash되면 최대 1s동안의 transaction을 잃을 수 있으나, MySQL 장애시에는 이미 OS 영역으로 data는 넘어갔기 때문에 안전할 수 있음
	- sync_binlog=1 + innodb_flush_log_at_trx_commit=1이 가장 안전

- innodb_max_dirty_page_pct=0~99.999 (default : 75)
	- buffer pool에서 dirty page를 몇 %까지 허용할지 결정
	- 비율 초과 시 innodb_io_capacity parameter로 정한 값 만큼 한번에 page를 flush 시킴
	- 이 값을 낮출 시 buffer pool의 효율성이 떨어지게 되여 Disk I/O가 높아질수 있으나, 너무 높으면 redo file이 full이 나서 log기록을 할 수 없는 현상 발생할 수 있음

- innodb_io_capacity=100~2^64-1 (default: 200)
	- innodb_max_dirty_page_pct parameter로 명시한 값 만큼 buffer pool내 dirty page가 늘어나면 이 parameter 값 만큼 한번에 page를 flush 시킴

- innodb_log_buffer_size
	- redo(리두) log를 file에 직접 기록하기 전, memory 상에서 buffering을 하는데 이를 위한 buffer size

- innodb_log_files_in_group (default: 2)
	- redo log의 갯수

- innodb_log_file_size
	- redo log file 크기로 일반적으로 innodb_buffer_pool_size / innodb_log_files_in_group을 적정 값으로 봄

> 출처: https://sarc.io/index.php/mariadb/1146-innodb-redo-logging-process
