## 环境搭建
1. 导入各种`jar`包
2. 在`src`下新建全局配置文件(编写`JDBC`四个变量)
    1. 没有名称和地址要求
    2. 在全局配置文件中引入`DTD`或`schema`
        1. 若导入`dtd`之后没有提示
        ```
        window-->preference-->XML-->XML catalog-->add
        //添加dtd
        ```
    3. 全局配置文件内容
    ```XML
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE configuration 
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
        <configuration>
        <!-- default 引用 environment 的 id,当前所使用的环境 -->
            <environments default="default">
        <!-- 声明可以使用的环境 -->
                <environment id="default">
        <!-- 使用原生 JDBC 事务 -->
                    <transactionManager type="JDBC"></transactionManager>
                    <dataSource type="POOLED">
                        <property name="driver" value="com.mysql.jdbc.Driver"/>
                        <property name="url" value="jdbc:mysql://localhost:3306/ssm"/>
                        <property name="username" value="root"/>
                        <property name="password" value="smallming"/>
                    </dataSource>
                </environment>
            </environments>
        </configuration>
    ```
3. 新建以`mapper`结尾的包，在包下新建:`实体类名+Mapper.xml`
    1. 文件作用：编写需要执行的`SQL命令`
    2. 把`xml`文件理解成实体类
    3. `xml`文件内容
    ```XML
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <!-- namesapce:理解成实现类的全路径(包名+类名) -->
    <mapper namespace="a.b" >
    <!-- id:方法名
    parameterType:定义参数类型
    resultType:返回值类型. 如果方法返回值是 list,在 resultType 中写 List 的泛型,
    因为 mybatis对 jdbc 封装,一行一行读取数据
    -->
        <select id="selAll" resultType="com.bjsxt.pojo.Flower">
            select * from flower
        </select>
    </mapper>
    ```
4. 测试结果(只有在单独使用`mybatis`时使用，跟`ssm`整合时，下面代码不用写)
```Java
InputStream is = Resources.getResourceAsStream("myabtis.xml");
//使用工厂设计模式
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
//生产 SqlSession
SqlSession session=factory.openSession();
List<Flower> list = session.selectList("a.b.selAll");
for (Flower flower : list) {
    System.out.println(flower.toString());
}
session.close();
```

## 环境搭建详解
1. 全局配置文件中内容
    1. `<transactionManager/>`的 `type` 属性可取值
        1. `JDBC`,事务管理使用 `JDBC` 原生事务管理方式
        2. `MANAGED` 把事务管理转交给其他容器.原生 `JDBC `事务`setAutoMapping(false)`;
    2. `<dataSouce/>`的 `type `属性
        1. `POOLED` 使用数据库连接池
        2. `UNPOOLED` 不实用数据库连接池,和直接使用 `JDBC` 一样
        3. `JNDI` :java 命名目录接口技术

## 数据库连接池
1. 在内存中开辟一块空间,存放多个数据库连接对象. 
2. `JDBC Tomcat Pool`,直接由 `tomcat` 产生数据库连接池. 
3. 状态
    1. `active` 状态: 当前连接对象被应用程序使用中
    2. `Idle` 空闲状态: 等待应用程序使用
4. 使用数据库连接池的目的
    1. 在高频率访问数据库时,使用数据库连接池可以降低服务器系
统压力,提升程序运行效率.
        1. 小型项目不适用数据库连接池. 
5. 实现 `JDBC tomcat Pool` 的步骤. 
    1. 在 `web` 项目的 `META-INF` 中存放 `context.xml`,在 `context.xml` 编写数据库连接池相关属性
    ```XML
    <?xml version="1.0" encoding="UTF-8"?>
    <Context>
    <Resource 
        driverClassName="com.mysql.jdbc.Driver"
        url="jdbc:mysql://localhost:3306/ssm"
        username="root"
        password="smallming"
        maxActive="50"
        maxIdle="20"
        name="test"
        auth="Container"
        maxWait="10000"
        type="javax.sql.DataSource"
    />
    </Context>
    ```
    2. 把项目发布到`tomcat`中，数据库连接池就产生了
6. 可以在`java`中使用`jndi`获取数据库连接池中对象
    1. `Context`: 上下文接口，`context.xml`文件对象类型
    2. 代码：
    ```Java
    Context cxt = new InitialContext();
    DataSource ds = (DataSource)
    cxt.lookup("java:comp/env/test");
    Connection conn = ds.getConnection();
    ```
    3. 当关闭连接对象时，把对象归还给数据库连接池，把状态改成`Idle`

## 三种查询方式
1. `selectList()`返回值为`List<resultType 属性控制>`
    1. 适用于查询结果都需要遍历的需求
    ```Java
    List<Flower> list = session.selectList("a.b.selAll");
    for (Flower flower : list) {
        System.out.println(flower.toString());
    }
    ```
2. `selectOne()`返回值为`Object`
    1. 适用于返回结果只是变量或一行数据时
    ```Java
    int count = session.selectOne("a.b.selById");
    System.out.println(count);
    ```
