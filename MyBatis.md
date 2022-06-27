# mybatis

Mapper接口开发需要遵循以下规范： 

1. Mapper.xml文件中的namespace与mapper接口的类路径相同。 

2. Mapper接口方法名和Mapper.xml中定义的每个statement的id相同 

3. Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql 的parameterType的类型相同 

4. Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同

## xml配置

### select 

![1645344942668](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645344942668.png)



![1645344979125](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645344979125.png)



### 参数传递

![1645345027941](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645345027941.png)



### insert

![1645345076706](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645345076706.png)



### 主键回填

![1645345106306](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645345106306.png)

```xml
<!--   主键回填
		useGeneratedKeys="true"   启用主键回填机制
		keyProperty="sid"   将得到的主键值赋给 对象的哪个属性中
	 -->
	
	<insert id="addStudent" parameterType="Student" useGeneratedKeys="true" 			                            keyProperty="sid">
	
		insert into student (sname,birthday,ssex,classid) 
		values(#{sname},#{birthday},#{ssex},#{classid})
	
	</insert>
```

### 自定义主键生成规则

![1645345179875](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645345179875.png)



### 结果集映射

![1645345343485](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645345343485.png)

![1645345399313](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645345399313.png)

```xml
<insert id="addSM" parameterType="SMaster">
		insert into schoolmaster (sm_name,smsex)
		values(#{smname},#{smsex})
	</insert>
	
	
	<resultMap type="SMaster" id="smasterMap">
	<!-- 单表情况下，只写映射不上的 -->
		<!-- id  主键 -->
		<!-- <id column="smid" property="smid"/> -->
		<result column="sm_name" property="smname"/>
		<!-- <result column="smsex" property="smsex"/> -->
	</resultMap>
	
	<select id="findAllSM" resultMap="smasterMap">
		select smid,sm_name,smsex from schoolmaster
	</select>
```



### 一对一级联映射

![1645345534712](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645345534712.png)

```xml
<!--  多表映射全部字段都要写 -->
	<resultMap type="Student" id="Student_Banji_Map">
		<result column="sid" property="sid"/>
		<result column="sname" property="sname"/>
		<result column="birthday" property="birthday"/>
		<result column="ssex" property="ssex"/>
		<result column="classid" property="classid"/>
		<!-- 一对一的映射 -->
		<association property="bj">
			<result column="classid" property="classid"/>
			<result column="classname" property="classname"/>
		</association>
	</resultMap>

	<select id="findAllStudent" resultMap="Student_Banji_Map">	
		select * from student 
		left join class on student.classid = class.classid
	</select>
```



### 一对多级联

![1645345569507](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645345569507.png)

```xml
<resultMap type="Banji" id="Banji_Student_Map">
		<result column="classid" property="classid"/>
		<result column="classname" property="classname"/>
		
		<!-- 一对多 -->
		<collection property="slist" ofType="Student" >
			<result column="sid" property="sid"/>
			<result column="sname" property="sname"/>
			<result column="birthday" property="birthday"/>
		</collection>
	
	</resultMap>
	
	<select id="findAllBanji" resultMap="Banji_Student_Map">
		select * from  class left join student on student.classid = class.classid
	</select>
```



## xml动态sql

### if

![1645346240377](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645346240377.png)



### choose

![1645346335189](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645346335189.png)



### where

![1645346410245](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645346410245.png)



### set

![1645346445267](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645346445267.png)

### trim

![1645346540300](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645346540300.png)

​		prefix 表示 前面添加一个什么
​		prefixOverrides 表示 前面去掉一个什么
​		suffix 表示 后台添加一个什么
​		suffixOverrides 表示 后面去掉一个什么





### foreache

![1645346569633](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645346569633.png)

![1645346588784](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645346588784.png)



### bind

![1645346621063](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645346621063.png)

![1645346674903](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645346674903.png)

