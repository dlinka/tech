.gitignore和exclude的区别

    两个文件的本质都是用来忽略一些项目文件,用处不同而已
    
    .gitignore文件会被提交到仓库,一般这个文件内容忽略的东西也是别人想忽略的
    exclude文件不需要提交到仓库,忽略个人文件

git命令

    //查看每行的提交记录
    git blame file

.gitignore规则

    *.class //忽略所有class
    !HelloWorld.class //HelloWorld.class不忽略
    /*.iml //只会忽略根目录的iml文件
    aaa/ //忽略任意aaa目录
    aaa //忽略任意aaa目录和文件
    