3. selectMap() 返回值 Map
    1. 适用于需要在查询结果中通过某列的值取到这行数据的需求.
    2. `Map<key,resultType 控制>`
        ```Java
        Map<Object, Object> map = session.selectMap("a.b.c","name123");
        System.out.println(map);
        ```
     
## 注解
1. 注解存在的意义：简化xml文件的开发
2. 注解在`servlet3.0`规范之后大力推广的
3. 注解前面的`@XXX` 表示引用一个`@interface` 
    1. `@interface` 表示注解的声明
4. 注解可以有属性，因为注解就是一个接口(类)
    1. 每次使用注解都要导包
5. 注解的语法： `@XXX(属性名 = 值)`
6. 值的分类
    1. 如果值是基本的数据类型或字符串：属性名=值
    2. 如果值是数组类型：属性名={值，值}
        1. 如果只有一个值可以省略大括号
    3. 如果值是类类型，属性名=@名称
7. 如果注解只需要给一个属性赋值，且这个属性是默认属性，可以省略属性名


## 路径
1. 编写路径为了告诉编译器如何找到其他资源. 
2. 路径分类
    1. 相对路径:从当前资源出发找到其他资源的过程
    2. 绝对路径: 从根目录(服务器根目录或项目根目录)出发找到其他资源的过程
        1. 标志: 只要以 `/` 开头的都是绝对路径
3. 绝对路径:
    1. 如果是请求转发 `/` 表示项目根目录(`WebContent`)
    2. 其他重定向,`<img/>` `<script/>`,`<style/>`,`location.href` 等`/`都表示
服务器根目录(`tomcat/webapps` 文件夹)
4. 如果客户端请求的控制器,控制器转发到JSP后,jsp中如果使用相对
路径,需要按照控制器的路径去找其他资源. 
    1. 保险办法:使用绝对路径,可以防止上面的问题

## Log4J
1. 由`apache`推出的开源免费日志处理的类库
2. 为什么需要日志：
    1. 在项目中编写`System.out.println();`输出到控制台，当项目发布到`tomcat`以后，没有控制台(在命令行界面能看见)，不容易观察一些输出结果
    2. log4j作用，不仅能把内容输出到控制台，还能把**内容输出到文件**中，便于观察结果
3. 使用步骤
    1. 导入`log4j-xxx.jar`
    2. 在`src` 下新建`log4j.properties`(路径和名称都不允许改变)
        1. `ConversionPattern`: 写表达式
        2. `log4j.appender.appender.LOGFTLE.File`：文件位置及名称(日志文件拓展名.log)
        ```Java
        log4j.rootCategory=DEBUG, CONSOLE ,LOGFILE //控制输出级别为info
        log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
        log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
        log4j.appender.CONSOLE.layout.ConversionPattern=%C %d{YYYY-MM-dd hh:mm:ss} %m %n
        log4j.appender.LOGFILE=org.apache.log4j.FileAppender
        log4j.appender.LOGFILE.File=E:/my.log
        log4j.appender.LOGFILE.Append=true
        log4j.appender.LOGFILE.layout=org.apache.log4j.PatternLayout
        log4j.appender.LOGFILE.layout.ConversionPattern=%C %m %L %n
        ```
4. `log4j`输出级别
    1. `fatal`(致命错误) > `error`(错误) > `warn`(警告) > `info`(普通信息) > `debug`(调试信息)
    2. 在`log4j.properties`的第一行控制输出级别
5. `log4j`输出目的地
    1. 在一行控制输出目的地
    ```Java
    log4j.rootCategory=DEBUG, CONSOLE ,LOGFILE
    //console对应文件中的log4j.appender.CONSOLE.*
    //logfile对应文件中的log4j.appender.LOGFILE.*
    ```
6. `pattern`中常用几个表达式
    1. `%C` 包名+类名
    2. `%d{YYYY-MM-dd HH:mm:ss}` 时间
    3. `%L` 行号
    4. `%m` 信息
    5. `%n` 换行

## <settings>标签
1. 在`mybatis`全局配置文件中通过`<settings>`标签控制`mybatis`全局开关
2. 在`mybatis.xml`中开启`log4j`
    1. 必须保证有`log4j.jar`
    2. 在`src`下有`log4j.properties`
    ```xml
    <settings>
        <setting name="logImpl" value="LOG4J"/>
    </settings>
    ```
3. `log4j`中可以输出指定内容的日志(控制某个局部内容的日志级别)
    1. 命名级别(包级别):`<mapper>namespace`中除了最后一个是类名。例如`namespace=”com.bjsxt.mapper.PeopleMapper`,其中包级别为`com.bjsxt.mapper` ,需要在 `log4j.propeties` 中
        1. 先在总体级别调成 `Error` 不输出无用信息
        2. 在设置某个指定位置级别为` DEBUG`
        ```
        log4j.rootCategory=ERROR, CONSOLE ,LOGFILE
        
       log4j.logger.com.bjsxt.mapper.mapper = DEBUG
        ```
    2. 类级别
        1. `namespace` 属性值 ,`namespace` 类名
    3. 方法级别
        1. 使用 `namespace` 属性值+标签 `id` 属性值