```xml
<!--  sql代码片段 -->
	<sql id="stusql">
		select 	sid, sname, birthday, ssex, classid from student
	</sql>

	<!--  if 标签 -->
	<select id="findAllStudent" parameterType="Student" resultType="Student">
		<!-- 
			OGNL 语言
			if 标签  test 判断拼接和不拼接的表达式  true 拼  false  不拼
			where 标签 1. 当where标签中什么都没有的情况下 where标签就当不存在
					   2. 当where标签中有条件 自动添加一个 WHERE 关键字
					   3. 自动去掉遇到的第一个 and 或者 or
		 -->
	
		<include refid="stusql"></include>
		<where>
			<if test="ssex != null">
				and	ssex = #{ssex} 
			</if>
			<if test="classid != 0" >
			 	and	classid = #{classid}
			</if>
		</where> 
	</select>
	
	
	<!-- choose 标签 -->
	<select id="findStudentChoose" parameterType="Student" resultType="Student">
		
		<include refid="stusql"></include>
		<where>
			<choose>
				<when test="sid != 0">
					and sid = #{sid}
				</when>
				<when test="sname != null">
					and sname = #{sname}
				</when>
				<when test="ssex != null">
					and ssex = #{ssex}
				</when>
			</choose>
		</where>
	</select>
	
	<!-- set 标签
					1. 添加一个 SET 关键字
					2. 去掉最后一个,(逗号)
	 -->
	<update id="eidtStudent" parameterType="Student">
		update student 
		<set>
			<if test="sname != null">sname = #{sname},</if>
			<if test="ssex != null">ssex = #{ssex},</if>
			<if test="birthday != null"> birthday = #{birthday},</if>
			<if test="classid != 0">classid = #{classid},</if>
		</set>
		 where sid = #{sid}
	</update>
	
	
	
	<!-- trim 
		prefix 表示 前面添加一个什么
		prefixOverrides 表示 前面去掉一个什么
		suffix 表示 后台添加一个什么
		suffixOverrides 表示 后面去掉一个什么
	
	代替查询的使用 -->
	<select id="findStudentTrim" resultType="Student" parameterType="Student">
		<include refid="stusql"></include>
		<trim prefix = "where " prefixOverrides="and">
			<if test="ssex != null">
			 and	ssex = #{ssex}
			</if>
			<if test="classid != 0">
			and		classid = #{classid}
			</if>
		</trim>
	</select>
	
	<!--  添加 经常使用trim 标签 -->
	
	<insert id="addStudentTrim" parameterType="Student">
		insert into student 
		<trim prefix="(" suffix=")" suffixOverrides=",">
			<if test="sname != null">sname,</if>
			<if test="birthday != null">birthday,</if>
			<if test="ssex != null">ssex,</if>
			<if test="classid != 0">classid,</if>
		</trim>
		values
		<trim prefix="(" suffix=")" suffixOverrides=",">
			<if test="sname != null">#{sname},</if>
			<if test="birthday != null">#{birthday},</if>
			<if test="ssex != null">#{ssex},</if>
			<if test="classid != 0">#{classid},</if>
		</trim>
	</insert>
	
	<!-- 数组 foreach -->
	<select id="findStudentArray" resultType="Student" >
		<include refid="stusql"></include>
		<where>
			and sid
			<foreach collection="array" item="x" open=" in (" close=")" separator=",">
				#{x}
			</foreach>
		</where>
	</select>
	
	<!-- 集合 foreach -->
	<select id="findStudentList" resultType="Student">
		<include refid="stusql"></include>
		<where>
			and sid in
			<foreach collection="list" item="y" open="(" close=")" separator=",">
				#{y}
			</foreach>
		</where>
	</select>
	
	
	<select id="findStudentLike" parameterType="String" resultType="Student">
		<!-- 
		 1. java 逻辑中进行模糊字符串的处理 可
		select * from student where sname like #{v} -->
		
		<!-- 
		 2. 通过函数进行字符串拼接  可
		select * from student where sname like concat('%',#{v},'%') -->
		
		<!-- 
		 3. ${} 字符串的替换，不能防止sql注入  不行
		select * from student where sname like '%${v}%' -->
		
		<!-- 
		 4. 使用 "" 方案  可
		select * from student where sname like "%"#{v}"%" -->
		
		<!-- mybatis 推荐 -->
		<bind name="likemingzi" value="'%'+_parameter+'%'"/>
		select * from student where sname like #{likemingzi}
		
	</select>
```



## 注解实现

### 注解方式实现增删改查

```java
// 新增的语句映射
	@Insert("insert into student (sname,birthday,ssex,classid) "
			+ " values(#{sname},#{birthday},#{ssex},#{classid})")
	@Options(useGeneratedKeys = true, keyProperty = "sid")//主键回填
	public int  addStudent(Student s);
	
	// 删除的语句映射
	@Delete("delete from student where sid = #{v}")
	public int delStudent(int sid);
	
	// 修改的语句映射
	@Update("update student set sname=#{sname},birthday=#{birthday},"
			+ "ssex = #{ssex}, classid = #{classid} where sid= #{sid}")
	public int updateStudent(Student s);
	
	
	// 全部的学生信息
	@Select("select * from student")
	public List<Student> findAllStudent();
	
	// 根据sid 查询学生
	@Select("select * from student where sid = #{v}")
	public Student findStudentBysid(int sid);
	
	// 对学生个数的统计
	
	//多参
	// 1. javaBean
	// 2. Map
	// 3. @Param
	@Select("select * from student where ssex = #{xingbie} and classid = #{xxx}")
	public List<Student> findStudentBySexAndClassid(@Param("xingbie") String sex, 
			@Param("xxx") int banjiid);
	
	
```



