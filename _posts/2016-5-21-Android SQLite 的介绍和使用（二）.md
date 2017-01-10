---
layout: post
title: Android SQLite 介绍与简单使用(二)
tags:
- Android
categories: Android,SQLite
description: Android SQLite 介绍与简单使用
---

上一节简单介绍了一下SQLite，这一节我们开始SQLite在Android中的应用。

Android提供了一个数据库的帮助类 `SQLiteOpenHelper`，用于管理数据库的创建和版本管理。我们可以继承这个类，实现它的 `onCreate`和 `onUpgrade`方法。我们可以在这里设置数据库的版本，数据库名称，创建数据库表等。下面看代码：

	public class DBHelper extends SQLiteOpenHelper {
	    //数据库的版本号必须要大于1
	    public final static int DB_VERSION = 1;
	    //数据库的名称
	    public final static String DB_NAME = "user.db";
	    //数据库表名
	    public final static String TABLE_NAME = "userInfo";
	
	    public static DBHelper helper = null;
	
	    public static DBHelper getHelper() {
	        if (helper == null) {
	            helper = new DBHelper(App.getContext());
	        }
	        return helper;
	    }
	
	
	    private DBHelper(Context context) {
	        //调用父类方法创建数据库,CusorFactory 一般为空，使用默认的
	        //CursorFactory对象，用来构造查询完毕时返回的Cursor的子类对象，为null时使用默认的CursorFactory构造。
	        super(context, DB_NAME, null, DB_VERSION);
	    }
	
	
	    /**
	     * 数据库第一次创建时调用，我们在这里执行一些数据库表的创建，初始化等操作
	     */
	    @Override
	    public void onCreate(SQLiteDatabase db) {
	        createTableUser(db);
	    }
	
	    /**
	     * 当数据库更新时调用，我们可以在升级时执行数据库修改；比  如修改表，删除表，添加表等。
	     */
	    @Override
	    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
	        if (newVersion > oldVersion) {
	            dropTableUser(db);
	            createTableUser(db);
	        }
	    }
	
	    private void createTableUser(SQLiteDatabase db) {
	        String sql = "CREATE TABLE " + TABLE_NAME + "(_id integer primary key autoincrement ,NAME TEXT,AGE INT,ADDRESS TEXT);";
	        db.execSQL(sql);
	    }
	
	    private void dropTableUser(SQLiteDatabase db) {
	        String sql = "drop table if exists " + TABLE_NAME;
	        db.execSQL(sql);
	    }
	}

当我们创建好我们数据库帮助类后，我们就可以通过它来进行数据库的操作了。当然我们也可以把一些数据库 增、删、改、查 的方法放到这个数据库帮助类里。

###增 (insert)：
往数据库表里插入数据一般有两种方式，一个是使用 SQL语句，一个是使用Android 提供的insert方法。

##### 使用SQL 语句
使用SQL语句，我们首先需要组拼一下我们的SQL语句，我们使用StringBuilder:

	StringBuilder sql = new StringBuilder();
    sql.append("insert into " + DBHelper.TABLE_NAME + "(name,age,address) values(")
     	.append(" '张三'").append(",")
        .append("20").append(",")
        .append("'火星" + i + "号'")
        .append(" );")

然后 使用数据库的 execSQL 执行这个SQL语句。

	mDatabase.execSQL(sql.toString());

##### Android 提供的insert方法
使用Android 提供的insert方法，需要用到另一个对象ContentValues,它类似于Map，以键值对的形式保存数据，键代表的是表的列名，值代表该列插入的值。具体使用如下。

        ContentValues values = new ContentValues();
        values.put("name", "张三");
        values.put("age", 20);
        values.put("address", "火星1号");

然后调用insert方法：

	 mDatabase.insert(DBHelper.TABLE_NAME, null, values);

###删（delete）:
#####使用SQL：
组拼SQL：
		
 		StringBuilder builder = new StringBuilder();
        builder.append(" delete from ").append(DBHelper.TABLE_NAME).append(" where _id = ?");

大家应该注意到 SQL 语句末尾的where语句的条件是'？'，这里的'?'起到的是占位符的作用，我们在下面需要使用 "String[]"来填入具体的值，比如这里我们可以：

	String[] bindArgs = {"1"};

然后我们调用数据库的 `execSQL`方法来执行数据的删除：

	mDatabase.execSQL(builder.toString(), bindArgs);

它执行的SQL 语句相当于：
	
	delete from userInfo where _id = 1 ;
	