## parameterType 属性
1. 在`XXXMapper.xml`中`<select><delect>`等标签的`parameterType`可以控制参数类型
2. `SqlSession` 的 `selectList()`和 `selectOne()`的第二个参数和 `selectMap()`的第三个参数都表示方法的参数
    1. 示例
    ```Java
    People p = session.selectOne("a.b.selById",1);
    System.out.println(p
    ```
    2. 在` Mapper.xml` 中可以通过`#{}`获取参数
        1. `parameterType` 控制参数类型
        2. `#{}`获取参数内容
            1. 使用**索引**,从 `0` 开始` #{0}`表示第一个参数
            2. 也可以使用`#{param1}`第一个参数
            3. 如果只有一个参数(**基本数据类型**或 `String`),`mybatis`对`#{}`里面内容没有要求只要写内容即可
               ```xml
                <select id="selById" resultType="com.bjsxt.pojo.People" parameterType="int">
                    select * from people where id=#{0}
                </select>
                ```
            4. 如果参数是对象`#{属性名}`
            5. 如果参数是 `map `写成`#{key}`
3. `#{}` 和` ${}` 的区别
    1. `#{}` 获取参数的内容**支持索引获取**,`param1` 获取指定位置参数, 并且 `SQL` 使用`?占位符`
    2. `${}` 字符串拼接不使用`?`,默认找`${内容}`内容的` get/set` 方法,如果写数字,就是一个数字
4.  如果在 `xml` 文件中出现 `“<” , “>” `,双引号 等特殊字符时可以使用
`XML `文件转义标签(`XML 自身的`)**例如：
    1. `<![CDATA[ 内容 ]]>`
5. `mybatis` 中实现 mysql 分页写法
    1. `?`**不允许在关键字前后进行数学运算**,需要在代码中计算完成
后传递到 `mapper.xml `中
    2. 在 `java `代码中计算
        ```Java
            //显示几个
            int pageSize = 2;
            //第几页
            int pageNumber= 2;
            //如果希望传递多个参数,可以使用对象或map
            mapMap<String,Object> map = new HashMap<>();
            map.put("pageSize", pageSize);
            //每一页开始的编号
            map.put("pageStart", pageSize*(pageNumber-1));
            //第二个参数传递了map给xml
            List<People> p =session.selectList("a.b.page",map);
        ```
        3. 在`mapper.xml`中的代码
            ```xml
                <select id="page" resultType="com.bjsxt.pojo.People" parameterType="map">
                    select * from people limit #{pageStart},#{pageSize}
                </select>
            ```

## typeAliases 别名
1. 系统内置别名: 把类型全小写
2. 给某个类起别名
    1. alias="自定义"
        ```XML
        //给com.bjsxt.pojo.People起别名为peo
        <typeAliases>
            <typeAlias type="com.bjsxt.pojo.People"alias="peo"/>
        </typeAliase
        ```
    2. `mapper.xml` 中 `peo` 引用 `People` 类
        ```XML
        <select id="page" resultType="peo" parameterType="map">
            select * from people limit #{pageStart},#{pageSize}
        </select>
        ```
3. 直接给某个包下所有类起别名,**别名即为类名,不区分大小写**
    1. `mybatis.xml` 中配置
        ```xml
        <typeAliases>
            <package name="com.bjsxt.pojo" />
        </typeAliases>
        ```
    2. `mapper.xml` 中通过类名引用
        ```xml
        <select id="page" resultType="People" parameterType="map">
            select * from people limit #{pageStart},#{pageSize}
        </select>
        ```
    
## Mybatis实现新增
1. 概念
    1. 功能:从应用程序角度出发,软件具有哪些功能. 
    2. 业务:完成功能时的逻辑.对应 `Servic`e 中一个方法
    3. 事务:从数据库角度出发,完成业务时需要执行的 `SQL 集合`,统
称一个事务. 
        1. 事务回滚:如果在一个事务中某个` SQL 执行事务`,希望回
归到事务的原点,保证数据库数据的完整性. 
2. 在 `mybatis` 中默认是关闭了` JDBC `的自动提交功能
    1. 每一个` SqlSession `默认都是不自动提交事务. 
    2. `session.commit()`提交事务. 
    3. ` openSession(true)`;自动提交.`setAutoCommit(true)`;
3. `mybatis` 底层是对` JDBC `的封装. 
    1. `JDBC` 中 `executeUpdate()`执行新增,删除,修改的 `SQL`.返回值 `int`, 表示受影响的行数. 
    2. ` mybatis `中`<insert> <delete> <update>`标签没有 `resultType` 属性, 认为返回值都是 `int`
