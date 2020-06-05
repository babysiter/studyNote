@[TOC](MyBatis3学习笔记)

# 为什么要用mybatis
- MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的半自动化的持久层框架。 
- MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。 
- MyBatis可以使用简单的XML或注解用于配置和原始映射，将接口和Java的POJO（Plain Old Java Objects，普通的Java对象）映射成数据库中的记录
- sql和java编码分开，功能边界清晰，一个专注业务、一个专注数据
- JDBC的缺陷 
	– SQL夹在Java代码块里，耦合度高导致硬编码内伤 
	– 维护不易且实际开发需求中sql是有变化，频繁修改的情况多见 
- Hibernate和JPA的缺陷 
	– 长难复杂SQL，对于Hibernate而言处理也不容易 
	– 内部自动生产的SQL，不容易做特殊优化。 
	– 基于全映射的全自动框架，大量字段的POJO进行部分映射时比较困难。 导致数据库性能下降。 对开发人员而言，核心sql还是需要自己优化 


# MyBatis-全局配置文件

##	properties属性
properties属性可以直接加载外部的properties文件，还可以额外通过property的name和value来配置属性。
配置后的属性可以通过${name}来获取它的值。
```javascript
<properties resource="db.properties">
	<!--properties中还可以配置一些属性名和属性值  -->
	<!-- <property name="user" value="root"/> -->
</properties>
```

