
<!-- mtoc-start -->

* [postgresql](#postgresql)
  * [安装](#安装)
    * [集成PostgreSQL Docker镜像，已集成热门插件和工具](#集成postgresql-docker镜像已集成热门插件和工具)
  * [插件](#插件)
    * [DuckDB](#duckdb)
    * [nextgres：允许MySQL应用程序在PostgreSQL上无缝运行，无需任何代码更改](#nextgres允许mysql应用程序在postgresql上无缝运行无需任何代码更改)
  * [OLAP](#olap)
  * [隔离级别](#隔离级别)
  * [第三方工具](#第三方工具)
* [reference](#reference)

<!-- mtoc-end -->

# postgresql

## 安装

- [archlinux安装](https://www.netarky.com/programming/arch_linux/Arch_Linux_PostgreSQL_database_setup.html)

- [IT 邦德： 避坑！手把手教你7种安装PG16的手艺](https://mp.weixin.qq.com/s/QxnABtyPlmEMuvopmzNcVQ)

### 集成PostgreSQL Docker镜像，已集成热门插件和工具

```sh
# 拉取镜像, 第一次拉取一次即可. 或者需要的时候执行, 将更新到最新镜像版本.
docker pull registry.cn-hangzhou.aliyuncs.com/digoal/opensource_database:pg14_with_exts_arm64

# 启动容器
docker run -d -it -P --cap-add=SYS_PTRACE --cap-add SYS_ADMIN --privileged=true --name pg --shm-size=1g registry.cn-hangzhou.aliyuncs.com/digoal/opensource_database:pg14_with_exts_arm64

##### 如果你想学习备份恢复、修改参数等需要重启数据库实例的case, 换个启动参数, 使用参数--entrypoint将容器根进程换成bash更好. 如下:
docker run -d -it -P --cap-add=SYS_PTRACE --cap-add SYS_ADMIN --privileged=true --name pg --shm-size=1g --entrypoint /bin/bash registry.cn-hangzhou.aliyuncs.com/digoal/opensource_database:pg14_with_exts_arm64
##### 以上启动方式需要进入容器后手工启动数据库实例: su - postgres; pg_ctl start;

# 进入容器
docker exec -ti pg bash

# 连接数据库
psql
```

## 插件

![image](./Pictures/postgresql/插件-生态.avif)

![image](./Pictures/postgresql/插件-clickhouse性能对比.avif)

- [非法加冯：技术极简主义：一切皆用Postgres](https://mp.weixin.qq.com/s?__biz=MzU5ODAyNTM5Ng==&mid=2247486931&idx=1&sn=91dbe43bb6d26c760c532f4aa8d6e3cb&chksm=fe4b3808c93cb11e00194655a49bf7aa0d4d05a61a9b06ffcc57017c633de17066443ec62b6d&scene=21#wechat_redirect)

- [apache age：图数据库，可以使用图模型查询](https://age.apache.org/)

- [FerretDB](https://github.com/FerretDB/FerretDB)
    - 底层采用 PostgreSQL 作为存储引擎，用 Go 语言实现了 MongoDB 协议

### DuckDB

- 就像做在线业务时，首选数据库是 PostgreSQL 一样，如今做分析时的 “默认之王” 就是 DuckDB。以前大家可能还会说用 Pandas，但现在几乎一开口就是“DuckDB 走起”。这货特别轻便，所以很多人想把它塞进那些本身对 OLAP 支持不是特别好的数据库。今年，我们就看到四款把 DuckDB 集成到 Postgres 的扩展相继亮相。

    - 第一枪是 2024 年 5 月，Crunchy Data 宣布做了个专有扩展，把 Postgres 重定向到 DuckDB 来处理 OLAP 查询。随后他们又搞了个更厉害的版本，利用 DuckDB 的空间扩展 来加速 PostGIS 查询。

    - 2024 年 6 月，ParadeDB发布了一个开源扩展（pg_analytics），用 Postgres 的 FDW API 去调用 DuckDB。在此之前，他们用的是 DataFusion（pg_lakehouse），后来改用 DuckDB。

    - 8 月，官方版的 DuckDB-for-Postgres 出炉了（pg_duckdb），托管在 DuckDB Labs 的 GitHub 下，算是名正言顺的 DuckDB 官方插件。原本宣传说这是 MotherDuck、Hydra、Microsoft 和 Neon 联合开发，结果后来据说 Microsoft 和 Neon 因为开发管理问题被“踢出去”了，就跟 阿拉伯王子 离开 NWA 一样。现在只剩 MotherDuck 和 Hydra 继续干。

    - 11 月又来一个 pg_mooncake 插件（博文），这次是 Mooncake Labs 出品。它跟前面三个不太一样，是可以通过 Postgres 把数据写进 Iceberg 表里，还支持事务。

### nextgres：允许MySQL应用程序在PostgreSQL上无缝运行，无需任何代码更改

## OLAP

- 现在 PostgreSQL 生态的一个主要遗憾是，缺少一个足够好的列式存储分析插件来做 OLAP 分析。

    - 然而， ParadeDB 和 DuckDB 的出现改变了这一点！

    - ParadeDB 提供的 PG 原生扩展 pg_analytics 实现了第二梯队（x10）的性能水准，与第一梯队只有 3～4 倍的性能差距。相对于其他功能上的收益，这种程度的性能差距通常是可以接受的 —— ACID，新鲜性与实时性，无需 ETL、额外学习成本、维护独立的新服务，更别提它还提供了 ElasticSearch 质量的全文检索能力。
    - 而 DuckDB 则专注于 OLAP ，将分析性能这件事做到了极致（x3.2） —— 略过第一名 Umbra 这种学术研究型闭源数据库，DuckDB 也许是 OLAP 实战性能最快的数据库了。它并不是 PG 的扩展插件，但它是一个嵌入式文件数据库，而 DuckDB FDW 以及 pg_quack 这样的 PG 生态项目，能让 PostgreSQL 充分利用 DuckDB 带来的完整分析性能红利！

## 隔离级别

- [AustinDatabases：PostgreSQL 为什么也不建议 RR隔离级别，MySQL别笑](https://mp.weixin.qq.com/s/X51JO1-eg1cPPs91pTCfIA)

## 第三方工具

- [pgbackweb：自动化备份](https://github.com/eduardolat/pgbackweb)

- 客户端

    - [rainfrog：tui支持类似 Vim 的快捷键、关键字高亮和历史记录等人性化功能。](https://github.com/achristmascarl/rainfrog)

    - [pgmanage：gui](https://github.com/commandprompt/pgmanage)

# reference

- [PostgreSQL码农集散地：总算摸到门了！PostgreSQL及国产开源数据库学习进阶之路。](https://mp.weixin.qq.com/s/RCg5tdn1i7yBl2ZZy-ZaIw)