4. 在` openSession()`时` Mybatis` 会创建 `SqlSession` 时同时创建一个
`Transaction(事务对象)`,同时` autoCommit` 都为` false`
    1. **如果出现异常,应该 `session.rollback()`回滚事务**
5. 实现新增的步骤
    1. 在 `mapper.xml` 中提供`<insert>`标签,标签没有返回值
        ```xml
        <insert id="ins" parameterType="People">
            insert into people values(default,#{name},#{age})
        </insert>
        ```
    2. 通过 `session.insert()`调用新增方法
        ```Java
        //传给xml文件一个p对象
        int index1 = session.insert("a.b.ins", p);
        if(index1>0){
            System.out.println("成功");
        }else{
            System.out.println("失败");
        }    
        ```
        
## MyBatis实现修改
1. 在 `mapper.xml` 中提供`<update>`标签
    ```xml
    <update id="upd" parameterType="People">
        update people set name = #{name} where id = #{id}
    </update>
    ```
2. 测试代码
    ```Java
        People peo = new People();
        peo.setId(3);
        peo.setName("王五");
        int index = session.update("a.b.upd", peo);
        if(index>0){
            System.out.println("成功");
        }else{
            System.out.println("失败");
        }
        //MyBatis默认不自动提交
        session.commit();
    ```
    
## MyBatis实现删除
1. 在 `mapper.xml` 提供`<delete>`
    ```xml
    <delete id="del" parameterType="int">
        delete from people where id = #{0}
    </delete>
    ```
2. 测试代码
    ```Java
        int del = session.delete("a.b.del",3);
        if(del>0){
            System.out.println("成功");
        }else{
            System.out.println("失败");
        }
        session.commit();
    ```
    
## MyBatis 接口绑定方案及多参数传递

1. 作用：实现创建一个接口后把`mapper.xml`由`mybatis`生成接口的实现类，通过调用接口对象就可以获取`mapper.xml`中编写的`sql`
2. 后面`mybatis`和`spring`整合时使用的是这个方案
3. 实现步骤：
    1. 创建一个接口
        1. 接口包名和接口名与` mapper.xml` 中`<mapper>namespace`
相同
        2. 接口中方法名和 `mapper.xml` 标签的 `id` 属性相同
    2. 在 `mybatis.xml` 中使用`<package>`进行扫描接口和 `mapper.xml`
4. 代码实现步骤：
    1. 在`mybatis.xml`中`<mappers>`下使用`<package>`
        ```xml
        <mappers>
            <package name="com.xxx.mapper"/>
        </mappers>
        ```
    2. 在 `com.xxx.mapper` 下新建接口
        ```Java
        public interface LogMapper {
            List<Log> selAll();
        }
        ```
    3. 在`com.xxx.mapper`新建一个 `LogMapper.xml`
        1. **`namespace` 必须和接口全限定路径(包名+类名)一致**
        2. **`id` 值必须和接口中方法名相同**
        3. 如果接口中方法为多个参数,可以省略 `parameterType`
        ```xml
        <mapper namespace="com.xxx.mapper.LogMapper">
            <select id="selAll" resultType="log">
                select * from log
            </select>
        </mapper>
        ```
5. 多参数的实现方法
    1. 在接口中声明方法
    ```Java
    List<Log> selByAccInAccout(String accin,String accout);
    ```
    2. 在 mapper.xml 中添加
        1. `#{}`中使用 `0,1,2` 或 `param1,param2`
        ```xml
        <!-- 当多参数时,不需要写 parameterType -->
        <select id="selByAccInAccout" resultType="log" >
            select * from log where accin=#{0} and accout=#{1}
        </select>
        ```
6. 可以使用注解的方式
    1. 在接口中声明方法
    ```Java
    /**
    * mybatis 把参数转换为 map 了,其中@Param("key") 参数内
    容就是 map 的 value
    * @param accin123
    * @param accout3454235
    * @return
    */
    List<Log> selByAccInAccout(@Param("accin") String
    accin123,@Param("accout") String accout3454235);
    ```
    2. 在 `mapper.xml` 中添加
        1. `#{} `里面`写@Param(“内容”)`参数中**内容**
        ```xml
        <!-- 当多参数时,不需要写 parameterType -->
        <select id="selByAccInAccout" resultType="log" >
            select * from log where accin=#{accin} and accout=#{accout}
        </select>
        ```

## 动态SQL
1. 根据不同的条件需要执行不同的 `SQL 命令`.称为动态 `SQL`
2. `MyBatis` 中动态 `SQL` 在 `mapper.xml `中添加逻辑判断等. 
3. `<If> `使用
    ```xml
    <select id="selByAccinAccout" resultType="log">
            select * from log where 1=1
        <!-- OGNL 表达式,直接写 key 或对象的属性.不需要添加任何特字符号 -->
        <if test="accin!=null and accin!=''">
            and accin=#{accin}
        </if>
        <if test="accout!=null and accout!=''">
            and accout=#{accout}
        </if>
    </select>
    ```
