# SSM框架  
## Mybatis  
- jdbc问题总结如下：  
1、数据库连接创建、释放频繁造成系统资源浪费，从而影响系统性能。如果使用数据库连接池可解决此问题。  
2、Sql语句在代码中硬编码，造成代码不易维护，实际应用中sql变化的可能较大，sql变动需要改变java代码。  
3、使用preparedStatement向占有位符号传参数存在硬编码，因为sql语句的where条件不一定，可能多也可能少，修改sql还要修改代码，系统不易维护。  
4、对结果集解析存在硬编码（查询列名），sql变化导致解析代码变化，系统不易维护，如果能将数据库记录封装成pojo对象解析比较方便。  
- Mybatis架构  
![](http://oumcs7yiy.bkt.clouddn.com/SSM-1.png)  
- \#{}和${}  
```
#{}表示一个占位符号，通过#{}可以实现preparedStatement向占位符中设置值，自动进行java类型和jdbc类型转换。  
#{}可以有效防止sql注入。 #{}可以接收简单类型值或pojo属性值。 
如果parameterType传输单个简单类型值，#{}括号中可以是value或其它名称。  
${}表示拼接sql串，通过${}可以将parameterType 传入的内容拼接在sql中且不进行jdbc类型转换,   
${}可以接收简单类型值或pojo属性值，如果parameterType传输单个简单类型值，${}括号中只能是value。  

```

- 两种方式实现根据用户名模糊查询用户
```xml
    <!-- 如果返回多个结果，mybatis会自动把返回的结果放在list容器中 -->
    <!-- resultType的配置和返回一个结果的配置一样 -->
    <select id="queryUserByUsername1" parameterType="string"
        resultType="cn.itcast.mybatis.pojo.User">
        SELECT * FROM `user` WHERE username LIKE #{username}
    </select>
```
```java
@Test
    public void testQueryUserByUsername1() throws Exception {
        // 4. 创建SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        // 5. 执行SqlSession对象执行查询，获取结果User
        // 查询多条数据使用selectList方法
        List<Object> list = sqlSession.selectList("queryUserByUsername1", "%王%");

        // 6. 打印结果
        for (Object user : list) {
            System.out.println(user);
        }

        // 7. 释放资源
        sqlSession.close();
    }

```
```xml
    <!-- 如果传入的参数是简单数据类型，${}里面必须写value -->
    <select id="queryUserByUsername2" parameterType="string"
        resultType="cn.itcast.mybatis.pojo.User">
        SELECT * FROM `user` WHERE username LIKE '%${value}%'
    </select>
```
```java
@Test
public void testQueryUserByUsername2() throws Exception {
    // 4. 创建SqlSession对象
    SqlSession sqlSession = sqlSessionFactory.openSession();

    // 5. 执行SqlSession对象执行查询，获取结果User
    // 查询多条数据使用selectList方法
    List<Object> list = sqlSession.selectList("queryUserByUsername2", "王");

    // 6. 打印结果
    for (Object user : list) {
        System.out.println(user);
    }

    // 7. 释放资源
    sqlSession.close();
}
```

- DAO的Mapper动态代理方式  
Mapper接口开发需要遵循以下规范：  
1、  Mapper.xml文件中的namespace与mapper接口的类路径相同  
2、  Mapper接口方法名和Mapper.xml中定义的每个statement的id相同   
3、  Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql 的parameterType的类型相同  
4、  Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同  

- selectOne和selectList  
动态代理对象调用sqlSession.selectOne()和sqlSession.selectList()  是根据mapper接口方法的返回值决定，如果返回list则调用selectList方法，如果返回单个对象则调用selectOne方法。  

- properties和typeAliases  
```xml
    <properties resource="jdbc.properties"/>
    <!-- 别名 包以其子包下所有类   头字母大小都行-->
    <typeAliases>
<!--        <typeAlias type="com.itheima.mybatis.pojo.User" alias="User"/> -->
        <package name="com.itheima.mybatis.pojo"/>
    </typeAliases>
```


- sqlMapConfig中mapper配置  
使用mapper接口类路径  
如：`<mapper class="cn.itcast.mybatis.mapper.UserMapper"/>`  
注意：此种方法要求mapper接口名称和mapper映射文件名称相同，且放在同一个目录中。  
注册指定包下的所有mapper接口  
如：`<package name="cn.itcast.mybatis.mapper"/>`  
注意：此种方法要求mapper接口名称和mapper映射文件名称相同，且放在同一个目录中。  
package是主要方式  
