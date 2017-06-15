#mybatis准备
引用jar文件mybatis-3.4.1.jar
bean-dao-map.xml
通过dbconfig.properties进行数据库配置 乱码等问题这里解决

#mybatis
多数据库支持[<environments>]完成MySQL oracle的环境进行配置
map文件下bean的全类名可以通过<typeAliases>取别名 可以批量注册
bean和数据库的文件可以通过模糊命名来查询

#命名参数
单个参数或者bean类型的参数
直接使用#{name}或者${name}既可
多命名参数：明确指定封装参数时map的key；@Param("id") 在XML文件直接#{id}就可以

map.Java 
public Employee getEmpByIdAndLastName(@Param("id")Integer id,@Param("lastName")String lastName);

map.xml
	<select id="getEmpByMap" resultType="com.atguigu.mybatis.bean.Employee">
    select * from ${tableName} where id=${id} and last_name=#{lastName}
    </select>

#参数值的获取
 #{}：可以获取map中的值或者pojo对象属性的值；
 ${}：可以获取map中的值或者pojo对象属性的值；

	区别：
		#{}:是以预编译的形式，将参数设置到sql语句中；PreparedStatement；防止sql注入
		${}:取出的值直接拼装在sql语句中；会有安全问题；
		大多情况下，我们去参数的值都应该去使用#{}；
		
		原生jdbc不支持占位符的地方我们就可以使用${}进行取值
		比如分表、排序。。。；按照年份分表拆分
			select * from ${year}_salary where xxx;
			select * from tbl_employee order by ${f_name} ${order}

# #{}:更丰富的用法：
	规定参数的一些规则：
	javaType、 jdbcType、 mode（存储过程）、 numericScale、
	resultMap、 typeHandler、 jdbcTypeName、 expression（未来准备支持的功能）；

	jdbcType通常需要在某种特定的条件下被设置：
		在我们数据为null的时候，有些数据库可能不能识别mybatis对null的默认处理。比如Oracle（报错）；
		
		JdbcType OTHER：无效的类型；因为mybatis对所有的null都映射的是原生Jdbc的OTHER类型，oracle不能正确处理;
		
		由于全局配置中：jdbcTypeForNull=OTHER；oracle不支持；两种办法
		1、#{email,jdbcType=OTHER};
		2、jdbcTypeForNull=NULL
			<setting name="jdbcTypeForNull" value="NULL"/>
			
#select查询
返回的是list的情况 resulttype不是list类型 而是具体的bean类型
单个map 对应的是map
封装多个map集合 resulttype也是bean类型 MapKey通过注解的方式设置key的值通过什么来决定
如果不使用模糊命名 可以通过自定义resultmap进行封装

#动态SQL
    JSTL的语法形式？？？
##if标签
    根据动态查询条件经常查询
##where标签
    解决AND和OR的问题  如果AND不写在前面 可以采用trim的形式
##choose标签
    类似于switch-break 通过先后顺序满足进行匹配
##set标签
    通过set和if联合使用 update更新   
##foreach 批量查询和保存

#Cache
##一级缓存(本地缓存)
    sqlSession级别的缓存。一级缓存是一直开启的；SqlSession级别的一个Map
	 * 		与数据库同一次会话期间查询到的数据会放在本地缓存中。
	 * 		以后如果需要获取相同的数据，直接从缓存中拿，没必要再去查询数据库；
	 
##一级缓存失效的情况
     *      1、sqlSession不同。
	 * 		2、sqlSession相同，查询条件不同.(当前一级缓存中还没有这个数据)
	 * 		3、sqlSession相同，两次查询之间执行了增删改操作(这次增删改可能对当前数据有影响)
	 * 		4、sqlSession相同，手动清除了一级缓存（缓存清空）

##二级缓存（全局缓存）
    基于namespace级别的缓存：一个namespace对应一个二级缓存
    工作机制：
	 * 		1、一个会话，查询一条数据，这个数据就会被放在当前会话的一级缓存中；
	 * 		2、如果会话关闭；一级缓存中的数据会被保存到二级缓存中；新的会话查询信息，就可以参照二级缓存中的内容；
	 * 		3、sqlSession===EmployeeMapper==>Employee
	 * 						DepartmentMapper===>Department
	 * 			不同namespace查出的数据会放在自己对应的缓存中（map）
	 * 			效果：数据会从二级缓存中获取
	 * 				查出的数据都会被默认先放在一级缓存中。
	 * 				只有会话提交或者关闭以后，一级缓存中的数据才会转移到二级缓存中：
	 			
##开启二级缓存 二级缓存的参数设置
    在XML文件开启开启二级缓存
    在对应的POJO开启序列化implements Serializable
    如果有命中率为0.5 否则为0

    eviction:缓存的回收策略：
		• LRU – 最近最少使用的：移除最长时间不被使用的对象。
		• FIFO – 先进先出：按对象进入缓存的顺序来移除它们。
		• SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象。
		• WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
		• 默认的是 LRU。
	flushInterval：缓存刷新间隔
		缓存多长时间清空一次，默认不清空，设置一个毫秒值
	readOnly:是否只读：
		true：只读；mybatis认为所有从缓存中获取数据的操作都是只读操作，不会修改数据。
				 mybatis为了加快获取速度，直接就会将数据在缓存中的引用交给用户。不安全，速度快
		false：非只读：mybatis觉得获取的数据可能会被修改。
				mybatis会利用序列化&反序列的技术克隆一份新的数据给你。安全，速度慢
	size：缓存存放多少元素；
	type=""：指定自定义缓存的全类名；
			实现Cache接口即可；
##和缓存有关的设置/属性：
	 * 			1）、cacheEnabled=true：false：关闭缓存（二级缓存关闭）(一级缓存一直可用的)
	 * 			2）、每个select标签都有useCache="true"：
	 * 					false：不使用缓存（一级缓存依然使用，二级缓存不使用）
	 * 			3）、【每个增删改标签的：flushCache="true"：（一级二级都会清除）】
	 * 					增删改执行完成后就会清楚缓存；
	 * 					测试：flushCache="true"：一级缓存就清空了；二级也会被清除；
	 * 					查询标签：flushCache="false"：
	 * 						如果flushCache=true;每次查询之后都会清空缓存；缓存是没有被使用的；
	 * 			4）、sqlSession.clearCache();只是清楚当前session的一级缓存；
	 * 			5）、localCacheScope：本地缓存作用域：（一级缓存SESSION）；当前会话的所有数据保存在会话缓存中；
	 * 								STATEMENT：可以禁用一级缓存；		

##缓存原理

##整合第三方缓存ehcache/redis    
	 *      1）、导入第三方缓存包即可；
	 *		2）、导入与第三方缓存整合的适配包；官方有；
	 *		3）、mapper.xml中使用自定义缓存
	 *		<cache type="org.mybatis.caches.ehcache.EhcacheCache"></cache>

#Mybatis整合spring 
    导入mybatis的包  导入spring的包 导入mybatis-spring的适配包
    为什么要使用数据源？
    
#一对多查询
    Javabean中一对多关系 使用set/list进行一对多关系映射



