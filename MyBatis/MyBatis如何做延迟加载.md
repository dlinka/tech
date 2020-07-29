配置文件

    <settings>
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="aggressiveLazyLoading" value="false"/>
    </settings>

Mapper XML

    <resultMap id="userResultMap" type="User">
        <id column="user_id" property="userId"/>
        <result column="user_name" property="userName"/>
        //延迟加载company属性
        <association property="company" column="company_id" select="CompanyMapper.selectCompanyByCompanyId" />
        //eager会取消延迟加载
        <association property="company" column="company_id" select="CompanyMapper.selectCompanyByCompanyId" fetchType="eager"/>
    </resultMap>
    <select id="selectUser" resultMap="userResultMap">
        select * from user where user_id = #{userId}
    </select>

---
