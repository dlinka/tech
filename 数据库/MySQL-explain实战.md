前置知识

    了解B+Tree
    了解索引覆盖,回表查询概念
    了解type列,Extra列

测试脚本

    create table explain_test (
        i1 int,
        i2 int,
        d1 date,
        i3 int,
        primary key (i1, i2)
    ) engine = innodb

    insert into explain_test values
    (1, 1, '1998-01-01',1),(1, 2, '1999-01-01',2),(1, 3, '2000-01-01',1),(1, 4, '2001-01-01',2),(1, 5, '2002-01-01',1),
    (2, 1, '1998-01-01',2),(2, 2, '1999-01-01',1),(2, 3, '2000-01-01',2),(2, 4, '2001-01-01',1),(2, 5, '2002-01-01',2),
    (3, 1, '1998-01-01',1),(3, 2, '1999-01-01',2),(3, 3, '2000-01-01',1), (3, 4, '2001-01-01',2),(3, 5, '2002-01-01',1),
    (4, 1, '1998-01-01',2),(4, 2, '1999-01-01',1),(4, 3, '2000-01-01',2),(4, 4, '2001-01-01',1),(4, 5, '2002-01-01',2),
    (5, 1, '1998-01-01',1),(5, 2, '1999-01-01',2),(5, 3, '2000-01-01',1), (5, 4, '2001-01-01',2),(5, 5, '2002-01-01',1); 

    第4个实验后增加非聚集索引
    create index d1_index on explain_test(d1)

测试结果

1.type=ALL,Extra=Using where

    ALL代表全表扫描
    然后使用where进行过滤
    这种性能最差
    explain select * from explain_test where d1='2000-01-01'

2.type=ref,Extra=Using where

    ref使用到了primary key
    找到primary key中包含i1的所有行(i1非唯一,i1和i2是联合唯一索引)
    然后使用where进行过滤
    explain select * from explain_test where i1=1 and d1='2000-01-01'

3.type=ALL,Extra=Using where

    i2不是primary key的索引前缀
    explain select * from explain_test where i2=2 and d1='2000-01-01'

4.type=ref,Extra=Using index

    ref使用到了d1_index索引
    这种就是索引覆盖,表示通过一个索引树就可以查询到想要的结果
    explain select d1 from explain_test where d1='2000-01-01'

5.type=ref,Extra=NULL

    ref使用到了d1_index索引
    由于是select *,所以还需要去primary key索引上找其他列
    这种就是回表查询
    explain select * from explain_test where d1='2000-01-01'

6.type=range,Extra=Using where; Using index

    range使用到了d1_index索引,而且是范围查找
    扫描索引的过程中使用了where进行过滤
    explain select d1 from explain_test where d1 > '2020-01-01'

---