4. `<where>`
    1. 当编写 `where` 标签时,如果内容中第一个是 `and` 去掉第一个
`and`
    2. 如果`<where>`中有内容会生成 `where` 关键字,如果没有内容不
生成 `where` 关键字
    3. 使用示例
        1. 比直接使用`<if>`少写 `where 1=1`
        ```xml
        <select id="selByAccinAccout" resultType="log">
            select * from log
            <where>
                <if test="accin!=null and accin!=''">
                    and accin=#{accin}
                </if>
                <if test="accout!=null and accout!=''">
                    and accout=#{accout}
                </if>
            </where>
        </select>
        ```
5. `<choose> <when> <otherwise>`
    1. 只有有一个成立,其他都不执行. 
    2. 代码示例
        1. 如果 `accin` 和 `accout` 都不是 `null `或不是空生成的 `sql `中只有` where accin=?`(choose标签导致)
        ```xml
        <select id="selByAccinAccout" resultType="log">
            select * from log
            <where>
                <choose>
                    <when test="accin!=null and accin!=''">
                        and accin=#{accin}
                    </when>
                    <when test="accout!=null and accout!=''">
                        and accout=#{accout}
                    </when>
                </choose>
            </where>
        </select>
        ```
6. `<set>`用在修改 `SQL` 中 `set` 从句
    1. 作用:去掉最后一个逗号
    2. 作用:如果`<set>`里面有内容生成 `set` 关键字,没有就不生成
    3. 示例
        1. `id=#{id}` 目的防止`<set>`中没有内容,`mybatis `不生成 `set` 关键字,如果修改中没有 `set` 从句` SQL` 语法错误. 
        ```xml
        <update id="upd" parameterType="log" >
            update log
            <set>
                id=#{id},
                <if test="accIn!=null and accIn!=''">
                    accin=#{accIn},
                </if>
                <if test="accOut!=null and accOut!=''">
                    accout=#{accOut},
                </if>
            </set>
            where id=#{id}
        </update>
        ```
7. `Trim`标签
    1. `prefix` 在前面添加内容
    2. `prefixOverrides` 去掉前面内容
    3. `suffix `在后面添加内容
    4. `suffixOverrieds` 去掉后面内容
    5. 执行顺序:**去掉内容后添加内容**
    6. 代码示例(去掉最后一个逗号)
    ```xml
    <update id="upd" parameterType="log">
        update log
        <trim prefix="set" suffixOverrides=",">
            a=a,
        </trim>
        where id=100
    </update>
    ```
8. `<bind>`
    1. 作用:给参数重新赋值
    2. 使用场景:
        1. 模糊查询
        2. 在原内容前或后添加内容
    3. 示例
    ```xml
    <select id="selByLog" parameterType="log" resultType="log">
        <bind name="accin" value="'%'+accin+'%'"/>
        #{money}
    </select>
    ```
9. `<foreach>`标签
    1. 循环参数内容,还具备在内容的前后添加内容,还具备添加分
隔符功能. 
    2. 适用场景:**`in` 查询中**.批量新增中(`mybatis` 中 `foreach` 效率比较低)
        1. 如果希望批量新增,`SQL` 命令
        ```SQL
        insert into log VALUES (default,1,2,3),(default,2,3,4),(default,3,4,5)
        ```
        2. `openSession()`必须指定
            1. 底层 `JDBC `的 `PreparedStatement.addBatch();`
            ```Java
            factory.openSession(ExecutorType.BATCH);
            ```
   3. 示例
        1. `collection=”xx”` :要遍历的集合
        2. `item`: 迭代变量, `#{迭代变量名}`获取内容
        3.`open`: 循环后左侧添加的内容
        4. `close`: 循环后右侧添加的内容
        5. `separator`: 每次循环时,元素之间的分隔符
        ```xml
        <select id="selIn" parameterType="list" resultType="log">
            select * from log where id in
            <foreach collection="list" item="abc" open="(" close=")" separator=",">
                #{abc}
            </foreach>
        </select>
        ```
10. `<sql>` 和`<include>`
    1.  某些 `SQL` 片段如果希望复用,可以使用`<sql>`定义这个片段
    ```xml
    <sql id="mysql">
        id,accin,accout,money
    </sql>
    ```
   2. 在`<select>`或`<delete>`或`<update>`或`<insert>`中使用`<include>`引用
   ```xml
   <select id="abc">
        select <include refid="mysql"></include>
        from log
    </select>
   ```

## ThreadLocal
1. 线程容器,给线程绑定一个 `Object` 内容,后只要线程不变,可以随时
取出. 
    1. 改变线程,无法取出内容. 
2. 语法示例
```Java
final ThreadLocal<String> threadLocal = new ThreadLocal<>();
threadLocal.set("测试");
new Thread(){
    public void run() {
        String result = threadLocal.get();
        System.out.println("结果:"+result);
    };
}.start();
```

