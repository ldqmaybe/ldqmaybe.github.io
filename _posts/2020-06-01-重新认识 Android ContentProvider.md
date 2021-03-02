---
layout:     post
title:      重新认识 Android ContentProvider
subtitle:   
date:       2020-06-01
author:     dingqiang.l
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - Android
---
## 介绍

> 1、是什么？：ContentProvider是Android四大组件之一，也是五种数据存储的媒介之一，在Android中重中之重
> 2、作用？：进程间或进程内数据共享

本文简单介绍进程间数据共享，会使用到几个常用类和关键字，下面先介绍一下：

> 1、authorities：授权信息，在清单文件声明；
> 2、URI：统一资源标识符，格式：content://授权信息/表名；也可以指定表中的某条记录，也可以配置通配符。这里不展开
> 3、UriMatcher：在ContentProvider中声明，根据URI匹配对应的数据表
> 4、ContentResolver：通过匹配URI操作对应的ContentProvider

## 使用

### 一、创建数据提供者
这里使用的sqlite，所以先完成数据库的创建，再在ContentProvider操作。

**1、创建DbHelper**
表结构比较简单，只有3个字段 uid，name和phone。

```java
public class DbHelper extends SQLiteOpenHelper {
    //数据库文件的名称
    private static final String NAME = "name";
    //数据库版本
    private static final int DB_VERSION = 1;
    //表名
    private static final String TABLE_NAME = "user";

    public DbHelper(Context context) {
        super(context, NAME, null, DB_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        String sql = "create table " + TABLE_NAME +
                " (uid Integer primary key ," +
                "name char(32) ," +
                "phone char(32) )";
        db.execSQL(sql);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

    }
}
```

**2、自定义ContentProvider**
ContentProvider不能直接使用，所以需要自定义一个子类去继承它，并在AndroidManifest.xml中注册。

```java
public class AppContentProvider extends ContentProvider {
    private SQLiteDatabase db;
    private DbHelper helper;
    //Manifest文件中 android:authorities="appContentProvider"
    private static final String AUTHORITY = "appContentProvider";
    private static final int USER_URI_CODE = 1;
    private static final UriMatcher URI_MATCHER = new UriMatcher(UriMatcher.NO_MATCH);

    static {
        //  content://appContentProvider/user
        URI_MATCHER.addURI(AUTHORITY, "user", USER_URI_CODE);
    }

    @Override
    public boolean onCreate() {
        helper = new DbHelper(getContext());
        return true;
    }

    @Nullable
    @Override
    public Cursor query(@NonNull Uri uri, @Nullable String[] projection, @Nullable String selection, @Nullable String[] selectionArgs, @Nullable String sortOrder) {
        Cursor cursor = null;
        db = helper.getWritableDatabase();
        if (URI_MATCHER.match(uri) == USER_URI_CODE) {
            cursor = db.query("user", projection, selection, selectionArgs, null, null, sortOrder);
        }
        return cursor;
    }

    @Nullable
    @Override
    public String getType(@NonNull Uri uri) {
        return null;//略
    }

    @Nullable
    @Override
    public Uri insert(@NonNull Uri uri, @Nullable ContentValues values) {
        db = helper.getWritableDatabase();
        db.insert("user", null, values);
        return null;
    }

    @Override
    public int delete(@NonNull Uri uri, @Nullable String selection, @Nullable String[] selectionArgs) {
        return 0;//略
    }

    @Override
    public int update(@NonNull Uri uri, @Nullable ContentValues values, @Nullable String selection, @Nullable String[] selectionArgs) {
        return 0;//略
    }
}
```

清单文件中注册

```java
<provider
    android:name=".AppContentProvider"
    android:authorities="appContentProvider"
    android:exported="true"
    android:process=".provider" />
```

> authorities：授权信息
> exported：是否允许外部应用访问，默认false，这里设置为true，允许
> process：在新的进程中运行，和service类似。

至此，数据提供者已经准备好了，外部可以通过URI进行操作。

### 二、新建操作数据项目
这里只简单操作添加和查询，所以直接在界面上画连个丑陋的按钮，也直接在MainActivity读写了（正常不允许）。

```java
 @Override
    public void onClick(View v) {
        Uri uri = Uri.parse("content://appContentProvider/user");
        if (R.id.btn_select == v.getId()) {
            getUserList(uri);
        } else if (R.id.btn_add == v.getId()) {
            ContentValues values = new ContentValues();
            values.put("name", "小明");
            values.put("phone", "18012" + StringUtil.getCurrentTime("ddHHmmss"));
            getContentResolver().insert(uri, values);
        }
    }

    private List<User> getUserList(Uri uri) {
        List<User> userList = new ArrayList<>();
        try {
            cursor = getContentResolver().query(uri, null, null, null, null);
            if (cursor == null) {
                return userList;
            }
            while (cursor.moveToNext()) {
                User user = new User();
                user.setName(cursor.getString(1));
                user.setPhone(cursor.getString(2));
                userList.add(user);
                Log.i("json", user.toString());
            }
            return userList;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (cursor != null) {
                cursor.close();
            }
        }
        return null;
    }
```

点击两次添加按钮，在点查询，日志中打印如下：

```java
 I/json: User{name='小命', phone='20200601165850'}
 I/json: User{name='小明', phone='1801201165909'}
```
到这里，进程间用ContentProvider数据共享基本使用就完成了，当然也可以再起一个进程对要共享的数据进行改写，不出意外的话，这两个进程都能获取到对方改写的数据。

当然，作为Android的四大组件之一，ContentProvider肯定也不会就这么简单，比如ContentObserver和MIME等，这里就不展开了。

**欢迎关注我的个人微信公众号，【优了个秀】和你每天进步一点点**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165723960.jpg#pic_center =200x200)
