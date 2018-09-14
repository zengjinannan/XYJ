# SSM框架配置文件记录
Spring的有applicationContext.xml
springMVC的有applicationContext-mvc.xml
Mybatis的有mybatis-config.xml。
=====================================================================
或者有些项目拆分为（spring的配置自下而上）
Mybatis依旧是mybatis-config.xml
---------------------------------------------------------------------
Spring的applicationContext.xml拆为
spring-dao.xml：整合mybatis相关的配置（数据库的属性、数据库连接池、SqlSessionFactory对象，Dao接口包注入容器中）
spring-service.xml：配置service层，事务
---------------------------------------------------------------------
springMVC中的applicationContext-mvc.xml
spring-web.xml：配置springMVC


mybatis-config.xml
=====================================================================
=<?xml version="1.0" encoding="UTF-8" ?>
=<!DOCTYPE configuration
=  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
=  "http://mybatis.org/dtd/mybatis-3-config.dtd">
=<configuration>
=	<!-- 配置全局属性 -->
=	<settings>
=		<!-- 使用jdbc的getGeneratedKeys获取数据库自增主键值 -->
=		<setting name="useGeneratedKeys" value="true" />
=
=		<!-- 使用列别名替换列名 默认:true -->
=		<setting name="useColumnLabel" value="true" />
=
=		<!-- 开启驼峰命名转换:Table{create_time} -> Entity{createTime} -->
=		<setting name="mapUnderscoreToCamelCase" value="true" />
=		<!-- 打印查询语句 -->
=		<setting name="logImpl" value="STDOUT_LOGGING" />
=	</settings>
=</configuration>
=====================================================================


applicationContext.xml = spring-dao.xml + spring-service.xml
=====================================================================
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans.xsd 
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx 
       http://www.springframework.org/schema/tx/spring-tx.xsd">
    <!--数据源-链接数据库的基本信息,这里直接写,不放到*.properties资源文件中-->
    <bean id="dataSource"
          class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/javafeng" />
        <property name="username" value="root" />
        <property name="password" value="root" />
    </bean>
    <!-- 配置数据源,加载配置,也就是dataSource -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"></property>
        <!--mybatis的配置文件-->
        <property name="configLocation" value="classpath:mybatis/mybatis-config.xml" />
        <!--扫描 XXXmapper.xml映射文件,配置扫描的路径-->
        <property name="mapperLocations" value="classpath:com/javafeng/mapping/*.xml"></property>
    </bean>
    <!-- DAO接口所在包名，Spring会自动查找之中的类 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.javafeng.dao" />
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
    </bean>
    <!--下面这一部份可以单独放在spring-service.xml层-->
    <!--事务管理-->
    <bean id="transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--注入dataSource-->
        <property name="dataSource" ref="dataSource" />
    </bean>
    <!--开启事务注解扫描-->
    <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
</beans>
=====================================================================

applicationContext-mvc.xml  = spring-web.xml
=====================================================================
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans.xsd 
       http://www.springframework.org/schema/context 
       http://www.springframework.org/schema/context/spring-context.xsd 
       http://www.springframework.org/schema/mvc 
       http://www.springframework.org/schema/mvc/spring-mvc.xsd">
 
    <!-- 告知Spring，我们启用注解驱动 -->
    <mvc:annotation-driven/>
    <!-- org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler，
    它会像一个检查员，对进入DispatcherServlet的URL进行筛查，如果发现是静态资源的请求，
    就将该请求转由Web应用服务器默认的Servlet处理，如果不是静态资源的请求，才由DispatcherServlet继续处理。 -->
    <mvc:default-servlet-handler/>
    <!-- 指定要扫描的包的位置 -->
    <context:component-scan base-package="com.javafeng" />
    <!-- 对静态资源文件的访问,因为Spring MVC会拦截所有请求,导致jsp页面中对js和CSS的引用也被拦截,配置后可以把对资源的请求交给项目的
    默认拦截器而不是Spring MVC-->
    <mvc:resources mapping="/static/**" location="/WEB-INF/static/" />
 
    <!-- 配置Spring MVC的视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- 有时我们需要访问JSP页面,可理解为在控制器controller的返回值加前缀和后缀,变成一个可用的URL地址 -->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
=====================================================================

配置好SSM的相关配置文件，重点在于配置web.xml,用于加载SSM的配置
=====================================================================
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <!-- 加载Spring容器配置 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <!-- Spring容器加载所有的配置文件的路径 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath*:spring/applicationContext.xml</param-value>
    </context-param>
    <!-- 配置SpringMVC核心控制器,将所有的请求(除了刚刚Spring MVC中的静态资源请求)都交给Spring MVC -->
    <servlet>
        <servlet-name>springMvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath*:spring/applicationContext-mvc.xml</param-value>
        </init-param>
        <!--用来标记是否在项目启动时就加在此Servlet,0或正数表示容器在应用启动时就加载这个Servlet,
        当是一个负数时或者没有指定时，则指示容器在该servlet被选择时才加载.正数值越小启动优先值越高  -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <!--为DispatcherServlet建立映射-->
    <servlet-mapping>
        <servlet-name>springMvc</servlet-name>
        <!-- 拦截所有请求,千万注意是(/)而不是(/*) -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
 
    <!-- 设置编码过滤器 -->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
 
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
=====================================================================