## 缓存
1. 应用程序和数据库交互的过程是一个相对比较耗时的过程
2. 缓存存在的意义:让应用程序减少对数据库的访问,提升程序运行
效率
3. ` MyBatis` 中默认 `SqlSession` 缓存开启
    1. 同一个 `SqlSession` 对象调用同一个`<select`>时,只有第一次访问
数据库,第一次之后把查询结果缓存到 `SqlSession` 缓存区(内存)中
    2. 缓存的是 `statement` 对象.(简单记忆必须是用一个`<select>`)
        1. 在` myabtis` 时一个`<select>`对应一个` statement` 对象
    3. 有效范围必须是同一个 `SqlSession` 对象
4. 缓存流程
    1. 步骤一: 先去缓存区中找是否存在 `statement`
    2. 步骤二:返回结果
    3. 步骤三:如果没有缓存 ` statement` 对象,去数据库获取数据
    4. 步骤四:数据库返回查询结果
    5. 步骤五:把查询结果放到对应的缓存区中
5. `SqlSessionFactory` 缓存
    1. 又叫:二级缓存
    2. 有效范围:同一个 `factory` 内哪个` SqlSession `都可以获取
    3. 什么时候使用二级缓存:
        1. 当数据频繁被使用,很少被修改
    4. 使用二级缓存步骤
        1. 在 `mapper.xml` 中添加
        ```
        <cache readOnly="true"></cache>
        ```
        2. 如果不写 `readOnly=”true”`需要把实体类序列化
5. 当 `SqlSession` 对象 `close()`时或 `commit()`时会把 `SqlSession` 缓存的数据刷(flush)到 `SqlSessionFactory` 缓存区中

## Mybatis实现多表查询
1. Mybatis 实现多表查询方式
    1. 业务装配.对两个表编写单表查询语句,在业务(`Service`)把查询
的两个结果进行关联. 
    2. 使用 `Auto Mapping` 特性,在实现两表联合查询时通过别名完成
映射. 
    3. 使用 `MyBatis` 的`<resultMap>`标签进行实现.
2. 多表查询时,类中包含另一个类的对象的分类
    1.  单个对象
    2.  集合对象.

## resultMap标签
1. `<resultMap>`标签写在`mapper.xml`中,由程序员控制`SQL`查询结果与
实体类的映射关系. 
    1. 默认 `MyBatis` 使用 `Auto Mapping` 特性. 
2. 使用`<resultMap>`标签时,`<select>`标签不写 `resultType` 属性,而是使
用 `resultMap `属性引用`<resultMap>`标签. 
3. 使用` resultMap` 实现单表映射关系
    1. 数据库设计
    
    id | name
    ---|---
    1 | 老师1
    2 | 老师2
    2. 实体类设计
    ```Java
    public class Teacher{
        private int id1;
        private String name1;
        //get和set方法
        ...
    }
    ```
    3. `mapper.xml` 代码
    ```xml
    <resultMap type="teacher" id="mymap">
    <!-- 主键使用 id 标签配置映射关系 -->
        <id column="id" property="id1" />
    <!-- 其他列使用 result 标签配置映射关系 -->
        <result column="name" property="name1"/>
    </resultMap>
    <select id="selAll" resultMap="mymap">
        select * from teacher
    </select>
    ```
4. 使用 `resultMap` 实现关联单个对象(`N+1 `方式)
    1. ` N+1` 查询方式,先查询出某个表的全部信息,根据这个表的信息
查询另一个表的信息. 
    2. 与业务装配的区别:
        1.  在 `service` 里面写的代码,由 `mybatis` 完成装配
    3.  实现步骤:
        1. 在 `Student` 实现类中包含了一个` Teacher `对象
        ```Java
        public class Student {
            private int id;
            private String name;
            private int age;
            private int tid;
            private Teacher teacher;
            //get 和 set 方法
            ...
        }
        ```
        2. 在 `TeacherMapper` 中提供一个查询
        ```xml
        <select id="selById" resultType="teacher" parameterType="int">
            select * from teacher where id=#{0}
        </select>
        ```
        3. 在 `StudentMapper` 中
            1. ` <association>` 装配一个对象时使用
            2. `property`: 对象在类中的属性名
            3. `select`:通过哪个查询查询出这个对象的信息
            4. `column`: 把当前表的哪个列的值做为参数传递给另
一个查询
            5. 大前提使用 `N+1` 方式.时如果列名和属性名相同可
以不配置,使用 `Auto mapping` 特性.但是 `mybatis` 默认只会给列
专配一次
            ```xml
            <resultMap type="student" id="stuMap">
                <id property="id" column="id"/>
                <result property="name" column="name"/>
                <result property="age" column="age"/>
                <result property="tid" column="tid"/>
                <!-- 如果关联一个对象 -->
                <association property="teacher" select="com.bjsxt.mapper.TeacherMapper.selById" column="tid"></association>
            </resultMap>
            <select id="selAll" resultMap="stuMap">
                select * from student
            </select>
            ```
            6. 把上面代码简化成
            ```xml
            <resultMap type="student" id="stuMap">
                <result column="tid" property="tid"/>
                <!-- 如果关联一个对象 -->
                <association property="teacher" select="com.bjsxt.mapper.TeacherMapper.selById" column="tid"></association>
            </resultMap>
            <select id="selAll" resultMap="stuMap">
                select * from student
            </select>
            ```
