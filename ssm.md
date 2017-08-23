# SSM框架  
## Mybatis  
- jdbc问题总结如下：  
1、数据库连接创建、释放频繁造成系统资源浪费，从而影响系统性能。如果使用数据库连接池可解决此问题。  
2、Sql语句在代码中硬编码，造成代码不易维护，实际应用中sql变化的可能较大，sql变动需要改变java代码。  
3、使用preparedStatement向占有位符号传参数存在硬编码，因为sql语句条件不一定，还要修改代码，系统不易维护。  
4、对结果集解析存在硬编码，sql变化导致解析代码变化，系统不易维护，将数据库记录封装成pojo对象比较方便。  
- Mybatis架构  
![](http://oumcs7yiy.bkt.clouddn.com/SSM-1.png)  

- mapper入门  
parameterType传参类型，resultType自动返回类型，要求数据库字段和pojo一一对应  
resultType设置了别名就不用写全类名了  
```xml
<select id="findUserById" parameterType="Integer" resultType="User">
        select * from user where id = #{v}
</select>
```

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
- 增删改查  
添加用户并返回用户的id  
改用户信息，要传入user对象     
```xml
<!-- 写Sql语句   -->
<mapper namespace="com.itheima.mybatis.mapper.UserMapper">
    <!-- 通过ID查询一个用户 -->
    <select id="findUserById" parameterType="Integer" resultType="User">
        select * from user where id = #{v}
    </select>
    
    <!-- //根据用户名称模糊查询用户列表
    #{}    select * from user where id = ？    占位符  ? ==  '五'
    ${}    select * from user where username like '%五%'  字符串拼接  
    
     -->
    <select id="findUserByUsername" parameterType="String" resultType="com.itheima.mybatis.pojo.User">
        select * from user where username like "%"#{haha}"%"
    </select>
    
    <!-- 添加用户 -->
    <insert id="insertUser" parameterType="com.itheima.mybatis.pojo.User">
        <selectKey keyProperty="id" resultType="Integer" order="AFTER">
            select LAST_INSERT_ID()
        </selectKey>
        insert into user (username,birthday,address,sex) 
        values (#{username},#{birthday},#{address},#{sex})
    </insert>
    
    <!-- 更新 -->
    <update id="updateUserById" parameterType="com.itheima.mybatis.pojo.User">
        update user 
        set username = #{username},sex = #{sex},birthday = #{birthday},address = #{address}
        where id = #{id}
    </update>
    
    <!-- 删除 -->
    <delete id="deleteUserById" parameterType="Integer">
        delete from user 
        where id = #{vvvvv}
    </delete>


</mapper>
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
typeAliases设置包内的类别名不区分大小写  
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
- mapper的动态代理开发  
applicationContext  
```xml
    <context:property-placeholder location="classpath:db.properties"/>
    
    <!-- 数据库连接池 -->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close">
        <property name="driverClassName" value="${jdbc.driver}" />
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
        <property name="maxActive" value="10" />
        <property name="maxIdle" value="5" />
    </bean>
    
    <!-- Mybatis的工厂 -->
    <bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!-- 核心配置文件的位置 -->
        <property name="configLocation" value="classpath:sqlMapConfig.xml"/>
    </bean>
    
    <!-- Dao原始Dao -->
    <bean id="userDao" class="com.itheima.mybatis.dao.UserDaoImpl">
        <property name="sqlSessionFactory" ref="sqlSessionFactoryBean"/>
    </bean>
    <!-- Mapper动态代理开发 -->
    <bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
        <property name="sqlSessionFactory" ref="sqlSessionFactoryBean"/>
        <property name="mapperInterface" value="com.itheima.mybatis.mapper.UserMapper"/>
    </bean>
    
    <!-- Mapper动态代理开发   扫描 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- 基本包 -->
        <property name="basePackage" value="com.itheima.mybatis.mapper"/>
    </bean>
    
```
sqlMapConfig
```xml
<configuration>
    <!-- 设置别名 -->
    <typeAliases>
        <!-- 2. 指定扫描包，会把包内所有的类都设置别名，别名的名称就是类名，大小写不敏感 -->
        <package name="com.itheima.mybatis.pojo" />
    </typeAliases>
    
    <mappers>
        <package name="com.itheima.mybatis.mapper"/>
    </mappers>

</configuration>
```
## spring  
- spring的优点  
1、方便解耦，简化开发  （高内聚低耦合）  
　　Spring就是一个大工厂（容器），可以将所有对象创建和依赖关系维护，交给Spring管理  
　　spring工厂是用于生成bean  
2、AOP编程的支持  
　　Spring提供面向切面编程，可以方便的实现对程序进行权限拦截、运行监控等功能  
3、声明式事务的支持  
　　只需要通过配置就可以完成对事务的管理，而无需手动编程  
4、方便程序的测试  
　　Spring对Junit4支持，可以通过注解方便的测试Spring程序  
5、方便集成各种优秀框架  
　　Spring不排斥各种优秀的开源框架，其内部提供了对各种优秀框架（如：Struts、Quartz等）的直接支持  
6、降低JavaEE API的使用难度  
　　Spring 对JavaEE开发中非常难用的一些API（JDBC、JavaMail、远程调用等），都提供了封装  

- IOC和DI  
IOC:控制反转,将对象的创建权交给了Spring  
DI:Dependency Injection依赖注入.需要有IOC的环境,Spring创建这个类的过程中,Spring将类的依赖的属性设置进去  

- Bean元素  
尽量使用name属性  
```xml
    <!-- 将User对象交给spring容器管理 -->
    <!-- Bean元素:使用该元素描述需要spring容器管理的对象
            class属性:被管理对象的完整类名.
            name属性:给被管理的对象起个名字.获得对象时根据该名称获得对象.  
                    可以重复.可以使用特殊字符.
            id属性: 与name属性一模一样. 
                    名称不可重复.不能使用特殊字符.
            结论: 尽量使用name属性.
      -->
    <bean  name="user" class="cn.itcast.bean.User" ></bean>
    <!-- 导入其他spring配置文件 -->
    <import resource="cn/itcast/b_create/applicationContext.xml"/>
```

- 三种对象创建方式  
```xml
    <!-- 创建方式1:空参构造创建  -->
    <bean  name="user" class="cn.itcast.bean.User"
         init-method="init" destroy-method="destory" ></bean>
    <!-- 创建方式2:静态工厂创建 
          调用UserFactory的createUser方法创建名为user2的对象.放入容器
     -->
    <bean  name="user2" 
        class="cn.itcast.b_create.UserFactory" 
        factory-method="createUser" ></bean>
    <!-- 创建方式3:实例工厂创建 
         调用UserFactory对象的createUser2方法创建名为user3的对象.放入容器
     -->
    <bean  name="user3" 
        factory-bean="userFactory"
        factory-method="createUser2" ></bean>
        
    <bean  name="userFactory" 
        class="cn.itcast.b_create.UserFactory"   ></bean>
```
```java
    @Test
    public void fun1() {
        // 1 创建容器对象
        ApplicationContext ac = new ClassPathXmlApplicationContext("cn/itcast/b_create/applicationContext.xml");
        // 2 向容器"要"user对象
        User u = (User) ac.getBean("user");

        // 3 打印user对象
        System.out.println(u);
    }

    // 创建方式2:静态工厂
    @Test
    public void fun2() {
        // 1 创建容器对象
        ApplicationContext ac = new ClassPathXmlApplicationContext("cn/itcast/b_create/applicationContext.xml");
        // 2 向容器"要"user对象
        User u = (User) ac.getBean("user2");
        // 3 打印user对象
        System.out.println(u);
    }

    // 创建方式3:实例工厂
    @Test
    public void fun3() {
        // 1 创建容器对象
        ApplicationContext ac = new ClassPathXmlApplicationContext("cn/itcast/b_create/applicationContext.xml");
        // 2 向容器"要"user对象
        User u = (User) ac.getBean("user3");
        // 3 打印user对象
        System.out.println(u);
    }

```
- scope属性  
singleton  
默认值，单例的  
prototype  
多例的，每次创建新的对象，整合struts2，ActionBean必须配置为多例的  
request  
WEB项目中,Spring创建一个Bean的对象,将对象存入到request域中  
session  
WEB项目中,Spring创建一个Bean的对象,将对象存入到session域中  

- 初始化和销毁  
ClassPathXmlApplicationContext实现类中才有close方法  
```xml
    <!-- 创建方式1:空参构造创建  -->
    <bean  name="user" class="cn.itcast.bean.User"
         init-method="init" destroy-method="destory" ></bean>
```
```java
    public void fun5() {
        // 1 创建容器对象
        ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext(
                "cn/itcast/b_create/applicationContext.xml");
        // 2 向容器"要"user对象
        User u = (User) ac.getBean("user");
        // 3 打印user对象
        System.out.println(u);
        // 关闭容器,触发销毁方法
        ac.close();
    }
```

- 导入配置  
```xml
    <!-- 导入其他spring配置文件 -->
    <import resource="cn/itcast/b_create/applicationContext.xml"/>
```
- set方式注入属性  
```xml
    <!-- set方式注入: -->
    <bean name="user" class="cn.itcast.bean.User">
        <!--值类型注入: 为User对象中名为name的属性注入tom作为值 -->
        <property name="name" value="tom"></property>
        <property name="age" value="18"></property>
        <!-- 引用类型注入: 为car属性注入下方配置的car对象 -->
        <property name="car" ref="car"></property>
    </bean>

    <!-- 将car对象配置到容器中 -->
    <bean name="car" class="cn.itcast.bean.Car">
        <property name="name" value="兰博基尼"></property>
        <property name="color" value="黄色"></property>
    </bean>
```

- 构造函数注入  
```xml
<!-- 构造函数注入 -->
    <bean name="user2" class="cn.itcast.bean.User">
        <!-- name属性: 构造函数的参数名 -->
        <!-- index属性: 构造函数的参数索引 -->
        <!-- type属性: 构造函数的参数类型 -->
        <constructor-arg name="name" index="0" type="java.lang.Integer"
            value="999"></constructor-arg>
        <constructor-arg name="car" ref="car" index="1"></constructor-arg>
    </bean>
```

- p名称空间注入  
```xml
    <!-- p名称空间注入, 走set方法 1.导入P名称空间 xmlns:p="http://www.springframework.org/schema/p" 
        2.使用p:属性完成注入 |-值类型: p:属性名="值" |-对象类型: p:属性名-ref="bean名称" -->
    <bean name="user3" class="cn.itcast.bean.User" p:name="jack"
        p:age="20" p:car-ref="car">
    </bean>

```
- spel注入，可以取其他对象的值  
```xml
    <!-- spel注入: spring Expression Language sping表达式语言 -->
    <bean name="user4" class="cn.itcast.bean.User">
        <property name="name" value="#{user.name}"></property>
        <property name="age" value="#{user3.age}"></property>
        <property name="car" ref="car"></property>
    </bean>
```
- 复杂类型注入  
array、list、map、Properties  
```xml
<!-- 复杂类型注入 -->
    <bean name="cb" class="cn.itcast.c_injection.CollectionBean">
        <!-- 如果数组中只准备注入一个值(对象),直接使用value|ref即可 
        <property name="arr" value="tom" ></property> -->
        <!-- array注入,多个元素注入 -->
        <property name="arr">
            <array>
                <value>tom</value>
                <value>jerry</value>
                <ref bean="user4" />
            </array>
        </property>

        <!-- 如果List中只准备注入一个值(对象),直接使用value|ref即可 <property name="list" value="jack" 
            ></property> -->
        <property name="list">
            <list>
                <value>jack</value>
                <value>rose</value>
                <ref bean="user3" />
            </list>
        </property>
        <!-- map类型注入 -->
        <property name="map">
            <map>
                <entry key="url" value="jdbc:mysql:///crm"></entry>
                <entry key="user" value-ref="user4"></entry>
                <entry key-ref="user3" value-ref="user2"></entry>
            </map>
        </property>
        <!-- prperties 类型注入 -->
        <property name="prop">
            <props>
                <prop key="driverClass">com.jdbc.mysql.Driver</prop>
                <prop key="userName">root</prop>
                <prop key="password">1234</prop>
            </props>
        </property>
    </bean>
```

```java
    public String list() throws Exception {
            //获得spring容器=>从Application域获得即可
        
            //1 获得servletContext对象
            ServletContext sc = ServletActionContext.getServletContext();
            //2.从Sc中获得ac容器
            WebApplicationContext ac = WebApplicationContextUtils.getWebApplicationContext(sc);
            //3.从容器中获得CustomerService
            CustomerService cs = (CustomerService) ac.getBean("customerService");
    }
```

- 获得spring容器  
导包 4+2+1，四个核心两个依赖一个spring-web整合web  
web.xml配置spring  
```xml
  <!-- 可以让spring容器随项目的启动而创建,随项目的关闭而销毁 -->
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <!-- 指定加载spring配置文件的位置 -->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
  </context-param>
```
```java
        // 获得spring容器=>从Application域获得即可
        // 1 获得servletContext对象
        ServletContext sc = ServletActionContext.getServletContext();
        // 2.从Sc中获得ac容器
        WebApplicationContext ac = WebApplicationContextUtils.getWebApplicationContext(sc);
        // 3.从容器中获得CustomerService
        CustomerService cs = (CustomerService) ac.getBean("customerService");
```

- 注解配置spring  
主配置文件引入命名空间(约束)  
配置
```xml
<!-- 指定扫描cn.itcast.bean报下的所有类中的注解.
注意:扫描包时.会扫描指定报下的所有子孙包-->
<context:component-scan base-package="cn.itcast.bean"></context:component-scan>
```
导包4+2+1，spring-aop  
```java
//<bean name="user" class="cn.itcast.bean.User"  />
    @Component("user")
//  @Service("user") // service层
//  @Controller("user") // web层
//  @Repository("user")// dao层

    //指定对象的作用范围
    @Scope(scopeName="singleton")

    @Value("18")
    private Integer age;

    //@Autowired //自动装配
    //问题:如果匹配多个类型一致的对象.将无法选择具体注入哪一个对象.
    //@Qualifier("car2")//使用@Qualifier注解告诉spring容器自动装配哪个名称的对象
    
    @Resource(name="car")//手动注入,指定注入哪个名称的对象

    @PostConstruct //在对象被创建后调用.init-method
    public void init(){
        System.out.println("我是初始化方法!");
    }
    @PreDestroy //在销毁之前调用.destory-method
    public void destory(){
        System.out.println("我是销毁方法!");
    }

```
- spring整合junit  
不用每一个测试都创建容器并获取对象  
导包4+2+aop+test  
```java
@RunWith(SpringJUnit4ClassRunner.class)
//指定创建容器时使用哪个配置文件
@ContextConfiguration("classpath:applicationContext.xml")
public class Demo {
    //将名为user的对象注入到u变量中
    @Resource(name="user")
    private User u;
    @Test
    public void fun1(){
        System.out.println(u);
    }
}
```
- spring实现aop的原理  
动态代理(优先)  
　　被代理对象必须要实现接口,才能产生代理对象.如果没有接口将不能使用动态代理技术  
cglib代理(没有接口)  
　　第三方代理技术,cglib代理.可以对任何类生成代理.代理的原理是对目标对象进行继承代理.  
　　如果目标对象被final修饰.那么该类无法被cglib代理.  

- AOP开发中相关术语  
Joinpoint(连接点):目标对象中所有可以增强的方法  
Pointcut(切入点):目标对象需要增强的方法  
Advice(通知/增强)：需要加入的增强代码  
Target(目标对象):被代理的对象  
Weaving(织入):将通知应用到切入点的过程  
Proxy(代理):将通知织入目标对象之后，形成代理对象  
Aspect(切面):切入点+通知  

- spring中AOP的实现  
导包 4+2+2  
　　spring-aspects  
　　spring-aop  
　　com.springsource.org.aopalliance  
　　com.springsource.org.aspectj.weaver  
准备通知  
```java
    // 前置通知
    // |-目标方法运行之前调用
    // 后置通知(如果出现异常不会调用)
    // |-在目标方法运行之后调用
    // 环绕通知
    // |-在目标方法之前和之后都调用
    // 异常拦截通知
    // |-如果出现异常,就会调用
    // 后置通知(无论是否出现 异常都会调用)
    // |-在目标方法运行之后调用
    // ----------------------------------------------------------------
    // 前置通知
    public void before() {
        System.out.println("这是前置通知!!");
    }

    // 后置通知
    public void afterReturning() {
        System.out.println("这是后置通知(如果出现异常不会调用)!!");
    }

    // 环绕通知
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("这是环绕通知之前的部分!!");
        Object proceed = pjp.proceed();// 调用目标方法
        System.out.println("这是环绕通知之后的部分!!");
        return proceed;
    }

    // 异常通知
    public void afterException() {
        System.out.println("出事啦!出现异常了!!");
    }

    // 后置通知
    public void after() {
        System.out.println("这是后置通知(出现异常也会调用)!!"); 
    }
}
```
配置目标对象、通知对象、将通知织入　　
```xml
<!-- 准备工作: 导入aop(约束)命名空间 -->
<!-- 1.配置目标对象 -->
    <bean name="userService" class="cn.itcast.service.UserServiceImpl" ></bean>
