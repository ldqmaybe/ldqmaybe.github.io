---
layout:     post
title:      Java后端之路--SpringBoot整合Mybatis-Plus
subtitle:   
date:       2020-05-04
author:     dingqiang.l
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Java
---
SpringBoot整合mybatis-plus，实现简单的用户CRUD操作，直接开始吧。

## 准备环境
 - 系统：Win10 
 - 开发工具：IntellIJ IDEA 2019.3 
 - Java版本：open jdk1.8 
 - 项目管理工具：Maven

## 新建项目
此次以多模块管理，根目录下的pom.xml负责依赖版本管理，Modules需要哪些依赖直接在Module下的pom.xml引入即可，不用再重复管理版本号了。新建好项目后如下图所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020050413444089.png)
这里简单做了些配置，首先将application.properties改为yml格式（个人习惯），并简单配置了mybatis-plus的mapping扫描路径、mysql连接和log日志配置。

```java
#mybatis-plus mapping
mybatis-plus:
  mapper-locations: classpath*:com/hotch/springmybatis/**/xml/*Mapper.xml
#数据库连接
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/hotch?autoReconnect=true&useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=CONVERT_TO_NULL&useSSL=false&serverTimezone=UTC
    username: root
    password: root
#日志
logging:
  config: classpath:logback.xml
  
```

## 新建数据库
这里简单创建个user只包含id,用户名和密码的表
```sql
CREATE TABLE `tb_user`  (
  `id` bigint(8) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '用户主键',
  `username` varchar(47) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '用户登录名',
  `password` varchar(67) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '用户密码',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 6 CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '用户信息' ROW_FORMAT = Dynamic;
```
## 代码生成
有了上面创建好的tb_user表，这里就可以使用mybatis-plus提供的代码生成器替我们快速生成我们需要的Controller、Mapper和Service了

```java
public class EntityGenerator {
    // 生成输出目录，定位到工程的java目录下
    private String outputDir = "D:\\Android\\IdeaWorkspace\\hotch-parent\\spring-mybatis\\src\\main\\java";
    // 生成类的作者
    private String author = "lindq";
    // 数据源相关配置
    private String url = "jdbc:mysql://127.0.0.1:3306/hotch?autoReconnect=true&useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=CONVERT_TO_NULL&useSSL=false&serverTimezone=UTC";
    private String driverName = "com.mysql.cj.jdbc.Driver";
    private String userName = "root";
    private String userPwd = "root";
    // DAO的包路径
    private String daoPackage = "com.hotch.springmybatis.dao";
    // 待生成的表名，注意是覆盖更新
    private static String[] tableNames;

    static {
        tableNames = new String[]{"tb_user"};
    }

    @Test
    public void entityGenerator() {
        AutoGenerator mpg = new AutoGenerator();
        mpg.setTemplateEngine(new BeetlTemplateEngine());
        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        gc.setOutputDir(outputDir);
        gc.setFileOverride(true);
        gc.setActiveRecord(true);
        gc.setEnableCache(false);
        gc.setBaseResultMap(true);
        gc.setBaseColumnList(false);
        gc.setAuthor(author);
        mpg.setGlobalConfig(gc);
        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl(url);
        // dsc.setSchemaName("public");
        dsc.setDriverName(driverName);
        dsc.setUsername(userName);
        dsc.setPassword(userPwd);
        mpg.setDataSource(dsc);
        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        //strategy.setTablePrefix(new String[]{"_"});// 此处可以修改为您的表前缀
        strategy.setNaming(NamingStrategy.underline_to_camel);// 表名生成策略
        strategy.setInclude(tableNames);
        mpg.setStrategy(strategy);
        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setParent(null);
        pc.setEntity(daoPackage + ".entity");
        pc.setMapper(daoPackage + ".mapper");
        pc.setXml(daoPackage + ".mapper.xml");
        mpg.setPackageInfo(pc);
        // 注入自定义配置，可以在 VM 中使用 cfg.abc 设置的值
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                Map<String, Object> map = new HashMap<>();
                map.put("abc", this.getConfig().getGlobalConfig().getAuthor() + "-mp");
                this.setMap(map);
            }
        };
        mpg.setCfg(cfg);
        // 执行生成
        mpg.execute();
        // 打印注入设置
        System.err.println(mpg.getCfg().getMap().get("abc"));
    }
}
```