5. 使用 `resultMap` 实现关联单个对象(**联合查询方式**)
    1. 只需要编写一个 `SQL`,在` StudentMapper `中添加下面效果
        1. `<association/>`只要专配一个对象就用这个标签
        2. 此时把`<association/>`当作小的`<resultMap>`看待
        3. `javaType` 属性:`<association/>`专配完后返回一个什么类型
的对象.取值是一个类(或类的别名)
        ```xml
        <resultMap type="Student" id="stuMap1">
            <id column="sid" property="id"/>
            <result column="sname" property="name"/>
            <result column="age" property="age"/>
            <result column="tid" property="tid"/>
            <association property="teacher" javaType="Teacher" >
                <id column="tid" property="id"/>
                <result column="tname" property="name"/>
            </association>
        </resultMap>
        <select id="selAll1" resultMap="stuMap1">
            select s.id sid,s.name sname,age age,t.id
            tid,t.name tname FROM student s left outer join teacher
            t on s.tid=t.id
        </select>
        ```
6. `N+1` 方式和联合查询方式对比
    1. `N+1`:需求不确定时. 
    2. 联合查询:需求中确定查询时两个表一定都查询. 
7. `N+1` 名称由来
    1. 举例:学生中有 3 条数据
    2. 需求:查询所有学生信息级授课老师信息
    3. 需要执行的 `SQL` 命令
        1. 查询全部学生信息:`select * from 学生`
        2. 执行 3 遍 `select * from 老师 where id=学生的外键`
    4. 使用多条 `SQl` 命令查询两表数据时,如果希望把需要的数据都
查询出来,需要执行` N+1` 条 `SQl `才能把所有数据库查询出来.
    5. 缺点:
        1. 效率低
    6. 优点:
        1. 如果有的时候不需要查询学生是同时查询老师.只需要
执行一个` select * from student`;
    7. 适用场景: 有的时候需要查询学生同时查询老师,有的时候只
需要查询学生. 
    8. 如何解决 `N+1` 查询带来的效率低的问题
        1. 默认带的前提: 每次都是两个都查询. 
        2. 使用两表联合查询. 

## 使用<resultMap>查询关联集合对象(N+1)
1. 在` Teacher `中添加 `List<Student>`
```Java
public class Teacher {
    private int id;
    private String name;
    private List<Student> list;
    //get和set方法
    ...
```
2. 在 `StudentMapper.xml` 中添加通过 `tid` 查询
```xml
<select id="selByTid" parameterType="int" resultType="student">
    select * from student where tid=#{0}
</select>
```
3. 在 `TeacherMapper.xml `中添加查询全部
    1. `<collection/>` 当**属性是集合类型时使用的标签**.
    ```xml
    <resultMap type="teacher" id="mymap">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <collection property="list" select="com.bjsxt.mapper.StudentMapper.selByTid" column="id"></collection>
    </resultMap>
    <select id="selAll" resultMap="mymap">
        select * from teacher
    </select>
    ```

## 使用<resultMap>实现加载集合数据(联合查询方式)
1. 在 `teacherMapper.xml` 中添加
    1. `mybatis` 可以通过主键判断对象是否被加载过. 
    2. 不需要担心创建重复 `Teacher`
    ```xml
    <resultMap type="teacher" id="mymap1">
        <id column="tid" property="id"/>
        <result column="tname" property="name"/>
        <collection property="list" ofType="student" >
            <id column="sid" property="id"/>
            <result column="sname" property="name"/>
            <result column="age" property="age"/>
            <result column="tid" property="tid"/>
        </collection>
    </resultMap>
    <select id="selAll1" resultMap="mymap1">
        select t.id tid,t.name tname,s.id sid,s.name
        sname,age,tid from teacher t LEFT JOIN student s on
        t.id=s.tid;
    </select>
    ```

## 使用 Auto Mapping 结合别名实现多表查询.
1. 只能使用多表联合查询方式. 
2. 要求:查询出的列别和属性名相同. 
3.  实现方式
    1. 在` SQL `是关键字符,两侧添加反单引号
    ```xml
    <select id="selAll" resultType="student">
        select t.id `teacher.id`,t.name
        `teacher.name`,s.id id,s.name name,age,tid
        from student s LEFT JOIN teacher t on t.id=s.tid
    </select>
    ```

## Mybatis注解
1. 注解:为了简化配置文件.
2.` Mybatis `的注解简化 `mapper.xml` 文件. 
    1. 如果涉及动态 `SQL` 依然使用 `mapper.xml`
