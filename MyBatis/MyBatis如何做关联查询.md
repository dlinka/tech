1.级联的方式

    <resultMap id="UserResultMap" type="User">
        <id column="user_id" property="userId"/>
        <result column="user_name" property="userName"/>
        <result column="company_id" property="company.companyId"/>
        <result column="company_name" property="company.companyName"/>
    </resultMap>
    <select id="selectUser" resultMap="UserResultMap">
        SELECT user_id, user_name, c.company_id AS company_id, company_name
        FROM user AS u
        INNER JOIN company AS c ON c.company_id=u.company_id
        WHERE u.user_id = #{userId}
    </select>
    
2.利用association

    <resultMap id="UserResultMap" type="User">
        <id column="user_id" property="userId"/>
        <result column="user_name" property="userName"/>
        <association property="company" javaType="Company">
            <id column="company_id" property="companyId"/>
            <result column="company_name" property="companyName"/>
        </association>
    </resultMap>
    <select id="selectUser" resultMap="UserResultMap">
        SELECT user_id, user_name, c.company_id AS company_id, company_name
        FROM user AS u
        INNER JOIN company AS c ON c.company_id = u.company_id
        WHERE u.user_id = #{userId}
    </select>

3.利用association

    <resultMap id="UserResultMap" type="User">
        <id column="user_id" property="userId"/>
        <result column="user_name" property="userName"/>
        <association property="company" select="CompanyMapper.selectCompanyByCompanyId" column="company_id"/>
    </resultMap>
    <select id="selectUser" resultMap="UserResultMap">
        select * from user where user_id = #{userId}
    </select>
    
---

查询集合属性

    <resultMap id="CompanyResultMap" type="Company">
        <id column="company_id" property="companyId"/>
        <result column="company_name" property="companyName"/>
        <collection property="userList" ofType="User">
            <id column="user_id" property="userId"/>
            <result column="user_name" property="userName"/>
        </collection>
    </resultMap>
    <select id="selectCompany" resultMap="CompanyResultMap">
        SELECT c.company_id AS company_id, company_name, user_id, user_name
        FROM company AS c
        INNER JOIN user u ON u.company_id = c.company_id
        WHERE c.company_id = #{companyId}
    </select>

    <resultMap id="CompanyResultMap" type="Company">
        <id column="company_id" property="companyId"/>
        <result column="company_name" property="companyName"/>
        <collection property="userList" select="UserMapper.selectUserListByCompanyId" column="company_id"/>
    </resultMap>
    <select id="selectCompany" resultMap="CompanyResultMap">
        select * from company where company_id = #{companyId}
    </select>
    
---