### 主键回填

![1645347197509](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645347197509.png)

### 主键自增

![1645347242875](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645347242875.png)

### 参数传递

![1645347280528](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645347280528.png)

### @Results结果集映射

![1645347346972](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645347346972.png)



![1645347431051](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645347431051.png)

```java
	@Results(id = "smMap",value = {
		@Result(column = "smid",property = "smid"),
		@Result(column = "sm_name",property = "smname"),
		@Result(column = "smsex",property = "smsex")
	})
	@Select("select * from schoolmaster")
	public List<SMaster> findAllSmaster();
	

	@Select("select * from schoolmaster where smid = #{v}")
	@ResultMap("smMap")//复用
	public SMaster findSmasterBysmid(int smid);
```



### 一对一映射

![1645347455473](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645347455473.png)

![1645347498573](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645347498573.png)

```java
public class Student {
	private int sid;
	private String sname;
	private Date birthday;
	private String ssex;
	private int classid;
	// 外键属性
	private Banji bj;
}
public class Banji {
	private int classid;
	private String classname;
	// 外键属性
	private List<Student> stulist;
}

	@Results({
		@Result(column = "classid", property = "classid"),
		@Result(property = "bj", column = "classid",
				one = @One(select = "com.ape.mapper.BanjiMapper.findBanjiByclassid")
				)
	})
	@Select("select * from student")
	public List<Student> findAllStudent();
	
	
```



### 一对多映射

![1645347578843](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645347578843.png)

![1645347612834](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645347612834.png)

```java
public class Student {
	private int sid;
	private String sname;
	private Date birthday;
	private String ssex;
	private int classid;
	// 外键属性
	private Banji bj;
}
public class Banji {
	private int classid;
	private String classname;
	// 外键属性
	private List<Student> stulist;
}

	
	@Results(id = "class_stu_Map",value = {
		@Result(column = "classid",property = "classid"),
		@Result(property = "stulist",column = "classid",
			many = @Many(select ="com.ape.mapper.StudentMapper.findStudentByClassid"))
	})
	@Select("select * from class")
	public List<Banji> findAllBanji();
```



## 注解实现动态sql

## 

### 1、脚本动态sql

![1645348473649](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645348473649.png)

```java
// 方式1 ：使用脚本
	// 查询学生
	@Select("<script>"
			+ "select * from student\r\n"
			+ "		<where>\r\n"
			+ "			<choose>\r\n"
			+ "				<when test=\"sid != 0\">\r\n"
			+ "					and sid = #{sid}\r\n"
			+ "				</when>\r\n"
			+ "				<when test=\"sname != null\">\r\n"
			+ "					and sname = #{sname}\r\n"
			+ "				</when>\r\n"
			+ "				<when test=\"ssex != null\">\r\n"
			+ "					and ssex = #{ssex}\r\n"
			+ "				</when>\r\n"
			+ "			</choose>\r\n"
			+ "		</where>"
			+ "</script>")
	public List<Student> findStudentScript(Student s);
```



### 2、方法中构建动态sql

![1645348670640](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645348670640.png)

```java
// 方式2：方法构建sql语句
	@SelectProvider(type = StuSql.class,method = "funcStusql")
	public List<Student> findStudentFunction(Student s);
	
	class StuSql{//内部类
		public String funcStusql(Student s) {
			String sql = "select * from student where 1=1 ";
			if(s.getClassid() != 0) {
				sql += " and classid = #{classid}";
			}
			if(s.getSsex() != null) {
				sql += " and ssex = #{ssex}";
			}
			return sql;
		}
```



### 3、SQL 语句构造器

![1645348719545](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645348719545.png)

![1645348766936](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645348766936.png)