<!-- 2.配置通知对象 -->
    <bean name="myAdvice" class="cn.itcast.d_springaop.MyAdvice" ></bean>
<!-- 3.配置将通知织入目标对象 -->
    <aop:config>
        <!-- 配置切入点 
            public void cn.itcast.service.UserServiceImpl.save() 
            void cn.itcast.service.UserServiceImpl.save()
            * cn.itcast.service.UserServiceImpl.save()
            * cn.itcast.service.UserServiceImpl.*()
            
            * cn.itcast.service.*ServiceImpl.*(..)
            * cn.itcast.service..*ServiceImpl.*(..)
        -->
        <aop:pointcut expression="execution(* cn.itcast.service.*ServiceImpl.*(..))" id="pc"/>
        <aop:aspect ref="myAdvice" >
            <!-- 指定名为before方法作为前置通知 -->
            <aop:before method="before" pointcut-ref="pc" />
            <!-- 后置 -->
            <aop:after-returning method="afterReturning" pointcut-ref="pc" />
            <!-- 环绕通知 -->
            <aop:around method="around" pointcut-ref="pc" />
            <!-- 异常拦截通知 -->
            <aop:after-throwing method="afterException" pointcut-ref="pc"/>
            <!-- 后置 -->
            <aop:after method="after" pointcut-ref="pc"/>
        </aop:aspect>
    </aop:config>
