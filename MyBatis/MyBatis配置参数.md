typeAliases

    <typeAliases>
        //为com.cr.entity包下面所有的类包括子包里面的类起别名
        //默认别名为类名,不区分大小写
        //如果存在别名冲突的话,可以使用@Alias注解为某个类单独指定别名
        <package name="com.cr.entity"/>
    </typeAliases>
    
mappers

    //映射文件放到resources目录下,目录结构和接口的包名一致,映射文件和接口同名
    <mappers>
        <package name="com.cr.dao"/>
    </mappers>
