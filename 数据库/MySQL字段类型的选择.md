时间类型

    timestamp占用4个字节
    datetime占用8个字节
    
    假设我的一个项目有十个表,每个表都有create_time,update_time,每个表最少一千万的数据
    使用timestamp会比使用datetime剩下多少空间的大小
    4*2(个字段)*10(个表)*10000000=8亿个字节
    8亿个字节约等于800MB

---

DECIMAL

    1~2位,用1个字节
    3~4位,用2个字节
    5~6位,用3个字节
    7~9位,用4个字节
    根据上面规则计算DECIMAL(20,6)的占用空间
    小数部分6位数字,整数部分14位数字
    小数部分6位根据上面规则需要3个字节
    整数部分9位需要4个字节,其余5位需要3个字节
    上面合起来就是10个字节


    我记性不是很好,经常忘记DECIMAL(M,N)中M,N的含义🤣🤣🤣
    但是记不住我也不担心,因为我知道怎么去验证它
    create table test(
        id int(11) not null,
        decimal_field decimal(10,2) not null,
        primary key(id)
    )
    -- 插入整数10位
    -- Out of range value for column 'decimal_field' at row 1
    insert into test(id,decimal_field) values(1, 1234567890);

    -- 插入整数8位
    -- 数据库查询结果12345678.00
    insert into test(id,decimal_field) values(1, 12345678);
    
    -- 插入整数2位,小数1位
    -- 数据库查询结果12.10
    insert into test(id,decimal_field) values(2, 12.1);
    
    -- 插入整数8位,小数5位
    -- 数据库查询结果12345678.12,发生了截断
    insert into test(id,decimal_field) values(3, 12345678.12345);

---