```
- aop注解方式  
开启注解  
```xml
<!-- 准备工作: 导入aop(约束)命名空间 -->
<!-- 1.配置目标对象 -->
    <bean name="userService" class="cn.itcast.service.UserServiceImpl" ></bean>
<!-- 2.配置通知对象 -->
    <bean name="myAdvice" class="cn.itcast.e_annotationaop.MyAdvice" ></bean>
<!-- 3.开启使用注解完成织入 -->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```
```java
//通知类
@Aspect
//表示该类是一个通知类
public class MyAdvice {
    @Pointcut("execution(* cn.itcast.service.*ServiceImpl.*(..))")
    public void pc(){}
    //前置通知
    //指定该方法是前置通知,并制定切入点
    @Before("MyAdvice.pc()")
    public void before(){
        System.out.println("这是前置通知!!");
    }
    //后置通知
    @AfterReturning("execution(* cn.itcast.service.*ServiceImpl.*(..))")
    public void afterReturning(){
        System.out.println("这是后置通知(如果出现异常不会调用)!!");
    }
    //环绕通知
    @Around("execution(* cn.itcast.service.*ServiceImpl.*(..))")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("这是环绕通知之前的部分!!");
        Object proceed = pjp.proceed();//调用目标方法
        System.out.println("这是环绕通知之后的部分!!");
        return proceed;
    }
    //异常通知
    @AfterThrowing("execution(* cn.itcast.service.*ServiceImpl.*(..))")
    public void afterException(){
        System.out.println("出事啦!出现异常了!!");
    }
    //后置通知
    @After("execution(* cn.itcast.service.*ServiceImpl.*(..))")
    public void after(){
        System.out.println("这是后置通知(出现异常也会调用)!!");
    }
}
```

- spring整合JDBC  
导包  
4+2  
spring-test  
spring-aop  
junit4类库  
c3p0连接池  
JDBC驱动  
spring-jdbc  
spring-tx事务  
　　  
jdbctemplate提供的api  
```java
public class UserDaoImpl extends JdbcDaoSupport implements UserDao {
    @Override
    public void save(User u) {
        String sql = "insert into t_user values(null,?) ";
        super.getJdbcTemplate().update(sql, u.getName());
    }
    @Override
    public void delete(Integer id) {
        String sql = "delete from t_user where id = ? ";
        super.getJdbcTemplate().update(sql,id);
    }
    @Override
    public void update(User u) {
        String sql = "update  t_user set name = ? where id=? ";
        super.getJdbcTemplate().update(sql, u.getName(),u.getId());
    }
    @Override
    public User getById(Integer id) {
        String sql = "select * from t_user where id = ? ";
        return super.getJdbcTemplate().queryForObject(sql,new RowMapper<User>(){
            @Override
            public User mapRow(ResultSet rs, int arg1) throws SQLException {
                User u = new User();
                u.setId(rs.getInt("id"));
                u.setName(rs.getString("name"));
                return u;
            }}, id);
        
    }
    @Override
    public int getTotalCount() {
        String sql = "select count(*) from t_user  ";
        Integer count = super.getJdbcTemplate().queryForObject(sql, Integer.class);
        return count;
    }

