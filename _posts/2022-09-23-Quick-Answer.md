---
layout: post
title: 🔖 Quick Answer
date: 2022-09-23 00:00
---

## Summary
This post contains some short answers of the technology questions I met in my daily work, including some other LINK for further details answers.

#### Why we need replication (master-slave) of database? 
1. Avoid single point failure
2. Scalability
3. Performance
4. Data security when running backup
5. Analytics

- [Mysql Replication](https://dev.mysql.com/doc/refman/8.0/en/replication.html)
- [Master-Slave Replication](https://hevodata.com/learn/mysql-master-slave-replication/#a2)

#### How replication synchronize data from master?
Basically, all the events ran in master db, will re-run in replication db as well to achieve synchronization. Every time a event happened in master db, will be written to bin-log. Then the replication read the bin-log and run the events.

- [How replication runs](https://dev.mysql.com/doc/refman/8.0/en/replication-formats.html)

#### Replication format?
Bin-log have two different formats, could choose one of them or mix of them. 
1. Statement-based
2. Row-based

- [Compare Statement-based and Row-based](https://dev.mysql.com/doc/refman/8.0/en/replication-sbr-rbr.html)