```java
// 方式3： sql构造器
	//查询
	@SelectProvider(type = StuSql.class,method = "gzfindStudentSql")
	public List<Student> findStudentGZQ(Student s);
	//新增
	@InsertProvider(type = StuSql.class,method = "gzaddStudentSql")
	public int addStudentGZQ(Student s);
	//修改
	@UpdateProvider(type = StuSql.class,method = "gzupdateSTudentSql")
	public int editStudentGZQ(Student s);
	//删除
	@DeleteProvider(type = StuSql.class,method = "gzdelStudentSql")
	public int delStudentGZQ(int sid);
	
	class StuSql{
		// 构造器查询
		public String gzfindStudentSql(Student s) {
			return new SQL() {
				{
					// SELECT 查询字段名
					SELECT("sid");
					SELECT("sname");
					SELECT("birthday,ssex,classid");
					// FROM 表名
					FROM("student");
					if(s.getSsex() != null) {
						WHERE("ssex = #{ssex}");
					}
					if(s.getClassid() != 0) {
						WHERE("classid = #{classid}");
					}
				}
			}.toString();
		}

		// 构造器新增
		public String gzaddStudentSql(Student s) {
			return new SQL() {
				{
					INSERT_INTO("Student");
					if(s.getSname() != null) {
						VALUES("sname","#{sname}");
					}
					if(s.getBirthday() != null) {
						VALUES("birthday","#{birthday}");
					}
					if(s.getSsex() != null) {
						VALUES("ssex","#{ssex}");
					}
					if(s.getClassid() != 0) {
						VALUES("classid","#{classid}");
					}
				}
			}.toString();
		}
		
		// 构造器修改
		public String gzupdateSTudentSql(Student s) {
			return new SQL() {
				{
					UPDATE("Student");
					if(s.getSname() != null) {
						SET("sname = #{sname}");
					}
					if(s.getBirthday() != null) {
						SET("birthday = #{birthday}");
					}
					if(s.getSsex() != null) {
						SET("ssex = #{ssex}");
					}
					if(s.getClassid() != 0) {
						SET("classid = #{classid}");
					}
					WHERE("sid = #{sid}");
					
				}
			}.toString();
		}
		
		// 构造器删除
		public String gzdelStudentSql(int sid) {
			return new SQL() {
				{
					DELETE_FROM("Student");
					WHERE("sid = #{v}");
				}
				
			}.toString();
		}
		
	}
	
```



## 分页

![1645349169846](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645349169846.png)

![1645349187209](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645349187209.png)



## 延 迟 加 载 和 立 即 加 载

立即加载是： 不管用不用信息，只要调用，马上发起查询并进行加载 

比如： 当我们查询学生信息时，就需要知道学生在哪个班级中，所以就需要立马去查询班级的信息 

通常：当 一对一或者 多对一的时候需要立即加载



延迟加载是： 在真正使用数据时才发起查询，不用的时候不查询，按需加载（也叫 懒加载） 

比如： 

在查询班级信息，每个班级都会有很多的学生（假如每个班有100个学生），如果我们只是查看 

班级信息，但是学生对象也会加载到内存中，会造成浪费。 

所以我门需要进行懒加载，当确实需要查看班级中的学生信息，我门在进行加载班级中的学生信息。 

通常： 一对多，或者多对多的是需要使用延迟加载

![1645349413840](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645349413840.png)

```java
// 1.传统方式 limit
	@Select("select * from student limit #{xx},#{yy}")
	public List<Student> findStudentPage(@Param("xx") int curpage,
										 @Param("yy") int sizepage);
	
	// 2.RowBounds 分页
	@Select("select * from student")
	public List<Student> findStudentPage2(RowBounds rb);
	
	// 查询所有的学生
	// FetchType.DEFAULT 根据主配置文件的方式来进行加载
	// FetchType.EAGER 立即加载
	// FetchType.LAZY 延迟加载
	@Results({
		@Result(column = "classid",property = "classid"),
		@Result(column = "classid", property = "bj",
			one=@One(
					select = "com.ape.mapper.BanjiMapper.findBanjiByClassid",
					fetchType = FetchType.LAZY
			)
		)
	})
	@Select("select * from student")
	public List<Student> findStudent();
```



## 缓存

缓存（cache），数据交换的缓冲区，当应用程序需要读取数据时，先从数据库中将数据取出，放置在 

缓冲区中，应用程序从缓冲区读取数据。 



### 一级缓存

会话 session 级别的缓存，针对一 次会话操作内

默认开启，一级缓存只是相对同一个 SqlSession 对象而言

注意：当调用sqlsession的修改，添加，删除，commit（），close（） 等方法是， 就会清空一级缓存

### 二级缓存

映射器级别的缓存，针对不同 Namespace 的映射器

一个 namespace 下的所有操作语句，都影响着同一个Cache

![1645349782054](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645349782054.png)

![1645349799987](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1645349799987.png)

```xml
	<settings>
		<setting name="cacheEnabled" value="true"/>
		<setting name="lazyLoadingEnabled" value="true"/>
		<setting name="aggressiveLazyLoading" value="false"/>
	</settings>
```

