---
description: >-
  For those who stuck with AMIv1, and their application is connecting to a
  PostgreSQL 15 database.
---

# 🐘 PostgreSQL12 on Amazon Linux 1

Current AMIv1 only contains up to PostgreSQL version 9.6 as per: [https://aws.amazon.com/amazon-linux-ami/2018-03-packages/](https://aws.amazon.com/amazon-linux-ami/2018-03-packages/)

```bash
yum search postgresql

...
postgresql95.x86_64 : PostgreSQL client programs
postgresql95-contrib.x86_64 : Extension modules distributed with PostgreSQL
postgresql95-devel.x86_64 : PostgreSQL development header files and libraries
...
postgresql96-server.x86_64 : The programs needed to create and run a PostgreSQL server
postgresql96-static.x86_64 : Statically linked PostgreSQL libraries
postgresql96-test.x86_64 : The test suite distributed with PostgreSQL
...
```

thus for application required connection to PostgreSQL 15 would fail miserably with this error:

```
CRAM authentication requires libpq version 10 or above (PG::ConnectionBad)
```

PostgreSQL version before 12 is no longer available on all PostgreSQL repositories. Yet, the archive yum repository is currently in-accessiable:

```
https://yum-archive.postgresql.org/
```

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption><p>no hope</p></figcaption></figure>

The only way right now is to install postgresql 12 and pray that nothing will break (yes, jumping 3 major version is something need to be carefully evaluated)

```
sudo yum install -y https://download.postgresql.org/pub/repos/yum/12/redhat/rhel-6-x86_64/postgresql12-libs-12.9-1PGDG.rhel6.x86_64.rpm
sudo yum install -y https://download.postgresql.org/pub/repos/yum/12/redhat/rhel-6-x86_64/postgresql12-12.9-1PGDG.rhel6.x86_64.rpm
sudo yum install -y https://download.postgresql.org/pub/repos/yum/12/redhat/rhel-6-x86_64/postgresql12-devel-12.9-1PGDG.rhel6.x86_64.rpm
```
