
## 1.0版本控制  

1. 分布式项目1-3

```sql
-- 01 创建临时表空间
create temporary tablespace bhz_temp 
tempfile 'D:\006_design\bhz_temp_01_20151130.dbf' size 100m autoextend on next 50m maxsize 200m;
--drop  tablespace bhz_temp including contents and datafiles;

-- 02 创建表空间
create tablespace bhz
datafile 'D:\006_design\bhz_01_20151130.dbf' size 200m autoextend on next 100m maxsize 400m;
--drop  tablespace bhz including contents and datafiles;
--alter tablespace bhz add datafile 'D:\006_design\bhz_02_20151130.dbf' size 200m autoextend on;

-- 03 创建用户并制定表空间
create user bhz identified by bhz
default tablespace bhz
temporary tablespace bhz_temp;

-- 04 赋权
grant dba to bhz;

```

bhz-parent父工程  

/bhz-parent/pom.xml  
<packaging>pom</packaging>相当于打包成空壳  
<build><finalName>bhz-parent</finalName>打成包之后的名字  

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>bhz</groupId>
  <artifactId>bhz-parent</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>pom</packaging>
  <name>bhz-parent</name>
  <url>http://maven.apache.org</url>

  <!-- 进行聚合的子项目 --> 
  <!--  
  <modules>
  	<module>../bhz-com</module>
  	<module>../bhz-sys</module>
   	<module>../bhz-sys-facade</module>
   	<module>../bhz-sys-service</module>   
  </modules>
  -->
  
  
  <!-- 配置私服工厂 -->
  <!--  
  <repositories>
		<repository>
			<id>nexus</id>
			<url>http://localhost:8081/nexus/content/groups/public/</url>
			<releases>
				<enabled>true</enabled>
			</releases>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</repository>
  </repositories>      
  <pluginRepositories>
		<pluginRepository>
			<id>nexus</id>
			<url>http://localhost:8081/nexus/content/groups/public/</url>
			<releases>
				<enabled>true</enabled>
			</releases>
			<snapshots>
				<enabled>true</enabled>
		   </snapshots>
		</pluginRepository>
  </pluginRepositories>
  -->
  <!-- 配置发布到私服 -->
  <distributionManagement>
		<repository>
			<id>nexus-releases</id>
			<name>Nexus Release Repository</name>
			<url>http://localhost:8081/nexus/content/repositories/releases/</url>
		</repository>
		<snapshotRepository>
			<id>nexus-snapshots</id>
			<name>Nexus Snapshot Repository</name>
			<url>http://localhost:8081/nexus/content/repositories/snapshots/</url>
		</snapshotRepository>
  </distributionManagement>     
  <properties>
  	<!-- base -->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <junit.version>4.11</junit.version>
    <spring.version>4.1.5.RELEASE</spring.version>
    <proxool.version>0.9.1</proxool.version>
    <commons-fileupload.version>1.2.2</commons-fileupload.version>
    <commons-lang3.version>3.3.1</commons-lang3.version>
    <commons-io.version>2.4</commons-io.version>
    <commons-dbcp.version>1.4</commons-dbcp.version>
    <commons-logging.version>1.1.3</commons-logging.version>
    <commons-codec.version>1.6</commons-codec.version>     
    <jackson.version>1.9.11</jackson.version>
    <fastjson.version>1.1.26</fastjson.version>
	<slf4j.version>1.7.6</slf4j.version>
    <log4j.version>1.2.15</log4j.version>
    <ojdbc7.version>1.0</ojdbc7.version>
    <jstl.version>1.2</jstl.version>
    <servlet-api.version>3.0-alpha-1</servlet-api.version>   
    
    <rocketmq.version>3.2.6</rocketmq.version>
    <httpclient.version>4.3.1</httpclient.version>
    <zookeeper.version>3.4.5</zookeeper.version>
    <zkclient.version>0.7</zkclient.version>
    <dubbo.version>2.8.4</dubbo.version>
    <tomcat_embed_version>8.0.11</tomcat_embed_version>
    <javassist.version>3.15.0-GA</javassist.version>
    <mina.version>1.1.7</mina.version>
    <grizzly.version>2.1.4</grizzly.version>
    <xstream.version>1.4.1</xstream.version>
    <bsf.version>3.1</bsf.version>
	<sorcerer.version>0.8</sorcerer.version>
	<curator.version>2.5.0</curator.version>
	<jedis.version>2.7.2</jedis.version>
	<xmemcached.version>1.3.6</xmemcached.version>
	<cxf.version>2.6.1</cxf.version>
	<thrift.version>0.8.0</thrift.version>
	<jfreechart.version>1.0.13</jfreechart.version>
	<hessian.version>4.0.7</hessian.version>
	<jetty.version>6.1.26</jetty.version>
	<validation.version>1.0.0.GA</validation.version>
	<hibernate-validator.version>4.2.0.Final</hibernate-validator.version>
	<jcache.version>0.4</jcache.version>
	<ws.version>2.0</ws.version>
	<resteasy.version>3.0.7.Final</resteasy.version>
	<kryo.version>2.24.0</kryo.version>
	<kryo-serializers.version>0.26</kryo-serializers.version>
	<fst.version>1.55</fst.version>    
  </properties>  
  
  
  <dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>${junit.version}</version>
			<scope>test</scope>
		</dependency>  
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>${spring.version}</version>
			<scope>test</scope> 
		</dependency>
		
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${spring.version}</version>
		</dependency>			
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<version>${spring.version}</version>
		</dependency>	
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-orm</artifactId>
			<version>${spring.version}</version>
		</dependency>	
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-aop</artifactId>
			<version>${spring.version}</version>
		</dependency>	
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-aspects</artifactId>
			<version>${spring.version}</version>
		</dependency>					
		<dependency>
			<groupId>com.cloudhopper.proxool</groupId>
			<artifactId>proxool</artifactId>
			<version>${proxool.version}</version>
		</dependency>
		<dependency>
			<groupId>commons-fileupload</groupId>
			<artifactId>commons-fileupload</artifactId>
			<version>${commons-fileupload.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-lang3</artifactId>
			<version>${commons-lang3.version}</version>
		</dependency>
		<dependency>
			<groupId>commons-io</groupId>
			<artifactId>commons-io</artifactId>
			<version>${commons-io.version}</version>
		</dependency>
		<dependency>
			<groupId>commons-dbcp</groupId>
			<artifactId>commons-dbcp</artifactId>
			<version>${commons-dbcp.version}</version>
		</dependency>
		<dependency>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
			<version>${commons-logging.version}</version>
		</dependency>
		<dependency>
			<groupId>commons-codec</groupId>
			<artifactId>commons-codec</artifactId>
			<version>${commons-codec.version}</version> 
		</dependency>		
		<dependency>
			<groupId>org.codehaus.jackson</groupId>
			<artifactId>jackson-mapper-asl</artifactId>
			<version>${jackson.version}</version>
		</dependency>	
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>${fastjson.version}</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>${slf4j.version}</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>jul-to-slf4j</artifactId>
			<version>${slf4j.version}</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>jcl-over-slf4j</artifactId>
			<version>${slf4j.version}</version>
		</dependency>				
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>${log4j.version}</version>
			<exclusions>
			    <exclusion>
			        <groupId>com.sun.jmx</groupId>
			        <artifactId>jmxri</artifactId>
			    </exclusion>
			    <exclusion>
			        <groupId>com.sun.jdmk</groupId>
			        <artifactId>jmxtools</artifactId>
			    </exclusion>
			    <exclusion>
			            <groupId>javax.jms</groupId>
			            <artifactId>jms</artifactId>
			    </exclusion>
			</exclusions>
		</dependency>		
		<dependency>
			<groupId>com.ojdbc7</groupId>
			<artifactId>ojdbc7</artifactId>
			<version>${ojdbc7.version}</version>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jstl</artifactId>
			<version>${jstl.version}</version>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>servlet-api</artifactId>
			<version>${servlet-api.version}</version>
			<scope>provided</scope>		
		</dependency>
		<dependency>
			<groupId>com.alibaba.rocketmq</groupId>
			<artifactId>rocketmq-client</artifactId>
			<version>${rocketmq.version}</version>
		</dependency>		
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpclient-cache</artifactId>
			<version>${httpclient.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpclient</artifactId>
			<version>${httpclient.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpcore</artifactId>
			<version>${httpclient.version}</version>
		</dependency>	
		<!-- zookeeper jar begin -->
		<dependency>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
			<version>${zookeeper.version}</version>
		</dependency>
		<dependency>
			<groupId>com.101tec</groupId>
			<artifactId>zkclient</artifactId>
			<version>${zkclient.version}</version>
		</dependency>
  		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-framework</artifactId>
			<version>2.4.2</version>
		</dependency>  	
  		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-recipes</artifactId>
			<version>2.4.2</version>
		</dependency> 		
		<!-- zookeeper jar end -->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>${jedis.version}</version>
        </dependency>		
		<!-- dubbox jar begin -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>${dubbo.version}</version>
        </dependency>
	    <dependency>
	        <groupId>org.apache.tomcat.embed</groupId>
	        <artifactId>tomcat-embed-core</artifactId>
	        <version>${tomcat_embed_version}</version>
	    </dependency>
	    <dependency>
	        <groupId>org.apache.tomcat.embed</groupId>
	        <artifactId>tomcat-embed-logging-juli</artifactId>
	        <version>${tomcat_embed_version}</version>
	    </dependency>         
        <dependency>
            <groupId>org.javassist</groupId>
            <artifactId>javassist</artifactId>
            <version>${javassist.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.mina</groupId>
            <artifactId>mina-core</artifactId>
            <version>${mina.version}</version>
        </dependency>
        <dependency>
            <groupId>org.glassfish.grizzly</groupId>
            <artifactId>grizzly-core</artifactId>
            <version>${grizzly.version}</version>
        </dependency>
        <dependency>
            <groupId>com.thoughtworks.xstream</groupId>
            <artifactId>xstream</artifactId>
            <version>${xstream.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.bsf</groupId>
            <artifactId>bsf-api</artifactId>
            <version>${bsf.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>${curator.version}</version>
        </dependency>
        <dependency>
            <groupId>com.googlecode.xmemcached</groupId>
            <artifactId>xmemcached</artifactId>
            <version>${xmemcached.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.cxf</groupId>
            <artifactId>cxf-rt-frontend-simple</artifactId>
            <version>${cxf.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.cxf</groupId>
            <artifactId>cxf-rt-transports-http</artifactId>
            <version>${cxf.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.thrift</groupId>
            <artifactId>libthrift</artifactId>
            <version>${thrift.version}</version>
        </dependency>
        <dependency>
            <groupId>com.caucho</groupId>
            <artifactId>hessian</artifactId>
            <version>${hessian.version}</version>
        </dependency>
        <dependency>
            <groupId>org.mortbay.jetty</groupId>
            <artifactId>jetty</artifactId>
             <version>${jetty.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>org.mortbay.jetty</groupId>
                    <artifactId>servlet-api</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>javax.validation</groupId>
            <artifactId>validation-api</artifactId>
            <version>${validation.version}</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>${hibernate-validator.version}</version>
        </dependency>
        <dependency>
            <groupId>javax.cache</groupId>
            <artifactId>cache-api</artifactId>
            <version>${jcache.version}</version>
        </dependency>
        <dependency>
            <groupId>javax.ws.rs</groupId>
            <artifactId>javax.ws.rs-api</artifactId>
            <version>${ws.version}</version>
        </dependency>
        <dependency>
            <groupId>org.jboss.resteasy</groupId>
            <artifactId>resteasy-jaxrs</artifactId>
            <version>${resteasy.version}</version>
        </dependency>
        <dependency>
            <groupId>org.jboss.resteasy</groupId>
            <artifactId>resteasy-client</artifactId>
            <version>${resteasy.version}</version>
        </dependency>
        <dependency>
            <groupId>org.jboss.resteasy</groupId>
            <artifactId>resteasy-netty</artifactId>
            <version>${resteasy.version}</version>
        </dependency>
        <dependency>
            <groupId>org.jboss.resteasy</groupId>
            <artifactId>resteasy-jdk-http</artifactId>
            <version>${resteasy.version}</version>
        </dependency>
        <dependency>
            <groupId>org.jboss.resteasy</groupId>
            <artifactId>resteasy-jackson-provider</artifactId>
            <version>${resteasy.version}</version>
        </dependency>
        <dependency>
            <groupId>org.jboss.resteasy</groupId>
            <artifactId>resteasy-jaxb-provider</artifactId>
            <version>${resteasy.version}</version>
        </dependency>
        <dependency>
            <groupId>com.esotericsoftware.kryo</groupId>
            <artifactId>kryo</artifactId>
            <version>${kryo.version}</version>
        </dependency>
        <dependency>
            <groupId>de.javakaffee</groupId>
            <artifactId>kryo-serializers</artifactId>
            <version>${kryo-serializers.version}</version>
        </dependency>
        <dependency>
            <groupId>de.ruedigermoeller</groupId>
            <artifactId>fst</artifactId>
            <version>${fst.version}</version>
        </dependency>
        <!-- dubbox jar end -->
	    <!-- 添加cas --> 
		<dependency>
			<groupId>org.jasig.cas.client</groupId>
			<artifactId>cas-client-core</artifactId>
			<version>3.1.12</version>
		</dependency> 
  </dependencies>
  <build>
    <finalName>bhz-parent</finalName>
   <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-war-plugin</artifactId>
            <version>2.4</version>
        </plugin>
		<!--自动打包发布插件 -->
		<!--
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-release-plugin</artifactId>
			<configuration>
				<tagBase>svn://192.168.100.27:8080/svn/development/java/credithc-mq/tags</tagBase>
			</configuration>
		</plugin>
		-->
		<!-- 解解决maven update project 后版本降低为1.5的bug -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.1</version>
            <configuration>
                <source>1.7</source>
                <target>1.7</target>
            </configuration>
        </plugin>
		<!-- 单元测试 -->
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-surefire-plugin</artifactId>
			<configuration>
				<skip>true</skip>
				<includes>
					<include>**/*Test*.java</include>
				</includes>
			</configuration>
		</plugin>
    	<plugin>
    		<groupId>org.apache.maven.plugins</groupId>
    		<artifactId>maven-source-plugin</artifactId>
    		<version>2.1.2</version>
    		<executions>
    			<!-- 绑定到特定的生命周期之后，运行maven-source-pluin 运行目标为jar-no-fork -->
				<execution>
					<phase>package</phase>
					<goals>
						<goal>jar-no-fork</goal>
					</goals>
				</execution>
			</executions>
    	</plugin>	
    </plugins>    
  </build>
</project>

```

2. 分布式项目4  
 
bhz-sys  

/bhz-sys/pom.xml  
继承父工程  
依赖com工程才能使用工具类  

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <!-- 继承父类 -->
  <parent>
  	<groupId>bhz</groupId>
  	<artifactId>bhz-parent</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<!-- 设置父类的绝对路径 -->
	<relativePath>../bhz-parent/pom.xml</relativePath>
  </parent>  
  <artifactId>bhz-sys</artifactId>
  <packaging>war</packaging>
  <dependencies>
  	<dependency>
  		<groupId>bhz</groupId>
  		<artifactId>bhz-com</artifactId>
  		<version>0.0.1-SNAPSHOT</version>
  	</dependency>  
  	<dependency>
  		<groupId>bhz</groupId>
  		<artifactId>bhz-sys-facade</artifactId>
  		<version>0.0.1-SNAPSHOT</version>
  	</dependency>
  </dependencies>
  <build>
    <finalName>bhz-sys</finalName>
  </build>
</project>

```

web.xml  
/bhz-sys/src/main/webapp/WEB-INF/web.xml  
一个tomcat部署多个应用最好加上webAppRootKey  


```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xmlns:jsp="http://java.sun.com/xml/ns/javaee/jsp" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
  <display-name>bhz-sys</display-name>
  <welcome-file-list>
    <welcome-file>default.jsp</welcome-file>
  </welcome-file-list>
  <context-param>
    <param-name>webAppRootKey</param-name>
    <param-value>bhz-sys.root</param-value>
  </context-param>
  <context-param>
    <param-name>log4jConfigLocation</param-name>
    <param-value>/WEB-INF/classes/log4j.properties</param-value>
  </context-param>
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath*:spring-context.xml</param-value>
  </context-param>
 
  <listener>
    <listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>
  </listener>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <servlet>
    <servlet-name>bhz-sys</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
     	<param-name>contextConfigLocation</param-name>
     	<param-value>classpath:spring-mvc.xml</param-value>
   	</init-param>    
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
      <servlet-name>bhz-sys</servlet-name>
      <url-pattern>*.html</url-pattern>
  </servlet-mapping>
  <servlet-mapping>
      <servlet-name>bhz-sys</servlet-name>
      <url-pattern>*.json</url-pattern>
  </servlet-mapping> 
 
  <filter>
      <filter-name>GzipFilter</filter-name>
      <filter-class>bhz.com.util.GzipFilter</filter-class>
      <init-param>
          <param-name>headers</param-name>
          <param-value>Content-Encoding=gzip</param-value>
      </init-param>
  </filter>
  <filter-mapping>
      <filter-name>GzipFilter</filter-name>
      <url-pattern>*.gzjs</url-pattern>
  </filter-mapping>
  <filter-mapping>
      <filter-name>GzipFilter</filter-name>
      <url-pattern>*.gzcss</url-pattern>
  </filter-mapping>
     
 <session-config>
   <session-timeout>15</session-timeout>
 </session-config>
</web-app>
```

config.properties  
/bhz-sys/src/main/resources/config.properties  

```
# Environment Config Properties.
env.defaultEncoding=UTF-8

# JDBC Database Config Properties.
jdbc.driver=oracle.jdbc.driver.OracleDriver
jdbc.url=jdbc:oracle:thin:@//192.168.1.200:1521/orcl
jdbc.username=bhz
jdbc.password=bhz
jdbc.minConnection=2
jdbc.maxConnection=40
jdbc.maxConnectionLife=1800000
jdbc.maxActiveTime=300000
jdbc.prototypeCount=1
jdbc.testSql=SELECT 1 FROM DUAL

# File Upload Config Properties.
upload.maxMemSize=102400
upload.maxFileSize=104857600
upload.tempDir=/WEB-INF/upload/

# Dubbox Config Properties.
dubbox.registry.address=zookeeper://192.168.1.111:2181?backup=192.168.1.112:2181,192.168.1.113:2181
dubbox.application=bhz-sys

```

dubbo-consumer.xml  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xmlns:util="http://www.springframework.org/schema/util"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
	http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-3.0.xsd">

    <!-- 引入配置文件 -->
    <context:property-placeholder location="classpath:config.properties" />

	<!-- 消费服务名称 -->
    <dubbo:application name="${dubbox.application}" owner="programmer" organization="dubbox"/>

	<!-- zookeeper注册中心  zookeeper://192.168.1.111:2181 -->
    <dubbo:registry address="${dubbox.registry.address}"/>
    
    <!-- 扫描dubbox注解位置 -->
    <dubbo:annotation package="bhz" />
    
    <!-- kryo实现序列化  -->
    <dubbo:protocol name="dubbo" serialization="kryo" optimizer="bhz.sys.serial.SerializationOptimizerImpl" />

	<!-- 生成远程服务接口配置 -->
	<dubbo:reference interface="bhz.sys.facade.SysUserFacade" id="sysUserFacade" />
</beans>
```

log4j.properties

```
log4j.rootLogger=INFO, console, file

log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d %p [%c] - %m%n

log4j.appender.file=org.apache.log4j.DailyRollingFileAppender
#log4j.appender.file.File=D:/002_developer/workspace_001/zcmoni.log
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d %p [%c] - %m%n

log4j.logger.org.springframework=WARN

```

spring-context.xml  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://www.springframework.org/schema/beans"
    xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="
    http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
    http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

	<!-- 引入配置文件  -->
	<context:property-placeholder location="classpath:config.properties" />
	
    <!-- 配置数据源 -->
    <bean id="dataSource" class="org.logicalcobwebs.proxool.ProxoolDataSource">
        <property name="driver" value="${jdbc.driver}" />
        <property name="driverUrl" value="${jdbc.url}" />
        <property name="user" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
        <property name="minimumConnectionCount" value="${jdbc.minConnection}" />
        <property name="maximumConnectionCount" value="${jdbc.maxConnection}" />
        <property name="maximumConnectionLifetime" value="${jdbc.maxConnectionLife}" />
        <property name="maximumActiveTime" value="${jdbc.maxActiveTime}" />
        <property name="prototypeCount" value="${jdbc.prototypeCount}" />
        <property name="houseKeepingTestSql" value="${jdbc.testSql}" />
    </bean>

	<!-- 配置事务管理器 -->
	<bean id="transactionManager"
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource" />
	</bean>

	<!-- 注解方式配置事物 -->
	<tx:annotation-driven transaction-manager="transactionManager" />

	<!-- 注解解析 -->
	<context:annotation-config />
	
	<!-- 扫描所有spring bean注解 -->
    <context:component-scan base-package="bhz" />
	
	<!-- 动态代理 -->
	<aop:aspectj-autoproxy/>
	
	<!-- 配置dubbo服务 -->
	<import resource="classpath:dubbo-consumer.xml" />
	
</beans>
```

spring-mvc.xml  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://www.springframework.org/schema/beans"
    xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
	http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
    
    <!-- 引入配置文件 -->
    <context:property-placeholder location="classpath:config.properties" />
    
    <!-- 进行springMvc必要配置 RequestMappingHandlerMapping 和 RequestMappingHandlerAdapter -->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping" />
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter" />
    
    <!-- 上传工具类 -->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="defaultEncoding" value="${env.defaultEncoding}" />
        <property name="maxInMemorySize" value="${upload.maxMemSize}" />
        <property name="maxUploadSize" value="${upload.maxFileSize}" />
        <property name="uploadTempDir" value="${upload.tempDir}" />
    </bean>
    
	<!-- spring view过滤  -->
    <bean id="pageResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
        <property name="prefix" value="/WEB-INF/pages/" />
        <property name="suffix" value=".jsp" />
        <property name="order" value="1" />
    </bean>
    
    <!-- spring 扫描所有bhz.sys.web下文件 -->
    <context:component-scan base-package="bhz" />
</beans>
```

3. 分布式项目5  
bhz-com  
打包方式为jar包  

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <!-- 继承父类 -->
  <parent>
  	<groupId>bhz</groupId>
  	<artifactId>bhz-parent</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<!-- 设置父类的绝对路径 -->
	<relativePath>../bhz-parent/pom.xml</relativePath>
  </parent>  
  <artifactId>bhz-com</artifactId>
  <packaging>jar</packaging>
  <build>
    <finalName>bhz-com</finalName>
  </build>
</project>

```

所有常量  
/bhz-com/src/main/java/bhz/com/constant/Const.java  

```java
package bhz.com.constant;

public final class Const {

    /**
     * <B>构造方法</B><BR>
     */
    private Const() {
    }

    /** 判断代码：是 */
    public static final String TRUE = "1";

    /** 判断代码：否 */
    public static final String FALSE = "0";

    /** 通用字符集编码 */
    public static final String CHARSET_UTF8 = "UTF-8";

    /** 中文字符集编码 */
    public static final String CHARSET_CHINESE = "GBK";

    /** 英文字符集编码 */
    public static final String CHARSET_LATIN = "ISO-8859-1";

    /** 根节点ID */
    public static final String ROOT_ID = "root";

    /** NULL字符串 */
    public static final String NULL = "null";

    /** 日期格式 */
    public static final String FORMAT_DATE = "yyyy-MM-dd";

    /** 日期时间格式 */
    public static final String FORMAT_DATETIME = "yyyy-MM-dd HH:mm:ss";

    /** 时间戳格式 */
    public static final String FORMAT_TIMESTAMP = "yyyy-MM-dd HH:mm:ss.SSS";
    
    /** JSON成功标记 */
    public static final String JSON_SUCCESS = "success";

    /** JSON数据 */
    public static final String JSON_DATA = "data";

    /** JSON数据列表 */
    public static final String JSON_ROWS = "rows";
    
    /** JSON总数 */
    public static final String JSON_TOTAL = "total";

    /** JSON消息文本 */
    public static final String JSON_MESSAGE = "message";
    
    public static final String TAG_SYS = "sys";
    
    public static final String TAG_MST = "mst";
    
    public static final String TAG_MQ = "mq";
    
    public static final String TAG_DAT = "dat";
    
    public static final String TAG_STA = "sta";
    
    public static final String TAG_INT = "int";  
    
    public static final String[] TAGS = {TAG_SYS, TAG_MST, TAG_MQ, TAG_DAT, TAG_STA, TAG_INT};
    
    /** Cookie键值：验证键值 */
    public static final String COOKIE_VALIDATE_KEY = "VALIDATE_KEY";

    /** Cookie键值：验证键值分割符 */
    public static final String COOKIE_VALIDATE_KEY_SPLIT = "$_";

    /** 请求属性键值：当前项目标识 */
    public static final String REQ_CUR_TAG = "REQ_CUR_TAG";
    
    /** 请求属性键值：当前用户标识 */
    public static final String REQ_CUR_USER_ID = "CUR_USER_ID";

    /** 请求属性键值：当前用户名称 */
    public static final String REQ_CUR_USER_NAME = "CUR_USER_NAME";

    /** 请求属性键值：当前机构标识 */
    public static final String REQ_CUR_ORG_ID = "CUR_ORG_ID";

    /** 请求属性键值：当前角色名称 */
    public static final String REQ_CUR_ROLE_CODE = "CUR_ROLE_CODE";

}
```

实现了spring提供的RowMapper  
/bhz-com/src/main/java/bhz/com/dao/JsonRowMapper.java  

```java
package bhz.com.dao;

import java.sql.ResultSet;
import java.sql.ResultSetMetaData;
import java.sql.SQLException;

import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.support.JdbcUtils;

import com.alibaba.fastjson.JSONException;
import com.alibaba.fastjson.JSONObject;

/**
 * <B>系统名称：</B>通用系统功能<BR>
 * <B>模块名称：</B>数据访问通用功能<BR>
 * <B>中文类名：</B>JSON数据行映射器<BR>
 * <B>概要说明：</B><BR>
 * 
 * @author 交通运输部规划研究院（bhz）
 * @since 2013-10-28
 */
public class JsonRowMapper implements RowMapper<JSONObject> {

    /**
     * <B>方法名称：</B>映射行数据<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param rs 结果集
     * @param row 行号
     * @return JSONObject 数据
     * @throws SQLException SQL异常错误
     * @see org.springframework.jdbc.core.RowMapper#mapRow(java.sql.ResultSet,
     *      int)
     */
    public JSONObject mapRow(ResultSet rs, int row) throws SQLException {
        String key = null;
        Object obj = null;
        JSONObject json = new JSONObject();
        ResultSetMetaData rsmd = rs.getMetaData();
        int count = rsmd.getColumnCount();
        for (int i = 1; i <= count; i++) {
            key = JdbcUtils.lookupColumnName(rsmd, i);
            obj = JdbcUtils.getResultSetValue(rs, i);
            try {
                json.put(key, obj);
            }
            catch (JSONException e) {
            }
        }
        return json;
    }
}
```

BaseJdbcDao.java  
getCurrentTime获取当前时间以数据库为准  
queryForJsonList(String sql, Object... args)可以有无限参数，返回一个json  
/bhz-com/src/main/java/bhz/com/dao/BaseJdbcDao.java  

```java
package bhz.com.dao;

import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.Date;
import java.util.Iterator;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map.Entry;
import java.util.Set;
import java.util.UUID;

import javax.sql.DataSource;

import org.apache.commons.lang3.StringUtils;
import org.apache.commons.lang3.time.DateFormatUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.BatchPreparedStatementSetter;
import org.springframework.jdbc.core.JdbcTemplate;

import bhz.com.constant.Const;

import com.alibaba.fastjson.JSONObject;

/**
 * <B>系统名称：</B>通用系统功能<BR>
 * <B>模块名称：</B>数据访问通用功能<BR>
 * <B>中文类名：</B>基础数据访问<BR>
 * <B>概要说明：</B><BR>
 * 
 * @author 交通运输部规划研究院（bhz）
 * @since 2013-10-28
 */

public class BaseJdbcDao {

    /** JSON数据行映射器 */
    private static final JsonRowMapper JSON_ROW_MAPPER = new JsonRowMapper();

    /** JDBC调用模板 */
    private JdbcTemplate jdbcTemplate;

    /** 启动时间 */
    private static Date startTime;

    /**
     * <B>方法名称：</B>初始化JDBC调用模板<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param dataSource 数据源
     */
    @Autowired
    public void initJdbcTemplate(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
        if (startTime == null) {
            startTime = getCurrentTime();
        }
    }

    /**
     * <B>取得：</B>JDBC调用模板<BR>
     * 
     * @return JdbcTemplate JDBC调用模板
     */
    public JdbcTemplate getJdbcTemplate() {
        return jdbcTemplate;
    }

    /**
     * <B>取得：</B>启动时间<BR>
     * 
     * @return Date 启动时间
     */
    public Date getStartTime() {
        return startTime;
    }

    /**
     * 
     * <B>方法名称：</B>获取数据库的当前时间<BR>
     * <B>概要说明：</B><BR>
     * 
     * @return Date 当前时间
     */
    public Date getCurrentTime() {
        return this.getJdbcTemplate().queryForObject("SELECT SYSTIMESTAMP FROM DUAL", Date.class);
    }

    /**
     * <B>方法名称：</B>查询JSON列表<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param sql SQL语句
     * @param args 参数
     * @return List<JSONObject> JSON列表
     */
    public List<JSONObject> queryForJsonList(String sql, Object... args) {
        return this.jdbcTemplate.query(sql, JSON_ROW_MAPPER, args);
    }

    /**
     * <B>方法名称：</B>查询JSON数据<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param sql SQL语句
     * @param args 参数
     * @return JSONObject JSON数据
     */
    public JSONObject queryForJsonObject(String sql, Object... args) {
        List<JSONObject> jsonList = queryForJsonList(sql, args);
        if (jsonList == null || jsonList.size() < 1) {
            return null;
        }
        return jsonList.get(0);
    }

    /**
     * <B>方法名称：</B>查询文本<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param sql SQL语句
     * @param args 参数
     * @return String 文本
     */
    public String queryForString(String sql, Object... args) {
        List<String> dataList = this.jdbcTemplate.queryForList(sql, args, String.class);
        if (dataList == null || dataList.size() < 1) {
            return null;
        }
        return dataList.get(0);
    }

    /**
     * <B>方法名称：</B>拼接分页语句<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param sql SQL语句
     * @param start 开始记录行数（0开始）
     * @param limit 每页限制记录数
     */
    public void appendPageSql(StringBuffer sql, int start, int limit) {
        sql.insert(0, "SELECT * FROM (SELECT PAGE_VIEW.*, ROWNUM AS ROW_SEQ_NO FROM (");
        sql.append(") PAGE_VIEW WHERE ROWNUM <= " + (start + limit));
        sql.append(") WHERE ROW_SEQ_NO > " + start);
    }

    /**
     * <B>方法名称：</B>拼接管理机构子查询语句<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param sql SQL语句
     * @param params 参数列表
     * @param orgId 管理机构标识
     */
    public void appendOrgSubQuerySql(StringBuffer sql, List<Object> params, String orgId) {
        sql.append("SELECT ORG_ID FROM MST_ORG_REF WHERE PARENT_ID = ?");
        params.add(orgId);
    }

    /**
     * <B>方法名称：</B>获取唯一键值<BR>
     * <B>概要说明：</B><BR>
     * @return String 唯一键值
     */
    public String generateKey() {
        String sql = "SELECT '0000' || TO_CHAR(SYSTIMESTAMP, 'YYYYMMDD') FROM DUAL ";
        String pre = this.getJdbcTemplate().queryForObject(sql, String.class);
        String uid = UUID.randomUUID().toString().replaceAll("-", "").toUpperCase();
        return pre + uid.substring(12);
    }

    /**
     * <B>方法名称：</B>用于 in 通配符(?) 的拼接<BR>
     * <B>概要说明：</B>字段 in(?,?,?,?,?)<BR>
     * 
     * @param sql sql
     * @param sqlArgs 参数容器
     * @param params 参数的个数
     */
    public void appendSqlIn(StringBuffer sql, List<Object> sqlArgs, String[] params) {
        if (params != null && params.length > 0) {
            sql.append(" (");
            for (int i = 0; i < params.length; i++) {
                if (i == 0) {
                    sql.append("?");
                }
                else {
                    sql.append(",?");
                }
                sqlArgs.add(params[i]);
            }
            sql.append(") ");
        }
    }

    /**
     * <B>方法名称：</B>适应SQL列名<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param c 原列名
     * @return String 调整后列名
     */
    public static String c(String c) {
        if (StringUtils.isBlank(c)) {
            return null;
        }
        return c.trim().toUpperCase();
    }

    /**
     * <B>方法名称：</B>适应SQL参数<BR>
     * <B>概要说明：</B>防止SQL注入问题<BR>
     * 
     * @param v 参数
     * @return String 调整后参数
     */
    public static String v(String v) {
        if (StringUtils.isBlank(v)) {
            return null;
        }
        return v.trim().replaceAll("'", "''");
    }

    /**
     * <B>方法名称：</B>获取日期文本值<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param rs 结果集
     * @param column 列名
     * @return String 文本值
     * @throws SQLException SQL异常错误
     */
    protected String getDate(ResultSet rs, String column) throws SQLException {
        Date date = rs.getDate(column);
        if (date == null) {
            return null;
        }
        return DateFormatUtils.format(date, Const.FORMAT_DATE);
    }

    /**
     * <B>方法名称：</B>获取日期时间文本值<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param rs 结果集
     * @param column 列名
     * @return String 文本值
     * @throws SQLException SQL异常错误
     */
    protected String getDateTime(ResultSet rs, String column) throws SQLException {
        Date date = rs.getDate(column);
        if (date == null) {
            return null;
        }
        return DateFormatUtils.format(date, Const.FORMAT_DATETIME);
    }

    /**
     * <B>方法名称：</B>获取时间戳文本值<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param rs 结果集
     * @param column 列名
     * @return String 文本值
     * @throws SQLException SQL异常错误
     */
    protected String getTimestamp(ResultSet rs, String column) throws SQLException {
        Date date = rs.getDate(column);
        if (date == null) {
            return null;
        }
        return DateFormatUtils.format(date, Const.FORMAT_TIMESTAMP);
    }
    
    /**
     * <B>方法名称：</B>单表INSERT方法<BR>
     * <B>概要说明：</B>单表INSERT方法<BR>
     * @param tableName 表名
     * @param data JSONObject对象
     */
    protected int insert(String tableName, JSONObject data) {
        
    	if (data.size() <= 0) {
            return 0;
        }
    	
        StringBuffer sql = new StringBuffer();
        sql.append(" INSERT INTO ");
        sql.append(tableName + " ( ");
    	
    	Set<Entry<String, Object>> set = data.entrySet();
    	List<Object> sqlArgs = new ArrayList<Object>();
    	for (Iterator<Entry<String, Object>> iterator = set.iterator(); iterator.hasNext();) {
			Entry<String, Object> entry = (Entry<String, Object>) iterator.next();
			sql.append(entry.getKey() + ",");
			sqlArgs.add(entry.getValue());
		}

        sql.delete(sql.length() - 1, sql.length());
        sql.append(" ) VALUES ( ");
        for (int i = 0; i < set.size(); i++) {
            sql.append("?,");
        }
        
        sql.delete(sql.length() - 1, sql.length());
        sql.append(" ) ");
        
        return this.getJdbcTemplate().update(sql.toString(), sqlArgs.toArray()); 
    }
    
    
    /**
     * <B>方法名称：</B>批量新增數據方法<BR>
     * <B>概要说明：</B><BR>
     * @param tableName 数据库表名称
     * @param list 插入数据集合
     */
    protected void insertBatch(String tableName, final List<LinkedHashMap<String, Object>> list) {
        
        if (list.size() <= 0) {
            return;
        }
        
        LinkedHashMap<String, Object> linkedHashMap = list.get(0);
        
        StringBuffer sql = new StringBuffer();
        sql.append(" INSERT INTO ");
        sql.append(tableName + " ( ");
        
        final String[] keyset =  (String[]) linkedHashMap.keySet().toArray(new String[linkedHashMap.size()]);
        
        for (int i = 0; i < linkedHashMap.size(); i++) {
            sql.append(keyset[i] + ",");
        }
        
        sql.delete(sql.length() - 1, sql.length());
       
        sql.append(" ) VALUES ( ");
        for (int i = 0; i < linkedHashMap.size(); i++) {
            sql.append("?,");
        }
        
        sql.delete(sql.length() - 1, sql.length());
        sql.append(" ) ");
        
        this.getJdbcTemplate().batchUpdate(sql.toString(), new BatchPreparedStatementSetter() {
            public void setValues(PreparedStatement ps, int i) throws SQLException {
                LinkedHashMap<String, Object>  map = list.get(i);
                Object[] valueset = map.values().toArray(new Object[map.size()]);
                int len = map.keySet().size();
                for (int j = 0; j < len; j++) {
                    ps.setObject(j + 1, valueset[j]);
                }
            }
            public int getBatchSize() {
                return list.size();
            }
      });
    } 
}
```

json转换工具类  
/bhz-com/src/main/java/bhz/com/util/FastJsonConvert.java  

```java
package bhz.com.util;

import java.util.ArrayList;
import java.util.List;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.serializer.SerializerFeature;

public class FastJsonConvert {


	private static final SerializerFeature[] featuresWithNullValue = { SerializerFeature.WriteMapNullValue, SerializerFeature.WriteNullBooleanAsFalse,
	        SerializerFeature.WriteNullListAsEmpty, SerializerFeature.WriteNullNumberAsZero, SerializerFeature.WriteNullStringAsEmpty };

	/**
	 * <B>方法名称：</B>将JSON字符串转换为实体对象<BR>
	 * <B>概要说明：</B>将JSON字符串转换为实体对象<BR>
	 * @param data JSON字符串
	 * @param clzss 转换对象
	 * @return T
	 */
	public static <T> T convertJSONToObject(String data, Class<T> clzss) {
		try {
			T t = JSON.parseObject(data, clzss);
			return t;
		} catch (Exception e) {
			e.printStackTrace();
			return null;
		}
	}
	
	/**
	 * <B>方法名称：</B>将JSONObject对象转换为实体对象<BR>
	 * <B>概要说明：</B>将JSONObject对象转换为实体对象<BR>
	 * @param data JSONObject对象
	 * @param clzss 转换对象
	 * @return T
	 */
	public static <T> T convertJSONToObject(JSONObject data, Class<T> clzss) {
		try {
			T t = JSONObject.toJavaObject(data, clzss);
			return t;
		} catch (Exception e) {
			e.printStackTrace();
			return null;
		}
	}

	/**
	 * <B>方法名称：</B>将JSON字符串数组转为List集合对象<BR>
	 * <B>概要说明：</B>将JSON字符串数组转为List集合对象<BR>
	 * @param data JSON字符串数组
	 * @param clzss 转换对象
	 * @return List<T>集合对象
	 */
	public static <T> List<T> convertJSONToArray(String data, Class<T> clzss) {
		try {
			List<T> t = JSON.parseArray(data, clzss);
			return t;
		} catch (Exception e) {
			e.printStackTrace();
			return null;
		}
	}
	
	/**
	 * <B>方法名称：</B>将List<JSONObject>转为List集合对象<BR>
	 * <B>概要说明：</B>将List<JSONObject>转为List集合对象<BR>
	 * @param data List<JSONObject>
	 * @param clzss 转换对象
	 * @return List<T>集合对象
	 */
	public static <T> List<T> convertJSONToArray(List<JSONObject> data, Class<T> clzss) {
		try {
			List<T> t = new ArrayList<T>();
			for (JSONObject jsonObject : data) {
				t.add(convertJSONToObject(jsonObject, clzss));
			}
			return t;
		} catch (Exception e) {
			e.printStackTrace();
			return null;
		}
	}

	/**
	 * <B>方法名称：</B>将对象转为JSON字符串<BR>
	 * <B>概要说明：</B>将对象转为JSON字符串<BR>
	 * @param obj 任意对象
	 * @return JSON字符串
	 */
	public static String convertObjectToJSON(Object obj) {
		try {
			String text = JSON.toJSONString(obj);
			return text;
		} catch (Exception e) {
			e.printStackTrace();
			return null;
		}
	}
	
	/**
	 * <B>方法名称：</B>将对象转为JSONObject对象<BR>
	 * <B>概要说明：</B>将对象转为JSONObject对象<BR>
	 * @param obj 任意对象
	 * @return JSONObject对象
	 */
	public static JSONObject convertObjectToJSONObject(Object obj){
		try {
			JSONObject jsonObject = (JSONObject) JSONObject.toJSON(obj);
			return jsonObject;
		} catch (Exception e) {
			e.printStackTrace();
			return null;
		}		
	}


	/**
	 * <B>方法名称：</B><BR>
	 * <B>概要说明：</B><BR>
	 * @param obj
	 * @return
	 */
	public static String convertObjectToJSONWithNullValue(Object obj) {
		try {
			String text = JSON.toJSONString(obj, featuresWithNullValue);
			return text;
		} catch (Exception e) {
			e.printStackTrace();
			return null;
		}
	}

	public static void main(String[] args) {
		System.err.println(System.getProperties());
	}
}
```


bhz-sys-facade  
一个新的项目有这三个包  
bhz.sys.entity，用jdbctemplate可以没有实体  
bhz.sys.facade，提供restful接口  
bhz.sys.serial，用于序列化  

/bhz-sys-facade/pom.xml  

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  
  <!-- 继承父类 -->
  <parent>
  	<groupId>bhz</groupId>
  	<artifactId>bhz-parent</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<!-- 设置父类的绝对路径 -->
	<relativePath>../bhz-parent/pom.xml</relativePath>
  </parent> 
  <artifactId>bhz-sys-facade</artifactId>
  <packaging>jar</packaging>

  <name>bhz-sys-facade</name>
  <build>
    <finalName>bhz-sys-facade</finalName>
  </build>
</project>
```

/bhz-sys-facade/src/main/java/bhz/sys/entity/SysUser.java  
实体类在使用jdbctemplate可以没有  

```java
package bhz.sys.entity;
import java.io.Serializable;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
import org.codehaus.jackson.annotate.JsonProperty;

//@XmlRootElement
public class SysUser implements Serializable{

    @NotNull
    private String id;

    @JsonProperty("name")
    //@XmlElement(name = "name")
    @NotNull
    @Size(min = 6, max = 50)
    private String name;
    
	public SysUser() {
	}
	
	public String getId() {
		return id;
	}
	public void setId(String id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
}
```

/bhz-sys-facade/src/main/java/bhz/sys/facade/SysUserFacade.java  
bhz.sys.facade，提供restful接口  

```java
package bhz.sys.facade;

import javax.ws.rs.Consumes;
import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

import com.alibaba.dubbo.rpc.protocol.rest.support.ContentType;

import bhz.sys.entity.SysUser;

@Path("/sysUserService")
@Consumes({MediaType.APPLICATION_JSON, MediaType.TEXT_XML})
@Produces({ContentType.APPLICATION_JSON_UTF_8, ContentType.TEXT_XML_UTF_8})
public interface SysUserFacade {

	@GET
    @Path("/testget")
	public void testget();
	
    @GET
    @Path("/getUser")
	public SysUser getUser();
	
	@GET
	@Path("/get/{id : \\d+}")
	public SysUser getUser(@PathParam(value = "id") Integer id);
	
	@GET
	@Path("/get/{id : \\d+}/{name}")
	public SysUser getUser(@PathParam(value = "id") Integer id, @PathParam(value = "name") String name);
	
    @POST
    @Path("/testpost")
	public void testpost();
	
    @POST
    @Path("/postUser")
	public SysUser postUser(SysUser user);
	
	@POST
	@Path("/post/{id}")
	public SysUser postUser(@PathParam(value = "id") String id);
	
}
```

/bhz-sys-facade/src/main/java/bhz/sys/serial/SerializationOptimizerImpl.java  
序列化  

```java
package bhz.sys.serial;

import java.util.Collection;
import java.util.LinkedList;
import java.util.List;
import bhz.sys.entity.*;
import com.alibaba.dubbo.common.serialize.support.SerializationOptimizer;

public class SerializationOptimizerImpl implements SerializationOptimizer {

    public Collection<Class> getSerializableClasses() {
        List<Class> classes = new LinkedList<Class>();
        //这里可以把所有需要进行序列化的类进行添加
        classes.add(SysUser.class);
        return classes;
    }
}
```


bhz-sys-service项目  

/bhz-sys-service/pom.xml  
注意这个service要打成jar包  

要有一个main方法入口  
<mainClass>com.alibaba.dubbo.container.Main</mainClass>  
依赖的包会放到target/lib下  
<classpathPrefix>lib/</classpathPrefix>  

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  
  <!-- 继承父类 -->
  <parent>
  	<groupId>bhz</groupId>
  	<artifactId>bhz-parent</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<!-- 设置父类的绝对路径 -->
	<relativePath>../bhz-parent/pom.xml</relativePath>
  </parent> 
  <artifactId>bhz-sys-service</artifactId>
  <packaging>jar</packaging>

  <name>bhz-sys-service</name>
  
	<build>
		<finalName>bhz-sys-service</finalName>
		<!-- 配置文件  -->
		<resources>
			<resource>
				<targetPath>${project.build.directory}/classes</targetPath>
				<directory>src/main/resources</directory>
				<filtering>true</filtering>
				<includes>
					<include>**/*.xml</include>
					<include>**/*.properties</include>
				</includes>
			</resource>
			<resource>
				<targetPath>${project.build.directory}/classes/META-INF/spring</targetPath>
				<directory>src/main/resources</directory>
				<filtering>true</filtering>
				<includes>
					<include>spring-context.xml</include>
				</includes>
			</resource>
		</resources>	
		
		<pluginManagement>
			<plugins>
				<!-- 解决Maven插件在Eclipse内执行了一系列的生命周期引起冲突 -->
				<plugin>
					<groupId>org.eclipse.m2e</groupId>
					<artifactId>lifecycle-mapping</artifactId>
					<version>1.0.0</version>
					<configuration>
						<lifecycleMappingMetadata>
							<pluginExecutions>
								<pluginExecution>
									<pluginExecutionFilter>
										<groupId>org.apache.maven.plugins</groupId>
										<artifactId>maven-dependency-plugin</artifactId>
										<versionRange>[2.0,)</versionRange>
										<goals>
											<goal>copy-dependencies</goal>
										</goals>
									</pluginExecutionFilter>
									<action>
										<ignore />
									</action>
								</pluginExecution>
							</pluginExecutions>
						</lifecycleMappingMetadata>
					</configuration>
				</plugin>
			</plugins>
		</pluginManagement>
		<plugins>
			<!-- 打包jar文件时，配置manifest文件，加入lib包的jar依赖 -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-jar-plugin</artifactId>
				<configuration>
					<classesDirectory>target/classes/</classesDirectory>
					<archive>
						<manifest>
							<mainClass>com.alibaba.dubbo.container.Main</mainClass>
							<!-- 打包时 MANIFEST.MF文件不记录的时间戳版本 -->
							<useUniqueVersions>false</useUniqueVersions>
							<addClasspath>true</addClasspath>
							<classpathPrefix>lib/</classpathPrefix>
						</manifest>
						<manifestEntries>
							<Class-Path>.</Class-Path>
						</manifestEntries>
					</archive>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-dependency-plugin</artifactId>
				<executions>
					<execution>
						<id>copy-dependencies</id>
						<phase>package</phase>
						<goals>
							<goal>copy-dependencies</goal>
						</goals>
						<configuration>
							<type>jar</type>
							<includeTypes>jar</includeTypes>
							<useUniqueVersions>false</useUniqueVersions>
							<outputDirectory>
								${project.build.directory}/lib
							</outputDirectory>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>  
  
  <dependencies>
  	<dependency>
  		<groupId>bhz</groupId>
  		<artifactId>bhz-com</artifactId>
  		<version>0.0.1-SNAPSHOT</version>
  	</dependency>
  	<dependency>
  		<groupId>bhz</groupId>
  		<artifactId>bhz-sys-facade</artifactId>
  		<version>0.0.1-SNAPSHOT</version>
  	</dependency> 
  </dependencies>
    
</project>
```

/bhz-sys-service/src/main/webapp/WEB-INF/web.xml  
BootstrapListener，dubbo的listener放到spring容器前  
dubbo访问的前缀规则  
/bhz-sys-service/*

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xmlns:jsp="http://java.sun.com/xml/ns/javaee/jsp" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
  <display-name>bhz-sys-service</display-name>

  <context-param>
    <param-name>webAppRootKey</param-name>
    <param-value>bhz-sys-service.root</param-value>
  </context-param>
  <context-param>
    <param-name>log4jConfigLocation</param-name>
    <param-value>/WEB-INF/classes/log4j.properties</param-value>
  </context-param>
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath*:spring-context.xml</param-value>
  </context-param>
  
  <!-- 配置dubbox remoting -->
  <!--this listener must be defined before the spring listener-->
  <listener>
      <listener-class>com.alibaba.dubbo.remoting.http.servlet.BootstrapListener</listener-class>
  </listener> 
  <listener>
    <listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>
  </listener>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  
  <!-- 配置dubbox -->	 
  <servlet>
  	<servlet-name>dubboxDispatcher</servlet-name>
  	<servlet-class>com.alibaba.dubbo.remoting.http.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>2</load-on-startup>
	</servlet>
  <servlet-mapping>
  	<servlet-name>dubboxDispatcher</servlet-name>
  	<url-pattern>/bhz-sys-service/*</url-pattern>
  </servlet-mapping>
  
</web-app>
```

/bhz-sys-service/src/main/resources/config.properties  
配置zookeeper、tomcat

```
# Environment Config Properties.
env.defaultEncoding=UTF-8

# JDBC Database Config Properties.
jdbc.driver=oracle.jdbc.driver.OracleDriver
jdbc.url=jdbc:oracle:thin:@//192.168.1.200:1521/orcl
jdbc.username=scott
jdbc.password=tiger
jdbc.minConnection=2
jdbc.maxConnection=40
jdbc.maxConnectionLife=1800000
jdbc.maxActiveTime=300000
jdbc.prototypeCount=1
jdbc.testSql=SELECT 1 FROM DUAL

# Dubbox Config Properties.
dubbox.registry.address=zookeeper://192.168.1.111:2181
dubbox.application=bhz-sys-service
dubbox.rest.server=tomcat
dubbox.rest.port=8888
dubbox.rest.contextpath=bhz-sys-service
dubbox.rest.threads=500
dubbox.rest.accepts=500
```

dubbo-provider，dubbo服务生产者的配置  
/bhz-sys-service/src/main/resources/dubbo-provider.xml  
bhz.sys.serial.SerializationOptimizerImpl类在bhz-sys-facade中，可以找到，因为maven依赖  
这段配置<!-- 具体的实现bean -->不应该这么写，在provider应该用注解实现  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xmlns:util="http://www.springframework.org/schema/util"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
	http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-3.0.xsd">
	
	<!-- 引入配置文件 -->
    <context:property-placeholder location="classpath:config.properties" />

    <dubbo:application name="${dubbox.application}" owner="programmer" organization="dubbox"/>

	<!-- zookeeper注册中心 -->
    <dubbo:registry address="${dubbox.registry.address}"/>

    <dubbo:annotation package="bhz" />
    
    <!-- kryo实现序列化 -->
    <dubbo:protocol name="dubbo" serialization="kryo" optimizer="bhz.sys.serial.SerializationOptimizerImpl" />
   
	<!-- rest -->
    <dubbo:protocol name="rest" server="${dubbox.rest.server}" port="${dubbox.rest.port}" contextpath="${dubbox.rest.contextpath}" threads="${dubbox.rest.threads}" accepts="${dubbox.rest.accepts}" />
	
     
   	<!-- 具体的实现bean -->
	<!--
	<bean id="sysUserService" class="bhz.sys.service.SysUserService" />
	--> 
	<!-- 声明需要暴露的服务接口,同时提供本地dubbo方法调用和rest方法调用 -->
	<!--  
	<dubbo:service interface="bhz.sys.facade.SysUserFacade" ref="sysUserService" protocol="rest,dubbo" />
	-->
</beans>
```

spring配置  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://www.springframework.org/schema/beans"
    xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="
    http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
    http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

	<!-- 引入配置文件  -->
	<context:property-placeholder location="classpath:config.properties" />
	
    <!-- 配置数据源 -->
    <bean id="dataSource" class="org.logicalcobwebs.proxool.ProxoolDataSource">
        <property name="driver" value="${jdbc.driver}" />
        <property name="driverUrl" value="${jdbc.url}" />
        <property name="user" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
        <property name="minimumConnectionCount" value="${jdbc.minConnection}" />
        <property name="maximumConnectionCount" value="${jdbc.maxConnection}" />
        <property name="maximumConnectionLifetime" value="${jdbc.maxConnectionLife}" />
        <property name="maximumActiveTime" value="${jdbc.maxActiveTime}" />
        <property name="prototypeCount" value="${jdbc.prototypeCount}" />
        <property name="houseKeepingTestSql" value="${jdbc.testSql}" />
    </bean>

	<!-- 配置事务管理器 -->
	<bean id="transactionManager"
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource" />
	</bean>
	
	<!-- 注解方式配置事物 -->
	<tx:annotation-driven transaction-manager="transactionManager" />
	
	<!-- 注解解析 -->
	<context:annotation-config />
	
	<!-- 扫描所有spring bean注解 -->
    <context:component-scan base-package="bhz" />
	
	<!-- 动态代理 -->
	<aop:aspectj-autoproxy/>
	
	<!-- 配置dubbo服务 -->
	<import resource="classpath:dubbo-provider.xml" />
	
</beans>
```

下面这两个类用作测试  
/bhz-sys-service/src/main/java/bhz/sys/dao/SysUserDao.java  

```java
package bhz.sys.dao;

import org.springframework.stereotype.Repository;

import bhz.com.dao.BaseJdbcDao;

@Repository("sysUserDao")
public class SysUserDao extends BaseJdbcDao {

}
```

/bhz-sys-service/src/main/java/bhz/sys/service/SysUserService.java  
实现这个SysUserFacade接口  

```java
package bhz.sys.service;

import org.springframework.stereotype.Service;

import bhz.sys.entity.SysUser;
import bhz.sys.facade.SysUserFacade;


@Service("sysUserService")
@com.alibaba.dubbo.config.annotation.Service(interfaceClass=bhz.sys.facade.SysUserFacade.class, protocol = {"rest", "dubbo"})
public class SysUserService implements SysUserFacade {


	public void testget() {
		//http://localhost:8888/bhz-sys-service/sysUserService/testget
		System.out.println("测试...get");
	}
	
	public SysUser getUser() {
		//http://localhost:8888/bhz-sys-service/sysUserService/getUser
    	SysUser user = new SysUser();
    	user.setId("1001");
    	user.setName("张三");
    	return user;
	}

	public SysUser getUser(Integer id) {
		//http://localhost:8888/bhz-sys-service/sysUserService/get/1001
		System.out.println(id);
		System.out.println("测试...get");
		SysUser user = new SysUser();
    	user.setId("1001");
    	user.setName("张三");
    	return user;
	}

	public SysUser getUser(Integer id, String name) {
		//http://localhost:8888/bhz-sys-service/sysUserService/get/1001/z3
		System.out.println(id);
		System.out.println(name);
		System.out.println("测试...get");
		SysUser user = new SysUser();
    	user.setId("1001");
    	user.setName("张三");
    	return user;
	}
	
	public void testpost() {
    	System.out.println("测试...post");
	}
    
	public SysUser postUser(SysUser user) {
    	System.out.println(user.getName());
    	System.out.println("测试...postUser");
    	SysUser user1 = new SysUser();
    	user.setId("1001");
    	user1.setName("张三");
    	return user1;
	}

	public SysUser postUser(String id) {
		System.out.println(id);
		System.out.println("测试...get");
		SysUser user = new SysUser();
    	user.setId("1001");
    	user.setName("张三");
    	return user;
	}
}
```

/bhz-sys-service/src/test/java/bhz/test/Provider.java  
这个测试类为了启动spring，并让服务一直开启  

```java
package bhz.test;


import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Provider {

	public static void main(String[] args) throws Exception {
		ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(
				new String[] { "spring-context.xml" });
		context.start();
		System.in.read(); // 为保证服务一直开着，利用输入流的阻塞来模拟
	}
}
```

4. 分布式项目6  

把dubbo部署到tomcat上，service打成jar包放到linux运行  

5. 分布式项目7  

web端服务调用  
/bhz-sys/pom.xml  
bhz-sys-facade依赖加进来  

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <!-- 继承父类 -->
  <parent>
  	<groupId>bhz</groupId>
  	<artifactId>bhz-parent</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<!-- 设置父类的绝对路径 -->
	<relativePath>../bhz-parent/pom.xml</relativePath>
  </parent>  
  <artifactId>bhz-sys</artifactId>
  <packaging>war</packaging>
  <dependencies>
  	<dependency>
  		<groupId>bhz</groupId>
  		<artifactId>bhz-com</artifactId>
  		<version>0.0.1-SNAPSHOT</version>
  	</dependency>  
  	<dependency>
  		<groupId>bhz</groupId>
  		<artifactId>bhz-sys-facade</artifactId>
  		<version>0.0.1-SNAPSHOT</version>
  	</dependency>
  </dependencies>
  <build>
    <finalName>bhz-sys</finalName>
  </build>
</project>

```

其他改动  

config.properties  
添加Dubbox的配置  

```
# Environment Config Properties.
env.defaultEncoding=UTF-8

# JDBC Database Config Properties.
jdbc.driver=oracle.jdbc.driver.OracleDriver
jdbc.url=jdbc:oracle:thin:@//192.168.1.200:1521/orcl
jdbc.username=bhz
jdbc.password=bhz
jdbc.minConnection=2
jdbc.maxConnection=40
jdbc.maxConnectionLife=1800000
jdbc.maxActiveTime=300000
jdbc.prototypeCount=1
jdbc.testSql=SELECT 1 FROM DUAL

# File Upload Config Properties.
upload.maxMemSize=102400
upload.maxFileSize=104857600
upload.tempDir=/WEB-INF/upload/

# Dubbox Config Properties.
dubbox.registry.address=zookeeper://192.168.1.111:2181?backup=192.168.1.112:2181,192.168.1.113:2181
dubbox.application=bhz-sys
```

spring-context.xml  

```xml
	<!-- 配置dubbo服务 -->
	<import resource="classpath:dubbo-consumer.xml" />
```

dubbo-consumer.xml  
新添加这个配置文件  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xmlns:util="http://www.springframework.org/schema/util"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
	http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-3.0.xsd">

    <!-- 引入配置文件 -->
    <context:property-placeholder location="classpath:config.properties" />

	<!-- 消费服务名称 -->
    <dubbo:application name="${dubbox.application}" owner="programmer" organization="dubbox"/>

	<!-- zookeeper注册中心  zookeeper://192.168.1.111:2181 -->
    <dubbo:registry address="${dubbox.registry.address}"/>
    
    <!-- 扫描dubbox注解位置 -->
    <dubbo:annotation package="bhz" />
    
    <!-- kryo实现序列化  -->
    <dubbo:protocol name="dubbo" serialization="kryo" optimizer="bhz.sys.serial.SerializationOptimizerImpl" />

	<!-- 生成远程服务接口配置 -->
	<dubbo:reference interface="bhz.sys.facade.SysUserFacade" id="sysUserFacade" />
</beans>
```

6. 分布式项目8  

之前是运行 java -jar bhz-sys-service.jar &  
现在编写脚本sys-service.sh  

```sh
#!/bin/sh
## java env
export JAVA_HOME=/usr/local/jdk1.7
export JRE_HOME=$JAVA_HOME/jre

## exec shell name
EXEC_SHELL_NAME=sys-service\.sh
## service name
SERVICE_NAME=bhz-sys-service
SERVICE_DIR=/usr/local/workspace/sys-service
JAR_NAME=$SERVICE_NAME\.jar
PID=$SERVICE_NAME\.pid

cd $SERVICE_DIR

case "$1" in

    start)
        nohup $JRE_HOME/bin/java -Xms256m -Xmx512m -jar $JAR_NAME >/dev/null 2>&1 &
        echo $! > $SERVICE_DIR/$PID
        echo "#### start $SERVICE_NAME"
        ;;

    stop)
        kill `cat $SERVICE_DIR/$PID`
        rm -rf $SERVICE_DIR/$PID
        echo "#### stop $SERVICE_NAME"

        sleep 5

        TEMP_PID=`ps -ef | grep -w "$SERVICE_NAME" | grep -v "grep" | awk '{print $2}'`
        if [ "$TEMP_PID" == "" ]; then
            echo "#### $SERVICE_NAME process not exists or stop success"
        else
            echo "#### $SERVICE_NAME process pid is:$TEMP_PID"
            kill -9 $TEMP_PID
        fi
        ;;

    restart)
        $0 stop
        sleep 2
        $0 start
        echo "#### restart $SERVICE_NAME"
        ;;

    *)
        ## restart
        $0 stop
	sleep 2
        $0 start
        ;;

esac
exit 0
```

修改log4j.properties  
修改log4j.appender.file.File为logs/bhz-sys-service.log  

```
log4j.rootLogger=INFO, console, file

log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d %p [%c] - %m%n
log4j.appender.file=org.apache.log4j.DailyRollingFileAppender
#log4j.appender.file.File=D:/002_developer/workspace_001/zcmoni.log
log4j.appender.file.File=logs/bhz-sys-service.log
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d %p [%c] - %m%n
log4j.logger.org.springframework=WARN
```

添加前端代码  
前端ext代码见项目  

/bhz-sys/src/main/webapp/WEB-INF/web.xml  
添加一个过滤器处理压缩的js  

```xml
 
  <filter>
      <filter-name>GzipFilter</filter-name>
      <filter-class>bhz.com.util.GzipFilter</filter-class>
      <init-param>
          <param-name>headers</param-name>
          <param-value>Content-Encoding=gzip</param-value>
      </init-param>
  </filter>
  <filter-mapping>
      <filter-name>GzipFilter</filter-name>
      <url-pattern>*.gzjs</url-pattern>
  </filter-mapping>
  <filter-mapping>
      <filter-name>GzipFilter</filter-name>
      <url-pattern>*.gzcss</url-pattern>
  </filter-mapping>
    
```

/bhz-com/src/main/java/bhz/com/util/GzipFilter.java  
过滤器的代码  

```java
package bhz.com.util;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;
import java.util.Map.Entry;
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletResponse;
public class GzipFilter implements Filter {

    /** 参数键值：头信息 */
    public static final String PARAM_KEY_HEADERS = "headers";

    /** 头信息 */
    private Map<String, String> headers;

    /**
     * <B>方法名称：</B>初始化<BR>
     * <B>概要说明：</B>初始化过滤器<BR>
     * 
     * @param fConfig 过滤器配置
     * @throws ServletException Servlet异常
     * @see javax.servlet.Filter#init(javax.servlet.FilterConfig)
     */
    public void init(FilterConfig fConfig) throws ServletException {
        String param = fConfig.getInitParameter(PARAM_KEY_HEADERS);
        if (param == null || param.trim().length() < 1) {
            return;
        }
        this.headers = new HashMap<String, String>();
        String[] params = param.split(",");
        String[] kvs = null;
        for (int i = 0; i < params.length; i++) {
            param = params[i];
            if (param != null && param.trim().length() > 0) {
                kvs = param.split("=");
                if (kvs != null && kvs.length == 2) {
                    headers.put(kvs[0], kvs[1]);
                }
            }
        }
        if (this.headers.isEmpty()) {
            this.headers = null;
        }
    }

    /**
     * <B>方法名称：</B>过滤处理<BR>
     * <B>概要说明：</B>设定编码格式<BR>
     * 
     * @param request 请求
     * @param response 响应
     * @param chain 过滤器链
     * @throws IOException IO异常
     * @throws ServletException Servlet异常
     * @see javax.servlet.Filter#doFilter(javax.servlet.ServletRequest,
     *      javax.servlet.ServletResponse, javax.servlet.FilterChain)
     */
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException,
            ServletException {
        if (this.headers != null) {
            HttpServletResponse res = (HttpServletResponse) response;
            for (Entry<String, String> header : this.headers.entrySet()) {
                res.addHeader(header.getKey(), header.getValue());
            }
        }
        chain.doFilter(request, response);
    }

    /**
     * <B>方法名称：</B>释放资源<BR>
     * <B>概要说明：</B>释放过滤器资源<BR>
     * 
     * @see javax.servlet.Filter#destroy()
     */
    public void destroy() {
        this.headers.clear();
        this.headers = null;
    }

}
```

## 2.0版本控制  

1. 分布式项目10  

创建数据库表  

```sql
/*==============================================================*/
/* DBMS name:      ORACLE Version 11g                           */
/* Created on:     2015/12/5 10:29:07                           */
/*==============================================================*/


alter table SYS_ROLE_FUNC
   drop constraint FK_SYS_ROLE_FUNC_FUNC;

alter table SYS_ROLE_FUNC
   drop constraint FK_SYS_ROLE_FUNC_ROLE;

alter table SYS_USER
   drop constraint FK_SYS_USER_ROLE;

drop table SYS_FILE cascade constraints;

drop table SYS_FUNC cascade constraints;

drop table SYS_LOG cascade constraints;

drop table SYS_PARAM cascade constraints;

drop table SYS_ROLE cascade constraints;

drop table SYS_ROLE_FUNC cascade constraints;

drop table SYS_TASK cascade constraints;

drop table SYS_USER cascade constraints;

/*==============================================================*/
/* Table: SYS_FILE                                              */
/*==============================================================*/
create table SYS_FILE 
(
   KEY                  VARCHAR2(32)         not null,
   TYPE                 VARCHAR2(1)          not null,
   NAME                 VARCHAR2(80)         not null,
   EXT                  VARCHAR2(8),
   BYTES                NUMBER(12)           not null,
   DATA                 BLOB                 not null,
   EXPIRED              TIMESTAMP,
   DESC_INFO            VARCHAR2(200),
   UPDATE_BY            VARCHAR2(40)         not null,
   UPDATE_TIME          TIMESTAMP            not null,
   constraint PK_SYS_FILE primary key (KEY)
);

/*==============================================================*/
/* Table: SYS_FUNC                                              */
/*==============================================================*/
create table SYS_FUNC 
(
   FUNC_CODE            VARCHAR2(40)         not null,
   FUNC_NAME            VARCHAR2(40),
   FUNC_TYPE            VARCHAR2(40),
   FUNC_PATH            VARCHAR2(40),
   ORDER_SEQ            NUMBER(10),
   DISABLE_FLAH         VARCHAR2(1),
   DESC_INFO            VARCHAR2(200),
   CREATE_BY            VARCHAR2(40)         not null,
   CREATE_TIME          TIMESTAMP            not null,
   UPDATE_BY            VARCHAR2(40)         not null,
   UPDATE_TIME          TIMESTAMP            not null,
   constraint PK_SYS_FUNC primary key (FUNC_CODE)
);

/*==============================================================*/
/* Table: SYS_LOG                                               */
/*==============================================================*/
create table SYS_LOG 
(
   LOG_TASK             VARCHAR2(100)        not null,
   LOG_TIME             TIMESTAMP            not null,
   LOG_TEXT             VARCHAR2(1000)       not null,
   REF01                VARCHAR2(200),
   REF02                VARCHAR2(200),
   REF03                VARCHAR2(200),
   REF04                VARCHAR2(200),
   REF05                VARCHAR2(200),
   REF06                VARCHAR2(200),
   REF07                VARCHAR2(200),
   REF08                VARCHAR2(200),
   REF09                VARCHAR2(200),
   REF10                VARCHAR2(200)
);

/*==============================================================*/
/* Table: SYS_PARAM                                             */
/*==============================================================*/
create table SYS_PARAM 
(
   PARAM_CODE           VARCHAR2(40)         not null,
   PARAM_NAME           VARCHAR2(40),
   PARAM_VALUE          VARCHAR2(40),
   DESC_INFO            VARCHAR2(200),
   CREATE_BY            VARCHAR2(40)         not null,
   CREATE_TIME          TIMESTAMP            not null,
   UPDATE_BY            VARCHAR2(40)         not null,
   UPDATE_TIME          TIMESTAMP            not null,
   constraint PK_SYS_PARAM primary key (PARAM_CODE)
);

/*==============================================================*/
/* Table: SYS_ROLE                                              */
/*==============================================================*/
create table SYS_ROLE 
(
   ROLE_CODE            VARCHAR(40)          not null,
   ROLE_NAME            VARCHAR(40),
   DISABLE_FLAG         VARCHAR(1),
   DESC_INFO            VARCHAR(200),
   CREATE_BY            VARCHAR2(40)         not null,
   CREATE_TIME          TIMESTAMP            not null,
   UPDATE_BY            VARCHAR2(40)         not null,
   UPDATE_TIME          TIMESTAMP            not null,
   constraint PK_SYS_ROLE primary key (ROLE_CODE)
);

/*==============================================================*/
/* Table: SYS_ROLE_FUNC                                         */
/*==============================================================*/
create table SYS_ROLE_FUNC 
(
   ROLE_CODE            VARCHAR2(40)         not null,
   FUNC_CODE            VARCHAR2(40)         not null,
   constraint PK_SYS_ROLE_FUNC primary key (ROLE_CODE, FUNC_CODE)
);

/*==============================================================*/
/* Table: SYS_TASK                                              */
/*==============================================================*/
create table SYS_TASK 
(
   TASK_CODE            VARCHAR2(40)         not null,
   TASK_NAME            VARCHAR2(100)        not null,
   LAST_TIME            TIMESTAMP,
   constraint AK_SYS_TASK_01 unique (TASK_CODE)
);

/*==============================================================*/
/* Table: SYS_USER                                              */
/*==============================================================*/
create table SYS_USER 
(
   USER_ID              VARCHAR2(40)         not null,
   PASSWORD             VARCHAR2(128)        not null,
   USER_NAME            VARCHAR2(40)         not null,
   ROLE_CODE            VARCHAR2(40),
   ORG_ID               VARCHAR2(32),
   EMAIL                VARCHAR2(40),
   LOGIN_COUNT          NUMBER(10),
   LAST_LOGIN_TIME      TIMESTAMP,
   LAST_LOGIN_IP        VARCHAR2(40),
   DISABLE_FLAG         VARCHAR2(1),
   DESC_INFO            VARCHAR2(200),
   CREATE_BY            VARCHAR2(40)         not null,
   CREATE_TIME          TIMESTAMP            not null,
   UPDATE_BY            VARCHAR2(40)         not null,
   UPDATE_TIME          TIMESTAMP            not null,
   constraint PK_SYS_USER primary key (USER_ID),
   constraint AK_SYS_USER_01 unique (USER_NAME)
);

alter table SYS_ROLE_FUNC
   add constraint FK_SYS_ROLE_FUNC_FUNC foreign key (FUNC_CODE)
      references SYS_FUNC (FUNC_CODE);

alter table SYS_ROLE_FUNC
   add constraint FK_SYS_ROLE_FUNC_ROLE foreign key (ROLE_CODE)
      references SYS_ROLE (ROLE_CODE);

alter table SYS_USER
   add constraint FK_SYS_USER_ROLE foreign key (ROLE_CODE)
      references SYS_ROLE (ROLE_CODE);
```



```sql
insert into SYS_ROLE (role_code, role_name, disable_flag, desc_info, create_by, create_time, update_by, update_time)
values ('SYS_ADMIN', '系统管理员', '0', '1', 'SYS_INIT', SYSTIMESTAMP, 'SYS_INIT', SYSTIMESTAMP);
insert into SYS_USER (user_id, password, user_name, role_code, org_id, email, login_count, last_login_time, last_login_ip, disable_flag, desc_info, create_by, create_time, update_by, update_time)
values ('admin', '21232f297a57a5a743894a0e4a801fc3', 'admin', 'SYS_ADMIN', null, null, null, null, null, '0', null, 'SYS_INIT', SYSTIMESTAMP, 'SYS_INIT', SYSTIMESTAMP);


INSERT INTO SYS_FUNC VALUES('01', '系统运行', '1', '', '', '0',  '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('02', '基础信息', '1', '', '', '0',  '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('03', '检测数据', '1', '', '', '0',  '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('04', '统计分析', '1', '', '', '0',  '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('05', '消息服务', '1', '', '', '0',  '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);

INSERT INTO SYS_FUNC VALUES('0101', '人员管理', '2', '', '1', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('0102', '权限配置', '2', '', '2', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('0103', '系统设置', '2', '', '3', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);

INSERT INTO SYS_FUNC VALUES('0201', '组织管理', '2', '', '1', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('0202', '站点管理', '2', '', '2', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);

INSERT INTO SYS_FUNC VALUES('0301', '数据报送', '2', '', '1', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);

INSERT INTO SYS_FUNC VALUES('0401', '综合查询', '2', '', '1', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('0402', '统计分析', '2', '', '2', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);

INSERT INTO SYS_FUNC VALUES('0501', 'ZK配置',   '2', '', '1', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('0502', 'MQ配置',   '2', '', '2', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);


INSERT INTO SYS_FUNC VALUES('010101', '用户列表', '3', '', '1', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('010102', '角色管理', '3', '', '2', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('010201', '人员权限', '3', '', '1', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('010202', '数据权限', '3', '', '2', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('010301', '系统参数', '3', '', '1', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);

INSERT INTO SYS_FUNC VALUES('020101', '机构列表', '3', '', '1', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('020201', '站点列表', '3', '', '1', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('020202', '设备列表', '3', '', '2', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);

INSERT INTO SYS_FUNC VALUES('030101', '检测数据', '3', '', '1', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);

INSERT INTO SYS_FUNC VALUES('040101', '单车数据查询',        '3', '', '1', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('040102', '日平均超限数据查询',  '3', '', '2', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('040201', '日数据',              '3', '', '1', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('040202', '月数据',              '3', '', '2', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('040203', '季度数据',            '3', '', '3', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);
INSERT INTO SYS_FUNC VALUES('040204', '年数据',              '3', '', '4', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);


INSERT INTO SYS_FUNC VALUES('050101', 'ZK列表', 			 '3', '', '1', '0', '', 'SYS_INIT',SYSTIMESTAMP,'SYS_INIT',SYSTIMESTAMP);

INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','01');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','02');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','03');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','04');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','05');

INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','0101');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','0102');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','0103');

INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','0201');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','0202');

INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','0301');

INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','0401');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','0402');

INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','0501');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','0502');

INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','010101');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','010102');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','010201');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','010202');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','010301');

INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','020101');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','020201');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','020202');

INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','030101');

INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','040101');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','040102');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','040201');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','040202');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','040203');
INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','040204');

INSERT INTO SYS_ROLE_FUNC VALUES('SYS_ADMIN','050101');

COMMIT;
```

/bhz-sys-facade/src/main/java/bhz/sys/serial/SerializationOptimizerImpl.java  
实体都变成JSONObject  

```java
package bhz.sys.serial;
import java.util.Collection;
import java.util.LinkedList;
import java.util.List;
import com.alibaba.dubbo.common.serialize.support.SerializationOptimizer;
import com.alibaba.fastjson.JSONObject;

public class SerializationOptimizerImpl implements SerializationOptimizer {

    public Collection<Class> getSerializableClasses() {
        List<Class> classes = new LinkedList<Class>();
        //这里可以把所有需要进行序列化的类进行添加
        classes.add(JSONObject.class);
        return classes;
    }
}

```

/bhz-sys-facade/src/main/java/bhz/sys/facade/SysUserFacade.java  
修改SysUserFacade  

```java
package bhz.sys.facade;
import java.util.List;
import javax.ws.rs.Consumes;
import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

import com.alibaba.dubbo.rpc.protocol.rest.support.ContentType;
import com.alibaba.fastjson.JSONObject;

@Path("/sysUserService")
@Consumes({MediaType.APPLICATION_JSON, MediaType.TEXT_XML})
@Produces({ContentType.APPLICATION_JSON_UTF_8, ContentType.TEXT_XML_UTF_8})
public interface SysUserFacade {
	
	@POST
	public String generateKey() throws Exception;
	
	@GET
	@Path("/getById/{id}")
	public JSONObject getById(@PathParam(value = "id") String id) throws Exception;
	
	@POST
	@Path("/getList")
	public List<JSONObject> getList() throws Exception;
	
	@POST
	public int insert(JSONObject jsonObject) throws Exception;
}
```

/bhz-sys-service/src/main/java/bhz/sys/dao/SysUserDao.java  
Dao都是用JSONObject  

```java
package bhz.sys.dao;
import java.util.List;
import org.springframework.stereotype.Repository;
import bhz.com.dao.BaseJdbcDao;
import com.alibaba.fastjson.JSONObject;

@Repository("sysUserDao")
public class SysUserDao extends BaseJdbcDao {
	
	private static final String SQL_TABLE_NAME = "SYS_USER";
	private static final String SQL_SELECT_SYS_USER = "SELECT * FROM SYS_USER";
	private static final String SQL_SELECT_SYS_USER_BYID = "SELECT * FROM SYS_USER WHERE USER_ID = ?";
	
	public List<JSONObject> getList() throws Exception {
		return super.queryForJsonList(SQL_SELECT_SYS_USER);
	}
	
	public JSONObject getById(String id){
		return super.queryForJsonObject(SQL_SELECT_SYS_USER_BYID, id);
	}
	
	public int insert(JSONObject jsonObject) throws Exception {
		return super.insert(SQL_TABLE_NAME, jsonObject);
	}
	
}
```

/bhz-sys-service/src/main/java/bhz/sys/service/SysUserService.java  
修改SysUserService  

```java
package bhz.sys.service;
import java.util.Collections;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import bhz.sys.dao.SysUserDao;
import bhz.sys.facade.SysUserFacade;
import com.alibaba.fastjson.JSONObject;

@Service("sysUserService")
@com.alibaba.dubbo.config.annotation.Service(interfaceClass=bhz.sys.facade.SysUserFacade.class, protocol = {"rest", "dubbo"})
public class SysUserService implements SysUserFacade {

	@Autowired
	private SysUserDao sysUserDao;
	
	@Override
	public String generateKey() throws Exception {
		return this.sysUserDao.generateKey();
	}
	
	@Override
	public JSONObject getById(String id) {
		//get
		//http://localhost:8888/bhz-sys-service/sysUserService/getById/{id}
		return this.sysUserDao.getById(id);
	}

	@Override
	public List<JSONObject> getList() throws Exception {
		//post
		//http://localhost:8888/bhz-sys-service/sysUserService/getById/getList
		List<JSONObject> list = this.sysUserDao.getList();
		if(!list.isEmpty()){
			return list;
		} else {
			return Collections.emptyList();
		}
	}

	@Override
	public int insert(JSONObject jsonObject) throws Exception {
		return this.sysUserDao.insert(jsonObject);
	}
}
```

## 3.0版本控制  

持续集成  
分布式项目11-15  
装环境见持续集成安装手册  
jenkins监控svn并用maven把项目进行编译打包部署  

分布式项目16  
ssh sites 配置连接其他服务器  
publish over插件可以把项目发布到其他服务器  

pre step构建之前选择在远程服务器上执行shell  
post step构建之后把sh脚本、编译的jar包、依赖的lib发送到相应服务器  
给脚本赋权，并执行脚本启动项目  


## 4.0版本控制  

CAS单点登录系统  


jdk生成证书  

```
keytool   -genkey   -keystore  "D:\keystore\hellocj.keystore"   -alias   tomcat   -keyalg   RSA   -validity  365      -dname  "CN=localhost, OU=org, O=org.cj, L=昆明, ST=云南, C=中国"   -keypass  testcj  -storepass   hellocj
```

证书变成cer文件  

```
keytool   -alias  "cjTomcat"   -exportcert   -keystore    D:\keystore\hellocj.keystore    -file  D:\keystore\cjTomcat.cer   -storepass   "hellocj"
```

导入jre证书库  
导入到CAS的server端  

```
keytool    -import     -alias    "cjTomcat"    -keystore   C:\Java\jdk1.8.0_40\jre\lib\security\cacerts   -file   D:\keystore\cjTomcat.cer    -trustcacerts    -storepass    changeit
```

server.xml  
支持https，指定证书keystoreFile

```xml
Window ：  
<Connector port="8443" maxHttpHeaderSize="8192"  
     maxThreads="150" minSpareThreads="25" maxSpareThreads="75"  
     enableLookups="false" disableUploadTimeout="true"  
     acceptCount="100" scheme="https" secure="true"  
     clientAuth="false" sslProtocol="TLS" keystoreFile="d:/my.keystore" keystorePass="changeit"/>  
 
Linux：  
<Connector port="8443" maxHttpHeaderSize="8192"  
     maxThreads="150" minSpareThreads="25" maxSpareThreads="75"  
     enableLookups="false" disableUploadTimeout="true"  
     acceptCount="100" scheme="https" secure="true"  
     clientAuth="false" sslProtocol="TLS" keystoreFile="~/my.keystore" keystorePass="changeit"/>

```

部署cas.war  

```
下载：cas-server-4.0.0-release.tar.gz 
解压后拷贝cas-server-4.0.0\modules下cas-server-webapp-4.0.0.war 
重命名为cas.war后拷贝到服务器Tomcat\webapps下即可 

然后在客户端服务器host加入 
服务器ip地址 www.bhz.com 

然后输入https://www.bhz.com:8443/cas/login
即可正常访问 
```

cas/WEB-INF/deployerConfigContext.xml  
修改配置文件，支持查询数据库，用于用户登录  
后面两个bean配置数据源和加密方式，视频中用的是MD5  

```xml
<!--
<bean class="org.jasig.cas.authentication.handler.support.SimpleTestUsernamePasswordAuthenticationHandler" />
-->
<bean class="org.jasig.cas.adaptors.jdbc.QueryDatabaseAuthenticationHandler">
	<property name="dataSource" ref="dataSource"></property>
	<property name="sql" value="select passwd from sampleUser where userName=?"></property>
	<property name="passwordEncoder" ref="SHAPasswordEncoder"></property>
</bean>

	<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
		<property name="driverClassName"><value>org.gjt.mm.mysql.Driver</value></property>
		<property name="url"><value>jdbc:mysql://192.168.56.101:3306/sample?characterEncoding=UTF-8</value></property>
		<property name="username"><value>root</value></property>
		<property name="password"><value>123</value></property>
	</bean>


	<bean id="SHAPasswordEncoder" class="org.jasig.cas.authentication.handler.DefaultPasswordEncoder">
		<constructor-arg index="0">
			<value>SHA-1</value>
		</constructor-arg>
	</bean>

```

最后一步  
把cas-server-support-jdbc-3.4.3.1.jar和mysql驱动的jar放到WEB-INF/lib  





客户端配置  

1、下载cas-client-3.3.3-release.zip 

2、Pom.xml添加依赖  
/bhz-parent/pom.xml  

```xml
<dependency> 
	<groupId>org.jasig.cas.client</groupId> 
	<artifactId>cas-client-core</artifactId> 
	<version>3.2.1</version> 
</dependency> 
```

3、web.xml添加以下配置  

/bhz-sys/src/main/webapp/WEB-INF/web.xml  

过滤器负责用户的认证工作  
casServerLoginUrl是cas服务器的地址，https://www.bhz.com:8443/cas/login  
过滤器负责对Ticket的校验工作  
casServerUrlPrefix，https://www.bhz.com:8443/cas  
还有两个过滤器可以获取用户名  

```xml

	<!-- ======================== 单点登录开始 ======================== -->  
  	<!-- 用于单点退出，该过滤器用于实现单点登出功能，可选配置-->  
	<listener>  
	    <listener-class>org.jasig.cas.client.session.SingleSignOutHttpSessionListener</listener-class>  
	</listener>  
	<!-- 该过滤器用于实现单点登出功能，可选配置。 -->  
	<filter>  
	    <filter-name>CAS Single Sign Out Filter</filter-name>  
	    <filter-class>org.jasig.cas.client.session.SingleSignOutFilter</filter-class>  
	</filter>  
	<filter-mapping>  
	    <filter-name>CAS Single Sign Out Filter</filter-name>  
	    <url-pattern>/*</url-pattern>  
	</filter-mapping>  
	<!-- 该过滤器负责用户的认证工作，必须启用它 -->  
	<filter>  
	    <filter-name>CASFilter</filter-name>  
	    <filter-class>org.jasig.cas.client.authentication.AuthenticationFilter</filter-class>  
	    <init-param>  
	        <param-name>casServerLoginUrl</param-name>  
	        <!--这里为CAS服务器的地址，必须使用所创建的域名，不然验证证书不通过 -->
	        <param-value>https://www.bhz.com:8443/cas/login</param-value>  
	    </init-param>  
	    <init-param>  
	        <param-name>serverName</param-name> 
	        <!--这里的server是服务端的IP-->   
	        <param-value>http://192.168.1.133:8080</param-value>  
	    </init-param>  
	</filter>  
	<filter-mapping>  
	    <filter-name>CASFilter</filter-name>  
	    <url-pattern>/*</url-pattern>  
	</filter-mapping>  
	<!-- 该过滤器负责对Ticket的校验工作，必须启用它 -->  
	<filter>  
	    <filter-name>CAS Validation Filter</filter-name>  
	    <filter-class>  
	        org.jasig.cas.client.validation.Cas20ProxyReceivingTicketValidationFilter</filter-class>  
	    <init-param>  
	        <param-name>casServerUrlPrefix</param-name>
	        <!--这里为CAS服务器的地址，必须使用所创建的域名，不然验证证书不通过 -->
	        <param-value>https://www.bhz.com:8443/cas</param-value>
	    </init-param>  
	    <init-param>  
	        <param-name>serverName</param-name>  
	        <param-value>http://192.168.1.133:8080</param-value>  
	    </init-param>  
	</filter>  
	<filter-mapping>  
	    <filter-name>CAS Validation Filter</filter-name>  
	    <url-pattern>/*</url-pattern>  
	</filter-mapping>  
	<!--  
	    该过滤器负责实现HttpServletRequest请求的包裹，  
	    比如允许开发者通过HttpServletRequest的getRemoteUser()方法获得SSO登录用户的登录名，可选配置。  
	-->  
	<filter>  
	    <filter-name>CAS HttpServletRequest Wrapper Filter</filter-name>  
	    <filter-class>  
	        org.jasig.cas.client.util.HttpServletRequestWrapperFilter</filter-class>  
	</filter>  
	<filter-mapping>  
	    <filter-name>CAS HttpServletRequest Wrapper Filter</filter-name>  
	    <url-pattern>/*</url-pattern>  
	</filter-mapping>  
	<!--  
	    该过滤器使得开发者可以通过org.jasig.cas.client.util.AssertionHolder来获取用户的登录名。  
	    比如AssertionHolder.getAssertion().getPrincipal().getName()。  
	-->  
	<filter>  
	    <filter-name>CAS Assertion Thread Local Filter</filter-name>  
	    <filter-class>org.jasig.cas.client.util.AssertionThreadLocalFilter</filter-class>  
	</filter>  
	<filter-mapping>  
	    <filter-name>CAS Assertion Thread Local Filter</filter-name>  
	    <url-pattern>/*</url-pattern>  
	</filter-mapping>   
	<!-- ======================== 单点登录结束 ======================== -->  
	<!-- 自动根据单点登录的结果设置本系统的用户信息 -->   

	<!-- 下面的过滤器是后加的暂时不用-->   
	 
    <filter>
        <filter-name>authValidateFilter</filter-name>
        <filter-class>bhz.com.util.AuthValidateFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>authValidateFilter</filter-name>
        <servlet-name>bhz-sys</servlet-name>
    </filter-mapping>	
  
```


客户端代码  

common下加两个util  
/bhz-com/src/main/java/bhz/com/util/RequestUtils.java  

```java
package bhz.com.util;
import java.io.File;
import java.io.IOException;
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletRequest;
import org.apache.commons.lang3.StringUtils;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.multipart.MultipartHttpServletRequest;
import bhz.com.constant.Const;

public final class RequestUtils {

    /**
     * <B>构造方法</B><BR>
     */
    private RequestUtils() {
    }
    
    /**
     * <B>方法名称：</B>获取当前系统标识<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 请求
     * @return String 当前用户标识
     */
    public static String getCurrentTag(HttpServletRequest request) {
        return (String) request.getAttribute(Const.REQ_CUR_TAG);
    }
    
    /**
     * <B>方法名称：</B>获取当前用户标识<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 请求
     * @return String 当前用户标识
     */
    public static String getCurrentUserId(HttpServletRequest request) {
        return (String) request.getAttribute(Const.REQ_CUR_USER_ID);
    }

    /**
     * <B>方法名称：</B>获取当前用户名称<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 请求
     * @return String 当前用户名称
     */
    public static String getCurrentUserName(HttpServletRequest request) {
        return (String) request.getAttribute(Const.REQ_CUR_USER_NAME);
    }

    /**
     * <B>方法名称：</B>获取当前机构标识<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 请求
     * @return String 当前机构标识
     */
    public static String getCurrentOrgId(HttpServletRequest request) {
        return (String) request.getAttribute(Const.REQ_CUR_ORG_ID);
    }

    /**
     * <B>方法名称：</B>获取当前角色代码<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 请求
     * @return String 当前角色代码
     */
    public static String getCurrentRoleCode(HttpServletRequest request) {
        return (String) request.getAttribute(Const.REQ_CUR_ROLE_CODE);
    }    
    
    /**
     * <B>方法名称：</B>获取客户端Cookie信息<BR>
     * <B>概要说明：</B>根据指定的名称，获取客户端Cookie信息。<BR>
     * 
     * @param request 请求
     * @param name Cookie名
     * @return Cookie Cookie信息
     */
    public static Cookie getCookieByName(HttpServletRequest request, String name) {
        if (StringUtils.isBlank(name)) {
            return null;
        }
        Cookie[] cookies = request.getCookies();
        if (cookies != null) {
            for (Cookie cookie : cookies) {
                if (cookie.getName().equals(name)) {
                    return cookie;
                }
            }
        }
        return null;
    }
    
    /**
     * <B>方法名称：</B>获取客户端Cookie值<BR>
     * <B>概要说明：</B>根据指定的名称，获取客户端Cookie值。<BR>
     * 
     * @param request 请求
     * @param name Cookie名
     * @return String Cookie值
     */
    public static String getCookieValue(HttpServletRequest request, String name) {
        Cookie cookie = getCookieByName(request, name);
        if (cookie == null) {
            return null;
        }
        return cookie.getValue();
    }
    
    /**
     * <B>方法名称：</B>获取Cookie验证值<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 请求
     * @return String Cookie验证值
     */
    public static String getValidateKey(HttpServletRequest request) {
        return getCookieValue(request, Const.COOKIE_VALIDATE_KEY);
    }
    
    /**
     * <B>方法名称：</B>获取会话属性值<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 请求
     * @param name 属性名
     * @return String 属性值
     */
    public static String getSessionValue(HttpServletRequest request, String name) {
        Object attr = request.getSession().getAttribute(name);
        if (attr == null) {
            return null;
        }
        return (String) attr;
    }

    /**
     * <B>方法名称：</B>获取客户端信息<BR>
     * <B>概要说明：</B>根据请求获取客户端信息。<BR>
     * 
     * @param request 请求
     * @return String 客户端信息
     */
    public static String getClientInfo(HttpServletRequest request) {
        String ip = request.getHeader("x-forwarded-for");
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("WL-Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getRemoteAddr();
        }
        return ip;
    }

    /**
     * <B>方法名称：</B>判断客户端是否为Firefox<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 请求
     * @return boolean 是否为Firefox
     */
    public static boolean isFirefox(HttpServletRequest request) {
        String agent = request.getHeader("USER-AGENT").toUpperCase();
        return (!StringUtils.isBlank(agent) && agent.indexOf("FIREFOX") > 0);
    }

    /**
     * <B>方法名称：</B>获取上传文件<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 请求
     * @return MultipartFile 上传文件
     */
    public static MultipartFile getUploadFile(HttpServletRequest request) {
        return getUploadFile(request, "file");
    }

    /**
     * <B>方法名称：</B>获取上传文件<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 请求
     * @param name 文件名
     * @return MultipartFile 上传文件
     */
    public static MultipartFile getUploadFile(HttpServletRequest request, String name) {
        MultipartHttpServletRequest multipartRequest = (MultipartHttpServletRequest) request;
        return multipartRequest.getFile(name);
    }

    /**
     * <B>方法名称：</B>保存上传文件<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 请求
     * @param path 保存路径
     * @throws IOException 预想外异常错误
     */
    public static void saveUploadFile(HttpServletRequest request, String path) throws IOException {
        MultipartFile file = getUploadFile(request);
        String sep = System.getProperty("file.separator");
        File filePath = new File(path);
        if (!filePath.exists()) {
            filePath.mkdirs();
        }
        String fullPath = path + sep + file.getOriginalFilename();
        File uploadedFile = new File(fullPath);
        file.transferTo(uploadedFile);
    }

}
```

/bhz-com/src/main/java/bhz/com/util/ResponseUtils.java  

```java
package bhz.com.util;
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import java.util.List;
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.apache.commons.lang3.StringUtils;
import bhz.com.constant.Const;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONException;
import com.alibaba.fastjson.JSONObject;

public final class ResponseUtils {

    /**
     * <B>构造方法</B><BR>
     */
    private ResponseUtils() {
    }

    /**
     * <B>方法名称：</B>设定Cookie值<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param response 响应
     * @param name Cookie名
     * @param value Cookie值
     */
    public static void setCookieValue(HttpServletResponse response, String name, String value) {
        if (StringUtils.isBlank(name) || StringUtils.isBlank(value)) {
            return;
        }
        Cookie cookie = new Cookie(name, value);
        cookie.setPath("/");
        response.addCookie(cookie);
    }

    /**
     * <B>方法名称：</B>设定验证键值<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param response 响应
     * @param validateKey 验证键值
     */
    public static void setValidateKey(HttpServletResponse response, String validateKey) {
        setCookieValue(response, Const.COOKIE_VALIDATE_KEY, validateKey);
    }

    /**
     * <B>方法名称：</B>设定HTML文本响应<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param response 响应
     * @param html HTML文本
     */
    public static void putHtmlResponse(HttpServletResponse response, String html) {
        response.setContentType("text/html;charset=" + Const.CHARSET_UTF8);
        response.setCharacterEncoding(Const.CHARSET_UTF8);
        try {
            response.getWriter().write(html);
        }
        catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * <B>方法名称：</B>设定文本响应<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param response 响应
     * @param text 文本信息
     */
    public static void putTextResponse(HttpServletResponse response, String text) {
        response.setContentType("text/plain;charset=" + Const.CHARSET_UTF8);
        response.setCharacterEncoding(Const.CHARSET_UTF8);
        try {
            response.getWriter().write(text);
        }
        catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * <B>方法名称：</B>设定JSON对象响应<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param response 响应
     * @param json JSON对象
     */
    public static void putJsonResponse(HttpServletResponse response, JSONObject json) {
        if (json == null) {
            putTextResponse(response, "{}");
        }
        else {
            putTextResponse(response, json.toString());
        }
    }

    /**
     * <B>方法名称：</B>设定JSON对象响应<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param response 响应
     * @param jsonList JSON列表
     */
    public static void putJsonResponse(HttpServletResponse response, List<JSONObject> jsonList) {
        if (jsonList == null || jsonList.size() < 1) {
            putTextResponse(response, "[]");
        }
        else {
            putJsonResponse(response, new JSONArray(FastJsonConvert.convertJSONToArray(jsonList.toString(), Object.class)));
        }
    }

    /**
     * <B>方法名称：</B>设定JSON对象响应<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param response 响应
     * @param jsonArray JSON数组
     */
    public static void putJsonResponse(HttpServletResponse response, JSONArray jsonArray) {
        if (jsonArray == null || jsonArray.size() < 1) {
            putTextResponse(response, "[]");
        }
        else {
            putTextResponse(response, jsonArray.toString());
        }
    }

    /**
     * <B>方法名称：</B>设定JSON对象响应<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param response 响应
     */
    public static void putJsonSuccessResponse(HttpServletResponse response) {
        putJsonSuccessResponse(response, null);
    }

    /**
     * <B>方法名称：</B>设定JSON对象响应<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param response 响应
     * @param json JSON
     */
    public static void putJsonSuccessResponse(HttpServletResponse response, JSONObject json) {
        JSONObject ret = new JSONObject();
        try {
            ret.put("success", true);
            ret.put("data", json);
        }
        catch (JSONException e) {
            throw new RuntimeException(e);
        }
        putTextResponse(response, ret.toString());
    }

    /**
     * <B>方法名称：</B>设定JSON对象错误响应<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param response 响应
     * @param message 错误消息
     */
    public static void putJsonFailureResponse(HttpServletResponse response, String message) {
        JSONObject ret = new JSONObject();
        try {
            ret.put("success", false);
            ret.put("message", message);
        }
        catch (JSONException e) {
            throw new RuntimeException(e);
        }
        putTextResponse(response, ret.toString());
    }

    /**
     * <B>方法名称：</B>设定文件响应<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 请求
     * @param response 响应
     * @param fileName 文件名称
     */
    public static void putFileResponse(HttpServletRequest request, HttpServletResponse response, String fileName) {
        putFileResponse(request, response, fileName, null, Const.CHARSET_UTF8);
    }

    /**
     * <B>方法名称：</B>设定文件响应<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 请求
     * @param response 响应
     * @param fileName 文件名称
     * @param inline 是否内嵌
     */
    public static void putFileResponse(HttpServletRequest request, HttpServletResponse response, String fileName,
            Boolean inline) {
        putFileResponse(request, response, fileName, inline, Const.CHARSET_UTF8);
    }

    /**
     * <B>方法名称：</B>设定文件响应<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 请求
     * @param response 响应
     * @param fileName 文件名称
     * @param inline 是否内嵌
     * @param charset 字符集
     */
    public static void putFileResponse(HttpServletRequest request, HttpServletResponse response, String fileName,
            Boolean inline, String charset) {

        int i = fileName.lastIndexOf(".") + 1;
        String ext = null;
        if (i > 0 && i < fileName.length()) {
            ext = fileName.substring(i).toLowerCase();
        }

        try {
            if (RequestUtils.isFirefox(request)) {
                fileName = new String(fileName.getBytes(Const.CHARSET_UTF8), Const.CHARSET_LATIN);
            }
            else {
                fileName = URLEncoder.encode(fileName, Const.CHARSET_UTF8);
            }
        }
        catch (UnsupportedEncodingException e) {
            throw new RuntimeException(e);
        }

        String type = (inline != null && inline) ? "inline" : "attachment";

        String mime = "application/octet-stream";
        if (StringUtils.isBlank(ext)) {
        }
        else if (ext.equals("txt")) {
            mime = "text/plain";
        }
        else if (ext.equals("htm")) {
            mime = "text/htm";
        }
        else if (ext.equals("html")) {
            mime = "text/html";
        }
        else if (ext.equals("jpg")) {
            mime = "image/jpeg";
        }
        else if (ext.equals("jpe")) {
            mime = "image/jpeg";
        }
        else if (ext.equals("jpeg")) {
            mime = "image/jpeg";
        }
        else if (ext.equals("png")) {
            mime = "image/png";
        }
        else if (ext.equals("gif")) {
            mime = "image/gif";
        }
        else if (ext.equals("bmp")) {
            mime = "image/bmp";
        }
        else if (ext.equals("tif")) {
            mime = "image/tiff";
        }
        else if (ext.equals("tiff")) {
            mime = "image/tiff";
        }
        else if (ext.equals("svg")) {
            mime = "image/svg+xml";
        }
        else if (ext.equals("pdf")) {
            mime = "application/pdf";
        }
        else if (ext.equals("doc")) {
            mime = "application/msword";
        }
        else if (ext.equals("docx")) {
            mime = "application/msword";
        }
        else if (ext.equals("xls")) {
            mime = "application/vnd.ms-excel";
        }
        else if (ext.equals("xlsx")) {
            mime = "application/vnd.ms-excel";
        }
        else if (ext.equals("ppt")) {
            mime = "application/vnd.ms-powerpoint";
        }
        else if (ext.equals("pptx")) {
            mime = "application/vnd.ms-powerpoint";
        }
        else if (ext.equals("mpp")) {
            mime = "application/vnd.ms-project";
        }
        else if (ext.equals("mppx")) {
            mime = "application/vnd.ms-project";
        }
        else if (ext.equals("mp4")) {
            mime = "video/mp4";
        }

        response.setContentType(mime + ";charset=" + charset);
        response.setCharacterEncoding(charset);
        response.setHeader("Cache-Control", "no-store");
        response.addDateHeader("Last-Modified", System.currentTimeMillis());
        response.addHeader("Content-Disposition", type + ";filename=" + fileName);
    }
  
}
```

common里加一个filter  
/bhz-com/src/main/java/bhz/com/util/AuthValidateFilter.java  

```java
package bhz.com.util;
import java.io.IOException;
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.apache.commons.lang3.StringUtils;
import org.springframework.stereotype.Component;
import bhz.com.constant.Const;

@Component
public class AuthValidateFilter implements Filter {


	private String basePath = null;
	
	private String indexPath = "/index.html";
	
	/**
     * <B>方法名称：</B>初始化<BR>
     * <B>概要说明：</B>初始化过滤器。<BR>
     * 
     * @param fConfig 过滤器配置
     * @throws ServletException Servlet异常
     * @see javax.servlet.Filter#init(javax.servlet.FilterConfig)
     */
    public void init(FilterConfig fConfig) throws ServletException {
        this.basePath = "/" + fConfig.getServletContext().getServletContextName();
    }

    /**
     * <B>方法名称：</B>过滤处理<BR>
     * <B>概要说明：</B>验证访问的合法有效性。<BR>
     * 
     * @param request 请求
     * @param response 响应
     * @param chain 过滤器链
     * @throws IOException IO异常
     * @throws ServletException Servlet异常
     * @see javax.servlet.Filter#doFilter(javax.servlet.ServletRequest,
     *      javax.servlet.ServletResponse, javax.servlet.FilterChain)
     */
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException,
            ServletException {

        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse res = (HttpServletResponse) response;
        
        System.out.println("--------basepath----------- : " + this.basePath);
        String uri = req.getRequestURI();
        System.out.println("--------uri----------- : " + uri);
        String remoteUser = req.getRemoteUser();
        System.out.println("--------remoteUser----------- : " + req.getRemoteUser());
        String validateKey = RequestUtils.getValidateKey(req);
        System.out.println("--------validateKey----------- :" + validateKey);
        
        //如果地址为初始登陆地址，并且当前cookie里没有验证信息,则进入index.html进行设值
        if(StringUtils.isBlank(validateKey)){
        	//url == "bjsxt-sys/index.html"
        	if(uri.equals(this.basePath + this.indexPath)){
            	//设置是否存在cookie验证信息和basePath
            	req.setAttribute("existsCookie", Const.FALSE);
            	req.setAttribute("basePath", this.basePath);
            	chain.doFilter(request, response);
            	return;        		
        	} else {
        		res.sendRedirect(this.basePath + this.indexPath);
        		return;
        	}
        } 
        
        //如果验证信息不为空,则从cookie中获取信息，进行设置
        if(!StringUtils.isBlank(validateKey)){
        	//设置是否存在cookie验证信息和basePath
        	req.setAttribute("existsCookie", Const.TRUE);
        	req.setAttribute("basePath", this.basePath);
        	String values[] = validateKey.split("\\" + Const.COOKIE_VALIDATE_KEY_SPLIT);
        	req.setAttribute(Const.REQ_CUR_USER_ID, values[0]);
        	req.setAttribute(Const.REQ_CUR_USER_NAME, values[1]);
        	if(values.length > 2){
        		req.setAttribute(Const.REQ_CUR_ROLE_CODE, values[2]);
        		if(values.length > 3){
        			req.setAttribute(Const.REQ_CUR_ORG_ID, values[3]);
        		}
        	}
        }
        chain.doFilter(request, response);
    }

    /**
     * <B>方法名称：</B>释放资源<BR>
     * <B>概要说明：</B>释放过滤器资源。<BR>
     * 
     * @see javax.servlet.Filter#destroy()
     */
    public void destroy() {
    }
}
```

web.xml  
也要添加这个filter  

```xml
    <filter>
        <filter-name>authValidateFilter</filter-name>
        <filter-class>bhz.com.util.AuthValidateFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>authValidateFilter</filter-name>
        <servlet-name>bhz-sys</servlet-name>
    </filter-mapping>	
```

common加一个controller  
/bhz-com/src/main/java/bhz/com/web/IndexController.java  

```java
package bhz.com.web;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;
import bhz.com.constant.Const;
import bhz.com.dao.UserComDao;
import bhz.com.util.ResponseUtils;
import com.alibaba.fastjson.JSONObject;

@Controller
public class IndexController {

	private UserComDao userComDao;
	
	public UserComDao getUserComDao() {
		return userComDao;
	}
	@Autowired
	public void setUserComDao(UserComDao userComDao) {
		this.userComDao = userComDao;
	}

	/**
     * <B>方法名称：</B>系统首页<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 页面请求
     * @param response 页面响应
     * @param dataYear 年份
     * @return ModelAndView 模型视图
     */
    @RequestMapping("/index.html")
    public ModelAndView index(HttpServletRequest request, HttpServletResponse response) {
        ModelAndView ret = new ModelAndView();
        String existsCookie = (String) request.getAttribute("existsCookie");
        String basePath = (String) request.getAttribute("basePath");
        String myTag = basePath.substring(basePath.indexOf("-")+1, basePath.length());
        if(existsCookie.equals(Const.FALSE)){
        	//设置cookie
        	String remoteUser = request.getRemoteUser();
        	JSONObject userInfo = this.userComDao.getUserInfo(remoteUser);
        	String userId = userInfo.getString("USER_ID");
        	String userName = userInfo.getString("USER_NAME");
        	String roleCode = userInfo.getString("ROLE_CODE");
        	String orgId = userInfo.getString("ORG_ID");
        	
        	StringBuffer validateKey = new StringBuffer();
        	validateKey.append(userId);
        	validateKey.append(Const.COOKIE_VALIDATE_KEY_SPLIT);
        	validateKey.append(userName);
        	if(!StringUtils.isBlank(roleCode)){
            	validateKey.append(Const.COOKIE_VALIDATE_KEY_SPLIT);
            	validateKey.append(roleCode);        		
        	}
        	if(!StringUtils.isBlank(orgId)){
            	validateKey.append(Const.COOKIE_VALIDATE_KEY_SPLIT);
            	validateKey.append(orgId);        		
        	}
        	ResponseUtils.setValidateKey(response, validateKey.toString());
        } 
        for(String tag : Const.TAGS){
        	if(myTag.equals(tag)){
        		String redirect = "/" + tag + "index.html";
        		System.out.println(redirect);
        		ret.setViewName("redirect:" + redirect);
        	}
        }
        
        return ret;
    }
}
```

common加一个dao  
通过id查询用户信息  
/bhz-com/src/main/java/bhz/com/dao/UserComDao.java  

```java
package bhz.com.dao;
import org.springframework.stereotype.Repository;
import com.alibaba.fastjson.JSONObject;

@Repository
public class UserComDao extends BaseJdbcDao{
	private static final String SQL_SELECT_USER_INFO = "SELECT USER_ID, USER_NAME, ROLE_CODE, ORG_ID, DISABLE_FLAG FROM SYS_USER WHERE USER_ID = ?";
	
	public JSONObject getUserInfo(String userId){
		return this.queryForJsonObject(SQL_SELECT_USER_INFO, userId);
	}
}
```

common再加一个dao  
查询角色和权限  
/bhz-com/src/main/java/bhz/com/dao/RoleFuncComDao.java  

```java
package bhz.com.dao;
import java.util.Collections;
import java.util.List;
import org.apache.commons.lang3.StringUtils;
import org.springframework.stereotype.Repository;
import com.alibaba.fastjson.JSONObject;

@Repository
public class RoleFuncComDao extends BaseJdbcDao{

	public List<JSONObject> getFuncList(String roleCode, String funcCode){
		
		if(StringUtils.isBlank(roleCode) || StringUtils.isBlank(funcCode)){
			return Collections.emptyList();
		}
		StringBuffer sql = new StringBuffer();
		sql.append(" SELECT F.* FROM SYS_ROLE R JOIN SYS_ROLE_FUNC RF   ");
		sql.append("            ON(R.ROLE_CODE = RF.ROLE_CODE)          ");
		sql.append("            JOIN SYS_FUNC F                         ");
		sql.append("            ON(RF.FUNC_CODE = F.FUNC_CODE)          ");
		sql.append("            WHERE R.ROLE_CODE = ?         			");
		sql.append("            AND F.FUNC_CODE LIKE ?             		");
		sql.append("            AND R.DISABLE_FLAG = '0'                ");
		sql.append("            AND F.DISABLE_FLAH = '0'                ");	

		funcCode = funcCode + "__";
		
		return this.queryForJsonList(sql.toString(), roleCode, funcCode);
	}
}
```

sys加入一个Controller  
/bhz-sys/src/main/java/bhz/sys/web/SysIndexController.java  

```java
package bhz.sys.web;

import java.util.ArrayList;
import java.util.List;
import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;
import bhz.com.dao.RoleFuncComDao;
import bhz.com.util.RequestUtils;
import bhz.com.util.ResponseUtils;
import bhz.sys.facade.SysUserFacade;
import com.alibaba.fastjson.JSONObject;

/**
 * <B>系统名称：</B><BR>
 * <B>模块名称：</B><BR>
 * <B>中文类名：</B><BR>
 * <B>概要说明：</B><BR>
 * @author bhz（Alienware）
 * @since 2016年2月29日
 */
@Controller
public class SysIndexController {
	
	@Resource
	private SysUserFacade sysUserFacade;

	private RoleFuncComDao roleFuncComDao;
	
	public RoleFuncComDao getRoleFuncComDao() {
		return roleFuncComDao;
	}
	@Autowired
	public void setRoleFuncComDao(RoleFuncComDao roleFuncComDao) {
		this.roleFuncComDao = roleFuncComDao;
	}
	
	/**
     * <B>方法名称：</B>系统首页<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 页面请求
     * @param response 页面响应
     * @param dataYear 年份
     * @return ModelAndView 模型视图
     */
    @RequestMapping("/sysindex.html")
    public ModelAndView index(HttpServletRequest request, HttpServletResponse response) throws Exception {
        ModelAndView ret = new ModelAndView();
        String userId = RequestUtils.getCurrentUserId(request);
        String userName = RequestUtils.getCurrentUserName(request);
        String roleCode = RequestUtils.getCurrentRoleCode(request);
        String orgId = RequestUtils.getCurrentOrgId(request);
        List<JSONObject> funcList = this.roleFuncComDao.getFuncList(roleCode, "01");
        ret.addObject("funcList", funcList.toString());     
        return ret;
    }
    
    @RequestMapping("/sysFuncList.json")
    public void sysFuncList(HttpServletRequest request, HttpServletResponse response, String id){
    	String roleCode = RequestUtils.getCurrentRoleCode(request);
    	List<JSONObject> records = new ArrayList<JSONObject>();
    	List<JSONObject> funcList = this.roleFuncComDao.getFuncList(roleCode, id);
    	for (JSONObject jsonObject : funcList) {
			JSONObject record = new JSONObject();
			//'id' ,'text' , 'type' , 'leaf', 'url'
			record.put("id", jsonObject.getString("FUNC_CODE"));
			record.put("text", jsonObject.getString("FUNC_NAME"));
			record.put("type", jsonObject.getString("FUNC_TYPE"));
			record.put("leaf", true);
			record.put("url", jsonObject.getString("FUNC_PATH"));
			records.add(record);
		}
    	ResponseUtils.putJsonResponse(response, records);
    }
}
```

修改sys中的jsp  
/bhz-sys/src/main/webapp/WEB-INF/pages/sysindex.jsp  	

```html
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@ taglib prefix="s" uri="http://www.springframework.org/tags"%>
<!DOCTYPE HTML>
<html>
<head>
<base href='<%=request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort() + request.getContextPath() + "/"%>' />
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>

<link rel="stylesheet" type="text/css" href="/bhz-sys/css/common-neptune.css" />
<link rel="stylesheet" type="text/css" href="/bhz-sys/extjs/resources/ext-theme-neptune/ext-theme-neptune-all.css" />
<!-- 
<link rel="stylesheet" type="text/css" href="/bhz-sys/css/common.css" />
<link rel="stylesheet" type="text/css" href="/bhz-sys/extjs/resources/ext-theme-classic/ext-theme-classic-all.css" />
-->
<script type="text/javascript" charset="utf-8" src="/bhz-sys/extjs/ext-all.gzjs"></script>
<script type="text/javascript" charset="utf-8" src="/bhz-sys/extjs/ext-lang-zh_CN.js"></script>
<script type="text/javascript" charset="utf-8">

Ext.onReady(function(){

	Ext.onReady(function(){
		Ext.QuickTips.init();
		Ext.Loader.setConfig({
			enabled:true
		});
		/**
		 * Menu数据模型
		 */
	  	Ext.define('Menu',{
	  		extend:'Ext.data.Model', 
	  		fields:['id' ,'text' , 'type' , 'leaf', 'url']
	  	});
		/**
		 * Menu数据集合
		 */
	  	Ext.define('MenuStore',{
	  		extend:'Ext.data.TreeStore' , 
			model:'Menu' , 
			//nodeParam:'id' , 		//指定节点参数名称
			proxy:{
				type:'ajax' , 
				url:'sysFuncList.json' , 
				reader:'json' 
			},
			autoLoad:false
	  	});
		/**
		 * treepanel组件
		 */
		Ext.define('MenuTree',{
			extend:'Ext.tree.Panel' ,
			title:'功能管理',
			border:false ,
			height:400,
			rootVisible:false ,
			listeners:{		
				'itemclick':function(view, record , item , index , e ){
					var panel = Ext.getCmp('main');			//取得主页面的main tabpanel面板
					if(panel.down('#tab_'+record.get('id'))){			//如果存在tab组件,设置为激活的状态
						panel.setActiveTab(panel.down('#tab_'+record.get('id')));
					} else {								//如果不存在tab组件 新增一个
						var tab = panel.add({
							id:'tab_'+record.get('id') ,
							title:record.get('text') ,
							html:'<iframe src= '+record.get('url') +' width=100% height=100%  marginwidth=0 marginheight=0 hspace=0 frameborder =0 allowtransparency=yes></iframe>' ,
							closable:true 
						});			
						panel.setActiveTab(tab);			//把tab设置为当前激活的状态
					}
				}
			},
			tools : [{
						type:'refresh',
						qtip: '刷新',
						handler: function(event, toolEl, header){
							this.up('treepanel').store.load();
						}
			}],				
			initComponent:function(arguments){
				var self = this; 
				self.callParent(arguments);
			}
		});	
		
		
		var funcList = '${funcList}';
		var accordionList = Ext.JSON.decode(funcList);
		var menuTreePanelArr = [];
		Ext.Array.each(accordionList,function(item){
			var menuStore = Ext.create('MenuStore');
			menuStore.proxy.extraParams = {id : item.FUNC_CODE} ;
			menuStore.load();
			var menuTreePanel = Ext.create('MenuTree',{
				title: item.FUNC_NAME,
				iconCls:'func',
				store: menuStore
			});
			menuTreePanelArr.push(menuTreePanel);
		});
		
		Ext.create('Ext.container.Viewport',{
			layout:'border' , 				//设置Border布局
			title:'数据中心' , 
			defaults:{
				collapsible: true , 		//展开形式
				split:true ,				//是否能被拖拽
				bodyStyle:'padding:1px' 	//边距
			},
			items:[{
				title:'数据交换中心平台' ,  
				region:'north' ,
				height:100 ,
				html:'<br><center><font size=6>数据交换中心平台</font></center>'
			},{
				title:'菜单' ,
				xtype:'panel',
				layout:'accordion',
				items: menuTreePanelArr ,
				//bodyStyle:'padding:0px' , 
				region: 'west' ,
				width:250
			},{
				title:'主界面' ,
				layout:'fit' ,				//fit布局填充主页面
				region:'center' ,
				id:'main',
				xtype:'tabpanel'
			}]
		});

		
	});
	
});
</script>
</head>
<body>
</body>
</html>
```


linux下配置cas  

1.把之前的keystore和tomcat.cer放到/usr/local/keys/  
2.把jdk下的cer删掉，替换成tomcat.cer  
3.配置好的cas项目放到tomcat  

## 5.0版本控制  

- 基础信息27  

只做了定时清理文件这一步，剩下的见实现步骤  

需要一个文件实体，不用和数据库实体名称对应，用jdbctemplate比较灵活  
/bhz-com/src/main/java/bhz/com/entity/SysFile.java  

```java

package bhz.com.entity;
import java.util.Date;
import org.apache.commons.lang3.StringUtils;
import org.apache.commons.lang3.time.DateFormatUtils;
import bhz.com.constant.Const;
import com.alibaba.fastjson.JSONException;
import com.alibaba.fastjson.JSONObject;
/**
 * <B>系统名称：</B>治超信息系统<BR>
 * <B>模块名称：</B>数据实体<BR>
 * <B>中文类名：</B>系统文件<BR>
 * <B>概要说明：</B><BR>
 */
public class SysFile implements java.io.Serializable {

    /** 默认序列版本标识 */
    private static final long serialVersionUID = 1L;

    /** 文件键值 */
    private String key;

    /** 文件分类 */
    private String type;

    /** 文件名称 */
    private String name;

    /** 文件扩展名 */
    private String ext;

    /** 文件大小 */
    private long bytes;

    /** 文件数据路径 */
    private String dataPath;
    
    /** 文件数据组 */
    private String dataGroup;
    
    /** 过期时间 */
    private Date expired;
    
    /** 备注 */
    private String descInfo;

    /** 更新人 */
    private String updateBy;

    /** 更新时间 */
    private Date updateTime;
    
    /**
     * <B>取得：</B>文件键值<BR>
     * 
     * @return String 文件键值
     */
    public String getKey() {
        return key;
    }

    /**
     * <B>设定：</B>文件键值<BR>
     * 
     * @param key 文件键值
     */
    public void setKey(String key) {
        this.key = key;
    }

    /**
     * <B>取得：</B>文件分类【GROUP】<BR>
     * 
     * @return String 文件分类【GROUP】
     */
    public String getType() {
        return type;
    }

    /**
     * <B>设定：</B>文件分类<BR>
     * 
     * @param type 文件分类
     */
    public void setType(String type) {
        this.type = type;
    }

    /**
     * <B>取得：</B>文件名称<BR>
     * 
     * @return String 文件名称
     */
    public String getName() {
        return name;
    }

    /**
     * <B>设定：</B>文件名称<BR>
     * 
     * @param name 文件名称
     */
    public void setName(String name) {
        this.name = name;
    }

    /**
     * <B>取得：</B>文件扩展名<BR>
     * 
     * @return String 文件扩展名
     */
    public String getExt() {
        return ext;
    }

    /**
     * <B>设定：</B>文件扩展名<BR>
     * 
     * @param ext 文件扩展名
     */
    public void setExt(String ext) {
        this.ext = ext;
    }


	/**
     * <B>取得：</B>文件大小<BR>
     * 
     * @return long 文件大小
     */
    public long getBytes() {
        return bytes;
    }

    /**
     * <B>设定：</B>文件大小<BR>
     * 
     * @param bytes 文件大小
     */
    public void setBytes(long bytes) {
        this.bytes = bytes;
    }

	/**
     * <B>取得：</B>文件路径<BR>
     * 
     * @return dataPath 文件路径
     */
    public String getDataPath() {
		return dataPath;
	}

    /**
     * <B>设定：</B>文件路径<BR>
     * 
     * @param dataPath 文件路径
     */
	public void setDataPath(String dataPath) {
		this.dataPath = dataPath;
	}
	
	/**
     * <B>取得：</B>文件组名<BR>
     * 
     * @return dataGroup 文件组名
     */
	public String getDataGroup() {
		return dataGroup;
	}
	
    /**
     * <B>设定：</B>文件组名<BR>
     * 
     * @param dataGroup 文件组名
     */
	public void setDataGroup(String dataGroup) {
		this.dataGroup = dataGroup;
	}

	/**
     * <B>取得：</B>过期时间<BR>
     * 
     * @return Date 过期时间
     */
    public Date getExpired() {
        return expired;
    }

    /**
     * <B>设定：</B>过期时间<BR>
     * 
     * @param expired 过期时间
     */
    public void setExpired(Date expired) {
        this.expired = expired;
    }

    /**
     * <B>取得：</B>更新人<BR>
     * 
     * @return String 更新人
     */
    public String getUpdateBy() {
        return updateBy;
    }

    /**
     * <B>设定：</B>更新人<BR>
     * 
     * @param updateBy 更新人
     */
    public void setUpdateBy(String updateBy) {
        this.updateBy = updateBy;
    }

    /**
     * <B>取得：</B>更新时间<BR>
     * 
     * @return Date 更新时间
     */
    public Date getUpdateTime() {
        return updateTime;
    }

    /**
     * <B>设定：</B>更新时间<BR>
     * 
     * @param updateTime 更新时间
     */
    public void setUpdateTime(Date updateTime) {
        this.updateTime = updateTime;
    }
    
    /**
     * <B>取得：</B>descInfo<BR>
     * @return String
     */
    public String getDescInfo() {
        return descInfo;
    }

    /**
     * <B>设定：</B>descInfo<BR>
     * @param descInfo 备注
     */
    public void setDescInfo(String descInfo) {
        this.descInfo = descInfo;
    }
}
```

/bhz-com/src/main/java/bhz/com/dao/SysFileComDao.java  
对文件进行添加删除、设置或清除文件过期时间等操作  
clearFileKeys方法使用JdbcTemplate调用oracle的存储过程PKG_SYS.sql，清除数据库中过期文件记录  

```java
package bhz.com.dao;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Date;
import java.util.List;
import java.util.Map;
import oracle.jdbc.OracleTypes;
import org.apache.commons.lang3.StringUtils;
import org.springframework.jdbc.core.SqlOutParameter;
import org.springframework.jdbc.core.simple.SimpleJdbcCall;
import org.springframework.stereotype.Repository;
import bhz.com.entity.SysFile;
import bhz.com.util.FastJsonConvert;
import com.alibaba.fastjson.JSONObject;

@Repository
public class SysFileComDao extends BaseJdbcDao {

    /** 系统文件行映射器 */
    private static final SysFileRowMapper SYS_FILE_ROW_MAPPER = new SysFileRowMapper();
    
    /**
     * <B>方法名称：</B>插入系统文件<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param data 系统文件
     * @return int 插入数量
     */
    public int insert(SysFile data) {
        if (data == null) {
            return 0;
        }
        String sql = "INSERT INTO SYS_FILE(KEY, TYPE, NAME, EXT, BYTES, DATA_PATH, DATA_GROUP, EXPIRED, DESC_INFO, UPDATE_BY, UPDATE_TIME) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, SYSTIMESTAMP)";
        Object[] args = new Object[10];
        args[0] = data.getKey();
        args[1] = data.getType();
        args[2] = data.getName();
        args[3] = data.getExt();
        args[4] = data.getBytes();
        args[5] = data.getDataPath();
        args[6] = data.getDataGroup();
        args[7] = data.getExpired();
        args[8] = data.getDescInfo();
        args[9] = data.getUpdateBy();
        return super.getJdbcTemplate().update(sql.toString(), args);
    }

    /**
     * <B>方法名称：</B>清除系统文件过期限制<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param key 文件键值
     * @return int 更新数量
     */
    public int clearExpired(String key) {
        if (StringUtils.isBlank(key)) {
            return 0;
        }
        return super.getJdbcTemplate().update("UPDATE SYS_FILE SET EXPIRED = NULL WHERE KEY = ?", key);
    }
    
    /**
     * <B>方法名称：</B>更新系统文件过期时间<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param key 文件键值
     * @param expired 过期时间（null为当前时间）
     * @return int 更新数量
     */
    public int expire(String key, Date expired) {
        if (StringUtils.isBlank(key)) {
            return 0;
        }
        if (expired == null) {
            return super.getJdbcTemplate().update("UPDATE SYS_FILE SET EXPIRED = SYSTIMESTAMP WHERE KEY = ?", key);
        }
        return super.getJdbcTemplate().update("UPDATE SYS_FILE SET EXPIRED = ? WHERE KEY = ?", expired, key);
    }   
   
    /**
     * <B>方法名称：</B>删除系统文件<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param key 文件键值
     * @return int 删除数量
     */
    public int delete(String key) {
        if (StringUtils.isBlank(key)) {
            return 0;
        }
        return super.getJdbcTemplate().update("DELETE FROM SYS_FILE WHERE KEY = ?", key);
    }
    
    /**
     * <B>方法名称：</B>获取系统文件<BR>
     * <B>概要说明：</B><BR>
     * @param key 文件键值
     * @return SysFile 系统文件
     */
    public SysFile get(String key) {
        if (StringUtils.isBlank(key)) {
            return null;
        }
        List<SysFile> dataList = getList(key.split(","));
        if (dataList == null || dataList.size() < 1) {
            return null;
        }
        return dataList.get(0);
    }

    /**
     * <B>方法名称：</B>获取系统文件列表<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param keys 文件键值集合
     * @return List<SysFile> 系统文件列表
     */
    public List<SysFile> getList(String[] keys) {
        List<Object> args = new ArrayList<Object>();
        StringBuffer sql = new StringBuffer();
        sql.append("SELECT KEY, TYPE, NAME, EXT, BYTES, DATA_PATH, DATA_GROUP, EXPIRED, DESC_INFO, UPDATE_BY, UPDATE_TIME ");
        sql.append(" FROM SYS_FILE WHERE KEY IN ");
        super.appendSqlIn(sql, args, keys);
        sql.append(" ORDER BY TYPE, NAME");
        return super.getJdbcTemplate().query(sql.toString(), SYS_FILE_ROW_MAPPER, args.toArray());
    }
    
    
    /**
     * <B>方法名称：</B>清空文件缓存方法<BR>
     * <B>概要说明：</B>清空文件缓存方法<BR>
     * @return List<String> 主键KEY集合
     */
    public List<String> clearFileKeys() {
        SimpleJdbcCall call = new SimpleJdbcCall(this.getJdbcTemplate()).
        		withCatalogName("PKG_SYS").withProcedureName("CLEAR_FILES").
        		declareParameters(new SqlOutParameter("P_RET_REFCUR", OracleTypes.CURSOR));
      	Map<String, Object> m = call.execute();
      	if(m.values().size() > 0){
          	String temp = m.values().toArray()[0].toString().replace("[", "").replace("]", "");
          	String[] keys = temp.split(",");
          	List<String> ret = new ArrayList<String>();
          	for(String key : keys){
          		String temp1 = key.replace("{", "").replace("}", "");
          		if(!StringUtils.isBlank(temp1)){
          			ret.add(temp1.split("\\=")[1]);
          		}
          	}  
          	return ret;
      	}
      	return Collections.emptyList();
    }
}
```

PKG_SYS.sql存储过程  
使用cursor遍历所有的需要删除的记录并删除  

```sql
CREATE OR REPLACE PACKAGE PKG_SYS AUTHID DEFINER IS

    /**************************************************************************
     -- Copyright 2013 TPRI. All Rights Reserved.
     -- Summary : 系统处理包
     -- Author  : 交通运输部规划研究院（白贺卓）
     -- Since   : 2013-08-09
    **************************************************************************/

    /**************************************************************************
     -- Summary : 参照游标
     -- return  : RET_REFCUR （返回参照游标数据）
    **************************************************************************/
    TYPE RET_REFCUR IS REF CURSOR;

    /**************************************************************************
     -- Summary : 生成实时数据（参考临时数据）
     -- Param   : P_TASK 任务名称
     -- Param   : P_TEXT 日志文本
     -- Param   : P_REF01 参考字段01（默认NULL）
     -- Param   : P_REF02 参考字段02（默认NULL）
     -- Param   : P_REF03 参考字段03（默认NULL）
     -- Param   : P_REF04 参考字段04（默认NULL）
     -- Param   : P_REF05 参考字段05（默认NULL）
     -- Param   : P_REF06 参考字段06（默认NULL）
     -- Param   : P_REF07 参考字段07（默认NULL）
     -- Param   : P_REF08 参考字段08（默认NULL）
     -- Param   : P_REF09 参考字段09（默认NULL）
     -- Param   : P_REF10 参考字段10（默认NULL）
    **************************************************************************/
    PROCEDURE PUT_LOG
    (
        P_TASK IN VARCHAR2,
        P_TEXT IN VARCHAR2,
        P_REF01 IN VARCHAR2 DEFAULT NULL,
        P_REF02 IN VARCHAR2 DEFAULT NULL,
        P_REF03 IN VARCHAR2 DEFAULT NULL,
        P_REF04 IN VARCHAR2 DEFAULT NULL,
        P_REF05 IN VARCHAR2 DEFAULT NULL,
        P_REF06 IN VARCHAR2 DEFAULT NULL,
        P_REF07 IN VARCHAR2 DEFAULT NULL,
        P_REF08 IN VARCHAR2 DEFAULT NULL,
        P_REF09 IN VARCHAR2 DEFAULT NULL,
        P_REF10 IN VARCHAR2 DEFAULT NULL
    );
    
    /**************************************************************************
     -- Summary : 清理文件数据
     -- Param   : P_DATE 参考时间（默认为当前时间）
     -- Param   : P_RET_REFCUR 参照游标输出参数
    **************************************************************************/
    PROCEDURE CLEAR_FILES(P_RET_REFCUR OUT RET_REFCUR);

END PKG_SYS;


CREATE OR REPLACE PACKAGE BODY PKG_SYS IS

    /**************************************************************************
     -- Summary : 生成实时数据（参考临时数据）
     -- Param   : P_TASK 任务名称
     -- Param   : P_TEXT 日志文本
     -- Param   : P_REF01 参考字段01（默认NULL）
     -- Param   : P_REF02 参考字段02（默认NULL）
     -- Param   : P_REF03 参考字段03（默认NULL）
     -- Param   : P_REF04 参考字段04（默认NULL）
     -- Param   : P_REF05 参考字段05（默认NULL）
     -- Param   : P_REF06 参考字段06（默认NULL）
     -- Param   : P_REF07 参考字段07（默认NULL）
     -- Param   : P_REF08 参考字段08（默认NULL）
     -- Param   : P_REF09 参考字段09（默认NULL）
     -- Param   : P_REF10 参考字段10（默认NULL）
    **************************************************************************/
    PROCEDURE PUT_LOG
    (
        P_TASK IN VARCHAR2,
        P_TEXT IN VARCHAR2,
        P_REF01 IN VARCHAR2 DEFAULT NULL,
        P_REF02 IN VARCHAR2 DEFAULT NULL,
        P_REF03 IN VARCHAR2 DEFAULT NULL,
        P_REF04 IN VARCHAR2 DEFAULT NULL,
        P_REF05 IN VARCHAR2 DEFAULT NULL,
        P_REF06 IN VARCHAR2 DEFAULT NULL,
        P_REF07 IN VARCHAR2 DEFAULT NULL,
        P_REF08 IN VARCHAR2 DEFAULT NULL,
        P_REF09 IN VARCHAR2 DEFAULT NULL,
        P_REF10 IN VARCHAR2 DEFAULT NULL
    ) IS
    BEGIN
        INSERT INTO SYS_LOG
            (LOG_TASK,
             LOG_TIME,
             LOG_TEXT,
             REF01,
             REF02,
             REF03,
             REF04,
             REF05,
             REF06,
             REF07,
             REF08,
             REF09,
             REF10)
        VALUES
            (SUBSTR(P_TASK, 1, 100),
             SYSTIMESTAMP,
             SUBSTR(P_TEXT, 1, 500),
             SUBSTR(P_REF01, 1, 100),
             SUBSTR(P_REF02, 1, 100),
             SUBSTR(P_REF03, 1, 100),
             SUBSTR(P_REF04, 1, 100),
             SUBSTR(P_REF05, 1, 100),
             SUBSTR(P_REF06, 1, 100),
             SUBSTR(P_REF07, 1, 100),
             SUBSTR(P_REF08, 1, 100),
             SUBSTR(P_REF09, 1, 100),
             SUBSTR(P_REF10, 1, 100));
    END PUT_LOG;

    /**************************************************************************
     -- Summary : 清理文件数据
     -- Param   : P_DATE 参考时间（默认为当前时间）
    **************************************************************************/
    PROCEDURE CLEAR_FILES(P_RET_REFCUR OUT RET_REFCUR) IS
        V_DATE TIMESTAMP := SYSTIMESTAMP;
        CURSOR CUR_EXPIRED_FILE IS
            SELECT KEY FROM SYS_FILE WHERE EXPIRED < V_DATE;
        V_EXPIRED_FILE CUR_EXPIRED_FILE%ROWTYPE;
    BEGIN
        OPEN P_RET_REFCUR FOR SELECT DATA_PATH FROM SYS_FILE WHERE EXPIRED < V_DATE;
        FOR V_EXPIRED_FILE IN CUR_EXPIRED_FILE LOOP
            DELETE FROM SYS_FILE WHERE KEY = V_EXPIRED_FILE.KEY;
            COMMIT;
        END LOOP;
    EXCEPTION
        WHEN OTHERS THEN
            ROLLBACK;
            PUT_LOG(P_TASK  => 'PKG_SYS.CLEAR_FILES',
                    P_TEXT  => 'EXCEPTION : [' || SQLCODE || ']' || SQLERRM,
                    P_REF01 => TO_CHAR(V_DATE, 'YYYY-MM-DD HH24:MI:SS.FF'));
            COMMIT;
    END CLEAR_FILES;

END PKG_SYS;

```

使用到了实体的映射实现RowMapper接口  
/bhz-com/src/main/java/bhz/com/dao/SysFileRowMapper.java  

```java

package bhz.com.dao;
import java.sql.ResultSet;
import java.sql.SQLException;
import org.springframework.jdbc.core.RowMapper;
import bhz.com.entity.SysFile;

/**
 * <B>系统名称：</B>通用系统功能<BR>
 * <B>模块名称：</B>数据访问通用功能<BR>
 * <B>中文类名：</B>系统文件行映射器<BR>
 * <B>概要说明：</B><BR>
 * @author bhz（Alienware）
 * @since 2016年5月23日
 */
public class SysFileRowMapper implements RowMapper<SysFile> {

    /**
     * <B>方法名称：</B>映射行数据<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param rs 结果集
     * @param row 行号
     * @return JSONObject 数据
     * @throws SQLException SQL异常错误
     * @see org.springframework.jdbc.core.RowMapper#mapRow(java.sql.ResultSet,
     *      int)
     */
    public SysFile mapRow(ResultSet rs, int row) throws SQLException {
        SysFile ret = new SysFile();
        ret.setKey(rs.getString("KEY"));
        ret.setType(rs.getString("TYPE"));
        ret.setName(rs.getString("NAME"));
        ret.setExt(rs.getString("EXT"));
        ret.setBytes(rs.getLong("BYTES"));
        ret.setDataPath(rs.getString("DATA_PATH"));
        ret.setDataGroup(rs.getString("DATA_GROUP"));
        ret.setExpired(rs.getDate("EXPIRED"));
        ret.setDescInfo(rs.getString("DESC_INFO"));
        ret.setUpdateBy(rs.getString("UPDATE_BY"));
        ret.setUpdateTime(rs.getDate("UPDATE_TIME"));
        return ret;
    }
}
```

/bhz-com/src/main/java/bhz/com/service/SysFileComService.java  
MultipartFile，通过平台传递的数据类型  
byte[]，通过流传输的字符数组  
定时清空时，调用clearFileKeys方法  

```java
package bhz.com.service;
import java.io.IOException;
import java.util.Date;
import java.util.List;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.PropertySource;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;
import com.alibaba.fastjson.JSONObject;
import bhz.com.dao.SysFileComDao;
import bhz.com.entity.SysFile;
import bhz.com.util.FastDFSClientUtils;
import bhz.com.util.FastJsonConvert;

@Service
@PropertySource("classpath:fastdfs.properties")
public class SysFileComService {

	@Autowired
	private SysFileComDao sysFileComDao;
	
	@Bean
	public static PropertySourcesPlaceholderConfigurer propertyConfigure(){
		return new PropertySourcesPlaceholderConfigurer();
	}
	
	/**
	 * FASTDFS_BASEURL
	 */
	@Value("${fastdfs.baseurl}")
	public String FASTDFS_BASEURL;
	
	
	public String key(){
		return this.sysFileComDao.generateKey();
	}
	
	/**
	 * <B>方法名称：</B>文件上传<BR>
	 * <B>概要说明：</B>文件上传<BR>
	 * @param key 键值
	 * @param type 类型
	 * @param userId 用户标识
	 * @param mf 文件对象	
	 * @param expired 过期时间
	 * @param descInfo 描述
	 * @return String 错误信息
	 * @throws Exception 输入输出异常
	 */
    public JSONObject put(String key, String type, String userId, MultipartFile mf, Date expired, String descInfo) throws Exception {
        
    	String filename = mf.getOriginalFilename();
    	String extName = getExtName(filename);
    	String fileId = FastDFSClientUtils.upload(mf.getBytes(), extName);
    	String dataGroup = fileId.substring(0,fileId.indexOf("/"));
    	long bytes = mf.getSize();
    	
        SysFile file = new SysFile();
        file.setKey(key);
        file.setName(filename);
        file.setType(type);
        file.setExt(extName);
        file.setBytes(bytes);
        file.setDataPath(FASTDFS_BASEURL + fileId);
        file.setDataGroup(dataGroup);
        file.setExpired(expired);
        file.setDescInfo(descInfo);
        file.setUpdateBy(userId);
        sysFileComDao.insert(file);
        
        JSONObject ret = new JSONObject();
        ret.put("success", true);
        ret.put("key", key);
        ret.put("type", type);
        ret.put("name", mf.getOriginalFilename());
        ret.put("bytes", mf.getSize());
        ret.put("dataPath", FASTDFS_BASEURL + fileId);
        ret.put("dataGroup", dataGroup);
        ret.put("expired", expired);
        return ret;
    }
    
    /**
     * <B>方法名称：</B>上传DATA数据文件<BR>
     * <B>概要说明：</B>上传DATA数据文件<BR>
     * 
     * @param key 键值
     * @param type 分类
     * @param userId 用户标识
     * @param fileName 文件名称
     * @param expired 过期时间
     * @param data 文件数据
     * @param descInfo 备注信息
     * @return String 错误信息
     * @throws IOException 输入输出异常
     */
    public String put(String key, String type, String userId, String fileName, Date expired, byte[] data, String descInfo)
            throws IOException {

        if (StringUtils.isBlank(fileName)) {
            return "未指定文件名称";
        }
        if (data == null || data.length < 1) {
            return "未指定文件数据";
        }
        
    	String fileId = FastDFSClientUtils.upload(data, getExtName(fileName));
    	String dataGroup = fileId.substring(0,fileId.indexOf("/"));

        SysFile file = new SysFile();
        file.setKey(key);
        file.setName(fileName);
        file.setType(type);
        file.setExt(getExtName(fileName));
        file.setExpired(expired);
        file.setBytes(data.length);
        file.setDataPath(FASTDFS_BASEURL + fileId);
        file.setDataGroup(dataGroup);
        file.setDescInfo(descInfo);
        file.setUpdateBy(userId);
        sysFileComDao.insert(file);

        return null;
    }    

    /**
     * <B>方法名称：</B>获取系统文件信息<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param key 文件键值
     * @return SysFile 系统文件基本信息
     */
    public SysFile getInfo(String key) {
        return sysFileComDao.get(key);
    }

    /**
     * <B>方法名称：</B>获取系统文件信息列表<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param keys 文件键值集合
     * @return List<SysFile> 系统文件信息列表
     */
    public List<SysFile> getInfoList(String... keys) {
        return sysFileComDao.getList(keys);
    }	
    
    /**
     * <B>方法名称：</B>删除系统文件<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param keys 文件键值
     */
    public void delete(String... keys) {
        if (keys == null || keys.length < 1) {
            return;
        }
        if (keys.length == 1) {
            keys = keys[0].split(",");
        }
        for (String key : keys) {
        	SysFile sysFile = sysFileComDao.get(key);
        	String dataPath = sysFile.getDataPath();
        	String fileId = dataPath.substring(FASTDFS_BASEURL.length());
        	String groupName = sysFile.getDataGroup();
        	FastDFSClientUtils.delete(groupName, fileId);
            sysFileComDao.delete(key);
        }
    }


    /**
     * <B>方法名称：</B>系统文件过期<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param keys 文件键值
     */
    public void expire(String... keys) {
        if (keys == null || keys.length < 1) {
            return;
        }
        if (keys.length == 1) {
            keys = keys[0].split(",");
        }
        for (String key : keys) {
            sysFileComDao.expire(key, null);
        }
    }

    /**
     * <B>方法名称：</B>系统文件过期<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param key 文件键值
     * @param expired 过期时间（null为当前时间）
     */
    public void expire(String key, Date expired) {
        if (StringUtils.isBlank(key)) {
            return;
        }
        String[] keys = key.split(",");
        for (String k : keys) {
            sysFileComDao.expire(k, expired);
        }
    }

    /**
     * <B>方法名称：</B>清除系统文件过期限制<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param keys 文件键值
     */
    public void clearExpired(String... keys) {
        if (keys == null || keys.length < 1) {
            return;
        }
        if (keys.length == 1) {
            keys = keys[0].split(",");
        }
        for (String key : keys) {
            sysFileComDao.clearExpired(key);
        }
    }    

    /**
     * <B>方法名称：</B>获取文件扩展名<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param filename 文件名
     * @return String 扩展名
     */
    public String getExtName(String filename) {
        if (StringUtils.isBlank(filename)) {
            return null;
        }
        int i = filename.indexOf(".");
        if (i >= 0) {
            return filename.substring(i + 1).trim().toLowerCase();
        }
        return null;
    }
    /**
     * <B>方法名称：</B>清空文件方法<BR>
     * <B>概要说明：</B>清空文件方法<BR>
     * @return List<String> 文件Key集合
     */
	public List<String> clearFileKeys(){
		return this.sysFileComDao.clearFileKeys();
	}
}
```

/bhz-com/src/main/java/bhz/com/web/CommController.java  

```java
package bhz.com.web;
import java.util.Date;
import java.util.List;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.multipart.MultipartFile;
import bhz.com.constant.Const;
import bhz.com.entity.SysFile;
import bhz.com.service.MstCodeComService;
import bhz.com.service.SysFileComService;
import bhz.com.util.RequestUtils;
import bhz.com.util.ResponseUtils;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONException;
import com.alibaba.fastjson.JSONObject;

@Controller
public class CommController {
	
	@Autowired
	private MstCodeComService mstCodeComService;
	
	@Autowired
	private SysFileComService sysFileComService;
	

	/**
     * <B>方法名称：</B>业务代码列表集合<BR>
     * <B>概要说明：</B><BR>
     * @param request 页面请求
     * @param response 页面响应
     * @param type 分类
     * @throws JSONException JSON异常错误
     */
    @RequestMapping("/comm/mst/code/list.json")
    public void mstCodeList(HttpServletRequest request, HttpServletResponse response, String type) throws JSONException {
        List<JSONObject> dataList = this.mstCodeComService.getTypeList(type);
        ResponseUtils.putJsonResponse(response, dataList);
    }
    
    /**
     * <B>方法名称：</B>获取文件信息列表<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 页面请求
     * @param response 页面响应
     * @param keys 键值
     * @throws Exception 预想外异常
     */
    @RequestMapping("/comm/file/list.json")
    public void list(HttpServletRequest request, HttpServletResponse response, String keys) throws Exception {

    	if(StringUtils.isBlank(keys)){
            ResponseUtils.putJsonFailureResponse(response, "未指定必要参数：键值");
            return;
    	}
    	
    	String[] keyarr = keys.split(",");
        JSONArray jsonArray = new JSONArray();
        List<SysFile> dataList = this.sysFileComService.getInfoList(keyarr);
        if (dataList != null && dataList.size() > 0) {
            JSONObject json = null;
            SysFile data = null;
            for (int i = 0; i < dataList.size(); i++) {
                data = dataList.get(i);
                json = new JSONObject();
                json.put("key", data.getKey());
                json.put("type", data.getType());
                json.put("name", data.getName());
                json.put("bytes", data.getBytes());
                json.put("dataPath", data.getDataPath());
                json.put("dataGroup", data.getDataGroup());
                json.put("expired", data.getExpired());
                jsonArray.add(i, json);
            }
        }
        ResponseUtils.putJsonResponse(response, jsonArray);
    }
    
    /**
     * <B>方法名称：</B>推送文件<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 页面请求
     * @param response 页面响应
     * @param type 分类
     * @param expired 是否过期
     * @param field 域名
     * @param descInfo 备注信息
     * @throws Exception 预想外异常
     */
    @RequestMapping("/comm/file/put.json")
    public void put(HttpServletRequest request, HttpServletResponse response, String type, Boolean expired, String field, String descInfo) throws Exception {
        if (StringUtils.isBlank(type)) {
            ResponseUtils.putJsonFailureResponse(response, "未指定必要参数：分类");
            return;
        }
        if (StringUtils.isBlank(field)) {
            field = "file";
        }
        String key = this.sysFileComService.key();
        String userId = RequestUtils.getCurrentUserId(request) == null ? Const.SYS_INIT : RequestUtils.getCurrentUserId(request);
        Date expireTime = new Date();
        MultipartFile mf = RequestUtils.getUploadFile(request);
    	if (mf == null || mf.isEmpty()) {
            ResponseUtils.putJsonFailureResponse(response, "未指定上传文件");
            return;
        }
        JSONObject ret = this.sysFileComService.put(key, type, userId, mf, expireTime, descInfo);
        ResponseUtils.putJsonResponse(response, ret);
    }

    /**
     * <B>方法名称：</B>删除指定文件<BR>
     * <B>概要说明：</B><BR>
     * 
     * @param request 页面请求
     * @param response 页面响应
     * @param keys 键值
     * @throws Exception 预想外异常
     */
    @RequestMapping("/comm/file/expire.json")
    public void expire(HttpServletRequest request, HttpServletResponse response, String[] keys) throws Exception {
        this.sysFileComService.expire(keys);
        ResponseUtils.putJsonSuccessResponse(response);
    }    
}
```

配置定时器  
/bhz-sys/src/main/resources/spring-scheduler.xml  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:jee="http://www.springframework.org/schema/jee" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:task="http://www.springframework.org/schema/task" xmlns:util="http://www.springframework.org/schema/util"
	xsi:schemaLocation="
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
	http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee.xsd
	http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
	http://www.springframework.org/schema/task
	http://www.springframework.org/schema/task/spring-task.xsd
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">

	<task:annotation-driven executor="executor" scheduler="scheduler" />
	
	<task:scheduler id="scheduler" pool-size="10"  />
	
	<!-- 
		定时器配置 
	    task:executor/@pool-size：可以指定执行线程池的初始大小、最大大小
	    task:executor/@queue-capacity：等待执行的任务队列的容量 
	    task:executor/@rejection-policy：当等待队已满时的策略，分为丢弃、由任务执行器直接运行等方式 
	    
	    @scheduler配置：
		//initialDelay :  表示第一次运行前需要延迟的时间，单位是毫秒
		//fixedDelay   :  表示从上一个任务完成到下一个任务开始的间隔, 单位是毫秒
		//fixedRate    :  表示从上一个任务开始到下一个任务开始的间隔, 单位是毫秒（如果某次任务开始时上次任务还没有结束，那么在上次任务执行完成时，当前任务会立即执行）
	    
    -->
	<task:executor id="executor"  pool-size="100-200"  queue-capacity="1000"/>
</beans>
```

spring配置文件  
spring-context.xml  

```xml
	<!-- 引入Schedule配置 -->
	<import resource="classpath:spring-scheduler.xml"/>
```

/bhz-com/src/main/java/bhz/com/task/ClearFilesTask.java  
fixedDelay是执行周期30秒，实际一天执行一次就行  

```java
package bhz.com.task;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import bhz.com.service.SysFileComService;
import bhz.com.util.FastDFSClientUtils;

@Component("clearFilesTask")
public class ClearFilesTask {

	private SysFileComService sysFileComService;
	
	public SysFileComService getSysFileComService() {
		return sysFileComService;
	}
	@Autowired
	public void setSysFileComService(SysFileComService sysFileComService) {
		this.sysFileComService = sysFileComService;
	}

	@Scheduled(initialDelay=5000, fixedDelay=1000*30)
	public void clearFiles(){
		System.out.println("定时清理数据库系统文件...");
		List<String> paths = this.sysFileComService.clearFileKeys();
		for (String p : paths) {
			String fileId = p.substring(this.sysFileComService.FASTDFS_BASEURL.length());
			String groupName = fileId.substring(0, fileId.indexOf("/"));
			System.out.println("fileId: " + fileId + ", groupName: " + groupName);
			int code = FastDFSClientUtils.delete(groupName, fileId);
			System.out.println("删除标识代码: " + code);
		}
	}
}
```


## 6.0版本控制  

缓存的实现  
/bhz-com/src/main/java/bhz/com/util/ClusterConfig.java  

```java
package bhz.com.util;
import java.util.HashSet;
import java.util.Set;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.Environment;
import redis.clients.jedis.HostAndPort;
import redis.clients.jedis.JedisCluster;
import redis.clients.jedis.JedisPoolConfig;

@Configuration
//启动缓存配置
@ComponentScan("bhz")
@PropertySource("classpath:redis.properties")
public class ClusterConfig {

	@Autowired
	private Environment environment;
	
	@Bean
	public JedisCluster getRedisCluster(){
	    Set<HostAndPort> jedisClusterNode = new HashSet<HostAndPort>();
	    
	    jedisClusterNode.add(new HostAndPort(environment.getProperty("redis.cluster.nodes1"), Integer.parseInt(environment.getProperty("redis.cluster.port1"))));
	    jedisClusterNode.add(new HostAndPort(environment.getProperty("redis.cluster.nodes2"), Integer.parseInt(environment.getProperty("redis.cluster.port2"))));
	    jedisClusterNode.add(new HostAndPort(environment.getProperty("redis.cluster.nodes3"), Integer.parseInt(environment.getProperty("redis.cluster.port3"))));
	    jedisClusterNode.add(new HostAndPort(environment.getProperty("redis.cluster.nodes4"), Integer.parseInt(environment.getProperty("redis.cluster.port4"))));
	    jedisClusterNode.add(new HostAndPort(environment.getProperty("redis.cluster.nodes5"), Integer.parseInt(environment.getProperty("redis.cluster.port5"))));
	    jedisClusterNode.add(new HostAndPort(environment.getProperty("redis.cluster.nodes6"), Integer.parseInt(environment.getProperty("redis.cluster.port6"))));
	    JedisPoolConfig cfg = new JedisPoolConfig();
	    cfg.setMaxTotal(Integer.parseInt(environment.getProperty("redis.cluster.config.max-total")));
	    cfg.setMaxIdle(Integer.parseInt(environment.getProperty("redis.cluster.config.max-idle")));
	    cfg.setMaxWaitMillis(Integer.parseInt(environment.getProperty("redis.cluster.config.max-waitmillis")));
	    cfg.setTestOnBorrow(Boolean.parseBoolean(environment.getProperty("redis.cluster.config.onborrow"))); 
	    JedisCluster jc = new JedisCluster(jedisClusterNode, Integer.parseInt(environment.getProperty("redis.cluster.timeout")), Integer.parseInt(environment.getProperty("redis.cluster.max-redirections")), cfg);
	    return jc;
	}
}
```

redis.properties  

```
redis.cluster.nodes1=192.168.1.117
redis.cluster.port1=7001
redis.cluster.nodes2=192.168.1.117
redis.cluster.port2=7002
redis.cluster.nodes3=192.168.1.117
redis.cluster.port3=7003
redis.cluster.nodes4=192.168.1.117
redis.cluster.port4=7004
redis.cluster.nodes5=192.168.1.117
redis.cluster.port5=7005
redis.cluster.nodes6=192.168.1.117
redis.cluster.port6=7006
redis.cluster.nodes1=192.168.1.117
redis.cluster.config.max-total=100
redis.cluster.config.max-idle=20
redis.cluster.config.max-waitmillis=-1
redis.cluster.config.onborrow=true
redis.cluster.timeout=6000
redis.cluster.max-redirections=100
```

MstCodeComService.java
缓存使用  

```java
package bhz.com.service;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import redis.clients.jedis.JedisCluster;
import bhz.com.dao.MstCodeComDao;
import bhz.com.util.FastJsonConvert;
import com.alibaba.fastjson.JSONObject;

@Service
public class MstCodeComService {

    /** 缓存标识前缀 */
    private static final String CACHE_ID = "MST_CODE";
    
    @Autowired
	private MstCodeComDao mstCodeComDao;

    @Autowired
    private JedisCluster jedisCluster;
	
	public List<JSONObject> getTypeList(String type){
		String ret = jedisCluster.hget(CACHE_ID, type);
		if(ret == null){
			System.out.println("----查询数据库----");
			List<JSONObject> list = this.mstCodeComDao.getTypeList(type);
			jedisCluster.hset(CACHE_ID, type, list.toString());
			ret = list.toString();
		}
		return FastJsonConvert.convertJSONToArray(ret, JSONObject.class);
	}
	
	//清空指定的hash key 自己实现...
	public void clearTypeList(String type){
	}
}
```