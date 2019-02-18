---
layout: post
title:  "用SQLiteOpenHelper实现操作SQLite"
date:   2019-02-18 12:00:00 +0800
categories: Android
tags: Kotlin, SQLite
---
Android内置数据库SQLite，SQLite是开源的SQL数据库。Android提供了SQLiteOpenHelper类来操作SQLite数据库，该类包含的方法有创建， 插入，删除，更新(CRUD)操作。  
现在，我们用SQLiteOpenHelper来实现SQLite的操作。

首先来看下布局代码，TextView显示数据库所有数据。  
``` xml
<?xml version="1.0" encoding="utf-8"?>
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

    <TextView
            android:id="@+id/display_name_tv"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_margin="16dp"
            android:text=""/>

    <EditText
            android:id="@+id/name_et"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginStart="8dp"
            android:layout_marginTop="16dp"
            android:layout_marginEnd="8dp"
            android:ems="10"
            android:hint="输入用户名称"
            android:inputType="textPersonName"
            android:text=""/>

    <Button
            android:id="@+id/add_to_db_btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="addUser"
            android:text="添加用户"/>

    <EditText
            android:id="@+id/id_et"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginStart="8dp"
            android:layout_marginTop="16dp"
            android:layout_marginEnd="8dp"
            android:ems="10"
            android:hint="输入要删除的用户ID"
            android:inputType="numberDecimal"/>

    <Button
            android:id="@+id/delete_btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            andrAndroid提供了SQLiteOpenHelper类来操作SQLite数据库，该类包含的方法有创建， 插入，删除，更新(CRUD)操作。oid:onClick="deleteUser"
            android:text="根据ID删除用户"/>

    <Button
            android:id="@+id/show_data_btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="showData"
            android:text="查看所有用户"/>
</LinearLayout>

```

接下来我们创建一个数据模型(model)
``` ktolin
data class User(
    var id: Int,
    var userName: String
) {
    constructor(userName: String): this(0,userName)
}
```

新建一个类继承SQLiteOpenHelper
``` Kotlin
class DBHelper(context: Context,
                           factory: SQLiteDatabase.CursorFactory?) :
    SQLiteOpenHelper(context, DATABASE_NAME,
        factory, DATABASE_VERSION) {
    override fun onCreate(db: SQLiteDatabase) {
        val CREATE_PRODUCTS_TABLE = ("CREATE TABLE " +
                TABLE_NAME + "("
                + COLUMN_ID + " INTEGER PRIMARY KEY," +
                COLUMN_NAME
                + " TEXT" + ")")
        db.execSQL(CREATE_PRODUCTS_TABLE)
    }
    override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
        db.execSQL("DROP TABLE IF EXISTS " + TABLE_NAME)
        onCreate(db)
    }

    fun addName(user: User) {
        val values = ContentValues()
        values.put(COLUMN_NAME, user.userName)
        val db = this.writableDatabase
        db.insert(TABLE_NAME, null, values)
        db.close()
    }

    fun deleteUserById(id: Int): Boolean {
        val db = this.readableDatabase
        return db.delete(TABLE_NAME, "id=${id}", null) > 0
    }

    fun getAllUser(): Cursor? {
        val db = this.readableDatabase
        return db.rawQuery("SELECT * FROM $TABLE_NAME", null)
    }

    companion object {
        private val DATABASE_VERSION = 1
        private val DATABASE_NAME = "sqldemo.db"
        val TABLE_NAME = "name"
        val COLUMN_ID = "id"
        val COLUMN_NAME = "username"
    }
}
```
现在我们通过Button的点击事件来对SQLite数据库进行操作。
``` kotlin
    fun addUser(view: View) {
        val dbHelper = DBHelper(this, null)
        val user = User(editText.text.toString())
        dbHelper.addName(user)
        Toast.makeText(this, "${user.userName}  添加到数据库", Toast.LENGTH_SHORT).show()
    }

    fun deleteUser(view: View) {
        val dbHelper = DBHelper(this, null)
        val id = editText2.text.toString()
        if (dbHelper.deleteUserById(id.toInt())) {
            Toast.makeText(this, "删除成功", Toast.LENGTH_SHORT).show()
            editText2.setText("")
        } else {
            Toast.makeText(this, "删除失败", Toast.LENGTH_SHORT).show()
        }
    }

    fun showData(view: View) {
        textView.text = ""

        val dbHandler = DBHelper(this, null)
        try {
            val cursor = dbHandler.getAllUser()
            cursor!!.moveToFirst()
            var name = User(
                cursor.getInt(cursor.getColumnIndex(DBHelper.COLUMN_ID)),
                cursor.getString(cursor.getColumnIndex(DBHelper.COLUMN_NAME))
            )
            textView.append("id: ${name.id} name: ${name.userName}")
            textView.append("\n")
            while (cursor.moveToNext()) {
                var name = User(
                    cursor.getInt(cursor.getColumnIndex(DBHelper.COLUMN_ID)),
                    cursor.getString(cursor.getColumnIndex(DBHelper.COLUMN_NAME))
                )
                textView.append("id: ${name.id} name: ${name.userName}")
                textView.append("\n")
            }
            cursor.close()
        } catch (e: CursorIndexOutOfBoundsException) {
            Toast.makeText(this, "没有数据", Toast.LENGTH_SHORT).show()
        }
    }
```
整个MainActivit的代码
``` kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var textView: TextView;
    private lateinit var editText: EditText;
    private lateinit var editText2: EditText

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        textView = findViewById(R.id.display_name_tv)
        editText = findViewById(R.id.name_et)
        editText2 = findViewById(R.id.id_et)
    }

    fun addUser(view: View) {
        val dbHelper = DBHelper(this, null)
        val user = User(editText.text.toString())
        dbHelper.addName(user)
        Toast.makeText(this, "${user.userName}  添加到数据库", Toast.LENGTH_SHORT).show()
    }

    fun deleteUser(view: View) {
        val dbHelper = DBHelper(this, null)
        val id = editText2.text.toString()
        if (dbHelper.deleteUserById(id.toInt())) {
            Toast.makeText(this, "删除成功", Toast.LENGTH_SHORT).show()
            editText2.setText("")
        } else {
            Toast.makeText(this, "删除失败", Toast.LENGTH_SHORT).show()
        }
    }

    fun showData(view: View) {
        textView.text = ""

        val dbHandler = DBHelper(this, null)
        try {
            val cursor = dbHandler.getAllUser()
            cursor!!.moveToFirst()
            var name = User(
                cursor.getInt(cursor.getColumnIndex(DBHelper.COLUMN_ID)),
                cursor.getString(cursor.getColumnIndex(DBHelper.COLUMN_NAME))
            )
            textView.append("id: ${name.id} name: ${name.userName}")
            textView.append("\n")
            while (cursor.moveToNext()) {
                var name = User(
                    cursor.getInt(cursor.getColumnIndex(DBHelper.COLUMN_ID)),
                    cursor.getString(cursor.getColumnIndex(DBHelper.COLUMN_NAME))
                )
                textView.append("id: ${name.id} name: ${name.userName}")
                textView.append("\n")
            }
            cursor.close()
        } catch (e: CursorIndexOutOfBoundsException) {
            Toast.makeText(this, "没有数据", Toast.LENGTH_SHORT).show()
        }
    }

}
```
运行代码。以上就通过SQLiteOpenHelper来实现对SQLite数据的操作。上述代码在我的[github][github]。

[github]: https://github.com/jessyuan24/SQLiteDemo.git
