# xxMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace 命名空间，作用就是对sql进行分类化管理,理解为sql隔离
 注意：使用mapper代理方法开发，namespace有特殊重要的作用
 -->
<mapper namespace="test">
    <!-- 在映射文件中配置很多sql语句 -->
    <!--需求:通过id查询用户表的记录 -->
    <!-- 通过select执行数据库查询
     id:标识映射文件中的sql，称为statement的id
     将sql语句封装到mappedStatement对象中，所以将id称为statement的id
     parameterType:指定输入参数的类型
     #{}标示一个占位符,
     #{value}其中value表示接收输入参数的名称，如果输入参数是简单类型，那么#{}中的值可以任意。

     resultType：指定sql输出结果的映射的java对象类型，select指定resultType表示将单条记录映射成java对象
     -->
    <select id="findUserById" parameterType="int" resultType="top.songfang.mybatistest.Psd">
        SELECT * FROM  psd  WHERE id=#{id}
    </select>

    <!-- 根据用户名称模糊查询用户信息，可能返回多条
	resultType：指定就是单条记录所映射的java对象类型
	${}:表示拼接sql串，将接收到参数的内容不加任何修饰拼接在sql中。
	使用${}拼接sql，引起 sql注入
	${value}：接收输入参数的内容，如果传入类型是简单类型，${}中只能使用value
	 -->
    <select id="findUserByAccount" parameterType="java.lang.String" resultType="top.songfang.mybatistest.Psd">
        SELECT * FROM psd WHERE account LIKE '%${value}%'
    </select>

    <!--
    插入数据 parameterType：指定输入数据类型，参数类型为pojo
    #{}中指定pojo的属性名，接受到pojo对象的属性值，mybatis通过OGNL获取对象的属性值
    -->
    <insert id="insertId" parameterType="top.songfang.mybatistest.Psd">
        <!--
        将插入数据的主键返回，返回到user对象中

        SELECT LAST_INSERT_ID():得到刚insert进去记录的主键值，只适用于自增主键
        keyProperty:将查询到主键值设置到parameterType指定的对象的哪个属性
        order：SELECT LAST_INSERT_ID()执行顺序，相对于insert语句来说它的执行顺序
        resultType：指定SELECT LAST_INSERT_ID()的结果类型
        -->
        <selectKey keyProperty="id" order="AFTER" resultType="int">
            SELECT LAST_INSERT_ID()
        </selectKey>
        INSERT INTO psd (account,password,tip) values (#{account},#{password},#{tips})
        <!--
    使用mysql的uuid（）生成主键
    执行过程：
    首先通过uuid()得到主键，将主键设置到user对象的id属性中
    其次在insert执行时，从user对象中取出id属性值
     -->
    <!--  <selectKey keyProperty="id" order="BEFORE" resultType="java.lang.String">
        SELECT uuid()
    </selectKey>
    insert into user(id,username,birthday,sex,address) value(#{id},#{username},#{birthday},#{sex},#{address}) -->
    </insert>

    <!-- 根据id进行删除 -->
    <delete id="deletePsd" parameterType="java.lang.Integer">
        delete from psd where id=#{id}
    </delete>

    <!-- 更新 -->
    <update id="updatePsd" parameterType="top.songfang.mybatistest.Psd">
        update psd set id=#{id},account=#{account},password=#{password},tip=#{tips} where id=#{id}
    </update>
</mapper>
```