3. `mapper.xml `和注解可以共存. 
4. 使用注解时 `mybatis.xml` 中`<mappers>`使用
    1. `<package/>`
    2. `<mapper class=””/>`
5. 实现查询
```Java
@Select("select * from teacher")
List<Teacher> selAll();
```
6. 实现新增
```Java
@Insert("insert into teacher
values(default,#{name})")
int insTeacher(Teacher teacher);
```
7. 实现修改
```Java
@Update("update teacher set name=#{name} where
id=#{id}")
int updTeacher(Teacher teacher);
```
8. 实现删除
```Java
@Delete("delete from teacher where id=#{0}")
int delById(int id);
```
9. 使用注解实现`<resultMap>`功能(**一般不用注解实现**)
    1. 以 `N+1` 举例
    2. 在 `StudentMapper` 接口添加查询
    ```Java
    @Select("select * from student where tid=#{0}")
    List<Student> selByTid(int tid);
    ```
    3. 在 `TeacherMapper` 接口添加
        1. `@Results()` 相当于`<resultMap>`
        2. `@Result()` 相当于`<id/>`或`<result/>`
            1. `@Result(id=true)` 相当与`<id/>`
        3. `@Many()` 相当于`<collection/>`
        4. `@One()` 相当于`<association/>`
        ```Java
        @Results(value={ @Result(id=true,property="id",column="id"),
        @Result(property="name",column="name"),
        @Result(property="list",column="id",many=@Many(select="com.bjsxt.mapper.StudentMapper.selByTid"))})
        @Select("select * from teacher")
        List<Teacher> selTeacher();
        ```
## Mybatis运行原理
1. 运行过程中涉及到的类
    1. `Resources`: `MyBatis` 中 `IO 流`的工具类,加载配置文件
    2. `SqlSessionFactoryBuilder()`: 构建器
        1. 作用:创建 `SqlSessionFactory` 接口的实现类
    3. `XMLConfigBuilder`: `MyBatis` 全局配置文件内容构建器类
       1. 作用: 负责读取流内容并转换为 `JAVA 代码`. 
    4. `Configuration` 封装了全局配置文件所有配置信息. 
        1. 全局配置文件内容存放在` Configuration` 中
    5. `DefaultSqlSessionFactory` 是`SqlSessionFactory`接口的实现类
    6. `Transaction` 事务类
        1. 每一个 `SqlSession` 会带有一个 `Transaction` 对象.
    7. `TransactionFactory` 事务工厂
       1. 作用： 负责生产 `Transaction`
    8. `Executor`: `MyBatis` 执行器
        1. 作用:负责执行 `SQL` 命令
        2. 相当于 `JDBC` 中 `statement` 对象(或 `PreparedStatement`
    或 `CallableStatement`)
        3. 默认的执行器 `SimpleExcutor`
        4. 批量操作 `BatchExcutor`
        5. 通过 `openSession(参数控制)`
    9. `DefaultSqlSession` 是` SqlSession` 接口的实现类
    10. `ExceptionFactory`: `MyBatis` 中异常工厂
2. 流程图
```
graph TD
A[Resources加载全局配置文件];
B[实例化SqlSessionFactoryBuilder构建器];
C[由XMLConfigBuilder解析配置文件流];
D[把配置信息放在Configuration中];
E[实例化SqlSessionFactory实现类DefaultSqlSessionFactory];
F[由TransactionFactory创建一个Transaction事物对象];
G[创建执行器Excutor];
H[创建SqlSession接口实现类DefaultSqlSession];
I[实现CURD];
J{执行是否成功};
K[事务提交];
L[关闭];
A-->B
B-->C
C-->D
D-->E
E-->F
F-->G
G-->H
H-->I
I-->F
I-->J
J--成功-->K
J--失败,事物回滚-->F
K-->L
```
3. 文字解释
在 `MyBatis` 运行开始时需要先通过` Resources `加载全局配置文件.下面
需要实例化 `SqlSessionFactoryBuilder `构建器.帮助` SqlSessionFactory` 接口实现类 `DefaultSqlSessionFactory`. 在实例化 `DefaultSqlSessionFactory` 之前需要先创建 `XmlConfigBuilder`
解析全局配置文件流,并把解析结果存放在 `Configuration` 中.之后把
`Configuratin` 传递给` DefaultSqlSessionFactory`.到此`SqlSessionFactory` 工
厂创建成功. 由` SqlSessionFactory `工厂创建` SqlSession`. 每次创建 `SqlSession `时,都需要由 `TransactionFactory `创建 `Transaction`
对象,同时还需要创建 `SqlSession `的执行器` Excutor`,最后实例化
`DefaultSqlSession`,传递给 `SqlSession` 接口. 根据项目需求使用 `SqlSession` 接口中的 `API` 完成具体的事务操作. 如果事务执行失败,需要进行 `rollback 回滚事务`. 如果事务执行成功提交给数据库.关闭 `SqlSession`