当然我们也可以自己把条件值直接组拼成String语句。  


#####使用Android提供的delete方法：
Android 提供的delete方法需要提供表名，where子句，where子句的条件，并且这个方法返回的是delete影响到的行数。

        //where子句
        String whereClause = "_id = ?";
        //where字句里的里占位符的值
        String[] whereArgs = {"1"};
        int affectedRows = mDatabase.delete(DBHelper.TABLE_NAME, whereClause, whereArgs);

###改（update)
#####使用SQL 语句
首先我们组拼SQL语句：

        StringBuilder builder = new StringBuilder();
        builder.append("update ").append(DBHelper.TABLE_NAME).append(" set name=?,age=?,address=? where _id=?;");

然后定义数组来表示 set 子句中 ‘？’占位符的值：

 	Object[] bindArgs = {"赵六", 13, "水星1号", 5};

然后我们调用数据库的 `execSQL`方法来执行数据的更新：

	mDatabase.execSQL(builder.toString(), bindArgs);

#####使用Android 自带的update方法
使用Android提供的update 方法，需要用到ContentValues对象，来确定需要更新的列和值。

        ContentValues values = new ContentValues();
        values.put("name", "钱八");
        values.put("age", 15);
        values.put("address", "水星2号");

然后我们需要准备好，where子句和where子句中占位的值：

        String whereClause = "_id=?";
        String[] whereArgs = {"4"};

然后调用数据库的update方法执行更新数据的操作：

      mDatabase.update(DBHelper.TABLE_NAME, values, whereClause, whereArgs);

###查（select）
#####使用SQL语句：
Android 查询数据库的结果返回到一个Cursor 对象里。它会定位到第一行之前。所有的数据都是通过下标取得。

首先我需要组拼SQL语句：

	 StringBuilder builder = new StringBuilder();
        builder.append(" select * from ").append(DBHelper.TABLE_NAME);

然后调用数据库的 `rawQuery` 方法来执行查询，它会返回一个Cursor：

 	Cursor cursor = mDatabase.rawQuery(builder.toString(), null);

由于Cursor获取数据都是通过下标来获取的，Cursor专门提供了一个通过列表获取下标的方法：

	cursor.getColumnIndex(列名)；

Cursor 定位是定在第一行之前的我们需要调用 moveToNext()定位到第一行，当然还有很多move开头方法，大家可以多研究一下。

这样我们就可以使用循环来获取我们需要的内容了，我们把数据表里的内容都保存到List里面。

        List<User> users = new ArrayList<>();
        while (cursor.moveToNext()) {
            String name = cursor.getString(cursor.getColumnIndex("name"));
            int age = cursor.getInt(cursor.getColumnIndex("age"));
            String address = cursor.getString(cursor.getColumnIndex("address"));
            User user = new User(name, age, address);
            users.add(user);
        }

最后 我们的Cursor对象需要 close 方法，把资源释放掉。

	cursor.close();

#####使用Android提供的query方法：
我们需要提供需要查询的列名：

	String[] columns = {"name", "age", "address"};

where条件：

	String selection = "_id=?";

where 条件中 '?'的替代值：

	String[] selectionArgs = {"1"};

我们调用query方法来执行查询：

 	Cursor cursor = mDatabase.query(DBHelper.TABLE_NAME, columns, selection, selectionArgs, null, null, null);

其中后面的三个数分别为 groupBy子句,having子句,orderBy子句，如果有这些需求可以自己定义。

然后我们再把数据从Cursor里取出来，放到List里。

        List<User> users = new ArrayList<>();
        while (cursor.moveToNext()) {
            String name = cursor.getString(cursor.getColumnIndex("name"));
            int age = cursor.getInt(cursor.getColumnIndex("age"));
            String address = cursor.getString(cursor.getColumnIndex("address"));
            User user = new User(name, age, address);
            users.add(user);
        }

最后关闭 Cursor，释放资源：

	 cursor.close();