## 编写测试单元
上面的工作准备就绪后，接下来简单几个User的增删查改方法

### 添加用户
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200504140912597.png)
发现如果直接run上面的addUser方法的话，userMapper会报空指针异常，原因在于UserTest类继承了SpringmybatisApplicationTests类，而SpringmybatisApplicationTests这个类从一开始我们就没有过任何配置，加上如下配置就好了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200504141725201.png)
addUser()方法执行成功后，打开数据库，发现数据库中已经成功添加了一条记录
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200504142101955.png)

### 查询用户
```java
@Test
    public void queryUser() {
        TbUser user = userMapper.selectById(6);
        log.info(user.toString());
        /*
        根据id查询
        TbUser{,id=6, username=优了个秀, password=e10adc3949ba59abbe56e057f20f883e}
         */

        //这里自定义查询
        TbUser user1 = userMapper.selectByName("优了个秀");
        log.info(user1.toString());
        /*
        TbUser{id=6, username=优了个秀, password=e10adc3949ba59abbe56e057f20f883e}
         */

        List<TbUser> userList = userMapper.selectList(null);
        log.info(userList.toString());
        /*
        查询所有
        [
            TbUser{,id=6, username=优了个秀, password=e10adc3949ba59abbe56e057f20f883e},
            TbUser{,id=7, username=zhangsan, password=670b14728ad9902aecba32e22fa4f6bd}
       ]
         */
    }
```
其中自定义查询需要在TbUserMapper接口中定义`selectByName`方法，并且在对应的Mapper.xml中编写我们sql，这里推荐一个插件`mybatis-helper`,它可以方便我们在Mapper接口和对应的xml相互跳转。

```java
public interface TbUserMapper extends BaseMapper<TbUser> {
    TbUser selectByName(String userName);
}

<select id="selectByName" resultType="com.hotch.springmybatis.dao.entity.TbUser">
    select id,username,password from tb_user where username = #{userName}
</select>
```

### 修改用户
```java
@Test
public void updateUser() {
    TbUser user = userMapper.selectById(7);
    user.setUsername("lisi");
    userMapper.updateById(user);
}
```
执行之后发现`id=7`的用户的名字变成了lisi，说明执行成功了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200504144811475.png)
### 删除用户
```java
@Test
public void deleteUser() {
    //删除id为7的用户
    userMapper.deleteById(7);
}
```


## 在客户端操作CRUD
### 新建Controller
这里新建一个UserController，并且新建对应的增删查改方法

### 添加用户

```java
@RequestMapping(value = "/addUser", method = RequestMethod.POST)
public TbUser addUser(@RequestBody TbUser user) {
    log.info(user.toString());
    user.setPassword(MD5Util.encrypt(user.getPassword()));
    userMapper.insert(user);
    return user;
}
```

这里建议使用`postman`或者它的老情人`postwoman`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200504150610503.png)
### 查询用户
```java
@RequestMapping(value = "/queryUser/{id}", method = RequestMethod.GET)
public TbUser queryUser(@PathVariable Long id) {
    TbUser user = userMapper.selectById(id);
    return user;
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200504152336783.png)

删除和修改这里就不写了，和测试单元里面的类似。

到此，Spring Boot整合Mybatis-Plus并实现简单的增删查改就完成了，其实其中也不仅仅这些，还包含了log日志的配置，resource资源的重新指向等等

**欢迎关注我的个人微信公众号，【优了个秀】和你每天进步一点点**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165723960.jpg#pic_center =200x200)