    @Override
    public List<User> getAll() {
        String sql = "select * from t_user  ";
        List<User> list = super.getJdbcTemplate().query(sql, new RowMapper<User>(){
            @Override
            public User mapRow(ResultSet rs, int arg1) throws SQLException {
                User u = new User();
                u.setId(rs.getInt("id"));
                u.setName(rs.getString("name"));
                return u;
            }});
        return list;
    }
}
```
配置到spring容器并测试  
```xml
<!-- 指定spring读取db.properties配置 -->
<context:property-placeholder location="classpath:db.properties"  />

<!-- 1.将连接池放入spring容器 -->
<bean name="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" >
    <property name="jdbcUrl" value="${jdbc.jdbcUrl}" ></property>
    <property name="driverClass" value="${jdbc.driverClass}" ></property>
    <property name="user" value="${jdbc.user}" ></property>
    <property name="password" value="${jdbc.password}" ></property>
</bean>


<!-- 2.将JDBCTemplate放入spring容器 -->
<bean name="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate" >
    <property name="dataSource" ref="dataSource" ></property>
</bean>

<!-- 3.将UserDao放入spring容器 -->
<bean name="userDao" class="cn.itcast.a_jdbctemplate.UserDaoImpl" >
    <!-- <property name="jt" ref="jdbcTemplate" ></property> -->
    <property name="dataSource" ref="dataSource" ></property>