##	settings属性
settings属性包含了mybatis的许多全局配置参数，想要打开延迟加载就需要在这里配置。
![setting属性一览表](https://img-blog.csdnimg.cn/20200520211446529.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxNzQyMTQzNzk3,size_16,color_FFFFFF,t_70#pic_center)
示例
```javascript
<settings>
		<!-- 实现延迟加载 -->
		<setting name="lazyLoadingEnabled" value="true"/>
		<setting name="aggressiveLazyLoading" value="false"/>
	</settings>
```
##	 typeAliases-别名处理器
* 类型别名是为 Java 类型设置一个短的名字，可以方便我们引用某个类。 
```javascript
<typeAliases>
	<typeAlias type="cn.itcast.mybatis.po.User" alias="user"/>
</typeAliases>
```
- 类很多的情况下，可以批量设置别名这个包下的每一个类创建一个默认的别名，就是类名其中首字母会变成小写。 
```javascript
	<typeAliases>
		<package name="zjut.lhd.po"/>
	</typeAliases>
```
- 也可以使用@Alias注解为其指定一个别名
```javascript
 @alias(“a”)
Public class abc{}
 ```
 - Mybatis也内部自己包含了许多别名
 
![别名](https://img-blog.csdnimg.cn/20200520212338903.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxNzQyMTQzNzk3,size_16,color_FFFFFF,t_70#pic_center)
##	typeHandlers类型处理器
Mybatis在预处理语句（PreparedStatement）中设置参数时或者时在从数据库得到的结果集中，都会用类型处理器以合适的方式转换为java类型

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200520212635855.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxNzQyMTQzNzk3,size_16,color_FFFFFF,t_70#pic_center)
###	日期类型的处理
JDK1.8以前，我们通常使用JSR310规范领导者Stephen Colebourne创建的Joda-Time来操作。1.8已经实现全部的JSR310规范了。 
我们可以使用MyBatis基于JSR310（Date and Time API）编写的各种日期时间类型处理器
MyBatis3.4以前的版本需要我们手动注册这些处理器，以后的版本都是自动注册的
```javascript
<typeHandlers>
	<typeHandler handler="org . apache . ibatis . type . InstantTypeHandler" />
	<typeHandler handler="org. apache. ibatis . type . LocalDateTimeTypeHandler" />
	<typeHandler handler= "org. apache . ibatis . type. LocalDateTypeHandler" />
	<typeHandler handler="org. apache. ibatis . type . LocalTimeTypeHandler" />
	<typeHandler handler="org . apache . ibatis . type .OffsetDateTimeTypeHandler" />
	<typeHandler handler= "org . apache . ibatis . type . ZonedDateTimeTypeHandler" />
	<typeHandler handler="org. apache. ibatis . type . YearTypeHandler" />
	<typeHandler handler="org. apache . ibatis . type . MonthTypeHandler" />
	<typeHandler handler="org . apache . ibatis . type . YearMonthTypeHandler" />
	<typeHandler handler="org . apache . ibatis . type .JapaneseDateTypeHandler" />
</typeHandlers>	
```
###	自定义TypeHandler处理枚举
我们可以通过自定义TypeHandler的形式来在设置参数或者取出结果集的时候自定义参数封装策略。 
1、实现TypeHandler接口或者继承BaseTypeHandler 
2、在全局配置文件中配置自定义类型处理器
```javascript
	<typeHandlers>
		<typeHandler handler= "zjut.typeHandler.myTypeHandler" javaType= "zjut.po.address" jdbcType="varchar"/>
	< /typeHandlers >
```
3、或者在mapper.xml文件中涉及到你自己定义好的javatype和jdbctype转换时，添加typeHandler。
```javascript
	<!-- 示例代码 -->
	<resultMap id="t" type="zjut.user">
        <result column="address" jdbcType="VARCHAR" property="address" typeHandler="zjut.typeHandler.myTypeHandler"></result>
    </resultMap>
    #{address,typeHandler="zjut.typeHandler.myTypeHandler"}
```
##	plugins插件
插件是MyBatis提供的一个非常强大的机制，我们可以通过插件来修改MyBatis的一些核心行为。插件通过动态代理机制，可以介入四大对象的任何一个方法的执行。
- Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed) 
- ParameterHandler (getParameterObject, setParameters) 
- ResultSetHandler (handleResultSets, handleOutputParameters) 
- StatementHandler (prepare, parameterize, batch, update, query)

## environments环境
MyBatis可以配置多种环境，比如开发、测试和生产环境需要有不同的配置。 
每种环境使用一个environment标签进行配置并指定唯一标识符 
可以通过environments标签中的default属性指定一个环境的标识符来快速的切换环境
### environment-指定具体环境 
- id：指定当前环境的唯一标识 
- transactionManager、和dataSource都必须有
```javascript
	<environments default= "deveLopment" >
		<environment id= "deveLopment">
			<transactionManager type= "JDBC" />
			<dataSource type= "POOLED">
				<property name= "driver" value= "${driver}" 
				<property name= "urL" value= "${urL}" />/>
				<property name= "username”value= "${username}" />
				<property name= "password" value= "${ password}"/>
			</dataSource>
		</environment>
	</environments>	
```
###	transactionManager
**type： JDBC | MANAGED | 自定义**
- **JDBC**：使用了 JDBC 的提交和回滚设置，依赖于从数据源得到的连接来管理事务范围。**dbcTransactionFactory 
-  **MANAGED**：不提交或回滚一个连接、让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。 ManagedTransactionFactory
-  **自定义**：实现TransactionFactory接口，type=全类名/别名
###	dataSource
**type： UNPOOLED | POOLED | JNDI | 自定义**
- **UNPOOLED**：不使用连接池，UnpooledDataSourceFactory 
- **POOLED**：使用连接池， PooledDataSourceFactory 
- **JNDI**： 在EJB 或应用服务器这类容器中查找指定的数据源 
- **自定义**：实现DataSourceFactory接口，定义数据源的获取方式。 

 **Note实际开发中我们使用Spring管理数据源和，并进行事务控制的配置来覆盖上述配置**
###	databaseIdProvider环境
MyBatis 可以根据不同的数据库厂商执行不同的语句。
**Type： DB_VENDOR** 
 使用MyBatis提供的VendorDatabaseIdProvider解析数据库厂商标识。也可以实现DatabaseIdProvider接口来自定义。 
 * Property-name：数据库厂商标识 
 * Property-value：为标识起一个别名，方便SQL语句使用databaseId属性引用
```javascript
<!-- 全局配置文件-->
<databaseIdProvider type= "DB_ VENDOR">
	<property name= "MySQL " value= "mysqL "/>
	<property name= "Oracle” value= "oracle "/ >
	<property name= "SQL Server" value= "sqlserver"/>
</databaseIdProvider>
<!--mapper.xml -->
<select id= "getEmpsByDeptId" resultType=”EmpparameterType= "Integer" databaseId= "mysqL " >
	SELECT * FROM tbl_ emp WHERE deptId=#{id}
</select >
```
####	MyBatis匹配规则如下： 
 - 如果没有配置databaseIdProvider标签，那么databaseId=null 
- 如果配置了databaseIdProvider标签，使用标签配置的name去匹配数据库信息，匹配上设置databaseId=配置指定的值，否则依旧为null 
- 如果databaseId不为null，他只会找到配置databaseId的sql语句 
- MyBatis 会加载不带 databaseId 属性和带有匹配当前数据库databaseId 属性的所有语句。如果同时找到带有 databaseId 和不带databaseId 的相同语句，则后者会被舍弃。
###	mapper
mapper逐个注册SQL映射文件 或者使用批量注册：这种方式要求SQL映射文件名必须和接口名相同并且在同一目录下
```javascript
	<mappers>
		<mapper class= "zjut. dao. UserMapper”
		resource= "classpath : mappers/UserMapper .xmL"
		<!--批量注最推荐使用需要xml文件和接口文件在同一个目录下--> / >
		<package name= "zjut . dao"/>
	</mappers>
```
#	MyBatis-映射文件（mapper.xml）
映射文件指导着MyBatis如何进行数据库增删改查，有着非常重要的意义；
- cache –命名空间的二级缓存配置 
- cache-ref – 其他命名空间缓存配置的引用。 
- resultMap – 自定义结果集映射 
- parameterMap – 已废弃！老式风格的参数映射 
- sql –抽取可重用语句块。 
- insert – 映射插入语句 
- update – 映射更新语句 
- delete – 映射删除语句 
- select -映射查询语句
##	insert、update、delete元素
**元素属性表**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020052022213469.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxNzQyMTQzNzk3,size_16,color_FFFFFF,t_70#pic_center)
若数据库支持自动生成主键的字段（比如 MySQL 和 SQL Server），则可以设置useGeneratedKeys=”true”，然后再把keyProperty 设置到目标属性上。
```javascript
	<insert id= "insertCustomer" databaseId= "mysql useGeneratedKeys= "true" keyProperty= "id">
	INSERT INTO customers2 ( last_ name, email, age) VALUES (#{lastName}, #{email}, #{age})
	</insert>
```
而对于不支持自增型主键的数据库（例如Oracle），则可以使用 selectKey 子元素：selectKey 元素将会首先运行，id 会被设置，然后插入语句会被调用
```javascript
	<insert id= "insertCustomer" databaseId= "oracle
	parameterType= "cus tomer">
		<selectKey order= "BEFORE" keyProperty= "id" resultType="_ int ">
		SELECT crm_ seq . nextval FROM dual
		</selectKey>
		INSERT INTO customers2 (id, 1ast_ name, email, age)	VALUES (#{id}, #{lastName}, #{email}, #{age})
	</insert>
```
###	selectKey	
**元素属性表**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200520222434420.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxNzQyMTQzNzk3,size_16,color_FFFFFF,t_70#pic_center)
##	参数（Parameters）传递
###	接口参数
**单个参数**
	 可以接受基本类型，对象类型，集合类型的值。这种情况MyBatis可直接使用这个参数，不需要经过任何处理。
 **多个参数**
	 任意多个参数，都会被MyBatis重新包装成一个Map传入。Map的key是param1，param2，0，1…，值就是参数的值。 
	 **命名参数**
	 为参数使用@Param起一个名字，MyBatis就会将这些参数封装进map中，key就是我们自己指定的名字 
	**POJO**
	当这些参数属于我们业务POJO时，我们直接传递POJO 
	**Map**
	我们也可以封装多个参数为map，直接传递
###	配置文件参数
-  #{key}：获取参数的值，预编译到SQL中。安全。 
 - ${key}：获取参数的值，拼接到SQL中。有SQL注入问题。ORDER BY ${name}
 ####	 参数位置支持的属性 
 - javaType、jdbcType：给参数指定数据类型
- Mode：IN|OUT | INOUT 
	如果参数为 OUT 或 INOUT，参数对象属性的真实值将会被改变，就像你在获取输出参数时所期望的那样。如果 mode 为 OUT（或 INOUT），而且 jdbcType 为 CURSOR(也就是 Oracle 的 REFCURSOR)，你必须指定一个 resultMap 来映射结果集到参数类型。要注意这里的 javaType 属性是可选的，如果左边的空白是 jdbcType 的 CURSOR 类型，它会自动地被设置为结果集。
- numericScale：对于数值类型,用来确定小数点后保留的位数
- resultMap：将返回的参数封装到这个resultMap指定的类中
- typeHandler：类型转换器,当javatype和jdbctype无法直接转换时需要设置。
- jdbcTypeName
- expression  

**Note:需要为可能为空的列名指定 jdbcType ，比如在插入数据时，int类型的列id是可能存在null这个情况的那么这个参数位置就要写成#{Id,jdbcType=INTEGER}。尽管所有这些强大的选项很多时候你只简单指定属性名，其他的事情 MyBatis 会自己去推断，最多你需要为可能为空的列名指定 jdbcType。**	
##	select元素
**元素属性表**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200520223459595.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxNzQyMTQzNzk3,size_16,color_FFFFFF,t_70#pic_center)
##	resultMap 
###	id & result
Id 和 result 映射一个单独列的值到简单数据类型(字符串,整型,双精度浮点数,日期等)的属性或字段
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200520223849221.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxNzQyMTQzNzk3,size_16,color_FFFFFF,t_70#pic_center)
###	Association
复杂对象映射 POJO中的属性可能会是一个对象 我们可以使用联合查询，并以级联属性的方式封装对象。**使用association标签定义对象的封装规则**
```javascript
	<resultMap id="ResultMapWithDept" type="zjut.po.Emp">
    <id column="emp_id" jdbcType="INTEGER" property="empId" />
    <result column="emp_name" jdbcType="VARCHAR" property="empName" />
    <association property="dept" javaType="zjut.po.Dept" column = "d_id">
    	<id column="dept_id" property="deptId"/>
    	<result column="dept_name" property="deptName"/>
    </association>
```
**association-分段查询&延迟加载**
```javascript
<association property="Dept" javaType="zjut.po.Dept"
    	select="zjut.dao.DeptMapper.selectByPrimaryKey" column = "d_id">
    </association>
```
	开启延迟加载和属性按需加载
	<settings>
		<!-- 实现延迟加载 -->
		<setting name="lazyLoadingEnabled" value="true"/>
		<setting name="aggressiveLazyLoading" value="false"/>
	</settings>
###		Collection-集合类型&嵌套结果集
**使用Collection标签定义集合的封装规则**
```javascript
	<resultMap type= "zjut.po. Department" id= "MyDept "
	<id column="d_ id" property= "id"/>
	<result column= "d_ deptName”property= "deptName "/ >
	<collection property= "emps”ofType= "zjut.po.Employee "columnPrefix="e_ ">
		<id column= "id" property= "id"/>
		<result column= "LastName”property= "LastName "/>
		<result column= "email” property="email "/>
		<result column= "gender" property= "gender"/>
	</collection>
	</resultMap>
```
**分段查询**
```javascript
<collection property= "emps”
	select= "zjut.dao.EmployeeMapper.getEmpsByDeptId" column= "id">
</collection>
```
**注意：association或者collection标签的，fetchType=eager/lazy可以覆盖全局的延迟加载策略，指定立即加载（eager）或者延迟加载（lazy）。
分步查询的时候通过column指定，将对应的列的数据传递过去，我们有时需要传递多列数据。可以使用**

#	MyBatis-动态SQL 
MyBatis 采用功能强大的基于 OGNL 的表达式来简化操作。

##	foreach
foreach 标签主要用于构建 in 条件，可在 sql 中对集合进行迭代。也常用到批量删除、添加等操作中。
```javascript
	<foreach close=")" collection=” cri terion. value'
	item= "ListItem"	open= (”separator= ,">
		#{listItem}
	</foreach>
```
-	collection：collection 属性的值有三个分别是 list、array、map 三种，分别对应的参数类型为：List、数组、map 集合。
-	item ：表示在迭代过程中每一个元素的别名
-	index ：表示在迭代过程中每次迭代到的位置（下标）
-	open ：前缀
-	close ：后缀
+	separator ：分隔符，表示迭代时每个元素之间以什么分隔

##	choose 标签
类似于java的switch，按顺序判断when标签内的条件，如果成立则结束，否则执行otherwise内的条件，如果没有otherwise标签，就什么也不执行
```javascript
	<choose>
       <when test="criterion.noValue">
           and ${criterion.condition}
       </when>
       <when test="criterion.singleValue">
           and ${criterion.condition} #{criterion.value}
      </when>
       <when test="criterion.betweenValue">
           and ${criterion.condition} #{criterion.value} and #{criterion.secondValue}
       </when>
       <when test="criterion.listValue">
           and ${criterion.condition}
         <foreach close=")" collection="criterion.value" item="listItem" open="(" separator=",">
             #{listItem}
         </foreach>
      </when>
   </choose>
```
##	If&Where&Trim
- if就只有一个内部属性test，内部存放着判断条件
- Where为动态sql语句添加Where关键字，在内部可以添加where的判定条件。
###	trim
主要应用于where和set标签的内部拼接（覆盖前缀或者后缀的无用语句）
**内部属性：**
- prefix：在trim标签内sql语句加上前缀
- suffix：在trim标签内sql语句加上后缀
- prefixOverrides：指定去除多余的前缀内容，如：prefixOverrides=“AND | OR”，去除trim标签内sql语句多余的前缀"and"或者"or"。
- suffixOverrides：指定去除多余的后缀内容。
```javascript
 <where>
      <foreach collection="oredCriteria" item="criteria" separator="or">
        <if test="criteria.valid">
          <trim prefix="(" prefixOverrides="and" suffix=")">
            <foreach collection="criteria.criteria" item="criterion">
              <choose>
                <when test="criterion.noValue">
                  and ${criterion.condition}
                </when>
                <when test="criterion.singleValue">
                  and ${criterion.condition} #{criterion.value}
                </when>
                <when test="criterion.betweenValue">
                  and ${criterion.condition} #{criterion.value} and #{criterion.secondValue}
                </when>
                <when test="criterion.listValue">
                  and ${criterion.condition}
                  <foreach close=")" collection="criterion.value" item="listItem" open="(" separator=",">
                    #{listItem}
                  </foreach>
                </when>
              </choose>
            </foreach>
          </trim>
        </if>
      </foreach>
    </where>
```
##	Set
类似于where，是作为update的关键字填充。
```javascript
 <update id="updateByExampleSelective" parameterType="map">
    update emp
    <set>
      <if test="record.empId != null">
        emp_id = #{record.empId,jdbcType=INTEGER},
      </if>
      <if test="record.empName != null">
        emp_name = #{record.empName,jdbcType=VARCHAR},
      </if>
      <if test="record.gender != null">
        gender = #{record.gender,jdbcType=CHAR},
      </if>
      <if test="record.email != null">
        email = #{record.email,jdbcType=VARCHAR},
      </if>
      <if test="record.dId != null">
        d_id = #{record.dId,jdbcType=INTEGER},
      </if>
    </set>
    <if test="_parameter != null">
      <include refid="Update_By_Example_Where_Clause" />
    </if>
  </update>
```
##	Bind
bind 元素可以从 OGNL 表达式中创建一个变量并将其绑定到上下文。
```javascript
	<select id= "getEmpByLas tNameL ike" resultType= "zjut.po. Employee">
		<bind name= "myLastName" value="'%'+_ LastName+ '%'"/>
		select * from employee where last_ name like #{ myLastName}
	</select>
```
##	不同的数据库厂商的不同语句支持
若在 mybatis 配置文件中配置了 databaseIdProvider , 则可以使用 “_databaseId”变量，这样就可以根据不同的数据库厂商构建特定的语句
```javascript
	<select id= "getEmpPage" resultType= "zjut.po. Employee">
		<if test=”databaseId== 'mysql '">
			select * from employee where limit 日,5
		</if>
		<if test=”databaseId== 'oracle ">
			select * from (select e.*, rownum as r1 from employee e where rownum &1t;= 5)
		where r1 &gt;= 1;
		</if>
	</select>
```
#	MyBatis-缓存机制
MyBatis系统中默认定义了两级缓存。 
- 默认情况下，只有一级缓存（SqlSession级别的缓存，也称为本地缓存）开启。 
- 二级缓存需要手动开启和配置，他是基于namespace级别的缓存。 
- 为了提高扩展性。MyBatis定义了缓存接口Cache。我们可以通过实现Cache接口来自定义二级缓存
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200520230738162.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxNzQyMTQzNzk3,size_16,color_FFFFFF,t_70#pic_center)
##	一级缓存
- 同一次会话期间只要查询过的数据都会保存在当前SqlSession的一个Map中
- 一级缓存(local cache), 即本地缓存, 作用域默认为sqlSession。**当 Session flush 或 close 后, 该Session 中的所有 Cache 将被清空。**
- 本地缓存不能被关闭, 但可以调用 clearCache()来清空本地缓存, 或者改变缓存的作用域. 
- **在mybatis3.1之后, 可以配置本地缓存的作用域，在 全局配置文件中配置。默认值为SESSION ,这种情况下会缓存一个会话中执行的所有查询若设置值为STATEMENT ,本地会话仅用在语句执行上,对相同SqlSession的不同调用将不会共享数据。**

###	一级缓存失效的四种情况

## 合理的创建标题，有助于目录的生成
- 不同的SqlSession对应不同的一级缓存 
- 同一个SqlSession但是查询条件不同 
- 同一个SqlSession两次查询期间执行了任何一次增删改操作 
- 同一个SqlSession两次查询期间手动清空了缓存
##	二级缓存
- 二级缓存(second level cache)，全局作用域缓存 
- 二级缓存默认不开启，需要手动配置 
- MyBatis提供二级缓存的接口以及实现，缓存实现要求POJO实现Serializable接口 
 - 二级缓存在 SqlSession 关闭或提交之后才会生效
###	使用步骤 
- 全局配置文件中开启二级缓存 <setting name="cacheEnabled" value="true"/> 
- 需要使用二级缓存的映射文件处使用cache配置缓存 <cache /> 
+ 注意：POJO需要实现Serializable接口

##	缓存相关属性
- eviction=“FIFO”：缓存回收策略：
 	*  LRU – 最近最少使用的：移除最长时间不被使用的对象。 
 	* FIFO – 先进先出：按对象进入缓存的顺序来移除它们。 
 	* SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象。 
	*  WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
默认的是 LRU。 
- flushInterval：刷新间隔，单位毫秒 
 默认情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句时刷新 
- size：引用数目，正整数 
 	代表缓存最多可以存储多少个对象，太大容易导致内存溢出 
- readOnly：只读，true/false 
 	* true：只读缓存；会给所有调用者返回缓存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势。 
	*  false：读写缓存；会返回缓存对象的拷贝（通过序列化）。这会慢一些，

直接输入1次<kbd>#</kbd>，并按下<kbd>space</kbd>后，将生成1级标题。
输入2次<kbd>#</kbd>，并按下<kbd>space</kbd>后，将生成2级标题。
以此类推，我们支持6级标题。有助于使用`TOC`语法后生成一个完美的目录。

##	缓存有关设置
* 全局setting的cacheEnable： 
	– 配置二级缓存的开关。一级缓存一直是打开的。 
* select标签的useCache属性： 
– 配置这个select是否使用二级缓存。一级缓存一直是使用的 
- sql标签的flushCache属性： 
– 增删改默认flushCache=true。sql执行以后，会同时清空一级和二级缓存。查询默认flushCache=false。 
- sqlSession.clearCache()： 
– 只是用来清除一级缓存。 
- 当在某一个作用域 (一级缓存Session/二级缓存Namespaces) 进行了 C/U/D 操作后，默认该作用域下所有 select 中的缓存将被clear

#	MyBatis-工作原理
##	总工作流程图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200520231909919.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxNzQyMTQzNzk3,size_16,color_FFFFFF,t_70#pic_center)
##	查询工作流程顺序图
**1、根据配置文件创建SQLSessionFactory**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200520232043934.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxNzQyMTQzNzk3,size_16,color_FFFFFF,t_70#pic_center)
**总结：把配置文件的信息解析并保存在Configuration对象中，返回包含了Configuration的DefaultSqlSession对象，其中Configuration封装了所有配置文件的信息包含mapper.xml。**

**2、返回SqlSession的实现类DefaultSqlSession对象。他里面包含了Executor和Configuration；Executor会在这一步被创建**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200520232216739.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxNzQyMTQzNzk3,size_16,color_FFFFFF,t_70#pic_center)
**3、getMapper返回接口的代理对象包含了SqlSession对象**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020052023224093.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxNzQyMTQzNzk3,size_16,color_FFFFFF,t_70#pic_center)
**4、查询流程**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020052023231186.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxNzQyMTQzNzk3,size_16,color_FFFFFF,t_70#pic_center)
**5、查询流程总结**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200520232513945.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxNzQyMTQzNzk3,size_16,color_FFFFFF,t_70#pic_center)
- StatementHandler：处理sql语句预编译，设置参数等相关工作
* ParameterHandler：设置预编译参数用的
* ResultHandler：处理结果集
* TypeHandler：在整个过程中，进行数据库类型和javaBean类型的映射

#	插件开发
MyBatis 允许在已映射语句执行过程中的某一点进行拦截调用。
默认情况下，MyBatis 允许使用插件来拦截的方法调用包括:对象（方法名）
- Executor (update, query, flushStatements, commit, rollback,getTransaction, close, isClosed) 
- ParameterHandler (getParameterObject, setParameters) 
- ResultSetHandler (handleResultSets, handleOutputParameters) 
- StatementHandler (prepare, parameterize, batch, update, query) 

##	插件开发步骤 
##	插件原理
- 按照插件注解声明，按照插件配置顺序调用插件plugin方法，生成被拦截对象的动态代理 
- 多个插件依次生成目标对象的代理对象，层层包裹，先声明的先包裹；形成代理链 
* 目标方法执行时依次从外到内执行插件的intercept方法。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200520233057178.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxNzQyMTQzNzk3,size_16,color_FFFFFF,t_70#pic_center)
- 多个插件情况下，我们往往需要在某个插件中分离出目标对象。可以借助MyBatis提供的SystemMetaObject类来进行获取最后一层的h以及target属性的值


**注：本文章很多图片和资料摘自网络资源，侵权请联系我删除，谢谢。**