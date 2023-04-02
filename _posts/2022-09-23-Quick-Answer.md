---
layout: post
title: 🗂 【Daily】Question with answer - 2022
date: 2022-09-23 00:00
---

## Summary
This post contains some short answers of the technology questions I met in my daily work in 2022, including some other LINK for further details answers. Will keep updating this post for a year.
### Why we need replication (master-slave) of database? 
1. Avoid single point failure
2. Scalability
3. Performance
4. Data security when running backup
5. Analytics

- [Mysql Replication](https://dev.mysql.com/doc/refman/8.0/en/replication.html)
- [Master-Slave Replication](https://hevodata.com/learn/mysql-master-slave-replication/#a2)

---

### How replication synchronize data from master?
Basically, all the events ran in master db, will re-run in replication db as well to achieve synchronization. Every time a event happened in master db, will be written to bin-log. Then the replication read the bin-log and run the events.

- [How replication runs](https://dev.mysql.com/doc/refman/8.0/en/replication-formats.html)

---

### Replication format?
Bin-log have two different formats, could choose one of them or mix of them. 
1. Statement-based
2. Row-based

- [Compare Statement-based and Row-based](https://dev.mysql.com/doc/refman/8.0/en/replication-sbr-rbr.html)

---
### What is message-oriented middleware?
Message-oriented middleware is a kind of infrastructure which supports sending and receiving data (message) to and from distributed systems. Both sending and receiving data is asynchronous in MOM, which means the sender application could just focusing on sending message, and doesn't need to put effect on how the message been received. Generally, a MOM consists of three main parts: **producer**, **message queue**, **consumer**.

Advantage:
1. Lower coupling
2. Scalability 
3. Asynchronous
4. Flexibility

Disadvantage:
5. Topology

![Message-Oriented Middleware](https://typora-1302119905.cos.ap-nanjing.myqcloud.com/Coding/MessageOrientedMiddleware.png)

- [Kafka as message-oriented middleware](https://www.oreilly.com/library/view/data-lake-for/9781787281349/01b64a9d-bb69-4df8-a6ed-cd15e2ff8e46.xhtml)
- [Message-oriented Architectures](https://blogs.msmvps.com/peterritchie/2011/07/14/message-oriented-architectures/)
- [What is Message Oriented Middleware (MOM)?](https://www.geeksforgeeks.org/what-is-message-oriented-middleware-mom/)