</bean>
```

```java
//演示JDBC模板
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class Demo {
    @Resource(name="userDao")
    private UserDao ud;

    @Test
    public void fun2() throws Exception{
        User u = new User();
        u.setName("tom");
        ud.save(u);
    }
    @Test
    public void fun3() throws Exception{
        User u = new User();
        u.setId(2);
        u.setName("jack");
        ud.update(u);
        
    }
    
    @Test
    public void fun4() throws Exception{
        ud.delete(2);
    }
    
    @Test
    public void fun5() throws Exception{
        System.out.println(ud.getTotalCount());
    }
    
    @Test
    public void fun6() throws Exception{
        System.out.println(ud.getById(1));
    }
    
    @Test
    public void fun7() throws Exception{
        System.out.println(ud.getAll());
    } 
}
```
- spring读取properties配置  
```xml
<!-- 指定spring读取db.properties配置 -->
<context:property-placeholder location="classpath:db.properties"  />

<!-- 1.将连接池放入spring容器 -->
<bean name="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" >
    <property name="jdbcUrl" value="${jdbc.jdbcUrl}" ></property>
    <property name="driverClass" value="${jdbc.driverClass}" ></property>
    <property name="user" value="${jdbc.user}" ></property>
    <property name="password" value="${jdbc.password}" ></property>
