插入

```sql
避免重复插入

    INSERT IGNORE INTO user(uid, username) VALUES(1, 'cr')
    
如果存在则更新

    INSERT INTO user(uid, username) VALUES(1, 'cr') ON DUPLICATE KEY UPDATE username='CR'
    
如果存在则删除再插入

    REPLACE INTO user(uid, username) VALUES(1, 'cr')
```



函数

IFNULL

```sql
这行SQL查不出来real_name等于NULL数据
SELECT * FROM user_profile WHERE real_name!=0
↓
↓
SELECT * FROM user_profile WHERE IFNULL(real_name, -1)!=0
```

