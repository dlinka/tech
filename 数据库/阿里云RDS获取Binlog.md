下载Binlog

    //查看Binlog名字
    show binary logs
    //下载Binlog到本地
    mysqlbinlog -u[$User] -p[$Password] -h[$Host] --read-from-remote-server mysql-bin.xxx > [$File_Name]
    上面这种方式经常下不完整,然后导致解析失败
    
    推荐使用阿里云控制台下载
    选择实例->备份恢复->日志备份
    
解析Binlog

    mysqlbinlog -v --base64-output=decode-rows [$File_Name]
    

[-v、-vv、--base64-output参数](https://blog.csdn.net/jolly10/article/details/80077366)