</bean>
```

- 事物  
事务特性:acid  
事务并发问题:脏读、不可重复读、幻读  
事务的隔离级别  
1 读未提交  
2 读已提交  
4 可重复读  
8 串行化  
事务操作对象：PlatformTransactionManager接口  

- 事物  
```xml
<!-- 配置事务通知 -->
<tx:advice id="txAdvice" transaction-manager="transactionManager" >
    <tx:attributes>
        <!-- 以方法为单位,指定方法应用什么事务属性
            isolation:隔离级别
            propagation:传播行为
            read-only:是否只读
         -->
        <tx:method name="save*" isolation="REPEATABLE_READ" propagation="REQUIRED" read-only="false" />
        <tx:method name="persist*" isolation="REPEATABLE_READ" propagation="REQUIRED" read-only="false" />
        <tx:method name="update*" isolation="REPEATABLE_READ" propagation="REQUIRED" read-only="false" />
        <tx:method name="modify*" isolation="REPEATABLE_READ" propagation="REQUIRED" read-only="false" />
        <tx:method name="delete*" isolation="REPEATABLE_READ" propagation="REQUIRED" read-only="false" />
        <tx:method name="remove*" isolation="REPEATABLE_READ" propagation="REQUIRED" read-only="false" />
        <tx:method name="get*" isolation="REPEATABLE_READ" propagation="REQUIRED" read-only="true" />
        <tx:method name="find*" isolation="REPEATABLE_READ" propagation="REQUIRED" read-only="true" />
        <tx:method name="transfer" isolation="REPEATABLE_READ" propagation="REQUIRED" read-only="false" />
    </tx:attributes>