全部代码如下：
	
	public class DBHelper extends SQLiteOpenHelper {
	    //数据库的版本号必须要大于1
	    public final static int DB_VERSION = 1;
	    //数据库的名称
	    public final static String DB_NAME = "user.db";
	    //数据库表名
	    public final static String TABLE_NAME = "userInfo";
	
	    public static DBHelper helper = null;
	    public SQLiteDatabase mDatabase;
	
	    public static DBHelper getHelper() {
	        if (helper == null) {
	            helper = new DBHelper(App.getContext());
	        }
	        return helper;
	    }
	
	
	    /**
	     * 通过SQLiteOpenerHelper创建数据库表，
	     * <p/>
	     * getWritableDatabase 与 getReadableDatabase 的区别
	     * <p/>
	     * getWritableDatabase取得的实例不是仅仅具有写的功能，而是同时具有读和写的功能
	     * 同样的，getReadableDatabase 取得的实例也是具对数据库进行读和写的功能
	     * 两者的区别在于
	     * getWritableDatabase取得的实例是以读写的方式打开数据库，如果打开的数据库磁盘满了，此时只能读不能写，此时调用了getWritableDatabase的实例，那么将会发生错误（异常）
	     * getReadableDatabase取得的实例是先调用getWritableDatabase以读写的方式打开数据库，如果数据库的磁盘满了，此时返回打开失败，继而用getReadableDatabase的实例以只读的方式去打开数据库
	     */
	    private DBHelper(Context context) {
	        //调用父类方法创建数据库,CusorFactory 一般为空，使用默认的
	        //CursorFactory对象，用来构造查询完毕时返回的Cursor的子类对象，为null时使用默认的CursorFactory构造。
	        super(context, DB_NAME, null, DB_VERSION);
	        //获取数据库对象。
	        mDatabase = getReadableDatabase();
	//        mDatabase = getWritableDatabase();
	    }
	
	
	    /**
	     * 数据库第一次创建时调用，我们在这里执行一些数据库表的创建，初始化等操作
	     */
	    @Override
	    public void onCreate(SQLiteDatabase db) {
	        createTableUser(db);
	    }
	
	    /**
	     * 当数据库更新时调用，我们可以在升级时执行数据库修改；比如修改表，删除表，添加表等。
	     */
	    @Override
	    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
	        if (newVersion > oldVersion) {
	            dropTableUser(db);
	            createTableUser(db);
	        }
	    }
	
	    public void createTableUser(SQLiteDatabase db) {
	        String sql = "CREATE TABLE " + TABLE_NAME + "(_id integer primary key autoincrement ,NAME TEXT,AGE INT,ADDRESS TEXT);";
	        db.execSQL(sql);
	    }
	
	    public void dropTableUser(SQLiteDatabase db) {
	        String sql = "drop table if exists " + TABLE_NAME;
	        db.execSQL(sql);
	    }
	
	
	    /**
	     * 查看是否存在某个数据库表
	     * <p/>
	     * sqlite_master: SQLite数据库都有一个叫 SQLITE_MASTER 的表， 它定义数据库的模式,这个表是只读的
	     *
	     * @param tableName
	     * @return
	     */
	    public boolean isTableExist(String tableName) {
	        String sql = "SELECT COUNT(*) AS c FROM sqlite_master WHERE type='table' AND name='" + tableName + "'";
	        Cursor cursor = mDatabase.rawQuery(sql, null);
	        if (cursor.moveToNext()) {
	            int count = cursor.getInt(0);
	            if (count > 0) {
	                return true;
	            }
	        }
	        return false;
	    }
	
	    /**
	     * 使用SQL 实现插入
	     */
	    public void insertWithSQL() {
	        SQLiteDatabase db = mDatabase;
	        db.beginTransaction();
	        try {
	            for (int i = 0; i < 10; i++) {
	                StringBuilder sql = new StringBuilder();
	                sql.append("insert into " + DBHelper.TABLE_NAME + "(name,age,address) values(")
	                        .append(" '张三'").append(",")
	                        .append("20").append(",")
	                        .append("'火星" + i + "号'")
	                        .append(" );");
	                db.execSQL(sql.toString());
	            }
	
	            db.setTransactionSuccessful();
	        } finally {
	            db.endTransaction();
	        }
	    }
	
	    /**
	     * 使用 ContentValues 实现插入数据
	     */
	    public void insert() {
	        ContentValues values = new ContentValues();
	        values.put("name", "张三");
	        values.put("age", 20);
	        values.put("address", "火星1号");
	        mDatabase.insert(DBHelper.TABLE_NAME, null, values);
	    }
	
	
	    /**
	     * 使用SQL 语句删除数据
	     */
	    public void deleteBySQL() {
	        StringBuilder builder = new StringBuilder();
	        builder.append(" delete from ").append(DBHelper.TABLE_NAME).append(" where _id = ?");
	        String[] bindArgs = {"1"};
	        mDatabase.execSQL(builder.toString(), bindArgs);
	    }
	
	    /**
	     * 使用Android 提供的delete 方法
	     *
	     * @return
	     */
	    public int delete() {
	        //where子句
	        String whereClause = "_id = ?";
	        //where字句里的里占位符的值
	        String[] whereArgs = {"1"};
	        int affectedRows = mDatabase.delete(DBHelper.TABLE_NAME, whereClause, whereArgs);
	        return affectedRows;
	    }
	
	
	    /**
	     * 使用SQL修改数据
	     */
	    public void updateBySQL() {
	        StringBuilder builder = new StringBuilder();
	        builder.append("update ").append(DBHelper.TABLE_NAME).append(" set name=?,age=?,address=? where _id=?;");
	        Object[] bindArgs = {"赵六", 13, "水星1号", 5};
	        mDatabase.execSQL(builder.toString(), bindArgs);
	    }
	
	    /**
	     * 使用Android 提供的update方法修改数据
	     */
	    public void update() {
	        ContentValues values = new ContentValues();
	        values.put("name", "钱八");
	        values.put("age", 15);
	        values.put("address", "水星2号");
	        String whereClause = "_id=?";
	        String[] whereArgs = {"4"};
	        mDatabase.update(DBHelper.TABLE_NAME, values, whereClause, whereArgs);
	    }
	
	
	    /**
	     * 使用 SQL 实现查询所有数据
	     */
	    public void queryDataBySql() {
	        StringBuilder builder = new StringBuilder();
	        builder.append(" select * from ").append(DBHelper.TABLE_NAME);
	        Cursor cursor = mDatabase.rawQuery(builder.toString(), null);
	
	        List<User> users = new ArrayList<>();
	        while (cursor.moveToNext()) {
	            String name = cursor.getString(cursor.getColumnIndex("name"));
	            int age = cursor.getInt(cursor.getColumnIndex("age"));
	            String address = cursor.getString(cursor.getColumnIndex("address"));
	            User user = new User(name, age, address);
	            users.add(user);
	        }
	
	        LogUtils.e(users + "");
	        cursor.close();
	    }
	
	    /**
	     * 使用 SQL 实现查询某条数据
	     */
	    public void queryDataBySql1() {
	        StringBuilder builder = new StringBuilder();
	        builder.append(" select name,age,address from ").append(DBHelper.TABLE_NAME).append(" where _id = ?");
	        String[] selectArgs = {"1"};
	        Cursor cursor = mDatabase.rawQuery(builder.toString(), selectArgs);
	
	        List<User> users = new ArrayList<>();
	        while (cursor.moveToNext()) {
	            String name = cursor.getString(cursor.getColumnIndex("name"));
	            int age = cursor.getInt(cursor.getColumnIndex("age"));
	            String address = cursor.getString(cursor.getColumnIndex("address"));
	            User user = new User(name, age, address);
	            users.add(user);
	        }
	
	        LogUtils.e(users + "");
	        cursor.close();
	    }
	
	
	    /**
	     * 使用Android提供的 query 方法 ，实现查询所有数据
	     */
	    public void queryAll() {
	        Cursor cursor = mDatabase.query(DBHelper.TABLE_NAME, null, null, null, null, null, null);
	        List<User> users = new ArrayList<>();
	        while (cursor.moveToNext()) {
	            String name = cursor.getString(cursor.getColumnIndex("NAME"));
	            int age = cursor.getInt(cursor.getColumnIndex("AGE"));
	            String address = cursor.getString(cursor.getColumnIndex("ADDRESS"));
	            User user = new User(name, age, address);
	            users.add(user);
	        }
	
	        LogUtils.e(users + "");
	        cursor.close();
	    }
	
	    /**
	     * 使用Android提供的 query 方法 ，实现查询某条数据
	     */
	    public void query() {
	
	        //想要查询的那些列
	        String[] columns = {"name", "age", "address"};
	        //where条件
	        String selection = "_id=?";
	        //where 条件中 '?'的替代值
	        String[] selectionArgs = {"1"};
	        Cursor cursor = mDatabase.query(DBHelper.TABLE_NAME, columns, selection, selectionArgs, null, null, null);
	
	        List<User> users = new ArrayList<>();
	        while (cursor.moveToNext()) {
	            String name = cursor.getString(cursor.getColumnIndex("name"));
	            int age = cursor.getInt(cursor.getColumnIndex("age"));
	            String address = cursor.getString(cursor.getColumnIndex("address"));
	            User user = new User(name, age, address);
	            users.add(user);
	        }
	        LogUtils.e(users + "");
	        cursor.close();
	    }
	}
