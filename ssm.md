# SSM框架  
## Mybatis  
- jdbc问题总结如下：  
1、数据库连接创建、释放频繁造成系统资源浪费，从而影响系统性能。如果使用数据库连接池可解决此问题。  
2、Sql语句在代码中硬编码，造成代码不易维护，实际应用中sql变化的可能较大，sql变动需要改变java代码。  
3、使用preparedStatement向占有位符号传参数存在硬编码，因为sql语句条件不一定，还要修改代码，系统不易维护。  
4、对结果集解析存在硬编码，sql变化导致解析代码变化，系统不易维护，将数据库记录封装成pojo对象比较方便。  
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
**package是主要方式**  


- parameterType(输入类型)pojo包装对象  
QueryVo包装类  
```java
public class QueryVo {
    // 包含其他的pojo
    private User user;

    public User getUser() {
        return user;
    }
    public void setUser(User user) {
        this.user = user;
    }
}
```
Mapper.xml，要通过user.username方式传递  
```xml
    <!-- 根据用户名模糊查询 -->
    <select id="findUserByQueryVo" parameterType="QueryVo" resultType="com.itheima.mybatis.pojo.User">
        select * from user where username like "%"#{user.username}"%"
    </select>
```

- resultType(输出类型)简单类型  
```java
public Integer countUser();
```
```xml
    <select id="countUser" resultType="Integer">
        select count(1) from user
    </select>

```
- resultMap  
字段名不一致要手动映射
一样的字段可以省略  
```xml
    <resultMap type="Orders" id="orders">
        <result column="user_id" property="userId"/>
    </resultMap>
    <select id="selectOrdersList" resultMap="orders">
        SELECT id, user_id, number, createtime, note FROM orders 
    </select>
```
- 动态sql，if和where  
where标签可以去掉第一个前and  
```xml
 <!--   根据性别和名字查询用户  where 可以去掉第一个前ANd   -->
     <select id="selectUserBySexAndUsername" parameterType="User" resultType="User">
        <include refid="selector"/>
        <where>
            <if test="sex != null and sex != ''">
                 and sex = #{sex} 
            </if>
            <if test="username != null and username != ''">
                 and username = #{username}
            </if>
        </where>
     </select>
```
- Sql片段  
调用上面的片段  
```xml
    <sql id="selector">
        select * from user
    </sql>

    <select id="selectUserBySexAndUsername" parameterType="User" resultType="User">
        <include refid="selector"/>
        <where>
            <if test="sex != null and sex != ''">
                 and sex = #{sex} 
            </if>
            <if test="username != null and username != ''">
                 and username = #{username}
            </if>
        </where>
     </select>
```
- foreach标签，多id查询，三种方式  
collection的值可为list、array、idslist(当用QueryVo传递时，使用QueryVo类中成员名)  
```xml
     <!-- 多个ID (1,2,3)-->
     <select id="selectUserByIds" parameterType="QueryVo" resultType="User">
        <include refid="selector"/>
        <where>
            <foreach collection="list" item="id" separator="," open="id in (" close=")">
                #{id}
            </foreach>
        </where>
     </select>
```
```java
public void testfindUserIDs() throws Exception {
        //加载核心配置文件
        String resource = "sqlMapConfig.xml";
        InputStream in = Resources.getResourceAsStream(resource);
        //创建SqlSessionFactory
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
        //创建SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        List<Integer> ids  = new ArrayList<>();
        ids.add(16);
        ids.add(22);
        ids.add(24);
        /*  QueryVo vo = new QueryVo();
        vo.setIdsList(ids);*/
        
//      List<User> users = userMapper.selectUserByIds(vo);
/*      Integer[] ids = new Integer[3];
        ids[0] = 16;
        ids[2] = 22;
        ids[1] = 24;*/
        List<User> users = userMapper.selectUserByIds(ids);
        for (User user2 : users) {
            System.out.println(user2);
        }
    }
```
```java
    public List<User> selectUserByIds(Integer[] ids);array
    public List<User> selectUserByIds(List<Integer> ids); 
    public List<User> selectUserByIds(QueryVo vo);
```
- 一对一关联  
一对一关联，要在order中内建user对象，用resultMap进行关联  
column和property框架不能自动补全  
```xml
    <!-- 
        //一对一关联 查询  以订单为中心 关联用户
    public List<Orders> selectOrders();
     -->
     <resultMap type="Orders" id="order">
        <result column="id" property="id"/>
        <result column="user_id" property="userId"/>
        <result column="number" property="number"/>
        <!-- 一对一 -->
        <association property="user" javaType="User">
            <id column="user_id" property="id"/>
            <result column="username" property="username"/>
        </association>
     </resultMap>
```

- 一对多关联  
ofType的值是list中的泛型  
```xml
     <!-- //一对多关联 public List<User> selectUserList(); -->
    <resultMap type="User" id="user">
        <id column="user_id" property="id"/>
        <result column="username" property="username"/>
        <!-- 一对多 -->
        <collection property="ordersList" ofType="Orders">
            <id column="id" property="id"/>
            <result column="number" property="number"/>
        </collection>
    </resultMap>
    <select id="selectUserList" resultMap="user">
        SELECT 
        o.id,
        o.user_id, 
        o.number,
        o.createtime,
        u.username 
        FROM user u
        left join orders o 
        on o.user_id = u.id
    </select>
```

- Mybatis整合spring  
整合思路  
1、SqlSessionFactory对象应该放到spring容器中作为单例存在。  
2、传统dao的开发方式中，应该从spring容器中获得sqlsession对象。  
3、Mapper代理形式中，应该从spring容器中直接获得mapper的代理对象。  
4、数据库的连接以及数据库连接池事务管理都交给spring容器来完成。  