</tx:advice>


<!-- 配置织入 -->
<aop:config  >
    <!-- 配置切点表达式 -->
    <aop:pointcut expression="execution(* cn.itcast.service.*ServiceImpl.*(..))" id="txPc"/>
    <!-- 配置切面 : 通知+切点
            advice-ref:通知的名称
            pointcut-ref:切点的名称
     -->
    <aop:advisor advice-ref="txAdvice" pointcut-ref="txPc" />
</aop:config>
```
- 注解方式配置事物  
```xml
<!-- 开启使用注解管理aop事务 -->
<tx:annotation-driven/>
```
```java
@Transactional(isolation=Isolation.REPEATABLE_READ,propagation=Propagation.REQUIRED,readOnly=true)
public class AccountServiceImpl implements AccountService {

    private AccountDao ad ;
    private TransactionTemplate tt;
    
    @Override
    @Transactional(isolation=Isolation.REPEATABLE_READ,propagation=Propagation.REQUIRED,readOnly=false)
    public void transfer(final Integer from,final Integer to,final Double money) {
                //减钱
                ad.decreaseMoney(from, money);
                int i = 1/0;
                //加钱
                ad.increaseMoney(to, money);
    }
}
```

##springmvc  
- 架构图  
![](http://oumcs7yiy.bkt.clouddn.com/SSM-2.png)  
- 前端控制器配置  
```xml
  <!-- 前端控制器 -->
  <servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 默认找 /WEB-INF/[servlet的名称]-servlet.xml -->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc.xml</param-value>
    </init-param>
  </servlet>
  
  <servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <!-- 
        1. /*  拦截所有   jsp  js png .css  真的全拦截   建议不使用
        2. *.action *.do 拦截以do action 结尾的请求     肯定能使用   ERP  
        3. /  拦截所有 （不包括jsp) (包含.js .png.css)  强烈建议使用     前台 面向消费者  www.jd.com/search   /对静态资源放行
     -->
    <url-pattern>*.action</url-pattern>
  </servlet-mapping>
```
通过拦截器拦截到ItemController  
```java
public class ItemController {
    //注释匹配浏览器url  
    @RequestMapping(value = "/item/itemlist.action")
    public ModelAndView itemList(){
        
        // 创建页面需要显示的商品数据
        List<Items> list = new ArrayList<Items>();
        list.add(new Items(1, "1华为 荣耀8", 2399f, new Date(), "质量好！1"));
        list.add(new Items(2, "2华为 荣耀8", 2399f, new Date(), "质量好！2"));
        list.add(new Items(3, "3华为 荣耀8", 2399f, new Date(), "质量好！3"));
        list.add(new Items(4, "4华为 荣耀8", 2399f, new Date(), "质量好！4"));
        list.add(new Items(5, "5华为 荣耀8", 2399f, new Date(), "质量好！5"));
        list.add(new Items(6, "6华为 荣耀8", 2399f, new Date(), "质量好！6"));
        
        ModelAndView mav = new ModelAndView();
        //数据
        mav.addObject("itemList", list);
        mav.setViewName("itemList");
        return mav;
    }
}
```
- springmvc.xml配置三大组件  
注解驱动相当于配置了映射器和适配器  
```xml
        <!-- 扫描@Controler  @Service   -->
        <context:component-scan base-package="com.itheima"/>
        
        <!-- 处理器映射器 -->
<!--         <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/> -->
        <!-- 处理器适配器 -->
<!--         <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/> -->
        <!-- 注解驱动 -->
        <mvc:annotation-driven/>
        
        <!-- 视图解释器 -->
        <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <property name="prefix" value="/WEB-INF/jsp/"/>
            <property name="suffix" value=".jsp"/>
        </bean>
```