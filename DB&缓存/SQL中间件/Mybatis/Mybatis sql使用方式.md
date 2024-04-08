- [整合 Mybatis 和 Spring Boot](#整合-mybatis-和-spring-boot)
  - [整合Mybatis](#整合mybatis)
  - [整合Mybatis-plus](#整合mybatis-plus)
- [执行 SQL 方式一：注解](#执行-sql-方式一注解)
- [执行 SQL 方式二：XML](#执行-sql-方式二xml)
- [执行 SQL 方式三：MyBatis-Plus](#执行-sql-方式三mybatis-plus)
- [BaseMapper \& Iservice \& ServiceImpl](#basemapper--iservice--serviceimpl)

# 整合 Mybatis 和 Spring Boot

## 整合Mybatis

第一步，在 pom.xml 文件中引入 starter

```XML
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.2</version>
</dependency>
```

第二步，在 application.yml 文件中添加数据库连接配置。

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: Huicheng123**
    url: jdbc:mysql://localhost:3306/codingmore-mybatis?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai&useSSL=false
```


## 整合Mybatis-plus

在 pom.xml 文件中引入 starter

```XML
 <dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.6</version>
</dependency>
```

# 执行 SQL 方式一：注解

```Java
public interface UserMapper {
    @Select("SELECT * FROM user")
    List<User> getAll();

    @Select("SELECT * FROM user WHERE id = #{id}")
    User getOne(Integer id);

    @Insert("INSERT INTO user(name,password,age) VALUES(#{name}, #{password}, #{age})")
    void insert(User user);

    @Update("UPDATE user SET name=#{name},password=#{password},age=#{age} WHERE id =#{id}")
    void update(User user);

    @Delete("DELETE FROM user WHERE id =#{id}")
    void delete(Integer id);
}

@SpringBootTest
@Slf4j
class CodingmoreMybatisApplicationTests {

	@Autowired
	private UserMapper userMapper;

	@Test
	void testInsert() {
		userMapper.insert(User.builder().age(18).name("沉默王二").password("123456").build());
		userMapper.insert(User.builder().age(18).name("沉默王三").password("123456").build());
		userMapper.insert(User.builder().age(18).name("沉默王四").password("123456").build());
		log.info("查询所有：{}",userMapper.getAll().stream().toArray());
	}

	@Test
	List<User> testQuery() {
		List<User> all = userMapper.getAll();
		log.info("查询所有：{}",all.stream().toArray());
		return all;
	}

	@Test
	void testUpdate() {
		User one = userMapper.getOne(1);
		log.info("更新前{}", one);
		one.setPassword("654321");
		userMapper.update(one);
		log.info("更新后{}", userMapper.getOne(1));
	}

	@Test
	void testDelete() {
		log.info("删除前{}", userMapper.getAll().toArray());
		userMapper.delete(1);
		log.info("删除后{}", userMapper.getAll().toArray());

	}
}
```

# 执行 SQL 方式二：XML

动态sql： 根据条件拼接 SQL 语句，例如 <if>, <choose>, <where> 等标签 

多表查询

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202406042039142.png)

```Java
public interface PostMapper {
    List<Posts> getAll();
    Posts getOne(Long id);
    void insert(Posts post);
    void update(Posts post);
    void delete(Long id);
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="top.codingmore.mapper.PostMapper">
    <resultMap id="BaseResultMap" type="top.codingmore.entity.Posts">
        <id column="posts_id" property="postsId"/>
        <result column="post_author" property="postAuthor"/>
        <result column="post_content" property="postContent"/>
        <result column="post_title" property="postTitle"/>
    </resultMap>

    <sql id="Base_Column_List">
        posts_id, post_author, post_content, post_title
    </sql>

    <select id="getAll" resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List" />
        from posts;
    </select>

    <select id="getOne" parameterType="java.lang.Long" resultMap="BaseResultMap" >
        SELECT
        <include refid="Base_Column_List" />
        FROM users
        WHERE id = #{id}
    </select>

    <insert id="insert" parameterType="top.codingmore.entity.Posts">
        insert into
            posts
            (post_author,post_content,post_title)
        values
            (#{postAuthor},#{postContent},#{postTitle});
    </insert>
    <update id="update" parameterType="top.codingmore.entity.Posts">
        update
            posts
        set
        <if test="postAuthor != null">post_author=#{postAuthor},</if>
        <if test="postContent != null">post_content=#{postContent},</if>
        post_title=#{postTitle}
        where id=#{id}
    </update>
    <delete id="delete">
        delete from
            posts
        where
            id=#{id}
    </delete>
</mapper>
```

# 执行 SQL 方式三：MyBatis-Plus

> https://baomidou.com/introduce/

- 强大的 CRUD 操作：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- 支持 Lambda 形式调用：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- 内置分页插件：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询

继承了 `BaseMapper` 类，可以直接使用 MyBatis-Plus 提供的 CRUD 方法。

```Java
public interface AgentMapper extends BaseMapper<AgentEntity> {

}
```

# BaseMapper & Iservice & ServiceImpl



![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202406122029880.png)