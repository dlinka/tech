如何查询是否存在

    生产环境有个10万行的表fitness_session

    SQL
    select count(*) from fitness_session where device_id=10000058 and status=2
    执行时长平均50ms,结果3644,表示所有满足条件的数量

    优化方案
    select 1 from fitness_session where device_id=10000058 and status=2 limit 1
    执行时长平均25ms,结果就是1
    
    让数据库查询时遇到1条就返回,不用继续查询还有多少条了

---

更新大量数据

    更新所有已过期优惠券
    如果行数量很大,这种SQL可能会导致MySQL卡死
    update status=2 from coupon WHERE expire_date <= now()
    
    解决方案:
    分批处理,for循环,1次更新100条

---

or不会使用索引

    i1和d1是一个组合索引
    此时并不会使用索引(type=ALL,Extra=Using where)
    select * from table where i1=1 or d1=1

    可以分别在i1和d1上建立索引
    然后使用如下SQL优化方案
    (select * from table where i1=1)
    union
    (select * from table where d1=1)

---

分页查询

    10万行的表
    //执行平均时长10ms
    select * from fitness_session order by session_id limit 0,10
    //执行平均时长45ms
    select * from fitness_session order by session_id limit 90000,10
    通过结果可以看出分页查询越到后面时间越长

    优化方案1
    //执行平均时长30ms
    select * from fitness_session where session_id >=
    (select session_id from fitness_session order by session_id limit 80000,1)
    order by session_id limit 0,10;

    优化方案2
    //假设session_id为连续的
    select * from fitness_session where session_id between 180072 and 180081;

---

利用位计算

    UPDATE table SET column=column|4
    27|4
    ↓
    0001 1011|0000 0100
    ↓
    0001 1111
    ↓
    31

    UPDATE table SET column=column|8 WHERE column&1=1
    3847|8
    ↓
    1111 0000 0111|0000 0000 1000
    ↓
    1111 0000 1111
    ↓
    3855

    3847&1
    ↓
    1111 0000 0111&0000 0000 0001
    ↓
    0000 0000 0001
    ↓
    1
