1.使用别名

    SELECT column_name AS columnName FROM table

2.配置文件

    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>

3.resultMap

    <resultMap type="User" id="ResultMapUser">
        <id column="user_id" property="userId"/>
        <result column="user_name" property="userName"/>
    </resultMap>
    <select id="selectUserByUserId" resultMap="ResultMapUser">
        select * from user where user_id=#{userId}
    </select>

---
