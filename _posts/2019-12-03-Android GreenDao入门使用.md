---
layout:     post
title:      Android GreenDao入门使用
subtitle:   
date:       2019-12-03
author:     dingqiang.l
header-img: img/post-2019-12-03.jpg
catalog: true
tags:
    - Android
---

最近负责的项目有需要用到数据库，原生sqlite写起来太麻烦了，所以找了用户群比较多的GreenDao和DBFlow，几经对比，最终选择了GreenDao。话不多说，开搞。

### 官方文档  ###
先来看看GreenDao的用法，打开 [GreenDao GitHub地址](https://github.com/greenrobot/greenDAO)，可以看到目前最新版本是3.2.2。根据文档先将依赖添加到我们的项目。

### 开始配置  ###
**1、配置project下的build.gradle**
    
    dependencies {
	    classpath 'com.android.tools.build:gradle:3.0.0'
	    classpath 'org.greenrobot:greendao-gradle-plugin:3.2.2' // add plugin
    }
**2、配置mudole下的build.gradle**
    
    apply plugin: 'com.android.application'
    apply plugin: 'org.greenrobot.greendao' // apply plugin
    ...
    dependencies {
   	 implementation 'org.greenrobot:greendao:3.2.2' // add library
    }
    
**3、配置数据库版本号**

同样在mudole下build.gradle的android{}下添加配置

    greendao {
        schemaVersion 1//数据库版本号
        daoPackage 'com.one.greendaohope.entity.greendao'//DaoMaster、DaoSession和UserEntityDao生成路径
        targetGenDir 'src/main/java' //生成文件路径
    }
此处需要注意的是如果daoPackage和targetGenDir 不配置，那么GreenDao注解生成文件将在app/buidl/generated 下，而不是我们自己定义的路径。

如果我们的gradle tools版本是3.3.0以上的版本，配置如上，build project的时候，会出现下面的警告

    WARNING: API 'variant.getJavaCompiler()' is obsolete and has been replaced with 'variant.getJavaCompileProvider()'.
    It will be removed at the end of 2019.
    For more information, see https://d.android.com/r/tools/task-configuration-avoidance.
    To determine what is calling variant.getJavaCompiler(), use -Pandroid.debug.obsoleteApi=true on the command line to display a stack trace.
    Affected Modules: app

这时千万别虚，稳住，上面warning的意思是这些过时的API将在2019年被移除，GreenDao使用了比较旧的API导致的。其实这并不影响使用，只是有强迫症的同学会比较难受，于是翻了下issues，看看作者是怎么解决的。哈哈哈，果然有不少人跳过坑。

看看大家怎么说issues,解决的办法有：

> 1. 去掉 targetGenDir
> 2. 降低gradle tools版本，降到3.3.0以下
> 3. 坐等作者更新
> 4. 锻炼一下你的强迫症

到此，配置工作完成了，总的来说还是非常简单的。

### 编码开始 ###
**1.创建持久化数据类**

新建好如下实体类，build project一下GreenDao会帮我们完成get，set和constructor工作。
    
    @Entity(nameInDb = "USER")
    public class UserEntity {
	    @Id(autoincrement = true)
	    private Long id;
	    private String userName;
	    private String phone;
	    private String address;
	}
**2.初始化GreenDao**

我们可以在自己的Application初始化我们的Session,之后使用就可通过appContext.getDaoSession()来对数据类进行增删查改的操作了。

    private DaoSession daoSession;

    private void initGreenDao() {
        DaoMaster.DevOpenHelper helper = new DaoMaster.DevOpenHelper(this, "greendao_db.db");
        SQLiteDatabase db = helper.getWritableDatabase();
        DaoMaster daoMaster = new DaoMaster(db);
        daoSession = daoMaster.newSession();
    }

    public DaoSession getDaoSession() {
        return daoSession;
    }

### 实体类注解解释
挑几个常用的简单解释。

**@Entity**
为GreenDao指明这是一个需要映射到数据库的实体类

    @Retention(RetentionPolicy.SOURCE)
    @Target(ElementType.TYPE)
    public @interface Entity {
	    String nameInDb() default "";//实体映射到数据库表的表名称，不填则默认是实体类名
	    Index[] indexes() default {};
	    boolean createInDb() default true;// 是否创建该实体数据库表，默认true
	    boolean active() default false;
	    boolean generateConstructors() default true;
	    boolean generateGettersSetters() default true;//是否生成get、set，默认true
	    Class protobuf() default void.class;//支持protobuf协议存储
    }
**@Id**
主键，用Long或long声明都可以，可以设置autoincrement = true实现自增，默认是false

**@Property**
数据库列的名称，如果没有该注解，则默认位该字段名称（大写）

**@Transient**
添加该注解，这个字段将不会映射到数据库中

**@Unique**
唯一约束，说明该列不允许重复

。。。其他注解以后用到在补上吧，有兴趣可以翻翻[官方API文档](http://greenrobot.org/files/greendao/javadoc/current/)

### 实现简单的增删查改

**添加**

        daoSession.insertOrReplace(entity);//已经存在key（通常是Id）则更新，否则插入
        daoSession.insert(entity);//插入一条新的数据
**删除**

        daoSession.delete(entity);//删除一条记录
        daoSession.deleteAll(UserEntity.class);//删除表中UserEntity所有记录
**更新**

        daoSession.update(entity);
**查询**

        String phoneNo = "13700011000";
        daoSession.load(entityClass, phoneNo);
        daoSession.queryRaw(entityClass, " where PHONE = ", phoneNo);
        daoSession.queryBuilder(entityClass).where(UserEntityDao.Properties.Phone.eq(phoneNo));
        daoSession.loadAll(entityClass);
**QueryBuilder**

开始之前可以先大致阅览一下QueryBuilder的相关方法，有SQL基础的同学基本能看的出这些方法对应的SQL语法，比如where()对应SQL的where的条件匹配语句，orderDesc()则对应的是排序语句。


举个简单的例子：查询Phone为13700011000的所有数据，并根据Id降序

        QueryBuilder queryBuilder = daoSession.queryBuilder(UserEntity.class);
        queryBuilder.where(UserEntityDao.Properties.Phone.eq("13700011000")).orderDesc(UserEntityDao.Properties.Id);
更多更好玩的用法有兴趣的同学可以试试。

### 版本升级

现在很多数据库的升级可能会有个很头疼的问题，就是新增字段后，历史数据会被清掉，不考虑这个问题的话将来有可能会被用户的口水淹死，手动捂脸。

解决的方案是有的：

> 1. 创建临时表TMP_,复制原来的数据库到临时表中；
> 1. 删除之前的原表；
> 1. 创建新表；
> 1. 将临时表中的数据复制到新表中，最后将TMP_表删除掉；

思路是有了，但是自己写起来又遇到各种坑，好在有大神已经给出了实现，不想自己实现的可以拿来就用，[stackoverflow地址](https://stackoverflow.com/questions/13373170/greendao-schema-update-and-data-migration/30334668#30334668)，自己实现DaoMaster.DevOpenHelper的onUpgrade方法即可。
    
    public class MyDevOpenHelper extends DaoMaster.DevOpenHelper {
    
	    public MyDevOpenHelper(Context context, String name, SQLiteDatabase.CursorFactory factory) {
	   	 super(context, name, factory);
	    }
	    
	    @Override
	    public void onUpgrade(Database db, int oldVersion, int newVersion) {
	    MigrationHelper.migrate(db, new MigrationHelper.ReCreateAllTableListener() {
	    @Override
	    public void onCreateAllTables(Database db, boolean ifNotExists) {
	    	DaoMaster.createAllTables(db, ifNotExists);
	    }
	    
	    @Override
	    public void onDropAllTables(Database db, boolean ifExists) {
	    	DaoMaster.dropAllTables(db, ifExists);
	    }
	    }, UserEntityDao.class);
	    }
    }
      
文章到此，对GreenDao的简单介绍算是完成了，希望对准备使用GreenDao的同学有所帮助。[源码传送门](https://github.com/ldqmaybe/ProjectHope)

**欢迎关注我的个人微信公众号，优了个秀和你每天进步一点点**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191203144730145.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70#pic_center)

