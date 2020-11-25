### Fragment

```kotlin
//动态添加Fragment标准步骤
 private fun replaceFragment(fragment: Fragment){//取得fragment实例
        val fragmentManager =supportFragmentManager//获取FragmentManager，java为getSupportFragmentManager
        val transaction = fragmentManager.beginTransaction()//开启一个事务
        transaction.replace(R.id.leftFrag,fragment)//使用replace添加或替换fragment(容器位置,内容对象)
        transaction.commit()//提交事务
    }
//Fragment添加返回栈
transaction.addToBackStack(null)//给Fragment添加返回栈，按下返回键回到上一个fragment
//activity中使用fragment
var fragment1=supportFragmentManager.findFragmentById(R.id.leftFrag)//通过supportFragmentManager在acticity中获取fragment中的控件 
        //使用了kotlin-android-extensions的写法
        //val fragment2 =leftFrag as leftFragment
//fragment中使用activity
fun getMyActivity(activity:Activity){
        if (activity!=null){
            val mainAcitvity = activity as MainActivity
        }
    }
```
屏幕特征|限定符|描述
-|-|-
大小|small|提供给小屏幕设备的资源
大小|normal|提供给中屏幕设备的资源
大小|large|提供给大屏幕设备的资源
大小|xlarge|提供给超大屏幕设备的资源
### 广播

```kotlin
//动态注册广播，当前在MainActivity的onCreate中进行注册，也就当我们打开MainActivity后才会开始
class MainActivity : AppCompatActivity() {
    lateinit var timeChangeReceiver:TimeChangeReceiver
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val intentFilter=IntentFilter()
        intentFilter.addAction("android.intent.action.TIME_TICK")//定义需要监听的广播action
        timeChangeReceiver = TimeChangeReceiver()
        registerReceiver(timeChangeReceiver,intentFilter)//向系统注册广播
    }

    override fun onDestroy() {
        super.onDestroy()
        unregisterReceiver(timeChangeReceiver)//退出时取消注册广播
    }
    inner class TimeChangeReceiver :BroadcastReceiver(){
        override fun onReceive(context: Context?, intent: Intent?) {//继承BroadcastReceiver所必须实现的函数，广播到来时执行
            Toast.makeText(this@MainActivity,"Time is changed",Toast.LENGTH_SHORT).show()
          //  Toast.makeText(context,"Time is changed",Toast.LENGTH_SHORT).show()
        }
    }
}
//路径：<Android SDK>/platforms/<任意 android api版本>/data/broadcast_actions.txt
//存放所以系统的广播列表
//当前系统路径:/Users/yuyuyu/Library/Android/sdk/platforms/android-29/data/broadcast_actions.txt
//静态注册，可以在app没启动的的情况下接收广播，但那些没有具体指定发送给哪个app的隐式广播都不允许用静态注册，只存在少量特殊的系统广播可以静态注册
```
![创建广播](image/guangbo.png)
用动态注册时创建的的内部类，现在用静态注册用视图直接创建一个广播类以在activity还没进入时就能完成广播接收。建好类后AndroidManifest.xml中自动添加广播注册。

```xml
<receiver
            android:name=".BootCompleteReceiver"
            android:enabled="true"
            android:exported="true"></receiver>
现在仅注册了广播还没设定具体的广播action
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.broadcastreceiver">
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>BOOT_COMPLETED需要先声明权限，
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.BroadcastReceiver">
        <receiver
            android:name=".BootCompleteReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED"/>//声明action，
              <action android:name="com.example.broadcastreceiver.My_Receiver"/>//自定义广播
              //当有Inten("com.example.broadcastreceiver.My_Receiver")通过sendBroadcast发送时就能接收到
            </intent-filter>
        </receiver>

        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

```kotlin
//自定义广播发送
				val intent=Intent("com.example.broadcastreceiver.My_Receiver")
        intent.setPackage(packageName)//指定能接收广播的包名，指定完广播就成了显式广播
        sendBroadcast(intent)//发送意图
```

### 文件读写

```kotlin
 //文件写入，java文件流操作参考https://www.cnblogs.com/jmcui/p/9096536.html
    fun save(output :String){
        try {
            val file = openFileOutput("myfile", Context.MODE_APPEND)//取得FileOutputStream对象
            val write = BufferedWriter(OutputStreamWriter(file))//用FileOutputStream对象构建出BufferedWriter对象
            write.use {
                it.write(output)//用BufferedWriter对象来将文本内容写入文件
                //use可以在代码执行完后自动关闭流
            }
        }catch (e:IOException){
            e.printStackTrace()
        }
    }
    
    fun load(){
        try {
            val file=openFileInput("myfile")
            val read=BufferedReader(InputStreamReader(file))
            read.use {
                tv.text=it.readText()
            }
        }catch (e:IOException){
            e.printStackTrace()
        }
    }
```

### 数据库

```sqlite
SQlite的数据类型：整型integer 浮点型real 文本型text 二进制型blob
```

1. 创建/更新

   ```kotlin
    val dbHelper = MyDatabaseHelper(this, "BookStore.db", 3)//取得DatabaseHelper实例
   	dbHelper.writableDatabase//创建或更新数据库，版本号上升时才能成功，第一次执行DatabaseHelper的onCreate(),数据库存在后就只会执行onUpgrade()更新数据库
   ```

2. 添加数据

   ```kotlin
   val db = dbHelper.writableDatabase//获取实例
               val values1 = ContentValues().apply {//添加封装好的数据
                   // 开始组装第一条数据
                   put("name", "The Da Vinci Code")
                   put("author", "Dan Brown")
                   put("pages", 454)
                   put("price", 16.96)
               }
              db.insert("Book", null, values1)//插入数据(表名,允许空的字段,数据)
   ```

3. 插入数据

   ```kotlin
    val values = ContentValues()
    values.put("price", 10.99)//添加数据
    val rows = db.update("Book", values, "name = ?", arrayOf("The Da Vinci Code"))
   //(表名,要插入的数据,where等条件,条件内容参数数组)
   ```

4. 删除数据

   ```kotlin
   db.delete("Book", "pages > ?", arrayOf("500"))//(表名,条件,条件内容参数数组)即使条件内容只有一个也要用array进行存放
   ```

5. 查询数据

   | query()方法参数 | 对应SQL部分               | 描述                        |
   | --------------- | ------------------------- | --------------------------- |
   | table           | from table_name           | 指定查询的表名              |
   | columns         | selection column1,coulmn2 | 指定查询的列名              |
   | selecton        | where column = value      | 指定where的约束条件         |
   | selectionArgs   | -                         | 为where的占位符提供具体内容 |
   | gropBy          | grop by column            | 指定需要grop by(分组)的类   |
   | having          | having column=value       | 对grop by后的结果进行约束   |
   | orderBy         | order by column1,colunm2  | 指定查询结果的排序方式      |

   ```kotlin
     val cursor = db.query("Book", null, null, null, null, null, null)
   //(表名,字段名/列名称数组,条件字句，相当于where,条件内容参数数组,分组列,分组条件,排序列,分页查询限制)
   /*java：Cursor query(String table, String[] columns,  
               String selection, String[] selectionArgs, String groupBy,  
               String having, String orderBy) */
   /*kotlin: Cursor query(String table, String[] columns, String selection,
               String[] selectionArgs, String groupBy, String having,
               String orderBy)*/
               if (cursor.moveToFirst()) {
                   do {
                       // 遍历Cursor对象，取出数据并打印
                       val name = cursor.getString(cursor.getColumnIndex("name"))
                       val author = cursor.getString(cursor.getColumnIndex("author"))
                       val pages = cursor.getInt(cursor.getColumnIndex("pages"))
                       val price = cursor.getDouble(cursor.getColumnIndex("price"))
                       Log.d("MainActivity", "book name is $name")
                       Log.d("MainActivity", "book author is $author")
                       Log.d("MainActivity", "book pages is $pages")
                       Log.d("MainActivity", "book price is $price")
                   } while (cursor.moveToNext())
               }
               cursor.close()
   ```

6. 替换数据

   ```kotlin
   db.beginTransaction() // 开启事务，缓存备份机制，只有setTransactionSuccessful()后endTransaction()才会保存过去
               try {
                   db.delete("Book", null, null)//直接删除表，不需要的参数直接null
                   val values = cvOf("name" to "Game of Thrones", "author" to "George Martin", "pages" to 720, "price" to 20.85)
                   db.insert("Book", null, values)
                   db.setTransactionSuccessful() // 事务已经执行成功
               } catch (e: Exception) {
                   e.printStackTrace()
               } finally {
                   db.endTransaction() // 结束事务
               }
   //自定了cvof，androidx的kotlin自带等效的ContentValiesof，简化了数据类型的判断
   fun cvOf(vararg pairs: Pair<String, Any?>) = ContentValues().apply {
       for (pair in pairs) {
           val key = pair.first
           val value = pair.second
           when (value) {
               is Int -> put(key, value)
               is Long -> put(key, value)
               is Short -> put(key, value)
               is Float -> put(key, value)
               is Double -> put(key, value)
               is Boolean -> put(key, value)
               is String -> put(key, value)
               is Byte -> put(key, value)
               is ByteArray -> put(key, value)
               null -> putNull(key)
           }
       }
   }
   ```

   

### 简化shp

```kotlin
//原版
val edit=getPreferences("data",Context.MODE_PRIVATE).edit()//先获取edit对象
edit.putString("name","Tom")//添加数据
edit.putInt("age",28)
edit.apply()//提交数据
//简化，使用高阶函数+扩展函数，定义一个SharedPreferences.open
fun SharedPreferences.open(block:SharedPreferences.Edit() -> Unit){//用拓展函数添加一个open
  val editor=edit()//像同个类下的a(){b()}这样直接使用SharedPreferences的edit()，
  edit.block()//取得传入的数据
  edit.apply()
}
//调用
getPreferences("data",Context.MODE_PRIVATE).open{
  edit.putString("name","Tom")//只需要添加数据
	edit.putInt("age",28)//数据通过SharedPreferences.Edit()打包
}
//androidx自带的简化
getPreferences("data",Context.MODE_PRIVATE).edit{//在implementation 'androidx.core:core-ktx:1.3.2'下
  edit.putString("name","Tom")
	edit.putInt("age",28)
}

```

### 进程

- 前台进程

  ```
  用户当前操作所必需的进程。如果一个进程满足以下任一条件，即视为前台进程：
  托管用户正在交互的 Activity（已调用 Activity 的 onResume() 方法）
  托管某个 Service，后者绑定到用户正在交互的 Activity 
  托管正在“前台”运行的 Service（服务已调用 startForeground()）
  托管正执行一个生命周期回调的 Service（onCreate()、onStart() 或 onDestroy()）
  托管正执行其 onReceive() 方法的 BroadcastReceiver
  通常，在任意给定时间前台进程都为数不多。只有在内存不足以支持它们同时继续运行这一万不得已的情况下，系统才会终止它们。 此时，设备往往已达到内存分页状态，因此需要终止一些前台进程来确保用户界面正常响应。
  ```

- 可见进程

  ```
  没有任何前台组件、但仍会影响用户在屏幕上所见内容的进程。 
  如果一个进程满足以下任一条件，即视为可见进程：               
  托管不在前台、但仍对用户可见的 Activity（已调用其 onPause() 方法）。例如，如果前台 Activity 启动了一个对话框，允许在其后显示上一 Activity，则有可能会发生这种情况。             
  托管绑定到可见（或前台）Activity 的 Service。              
  可见进程被视为是极其重要的进程，除非为了维持所有前台进程同时运行而必须终止，否则系统不会终止这些进程。          服务进程       
  正在运行已使用 startService() 方法启动的服务且不属于上述两个更高类别进程的进程。尽管服务进程与用户所见内容没有直接关联，但是它们通常在执行一些用户关心的操作（例如，在后台播放音乐或从网络下载数据）。因此，除非内存不足以维持所有前台进程和可见进程同时运行，否则系统会让服务进程保持运行状态。      
  ```

- 后台进程

  ```
  包含目前对用户不可见的 Activity 的进程（已调用 Activity 的 onStop() 方法）。这些进程对用户体验没有直接影响，系统可能随时终止它们，以回收内存供前台进程、可见进程或服务进程使用。 通常会有很多后台进程在运行，因此它们会保存在 LRU （最近最少使用）列表中，以确保包含用户最近查看的 Activity 的进程最后一个被终止。如果某个 Activity 正确实现了生命周期方法，并保存了其当前状态，则终止其进程不会对用户体验产生明显影响，因为当用户导航回该 Activity 时，Activity 会恢复其所有可见状态。 有关保存和恢复状态的信息，请参阅 Activity文档。
  ```

- 空进程

  ```
  不含任何活动应用组件的进程。保留这种进程的的唯一目的是用作缓存，以缩短下次在其中运行组件所需的启动时间。 为使总体系统资源在进程缓存和底层内核缓存之间保持平衡，系统往往会终止这些进程。
  ```

### Uri的使用场景

````kotlin
 Uri.parse(str); //将URI字符串解析成uri对象
````

1. 调web浏览器

   ```java
   Uri myBlogUri =Uri.parse(" http://xxxxx.com ");  
   returnIt = new Intent(Intent.ACTION_VIEW, myBlogUri);  
   ```

2. 地图

   ```java
   Uri mapUri = Uri.parse("geo:38.899533,-77.036476");  
   returnIt = new Intent(Intent.ACTION_VIEW, mapUri);  
   ```

3. 调用拨打电话页面

   ```java
   Uri telUri = Uri.parse("tel:100861");  
   returnIt = new Intent(Intent.ACTION_DIAL, telUri);  
   ```

4. 直接拨打电话

   ```java
   Uri telUri = Uri.parse("tel:100861");  
   returnIt = new Intent(Intent.ACTION_DIAL, telUri);  
   
   ```

5. 卸载

   ```java
   Uri uninstallUri = Uri.fromParts("package", "xxx", null);  
   returnIt = new Intent(Intent.ACTION_DELETE, uninstallUri);  
   ```

6. 安装

   ```java
   Uri uninstallUri = Uri.fromParts("package", "xxx", null);  
   returnIt = new Intent(Intent.ACTION_DELETE, uninstallUri);  
   ```

7. 播放

   ```java
   Uri playUri = Uri.parse("file:///sdcard/download/everything.mp3");  
   returnIt = new  Intent(Intent.ACTION_VIEW, playUri);  
   ```

8. 调用发邮件

   ```java
    Uri emailUri = Uri.parse("mailto:xxxx@gmail.com");  
    returnIt = new  Intent(Intent.ACTION_SENDTO, emailUri);  
   ```

   

9. 发邮件

   ```java
   returnIt = new Intent(Intent.ACTION_SEND);  
   
     String[] tos = {"xxxx@gmail.com" };  
   
     String[] ccs = {"xxxx@gmail.com" };  
   
     returnIt.putExtra(Intent.EXTRA_EMAIL, tos);  
   
     returnIt.putExtra(Intent.EXTRA_CC, ccs);  
   
     returnIt.putExtra(Intent.EXTRA_TEXT, "body");  
   
     returnIt.putExtra(Intent.EXTRA_SUBJECT, "subject");  
   
     returnIt.setType("message/rfc882");  
   
     Intent.createChooser(returnIt,
       "Choose Email Client");  
   
   ```

   

10. 发短信

    ```java
    Uri smsUri = Uri.parse("tel:100861");  
    
      returnIt = new
        Intent(Intent.ACTION_VIEW, smsUri);  
    
      returnIt.putExtra("sms_body", "yyyy");  
    
      returnIt.setType("vnd.android-dir/mms-sms");  
    ```

    

11. 直接发邮件

    ```java
    Uri smsToUri = Uri.parse("smsto://100861");  
    
      returnIt = new
        Intent(Intent.ACTION_SENDTO, smsToUri);  
    
      returnIt.putExtra("sms_body", "yyyy");  	
    ```

    

12. 发彩信

    ```java
    Uri mmsUri = Uri.parse("content://media/external/images/media/23");  
    
      returnIt = new Intent(Intent.ACTION_SEND);  
    
      returnIt.putExtra("sms_body", "yyyy");  
    
      returnIt.putExtra(Intent.EXTRA_STREAM, mmsUri);  
    
      returnIt.setType("image/png");
    ```

### Runtime Exception运行时异常

异常名|注释
:-|:-
ArithmeticException|算术异常 
 ArrayStoreException|将数组类型不兼容的值赋值给数组元素时抛出的异常
 BufferOverflowException |缓冲区溢出异常
 BufferUnderflowException |缓冲区下溢异常
 CannotRedoException |不能重复上一次操作异常
 CannotUndoException |不能撤销上一次操作异常
 ClassCastException |类型强制转换异常
 ClassNotFoundException |类没找到时,抛出该异常
 CMMException |CMM异常
 ConcurrentModificationException| 对Vector、ArrayList在迭代的时候如果同时对其进行修改就会抛出异常
 org.springframework.jdbc.CannotGetJdbcConnectionException |服务器端数据库连接不上时,抛出该异常
 CannotGetJdbcConnectionException| 网络没有连接或网络中断 
 DOMException| DOM异常 
 EOFException| 文件已结束异常 
 EmptyStackException| 空栈异常 
 FileNotFoundException| 文件未找到异常 
 IllegalArgumentException| 传递非法参数异常 
 IllegalMonitorStateException| 
 IllegalAccessException| 访问某类被拒绝时抛出的异常 
 IllegalPathStateException| 非法的路径声明异常 
 IllegalStateException| 非法声明异常 
 ImagingOpException| 成像操作异常 
 IndexOutOfBoundsException| 下标越界异常 
 IOException| 输入输出异常 
 NegativeArraySizeException| 数组负下标异常 
 NoSuchMethodException |在类中无法找到某一特定方法时,抛出该异常 
 NoSuchElementException| 方法未找到异常 
 NoSuchFieldException |类不包含指定名称的字段时产生的信号(bean中不存在这个属性)
 NumberFormatException | 字符串转换为数字异常 
 NullPointerException| 空指针异常 
 ProfileDataException| 没有日志文件异常 
 ProviderException| 供应者异常 
 RasterFormatException| 平面格式异常 
 SecurityException| 违背安全原则异常 
 SQLException| 操作数据库异常 
 SystemException| 系统异常 
 UndeclaredThrowableException| 
 UnmodifiableSetException| 
 UnsupportedOperationException| 不支持的操作异常

### 运行时权限

普通权限在manifest中声明后由系统授予，运行时权限声明后系统无法做主需要在要使用时弹出对话框进行申请

```kotlin
//先进行声明，声明一个拨打电话的权限
<uses-premission android:name="android.permission.CALL_PHONE"/>
//权限申请逻辑
if(ContextCompaat.checkSelfPermission(this,Manifest.permission.CELL_PHONE) !=PackageManager.PERMISSION_GRANTED){/判断权限是否已经允许过了
  //如果没允许过权限，进行申请
  ActivityCompat.requestPermissions(this,arrayof(Manifest.permission.CALL_PHONE),1)
  //使用ActivityCompat.requestPermissions(Activity,权限数组,请求码(需要唯一值))向用户申请权限  
  //requestPermissions()执行后会弹出权限申请的对话框，无论选允许还是拒绝都会调用OnRequestPermissionResult(),授权的结果在参数grantResults里                                                                                                      
}else{
  call()//如果没有权限直接cell()直接就SecurityException异常
}
//重写OnRequestPermissionResult()方法
override fun OnRequestPermissionResult(requestCode:Int,permissions:Array<String>,grantResults:IntArray){
  super.OnRequestPermissionResult(requesrCode,permissions,grantResults)
  when(requestCode){//判断grantResults
  1 -> {//	根据请求码判断具体是哪个requestPermissions()
    if(grantResults.isNotEmpty()&&
      grantResults[0]==PackageManager.PERMISSION_GRANTED){//判断是否取得权限
      //取得权限后继续执行需要权限的操作
      call()
    }else{//当然也可以根据权限数组中的每一个权限进行操作
      //没有权限,则无法执行call(),通知用户执行失败的原因
      Toats.makeText(this,"没有允许拨打电话权限",Toats.LENGTH_SHORT).show()
    }    	
   }
  }
}
//存储权限
saveButton.setOnClickListener {
            if (Build.VERSION.SDK_INT < 30 && ContextCompat.checkSelfPermission(
                    requireContext(),
                    android.Manifest.permission.WRITE_EXTERNAL_STORAGE
                ) != PackageManager.PERMISSION_GRANTED
            ) {
                requestPermissions(//直接requestPermissions
            //也不需要Content                 
                  arrayOf((android.Manifest.permission.WRITE_EXTERNAL_STORAGE)),
                    REQUEST_WRITE_EXERNAL_STORAGE
                )
            } else {
                save()
            }
        }
```

### ContentProvider

1. 使用ContentResover进行ContentProvider的增删改查

   ```kotlin
   val contentResover:ContentResover=Context.getContentResover()//取得ContentResover实例
   val uri=Uri.parse("content://com.example.app.provider/table1")
   val uriid=Uri.parse("content://com.example.app.provider/table1/id")
   //URI字符串 协议声明：content:// authority:com.example.app.provider path:/table1 id:1
   //authority指定要访问的App的包名 path指定该app中需要访问的表名 两者拼接在一起使用 id指定表中指定id的数据进行访问，可省略
   content://com.example.app.provider/*            //*表示匹配任意长度的任意字符，即所有表名
   content://com.example.app.provider/table1/#			//#表示匹配任意长度的数字，即所有id
   val cursor = contentResolver.query(uri,projection,selection,selectionArgs,sortOrder)
   //和数据库内容相同，uri指定表名，projection 指定列名
   //查，遍历数据库
   while(cursor.moveToNext){
     val column1=cursor.getString(cursor.getColumnIndex("column1"))//获取数据
     val column2=cursor.getInt(cursor.getColumnIndex("column2"))
   }
   cursor.close()//游标用完关闭
   //增
   val values =ContentValuesOf("column1" to "text","column2" to 1)//添加之前获取的数据
   contentResover.insert(uri,values)//insert(表，值)
   //改
   val values=ContentValuesOf("column1" to "")
   contentResover.update(uri,values,"column2 = ? and column2=?",arrayOf("text","1"))
   //把所有column1值为text并且column2值为1的行中column1的值改为""
   //删
   contentResolver.delete(uri,"column2=?",arrayOf("1"))//删除所有列column2的值为1的行
   ```

2. 创建自己的ContentProvider

   可以UI创建也可以直接建个类继承ContentProvider类来创建

   ```kotlin
   class MyContentProvider : ContentProvider() {
       val table1Dir = 0//根据自己想要给别人访问的表来创建变量
       val table1Item = 1
       val table2Dir = 2
       val table2Item = 3
       val uriMatcher = UriMatcher(UriMatcher.NO_MATCH)
       init {//uriMatcher.addURI(authority,path,自定义代码匹配成功后返回)
           uriMatcher.addURI("com.example.app.provider", "table1", table1Dir)
           uriMatcher.addURI("com.example.app.provider ", "table1/#", table1Item)
           uriMatcher.addURI("com.example.app.provider ", "table2", table2Dir)
           uriMatcher.addURI("com.example.app.provider ", "table2/#", table2Item)
       }
   ...
     override fun query(
           uri: Uri, projection: Array<String>?, selection: String?,
           selectionArgs: Array<String>?, sortOrder: String?
       ): Cursor? {
           when (uriMatcher.match(uri)) {//获取返回的uriMatcher.addURI()中的自定义代码来判断具体的uri是哪个
             table1Dir -> {//直接使用Database对象对自己数据库进行操作
                   // 查询table1表中的所有数据
               }
               table1Item -> {
                   // 查询table1表中的单条数据
               }
               table2Dir -> {
                   // 查询table2表中的所有数据
               }
               table2Item -> {
                   // 查询table2表中的单条数据
               }
           }
           return null
       }
   
     override fun getType(uri: Uri): String? = when (uriMatcher.match(uri)) {//根据uri生成MIME
           table1Dir -> "vnd.android.curror.dir/vnd.com/example.app.provider.table1"
           table1Item -> "vnd.android.curror.item/vnd.com/example.app.provider.table1"
           table2Dir -> "vnd.android.curror.dir/vnd.com/example.app.provider.table2"
           table2Item -> "vnd.android.curror.item/vnd.com/example.app.provider.table2"
           //1.以vnd.开头
           //2.根据uri的结尾是表名还是id区分android.curror.dir/和vnd.android.curror.item/
           //3.接上van.<authority>,<path>
           else -> null
       }
     
   ```

### 通知
   ```kotlin
   val manager=getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager//1.先取得一个NotificationManager实例
   //2.创建通知渠道
   if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){//判断安卓版本是否大于安卓O,O之后才有通知渠道
               val channel=NotificationChannel("normal","Normal",NotificationManager.IMPORTANCE_DEFAULT)
               val channel1=NotificationChannel("normal1","Normal",NotificationManager.IMPORTANCE_HIGH)
               //用NotificationChannel实例来创建一个ID为normal的通知渠道，name参数用来在系统中显示通知渠道的name，通知渠道只在第一次创建，
               //创建好后靠识别id来确认有没有创建过，使用id不能重复，name可以重复，NotificationManager.IMPORTANCE_DEFAULT为通知级别，现在是最低
               manager.createNotificationChannel(channel)
               manager.createNotificationChannel(channel1)
           }
   //3. 建立通知
   val intent=Intent(this,NotificationActivity::class.java)//创建一个跳转意图
               val pi=PendingIntent.getActivity(this,0,intent,0)//先通过getActivity()传入intent来获取实例
               //看具体是谁发通知，根据情况使用getActivity()或getService()或getBroadcast()
               //PendingIntent.getActivity(Context,一般用不到直接0,Intent,pi的行为)
               val notification=NotificationCompat.Builder(this,"normal")//传入context和通知渠道的id
                   .setContentTitle("Title")
                   .setContentText("This is content ! This is content ! This is content ! This is content ! This is content ! This is content !")
                   .setSmallIcon(R.drawable.ic_launcher_background)
                   .setLargeIcon(BitmapFactory.decodeResource(resources,R.drawable.ic_launcher_background))
                   .setContentIntent(pi)//设置意图，当点击通知的时候启动意图
                   .setAutoCancel(true)//执行完意图后通知还在，需要用setAutoCancel(true)清除通知
                   .build()
               manager.notify(1,notification)//这里的id是通知id，如果重复则表示同一个id，比如同一个人发来的信息进行更新用同id
   //setStyle()补充
   .setStyle(NotificationCompat.BigTextStyle().bigText("This is content ! This is content ! This is content ! This is content ! This is content ! This is content !"))
   //使用setStyle(NotificationCompat.BigTextStyle().bigText()使文本空间换行显示全部文本，用contentText的话只会显示一行后面用...省略
   .setStyle(NotificationCompat.BigPictureStyle().bigPicture(BitmapFactory.decodeResource(resources,R.drawable.my)))
   //setStyle(NotificationCompat.BigPictureStyle().bigPicture()来进行图片通知，需要先用BitmapFactory.decodeResource()转换bitmap对象
   ```

   ### assets和raw

```
assets目录创建在app/src/main中，于java、res目录平级
raw目录创建在res路径下，由于VideoView控件不支持assets下的视频资源播放，所以视频文件一般存放在raw中
```

![创建位置](image/屏幕快照 2020-11-09 上午9.00.32.png)

```kotlin
//assets下资源的使用
val assetManager=assets//使用getAssets()获取实例，该实例可读取路径下任何资源
val fd=assteManager.openFd("music.mp3")//openFd()打开文件句柄
//文件I/O中，要从一个文件读取数据，应用程序首先要调用操作系统函数并传送文件名，并选一个到该文件的路径来打开文件。该函数取回一个顺序号，即文件句柄（file handle），该文件句柄对于打开的文件是唯一的识别依据。
```

```kotlin
//raw可以像res目录下其他资源一样直接使用R.raw.xxx.mp4来使用
```

```
*res/raw和assets的相同点：两者目录下的文件在打包后会原封不动的保存在apk包中，不会被编译成二进制。    *res/raw和assets的不同点：
 1.res/raw中的文件会被映射到R.java文件中，访问的时候直接使用资源ID即R.id.filename；assets文件夹下的文件不会被映射到R.java中，访问的时候需要AssetManager类。
 2.res/raw不可以有目录结构，而assets则可以有目录结构，也就是assets目录下可以再建立文件夹    
*读取文件资源：    
1.读取res/raw下的文件资源，通过以下方式获取输入流来进行写操作       
InputStream is = getResources().openRawResource(R.id.filename);        
2.读取assets下的文件资源，通过以下方式获取输入流来进行写操作       
assets文件夹里面的文件都是保持原始的文件格式，需要用AssetManager以字节流的形式读取文件。
 1. 先在Activity里面调用getAssets()来获取AssetManager引用。
 2. 再用AssetManager的open(String fileName, int accessMode)方法则指定读取的文件以及访问模式                 就能得到输入流InputStream。 
 3. 然后就是用已经open file 的inputStream读取文件，读取完成后记得inputStream.close()。
 4.调用AssetManager.close()关闭AssetManager。
```

### 多线程

1. 创建线程

```kotlin
//定义线程的方法1
class MyThread : Thread(){//创建一个继承线程的类
  override fun run(){
    //编写具体逻辑
  }
}
//调用
MyThread.start()
//定义线程的方法2
class MyThresad :Runnable{//或是创建一个实现Runnable接口的类
    override fun run(){
    //编写具体逻辑
  }
}//调用
val myThread=Mythread()
Thread(myThread).start()
//定义线程的方法3
Thread{//使用Lambda的方式  
}.start()
//定义线程的方法4
thread{//kotlin提供了一个thread的顶层函数
}
```

2. 在子线程中更新UI

   - 异步消息处理机制

    ```kotlin
   Android不允许在子线程中更新UI，但可以让子线程通过Message让UI线程去更新
   1.Message
   	Message是在线程中传递消息，他可以在内部携带少量信息，用于在不同线程之间传递数据。
   	what,arg1,arg2字段传Int,obj字段传Object对象，当然都只能传一个
   2.Handler
   	Handler意为处理者，它主要是用于发送和处理消息
   	发送一般是使用sendMessage()、pust()等方法，
   	而发出发消息经过一系列的辗转处理后最终会传递到Handle的handleMessage()方法中
   3.MessageQueue
   	MessageQueue是消息队列，它主要用于存放所有通过Handler发送的消息。
   	这部分消息会一直存在于消息队列中，等待被处理。
   	每一个线程只会有一个MessageQueue对象。
   4.Looper
   	Looper是每个线程中MessageQueue的管家，调用Looper的loop()方法后，就会进入一个无限循环中，
   	然后每当发现MessageQueue中存放一条消息时，就将它取出，并传递到Handle的handleMessage()方法中
   	每个线程只会有一个Looper对象
    ```

   - AsyncTask

    ```kotlin
   Andorid对异步消息处理机制进行了封装，使用抽象类AsyncTask简单地进行上面的操作
   1.创建子类来继承抽象类AsyncTask
    ```

	class DownlondTask:AsyncTask<Unit,Int,Boolean>(){}
   	AsyncTask<Prams,Progress,Result>()中指定范型
   	Params:在执行AsyncTask时需要传入的参数类型
   	Progress:在后台执行任务时的进度的数据类型，以此类型作为进度单位
   	Result:任务执行完后需要返回的数据类型
   	AsyncTask<Unit,Int,Boolean>()
   	表示该AsyncTask执行时不需要传入参数，进度以Int作为单位，返回值类型为boolean
   2.重写AsyncTask的常用方法
   	onPreExecute():这个方法会在后台任务开始执行之前调用，用于进行一些界面上的初始化操作，比如显示一个进度条对话框
   	onInBackground(Params...):这个方法中的所有代码都会在子线程中运行，我们应该在这里处理所有的耗时任务
   	任务一旦完成，就可以通过return语句返回结果，如果AsyncTask的第三个参数为Unit，就可以不用返回。
   	onProgressUpdate(Progress...):当后台任务调用了publishprogress(Progress..)方法后，
   	onProgressUpdate(Progress...)方法就会很快被调用到，该方法中可以进行UI操作，可以根据参数值更新UI
   	onPostExecute(Result):当后台任务执行完毕并通过return进行返回后，这个方法会很快被调用，返回的值会被传到此方法中
   	可以根据返回数据进行UI操作，比如提醒任务结果，关闭对话框
   3.调用
   	DownloadTask().execute()//execute()中是可以传递参数的  
   
    ```
   
    ```


### Service

```kotlin
创建好Service类后除了必须要重写的onBind()方法，一般还会重写onCreate(),onStartCommand(),onDestroy
onCreate():在Service创建时调用，只会被创建一次，一个Service只会存在一个实例
onStartCommand():在每次Service启动的时候调用，一般我们需要Service执行什么动作就写在这里
onDestroy():在Service被销毁的时候调用，用于回收那些不再使用的资源
```

```kotlin
val intent=Intent(this,MyService::class.java)//先创建意图，创建方式同Activity
startService(intent)//启动Service
stopService(intent)//停止Service
```

Activity和Service进行通信

```kotlin
这时候就用到了那个必须重写的onBind()方法，通过onBind()方法给Servie绑定Activity
Service中建立Binder()的对象，再在onBind()中返回
	 private val mBinder=DownloadBinder()
   override fun onBind(intent: Intent): IBinder {
     return mBinder
    }
Activity中
先创建SercviceConnection的匿名类实现，并重写onServiceConnected()和onServiceDisconnected
	 private val connection =object :ServiceConnection{
        override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
          	downloadBinder=service as MyService.DownloadBinder//创建Binder实例
            downloadBinder.startDdownload()//有了实例就能使用该类的所有public方法
            downloadBinder.getProgress()
        }
     		//绑定成功时调用
        override fun onServiceDisconnected(name: ComponentName?) {}
     		//只有在Service创建进程崩溃或被杀掉时候调用
    }
再去执行绑定操作
 	 val intent=Intent(this,MyService::class.java)
   bindService(intent,connection,Context.BIND_AUTO_CREATE)//绑定Service，会回调Service里的onBind()
	 unbindService(connection)//解绑
bindService(Intent,SercviceConnection,标志位)
像Context.BIND_AUTO_CREATE就是在绑定后自动创建Service不需要再去执行一次startService(intent)
使用bindService()打开的Service要用unbindService()才能销毁
如果一个Service用了bindService()又用了startService()那么必须同时调用unbindService()和stopService()才能销毁
```

前台Service

```kotlin
先创建通知，因为前台服务需要通知来保持前台
原本通过manager.notify(1,notification)的方式生成通知
改为startForeground(1, notification)就变成了那种不能滑动关闭的前台服务通知
从安卓9.0开始要去manifest里进行权限声明
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
```

IntentService

```kotlin
Service无法处理一些耗时操作，前台服务20秒进入ANR，后台服务200秒，所以使用IntentService来创建额外线程处理耗时操作。当然也可以直接在Service中开线程，但不推荐。
1.首先创建一个继承IntentServiced的类
class MyIntentService:IntentService("MyIntentService"){}//父类构造函数里需要传入一个字符串，内容无所谓，只在调试的时候有用
2.重写抽象方法onHandleIntent()
override fun onHandleIntent(intent: Intent?) {
        //代码会在子线程中运行
    }
3.Activity中
  val intent=Intent(this,MyIntentService::class.java)
  startService(intent)
4.在manifest中声明Service
<service android:name=".MyIntentService"
            android:enabled="true"
            android:exported="true"/>
```

### 网络

webview的使用

```kotlin
webview.settings.javaScriptEnabled=true//打开js支持
webview.webViewClient= WebViewClient()//给webview传入一个WebViewClient()实例，
// 当我们进入网页跳转其他网页时，有这个实例就可以进行在当前activity中打开网页而不是跳转浏览器
webview.loadUrl("https://www.baidu.com")//传入网址
```

HTTP的使用

```kotlin
1.使用HttpURLConnection
	首先获取HttpURLConnection()实例
	val url = URL("https://www.baidu.com")
	val connection = url.openConnection() as Http URL Connection
	然后设置HTTP请求所使用的方法，主要是GET和POST。
	connection.requestMethod = "GET"
	下一步可以进行一些自定义操作，比如设置连接超时，读取超时的毫秒数，以及服务器希望得到的一些信息头。
	connection.connectTimeout = 8000
	connection.readTimeout = 8000
	之后调用getInputStream()方法就可以获取到服务器返回的输入流，并读取数据流
	val input= connection.inputStream
	val reader = BufferedReader(InputStreamReader(input))
	最后使用disconnect()关闭连接
	connection.disconnect()
2.OkHttp
	OkHttp项目主页: https://github.com/square/okhttp
	先在app/gradle里添加依赖
	dependencies{
    implementation 'com.squareup.okhttp3:okhttp:4.1.0'
  }
	创建实例
	val client = OkHttpClient()
	如果想发起一条HTTP请求，就需要创建一个request对象,一般GET直接写，POST看下面
	val request = Request.Builder().build()//这样仅仅是一个空对象
	val request = Request.Builder().url("https://www.baidu.com").build()//在bulid()之前可以对对象进行完善
	val response = client.newCall(request).execute()//创建Call()对象，调用它的execute()发送请求并获取服务器返回的数据
	val responseData = response.body?.string()//Response()对象就是服务器返回的数据
	如果是发起POST请求。则需要先构建一个Request Body对象来存放待提交的参数
	val requestBody = FormBody.Builder().add("usernaem","admin").add("password","123456").build()
	然后在Request.Builder中调用一些post()方法，并将RequestBody对象传入：
	val request = Request.Builder().url("https://www.baidu.com").post(requestBody).build()
	接下来就跟GET一样用Call对象发送请求
```

Android9.0后不再支持http传输的访问，需要在res下创建xml的directory，里面建个network_config.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
		<base-config cleartextTrafficPermitted = "true">
  			<trust-anchors>
      			<certifiates src="system"/>
      	</trust-anchors>
  	</base-config>
</network-security-config>
```

然后在manifest中<applicaation的属性中启用

```xml
<applicaation
       ...
       android:networkSercurityConfig="@xml/network_config">
</applicaation>
```

#### JSON解析

1. 使用JSONObject

   ```kotlin
   try{
     val jsonArray = JSONArray(jsonData)//json以数组形式存放,将json数据用数组对象存放
     for(i in 0 until jsonArray.length){//遍历json数组的所有项
       val jsonObject = jsonArray.getJSONObject(i)//获取当前数组内容
       val id = jsonObject.getString("id")//提取当前元素中键为“id”的值
       val name = jsonObject.getString("name")
       val version = jsonObject.getString("version")
     }
   }
   ```

2. 使用GSON

   ```kotlin
   先添加支持：implementation 'com.google.code.gson:gson:2.8.5'
   如JSON数据如下{"name":"tom","age":20}
   我们定义一个Person类带上name,age这两个参数
   val gson =Gson()
   val person = gson.fromJson(jsonData, Person::class.java)
   JSON数据就自动序列化成为一个person对象
   当JSON数据中存在多个对象数据时，如{"name":"tom","age":20},{"name":"mary","age":19}
   使用TypeToken来存放多个person对象
   val typeOf= object : TypeToken<List<person>>(){}.type
   val person = gson.fromJson<List<Person>>(jsonData, typeOf)
   ```

#### XML解析

1. Pull解析方式

   ```kotlin
   try{
   val factory = XmlPullParserFactory.newInstance()
   val xmlPullParser = factory.newPullParser()//创建实例
   xmlPullParser.setInput(StringReader(xmlData))//setInput设置要解析的xml
   var eventType = xmlPullparser.eventType//获取当前的解析事件
   var id = ""
   var name = ""
   var version = ""
   while(eventType != XmlPullParser.END_DOCUMENT){//遍历
     val nodeName = xmlPullParper.name//获取当前节点的名字
     when(eventType){
       //开始解析某个节点
       XmlPullParser.START_TAG -> {
         when(nodeName){//判断当前节点
           "id" -> id= xmlPullParser.nextText()
           "naem" -> name = xmlPullParser.nextText()
           "version" -> version = xmlPullParser.nextText()
         }
       }
       //完成解析某个节点
       XmlPullParser.END_TAG -> {
   			if("app" == nodeName){//这里的app是xml的大节点名称，一个app下面有id,name,version三个小节点
           log.d("MainActivity","id is $id")
         }//
       }
     }
     eventType = xmlPullParser.next()//进入下一个节点
   }
   }
   ```

2. SAX解析方式

   ```kotlin
   使用SAX解析通常要创建一个继承DefaultHandler的类，并重写父类的5个方法
   class MyHandler :DefaultHandler() {
       val nodeName = ""
       lateinit var id :StringBuilder
       lateinit var name :StringBuilder
       lateinit var version :StringBuilder
     	//在开始解析XML时调用
       override fun startDocument() {
         //里面可以进行数据初始化
           id=StringBuilder()
           name=StringBuilder()
           version=StringBuilder()
       }
   		//在开始解析某个节点的时候调用
       override fun startElement(
           uri: String?,
           localName: String?,
           qName: String?,
           attributes: Attributes?
       ) {
         nodeName=localName//获取节点名
         //可以打印日志来观察解析进度
       }
   		//在获取节点内容的时候调用，可能会被多次调用
       override fun characters(ch: CharArray?, start: Int, length: Int) {
         //根据节点名判断内容应该添加到哪个StringBuilder中
         when(nodeName){
           "id" -> id.append(ch,start,length)
         }
       }
   		//在某个节点解析结束时调用
       override fun endElement(uri: String?, localName: String?, qName: String?) {
         if("app" == localName)//判断是不是大节点解析完毕
         {	
           //输出数据到数据库等地方
           id.setLength(0)//输出完后清空StringBuilder才能保存下一个节点的数据
         }
       }
   		//解析完成整个xml时调用
       override fun endDocument() {
       }
   }
   ```

  #### 网络请求的回调

   ```kotlin
   网络请求常常时多次的，但代码是一样的，使用可以封装到方法里，但网络请求属于耗时操作需要开子线程，但方法在发送完请求后就执行完了，并不会有人来接收返回到数据。所以需要回调机制。
   先定义一个接口
   interface HttpCallbackListener{
     fun onFinish(responce : String)
     fun onError(e:Exception)
   }
   修改HttpURLConnection的发送请求函数
   fun sendHttpRequest(address: String, listener: HttpCallbackListener) {
           thread {
               var connection: HttpURLConnection? = null
               try {
                   val response = StringBuilder()
                   val url = URL(address)
                   connection = url.openConnection() as HttpURLConnection
                   connection.connectTimeout = 8000
                   connection.readTimeout = 8000
                   val input = connection.inputStream
                   val reader = BufferedReader(InputStreamReader(input))
                   reader.use {
                       reader.forEachLine {
                           response.append(it)
                       }
                   }
                   // 回调onFinish()方法，由于子线程中无法返回，所以靠onFinish()返回数据
                   listener.onFinish(response.toString())
               } catch (e: Exception) {
                   e.printStackTrace()
                   // 回调onError()方法
                   listener.onError(e)
               } finally {
                   connection?.disconnect()
               }
           }
       }
   //修改OkHttp的发送请求函数，OkHttp自带回调接口okhttp3.Callback
   fun sendOkHttpRequest(address: String, callback: okhttp3.Callback) {
           val client = OkHttpClient()
           val request = Request.Builder()
               .url(address)
               .build()
           client.newCall(request).enqueue(callback)//enqueue会自己开好子线程
       }
   ```

#### Retrofit

项目主页：https://github.com/square/retrofit

Retrofit可以配置好一个根路径，然后在指定服务器地址时使用相对路径，并且允许我们自行对服务器接口进行分类，将功能属于同一类的服务器接口定义到同一个接口文件中，可以通过注解的方式指定服务器接口提供所需参数

先添加依赖

```kotlin
implementation 'com.squareup.retrofit2:retrofit:2.6.1'
implementation 'com.squareup.retrofit2:converter-gson:2.6.1'
```

定义一个App类

```kotlin
class App(val id :String,val name :String,val version :String)
```

新建AppService接口

```kotlin
interface AppServicce{//通常Retrofit的接口都以文件名开头，Service结尾
  @GET("get_data.json")//当调用下面的getAppData()时发生一条GET请求，()里是请求内容
  fun getAppData() : Call<List<App>>//类型必须是Call，再通过范型来指定服务器数据该转换成什么类型的对象
}
```

调用

```kotlin
//先创建实例，用baseUrl指定跟路径，addConverterFactory指定解析数据用的转换库，两个都是必须实现的
val retrofit = Retrofit.Builder().baseUrl("http://10.0.2.2/").addConverterFactory(GsonConverterFactory.create()).build()
val appService = retrofit.create(AppService::class.java)//使用retrofit.creat传入具体Servvice对应的类来创建一个该接口的动态代理对象。，用它来使用接口的任意方法
//使用getAppData()会返回一个Callback<List<App>>再调用call对象的enqueue函数，retrofit就会根据注解配置服务器接口地址，服务器响应的数据会回调到enqueue函数中再传入Callback实现里面。发送请求时retrofit会自己开线程，callback后自动回到主线程
appService.getAppData().enqueue(object:Callback<List<App>>{
  //这里就是Callback中，下面重写函数的参数里都有传入响应回来的callback数据
  override fun onResponse(call : Call<List<App>>,response:Response<List<App>>){
    val list=response.body()
    if(list != null){
      for(app in list){
        log.d("MainActivity","id is ${app.id}")
      }
    }
  }
  override fun onFailure(call:Call<List<App>>,t:Throwable){
    t.printStackTrace()
  }
})
```

处理复杂的接口地址类型

```kotlin
上面的接口是静态的，而实际开发环境中的接口是千变万化的。
先定义个Data类包含id和content字段
class Data(val id :Int,val content :String)

GET http://example.com/<page>/get_data.json
<page>部分代表页数，我们传入不同的页数返回的数据自然也不同
实现接口：
interface ExampleService {
	@GET("{page}/get_data.json")//使用{page}占位符，再在getData()里添加page参数，
  fun getData(@Path("page")page:Int):Call<Data>//使用@Path("page")注解来声明这个参数
}

GET http://example.com/get_data.json?u=<user>&t=<token>
//也可以使用上面占位符形式处理，但Retrofit提供了专门的语法支持
interface ExampleService {
  @GET("get_data.json")
  fun getData(@Query("u")user:String,@Query("t")token:String):Call<Data>
  //使用@Query注解进行声明，执行getData时就会转换成get_data.json?u=<user>&t=<token>的格式
}
```

请求类型

| 请求名称 | 请求功能           | 注解    |
| -------- | :----------------- | ------- |
| GET      | 从服务器获取数据   | @GET    |
| POST     | 向服务器提交数据   | @POST   |
| PUT      | 修改服务器上的数据 | @PUT    |
| PATHC    | 修改服务器上的数据 | @PATCH  |
| DELETE   | 删除服务器上的数据 | @DELETE |

```kotlin
DELETE http://example.com/data/<id>
interface ExampleService{
  @DELETE("data/{id}")
  fun de;eteData(@Path("id") id :String):Call<ResponseBody>
}
使用GET时我们指定了Call的类型为Data类，而除GET外的请求都是对服务器的数据进行操作，而不是从服务器获取数据，所以它们通常对服务器响应的数据并不关心。这时候使用ResponseBody，表示能够接收任意类型的响应数据，并且不会的响应数据进行解析。

POST http://example.com/data/create{"id":1,"content":"this data"}
interface ExampleService{
  @POST("data/create")
  fun careatData(@Body data :Data):Call<ResponseBody>
}
声明了一个Data类型的参数并加上注解@Body,在发送POST时，会自动将data对象序列化成JSON，并放到HTTP请求的body部分。其他请求也支持这样发送。

GET http://example.com/get_data.json
User-Agent:okhttp
Cache-Control:max-age=0
有些服务器需要我们在HTTP请求中指定header参数
interface ExampleService{
  @Headers("User-Agent : okthhp","Cache-Control : max-age=0")
  @GET("get_data.json")
  fun getData():Call<Data>
}
@Headers是静态的写法，把参数值直接写死
使用@Header来动态指定参数
interface ExampleService{
  
  @GET("get_data.json")
  fun getData(@Header("User-Agent")usetAgebt:String,@Header("Cache-Control")cacheControl:String):Call<Data>
}
```

```kotlin
Retrofit的最佳写法
原本写法
val retrofit =Retrpfit.Builder().baseUrl("http://10.0.2.2/").addConverterFactory(GsonConverterFactory.creat()).build()
val appService = retrofit.creat(AppService::class.java)
创建retrofit实例的代码是一样的，而在不同类里面每次都需要重新创建，所以把它封装成一个单例类
新建ServiceCreator单例类
object ServiceCreator {

    private const val BASE_URL = "http://10.0.2.2/"

    private val retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(GsonConverterFactory.create())
        .build()

    fun <T> create(serviceClass: Class<T>): T = retrofit.create(serviceClass)
	//这时候的调用	val appService=ServiceCreator.creat（AppService::class.java)
    inline fun <reified T> create(): T = create(T::class.java)
	//加上范型之后的调用val appService=ServiecCreator.creat<AppService>()
}
```

### Material Design

#### toolbar

```kotlin
需要在没有actionbar的主题下使用
宽度需通过attr设为actionBarSize
android:layout_height="?attr/actionBarSize"
最后在Activity中使用setSupportActionBar设为该Activity的ActionBar
setSupportActionBar(toolbar)
其中菜单的使用，菜单布局的item可以通过app:showAsAction="always"属性显示到外面来
always:表示会一直显示
ifRoom:表示空间足够大的时候显示
never:表示只会显示在菜单列表里
使用onCreateOptionsMenu创建菜单
使用onOptionsItemSelected实现菜单功能
```

#### DrawerLayout

```kotlin
DrawerLayout作为容器，里面只能放置两个直接子控件。其中第二个字控件是放在滑出菜单里的，这个控件必须要指定android:layout_gravity，一般指定start，并且默认为深色透明背景
写好布局后就能直接从左边滑出，但还是需要提供个按钮让用户知道这边可以滑出菜单，在设置完actionbar后加上
supportActionBar?.let{//在supportActionBar存在时
            it.setDisplayHomeAsUpEnabled(true)//显示导航按钮
            it.setHomeAsUpIndicator(R.drawable.ic_baseline_menu_open_24)//设置导航按钮图标
        }
并在onOptionsItemSelected中加上
android.R.id.home ->drawerLayout.openDrawer(GravityCompat.START)//打开滑动菜单
drawerLayout.addDrawerListener(object :DrawerLayout.DrawerListener{//添加状态监听器，重写四个方法
            override fun onDrawerStateChanged(newState: Int) {
                TODO("Not yet implemented")
            }

            override fun onDrawerSlide(drawerView: View, slideOffset: Float) {
                TODO("Not yet implemented")
            }

            override fun onDrawerClosed(drawerView: View) {
                TODO("Not yet implemented")
            }

            override fun onDrawerOpened(drawerView: View) {
                TODO("Not yet implemented")
            }
        })
```

#### NavigationView

```kotlin
添加依赖
    implementation "com.google.android.material:material :1.1.0"
    implementation "de.hdodenhof:circleimageview:3.0.1"
然后将style.xml中的主题改成Theme.MaterialComponents.NoActionBar
再准备好一个menu和headerLayout，menu用来显示具体的菜单项，headerLayout用来显示头部布局
在<com.google.android.material.navigation.NavigationView中
app:menu="@menu/nav_meun"
app:headerLayout="@layout/nav_header"
来指定menu和header
给nav_meun添加点击事件
navView.setCheckedItem(R.id.navCall)//设置默认选择项，因为我们的menu的item设计的是一组单选按钮
navView.setNavigationItemSelectedListener { 
            drawerLayout.closeDrawers()//这里关闭菜单，也可以用when来实现具体逻辑
            true//返回true表示事件已经处理过来
        }
```

#### FloatonActionButton

```
可以通过app:elevation="8dp"设定悬浮高度，会模拟悬浮高度进行阴影显示
其他用法和普通Button一样
```

#### Snackbar

```
底部提示控件，就是删除图片时的撤销所用的控件，并不直接在布局上使用，而是像Toast一样使用
 fab.setOnClickListener { view->
            Snackbar.make(view,"提示信息",Snackbar.LENGTH_SHORT).setAction("撤销"){
                Toast.makeText(this, "撤销成功", Toast.LENGTH_SHORT).show()
            }.show()
        }
需要传入一个view给Snackbar使用，这个view只要是当前页面存在的任意view都可以，Snackbar会直接通过view找到最外层的布局来展示信息        
使用setAction来使用自带的按钮
```

#### CoordinatorLayout

```kotlin
上面的Snackbar弹出时会挡住fab按钮，如果我们使用CoordinatorLayout来代替帧布局就不会又这个问题
CoordinatorLayout在普通情况下和帧布局基本一致，但它有些额外功能
CoordinatorLayout可以监听其所有的子控件的各种事件，并自动做出最合理的响应
使用上也和帧布局一样
虽然Snackbar并不是CoordinatorLayout的子控件，但Snackbar触发用的view如果是它的子控件同样也能被监听到
```

#### 卡片式布局

```kotlin
MaterialCardView 使用app:cardCornerRadius="4dp"改圆角 他也有adnroid:elevatoin和app:cardElevation阴影深度的属性
```

#### AppBarLayout

```
toolbar会被RecyclerView挡住，使用AppBarLayout来让toolbar下滚的时候出现，上滑的时候消失
先把toolbar嵌套到AppBarLayout中
再给RecyclerView指定一个布局行为
在布局文件添加app:layout_behavior="@string/appbar_scrolling_view_behavior"
appbar_scrolling_view_behavior字符串也是由Material提供的
当RecyclerView发生滚动事件时就会去通知AppBarLayout,收到通知时子控件通过app:layout_scrollFlags进行响应
app:layout_scrollFlags="scroll|enterAlways|snap"
scroll表示当被观察对象向上滚动时,子控件会跟着上滚并隐藏
enterAlways表示当被观察对象向下滚动时,子控件会跟着下滚并重新显示
snap表示当子控件没有完全隐藏或显示时，根据当前滚动的距离自行判断是否隐藏显示
```

#### SwipeRefreshLayout

```kotlin
下滑刷新用的控件，嵌套到需要进行下滑刷新控件的外面一层，如果和AppBarLayout一起使用，那RecyclerView里面的app:layout_behavior属性就应该移到它里面
简易使用
swipeRefreshLayout.setColorSchemeResources(R.color.purple_500)//设置下拉刷新进度条颜色
swipeRefreshLayout.setOnRefreshListener { //下拉刷新监听器
            refreshFruits(adapter)
        }
private fun refreshFruits(adapter:FruitAadpter){
        thread{//通常情况刷新是去网络上请求最新数据，所以需要开线程来处理耗时操作
            Thread.sleep(2000)
            runOnUiThread {
                initFruitList()
                adapter.notifyDataSetChanged()//通知数据发生变化来刷新视图
                swipeRefreshLayout.isRefreshing = false//表示刷新事件结束，并隐藏刷新进度条
            }
        }
    }

```

#### 可折叠式标题栏

```xml
CollapsingToolbarLayout是一个作用于Toolbar基础上的布局，他也由Material库提供。
CollapsingToolbarLayout不能独立存在，只能作为AppBarLayout的直接子布局来使用，
而AppBarLayout又必须是CoordinatorLayout的子布局
<androidx.coordinatorlayout.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"> 表示该控件会延伸到状态栏，比如我们需要让图片延伸到状态栏那它的所以父布局都要加上android:fitsSystemWindows="true"，并且在主题中新建一个主题
  <style name="FruitActivityTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <item name="android:statusBarColor">@android:color/transparent</item>
    </style>
  来给需要的Activity使用

    <com.google.android.material.appbar.AppBarLayout
        android:id="@+id/appBar"
        android:layout_width="match_parent"
        android:layout_height="250dp"
        android:fitsSystemWindows="true">

        <com.google.android.material.appbar.CollapsingToolbarLayout
            android:id="@+id/collapsingToolbar"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar" 指定为ActionBar的主题
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorPrimary"  指定控件趋向于折叠以及折叠之后的颜色
            app:layout_scrollFlags="scroll|exitUntilCollapsed">
			scroll:表示会随着别人一起滚动
      exitUntilCollapsed：表示随着滚动完成就自动折叠保留在屏幕上，不再移出屏幕
            <ImageView
                android:id="@+id/fruitImageView"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scaleType="centerCrop"
                android:fitsSystemWindows="true"
                app:layout_collapseMode="parallax" />  parallax表示折叠过程中位置发生偏移

            <androidx.appcompat.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin" />    pin表示折叠过程中位置不变

        </com.google.android.material.appbar.CollapsingToolbarLayout>
       </com.google.android.material.appbar.AppBarLayout>

    <androidx.core.widget.NestedScrollView  自带滚动响应的ScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"> 指定滚动事件

        <LinearLayout
            android:orientation="vertical"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="15dp"
                android:layout_marginLeft="15dp"
                android:layout_marginRight="15dp"
                android:layout_marginTop="35dp"
                app:cardCornerRadius="4dp">

                <TextView
                    android:id="@+id/fruitContentText"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_margin="10dp" />

            </com.google.android.material.card.MaterialCardView>

        </LinearLayout>

    </androidx.core.widget.NestedScrollView>

    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:src="@drawable/ic_launcher_foreground"
        app:layout_anchor="@id/appBar"   指定一个锚点
        app:layout_anchorGravity="bottom|end" /> 自己相对于锚点点位置
      </androidx.coordinatorlayout.widget.CoordinatorLayout>
```

### jetpack

#### VeiwModel

```kotlin
添加依赖
implementation 'androidx.lifecycle:lifecycle-extensions:2.2.0'
 viewmodel = ViewModelProvider(this).get(MainViewModel::class.java)//创建实例
//第一个参数为你的Activity或Fragment实例，第二个参数为你的Viewmodel实例
接下去使用viewmodel.就能调用viewmodel中的参数

如果需要有带参数的构造函数，比如通过构造函数传入之前app退出保存的值
class MainViewModel(countReserved :Int):ViewModel() {
    var counter=countReserved
}修改构造函数
那么就要向构造函数传递数据，那就需要一个Factory类，创建类实现ViewModelProvider.Factory接口
class MainViewModelFactory(private val countReserved:Int):ViewModelProvider.Factory {
    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
      return MainViewModel(countReserves) as T//创建VM实例并传入参数
    }
}
在他的构造函数里传入我们需要传入viewmodel的值，再在creat中返回我们创建好的带参实例
创建带参实例
 viewmodel=ViewModelProvider(this,MainViewModelFactory(countReserved)).get(MainViewModel::class.java)  

//新创建方式1
先添加implementation 'androidx.fragment:fragment-ktx:1.3.0-beta01'如果仅在activity中使用可以不加
然后app build.gradle android下添加    
kotlinOptions {
        jvmTarget=1.8
    }
private val galleryViewModel by viewModels<GalleryViewModel> ()
val galleryViewModel by activityViewModels<GalleryViewModel> ()
//也能创建这种activityViewModel给多个Fragment使用，同一个activity中的不同fragment可以使用这种方式共享ViewModel，而普通的在fragment中创建出来的只能作用在fragment中，因为viewmodel是单例的，每一次创建的都不会是同一个
//新创建方式2
    private val galleryViewModel by lazy { ViewModelProvider(this,ViewModelProvider.AndroidViewModelFactory(requireActivity().application)).get(GalleryViewModel::class.java) }
   
```

#### Lifecycles

````kotlin
可以让一个非Activity类感知Activity的生命周期
如果使用手写监听器
class MyObserver{
  fun activityStart(){}
  fun activityStop(){}
}
然后再在Activity的生命周期中调用MyObserver中的方法来判断生命周期状态
如果使用Lifecycles
class MyObserver : LifecycleObserver{//实现LifecycleObserver接口
  @OnLifecycleEvent(Lifecycle.Event.ON_START)
  fun activityStart(){
    Log.d("MyObeserver","activityStrart")
  }
  @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
  fun activityStop(){}
}
让类实现接口，然后添加@OnLifecycleEvent注解，在对应的生命周期执行时会调用被注解的方法
当然写到现在还没有去指定监听的具体是哪个Activity
在Activity中添加lifecycle.addObserver(MyObserver())
全名应该是lifecycleOwner.lifecycle.addObserver(MyObserver())
但实际中我们不需要去实现lifecycleOwner，Activity等被观察对象如果是androidx版本那它自己本身就是个lifecycleOwner
到现在MyObserver能知道生命周期的变化了，但还不能主动获取生命周期
修改构造函数
class MyObserver(val lifecycle: Lifecycle) : LifecycleObserver {}
传入一个lifecycle参数，之后就能使用lifecycle.currentState来主动获取
lifecycle.currentState返回的状态是个枚举类型
共有5种状态类型来对应生命周期
````

#### LiveData

liveData是Jetpack提供的一种响应式编程组件，他可以包含任何类型的数据，并在数据发生变化是通知给观察者。绝大多数情况下在ViewModel里使用。

```kotlin
修改ViewModel，把counter变量改为LiveData实例
class MainViewModel(countReserved :Int):ViewModel() {
    var counter=MutableLiveData<Int>()//改为MutableLiveData对象
    //MutableLiveData是一种可变的LiveData，主要有
    //getValue:用于获取LiveData里的数据
    //setValue:给LiveData设置数据，但只能在主线程中调用
    //postValue://在非主线程里给LiveData设置数据
    init {
        counter.value=countReserved//kitlin自动转换的get/set
    }
    fun plusOne(){
        val count=counter.value?:0//getValue可能为空要进行非空判断
        counter.value=count + 1
    }
    fun clear(){
        counter.value=0
    }
}
Activity中
button.setOnClickListener {
            viewmodel.plusOne()//计算逻辑直接写在ViewModel里
        }
button2.setOnClickListener {
            viewmodel.clear()
        }
viewmodel.counter.observe(this, Observer { count->//当这样参数count发生变化是调用下面函数体
            textView.text=count.toString()//控件的刷新还是有Activity来执行
        })
但现在也有一个问题，如果是他人封装的ViewModel。那我们不知道它的参数叫count
在viewmodel声明count为外部不可见
    val counter: LiveData<Int>//声明一个不可变的LiveDate用它的get方法发送_counter
        get() = _counter
    private val _counter=MutableLiveData<Int>()
    init {
        _counter.value=countReserved//kitlin自动转换的get/set
    }
    fun plusOne(){
        val count=_counter.value?:0//getValue可能为空要进行非空判断
        _counter.value=count + 1
    }
    fun clear(){
        _counter.value=0
    }
这样我们实际的实例名为_counter，而外部调用是使用的是counter，也无法同过counter来获取执行逻辑操作后的数据，增强封装性，View部分不需要知道我们的参数数值和我们的点击事件具体干了什么逻辑
```

##### map和switchMap

1. map()将实际包含数据的LiveData和仅用于观察数据LiveData进行转换

   当有一个User()的data类，包含用户的姓名和年龄，data类会自动生成

   ```kotlin
   equals()
   hashCode()
   toString()
   componentN()
   copy()
   ```

   ```kotlin
   data class User(val firstName:String,val lastName:String ,val age:Int)
   ```

   在viewModel中创建相应的LiveData来包含User类型的数据

   ```kotlin
   val userLiveData = MutableLiveData<User>()
   ```

   此时，如果要让activity知道User的名字，而不知道年龄，那我们就不能把整个User传给activity

   ```kotlin
   private val userLiveData = MutableLiveData<User>()
   val userName:LiveData<String> =Transformations.map(userLiveData){ user->
           "${user.firstName} ${user.lastName}"
           }
   然后activity中观察userName来获取数据
   ```

2. switchMap()

   目前为止使用的LiveData对象都是在ViewModel中创建，如果是其他地方传进来的LiveData，

   新建一个Repository单例类

   ```kotlin
   object Repository {
   
       fun getUser(userId: String): LiveData<User> {
           val liveData = MutableLiveData<User>()
           liveData.value = User(userId, userId, 0)
           return liveData
       }
   
   }
   添加getUser方法接收一个userId参数，此时Repository返回的是一个带一个user对象的LiveData
   每次调用都会创建一个新的LiveData对象
   viewModel中写个get来获取Repository返回的对象
   fun getUser(userId :String):LiveData<User>{
     return Repository.getUser(userId)
   }
   此时我们的LiveData对象就是在外部获取的
   如果ativity中使用
   viewmodel.getUser(userId).observe(this){user ->}
   这当然是不行的，每次getUser(userId)的LiveData对象都不是同一个，而这里写的只能够观察到最老的那个对象
   使用switchMap()把LiveData转化为可观察的对象
    private val userIdLiveData = MutableLiveData<String>()
   
       val user: LiveData<User> = Transformations.switchMap(userIdLiveData) { userId ->
           Repository.getUser(userId)
       }
   
       fun getUser(userId: String) {
           userIdLiveData.value = userId
       }
   Transformations.switchMap()，第一个参数为被要被转换的观察对象。第二个参数为转换函数，这个转换函数需要返回一个LiveData对象
   activity中
   button4.setOnClickListener {
               val userId=(0..1000).random().toString()
               viewmodel.getUser(userId)
           }
           viewmodel.user.observe(this, Observer { user ->//后面的user当前观察的数据对象的实例
               textView2.text=user.firstName
           })
   
   在没有可观察数据的情况下
   private val refreshLiveData = MutableLiveData<Any?>()
   
       val refreshResult = Transformations.switchMap(refreshLiveData) {
           Repository.getUser("")//返回一个空的LiveData
       }
   
       fun refresh() {
           refreshLiveData.value = refreshLiveData.value
       }
   使用refresh()方法时，触发了setValue方法，就算我们的数据没有发生改变，但仍然会通知观察者
   并且由于要减少性能消耗，在avtivity处于不可见状态时，数据发生变化不会进行通知，而是当activity回到可见状态时才通知给观察者，并且之后通知最新的那一次。
   ```

#### Room

Room由Entity、Dao、Database3个部分组成

| Room     | 分工职能                                                     |
| -------- | ------------------------------------------------------------ |
| Entity   | 用于定义封装实际数据类型的实体类，每个实体类都会在数据库有一张对应的表，并且表中的列是根据实体类的字段自动生成的。 |
| Dao      | Dao是数据访问对象的意思，通常会在这里对数据库的各项操作进行封装，在实际编程的时候，逻辑层就不用和底层数据库打交道，直接和Dao层进行交互即可。 |
| Database | 用于定义数据库中的关键信息，包括数据库的版本号、包含哪些实体类、以及提供Dao层的访问实例。 |

```kotlin
添加依赖
implementation 'androidx.room:room-runtime:2.2.5'
kapt 'androidx.room:room-compiler:2.2.5'
要先在上方pluging中声明id 'kotlin-kapt'才能使用kapt指令，如果是java项目则使用annotationProcessor
plugins {
    id 'com.android.application'
    id 'kotlin-android'
    id 'kotlin-kapt'
}
```

1. 先定义实体类

   ```kotlin
   @Entity//声明为实体类
   data class User(var firstName:String, var lastName:String, var age:Int){
       @PrimaryKey(autoGenerate = true)//声明为主键，自动生成主键的值
       val id:Long=0
   }
   ```

2. 创建Dao接口

   ```kotlin
   @Dao//使用类@Dao Room才能将它识别为一个Dao
   interface UserDao {//根据需求对@Insert，@Update，@Query，@Delete进行封装
       @Insert
       fun insertUser(user: User):Long//表达参数传入对user会插入数据库还会将自动生成对ID返回
       @Update
       fun updataUser(newuser: User)//把参数传入对user对照数据库中同id对user进行更新
       @Query("select * from User")
       fun loadAllUsers():List<User>
       @Query("select * from User where age >:age")
       fun loadUserOlderThan(age:Int):List<User>
       @Delete
       fun deleteUser(user: User)//把参数传入对user从数据删除
       @Query("delete from user where lastName = :lastName")
       fun deleteUserByLastName(lastName:String):Int
       //Query必须配合SQL语句使用，如果传入的是实体类参数，那可以直接使用除Query外的那些注解，
       //如果传入一个String等其他参数，那都需要使用Query配合SQL语句来执行
   }
   ```

   

3. 定义Database

   ```kotlin
   @Database(version = 1,entities = [User::class])//声明版本号和实体类，多个实体类用逗号分隔
   abstract class AppDatabase: RoomDatabase() {//一定要声明成抽象类
       abstract fun userDao():UserDao//只需要声明，Room会自动获取之前编写的Dao实例
       companion object{//限制为单例，原则上全局应该只存在一个AppDatabase实例
           private var instance:AppDatabase?=null//用instance来缓存AppDatabase对象
   
           @Synchronized
           fun getDatabase(context: Context):AppDatabase{
               instance?.let {//如果实例已存在就直接返回它本身
                   return it
               }//没有就创建个新的
               return Room.databaseBuilder(context.applicationContext,AppDatabase::class.java,"app_database")
                   .build().apply { instance=this }
           }//Room.databaseBuilder(一定是applicationContext,AppDatabase的类，数据库名称).build()完成构建
       }
   }
   ```

4. 调用

   ```kotlin
           val userDao = AppDatabase.getDatabase(this).userDao()
           val user1 = User("Tom", "Brady", 40)
           val user2 = User("Tom", "Hanks", 63)
           addDataBtn.setOnClickListener {
               thread {
                   user1.id = userDao.insertUser(user1)
                   user2.id = userDao.insertUser(user2)
               }
           }
           updateDataBtn.setOnClickListener {
               thread {
                   user1.age = 42
                   userDao.updateUser(user1)
               }
           }
           deleteDataBtn.setOnClickListener {
               thread {
                   userDao.deleteUserByLastName("Hanks")
               }
           }
           queryDataBtn.setOnClickListener {
               thread {
                   for (user in userDao.loadAllUsers()) {
                       Log.d("MainActivity", user.toString())
                   }
               }
           }
           doWorkBtn.setOnClickListener {
               val request = OneTimeWorkRequest.Builder(SimpleWorker::class.java).build()
               WorkManager.getInstance(this).enqueue(request)
           }
   ```

5. 数据库升级

   ```kotlin
   在Databaes创建的时候使用.fallbackToDestructiveMigration
   Room.databaseBuilder(context.applicationContext,AppDatabase::class.java,"app_database")
                   .fallbackToDestructiveMigration
                   .build()
   销毁当前数据库，重新建个新的
   但这并不上升级应该使用MIGRATION
   companion object {//也写在companion object中
   
           private val MIGRATION_1_2 = object : Migration(1, 2) {
               override fun migrate(database: SupportSQLiteDatabase) {
                   database.execSQL("create table Book (id integer primary key autoincrement not null, name text not null, pages integer not null)")
               }
           }}
   把注解里的版本号改为2，并在darabase实例创建过程中加入addMigrations(MIGRATION_1_2, MIGRATION_2_3)
   return Room.databaseBuilder(context.applicationContext, AppDatabase::class.java, "app_database")
                   .addMigrations(MIGRATION_1_2)
   //如果是2升级3，则是.addMigrations(MIGRATION_1_2, MIGRATION_2_3)
                   .build().apply {
                   instance = this
   ```

   

#### WorkManager

```kotlin
workManager是一个处理定时任务的工具，会根据系统版本自动使用对应版本的后台管理来保持自己存活，他可以保证在应用退出甚至是手机重启的情况下，之前注册的任务依然能得到执行。所以很适合做一些定期与服务器交互的任务，比如周期性的数据同步。当然在国产系统下可能很难存活。
```

##### 基本用法

```kotlin
添加依赖
implementation 'androidx.work:work-runtime:2.4.0'
```

流程

1. 定义一个后台任务，并实现具体的任务逻辑

   ```kotlin
   class SimpleWorker(context: Context,params :WorkerParameters): Worker(context,params) {
    override fun doWork(): Result {//里面写具体的后台任务逻辑,不在主线程中运行，所以可以执行耗时任务
           Log.d("SimpleWorker", "doWork: ")
           return Result.success()
       }//共有Result.success(),Result.failure(),Result.retry()三种
       //Result.success()表示成功，Result.failure()表示失败，Result.retry()也是失败了，但能再来一次
       //再来一次需要配合WorkRequest.Builder的setBackoffCriteria()来使用
   }
   ```

2. 配置该后台任务的运行条件和约束条件，并构建后台任务请求

   ````kotlin
    val request=OneTimeWorkRequest.Builder(SimpleWorker::class.java).build()
           //OneTimeWorkRequest.Builder是WorkRequest.Builder的子类，用于创建单次运行的后台任务请求
           //WorkRequest.Builder还有个子类PeriodicWordRequest.Buider用于构建周期性的后台任务请求
   val request2=PeriodicWorkRequest.Builder(SimpleWorker::class.java,15,TimeUnit.MINUTES)
           //(work类,时间值,时间单位),运行周期不能短于15分钟
   ````

   

3. 将后台任务请求传入WorkManager的enqueue()方法，系统会在合适的时间运行

   ```kotlin
   WorkManager.getInstance(this).enqueue(request)
   ```

##### 复杂任务

- setInitialDelay()

  ```kotlin
   val request=OneTimeWorkRequest.Builder(SimpleWorker::class.java)
  							.setInitialDelay(5,TimeUnit.MINUTES)//指定任务延迟时间
  							.build()
  ```

  

- addTag

  ```kotlin
   val request=OneTimeWorkRequest.Builder(SimpleWorker::class.java)
  							.addTag("simple")//添加标签
  							.build()
  通过标签取消任务,多个任务用标签进行分组，可以一次性关闭
  WorkManager.getInstance(this).cancelAllworkByTag("simple")
  如果是单个任务可以直接使用id关闭
  WorkManager.getInstance(this).cancelAllworkById(request.id)
  ```

- setBackoffCriteria()

  ```kotlin
   val request=OneTimeWorkRequest.Builder(SimpleWorker::class.java)
  							.setBackoffCriteria(BackoffPolicy.LINEAAR,10,TimeUnit.SECONDS)
  							.build()
  setBackoffCriteria()
  setBackoffCriteria(指定任务如果再次失败下次重试的时间以什么样的形式执行,时间值,时间单位)
  最短10分钟，第一个参数共两个选项
  BackoffPolicy.LINEAAR:重试时间以线性方式延迟
  BackoffPolicy.EPONENTIAL:重试时间以指数方式延迟
  ```

- doWork()返回值的作用

  ```kotlin
  override fun doWork(): Result {
          return Result.success()
          }
  可以使用observe进行观察，来观察运行结果
  WorkManager,getInstance(this).getWorkinfoByIdLiveData(request.id).observ(this){workInfo->
    if(workInfo.state == WorkInfo.State.SUCCEEDED){
      Log.d("MainActivity","do work succeeded")
    }else if(workInfo.state == WorkInfo.State.FAILED){
      Log.d("MainActivity","do work failed")
    }                                                                           
  }
  这里使用了getWorkinfoByIdLiveData，同理也可以使用getWorkinfoByTagLiveData来观察同一标签下的任务请求运行情况
  ```

- 链式任务

  ```kotlin
  假设定义三个独立的后台任务：同步数据、压缩数据、上传数据
  想要实现先同步再压缩后上传，就可以使用链式任务
  val sync=...
  val compress=...
  val upload=...
  WorkManager.getInstance(this)
  					 .beginWith(sync)//开启一个链式任务
  					 .then(compress)//上一个任务完成后运行
  					 .then(upload)
  					 .enqueue()
  ```

  

### 序列化

```kotlin
由于kotlin提供了@Paecelize注解，使得parcelize用起来很方便，并且效率上比Serializable高，所以一般直接使用parcelize进行序列化。
@Paecelize
class Person(var name :String,var age : Int) : Parcelable
只需要加上注解并继承Parcelable就能自动实现
反序列化
val person = intent.getParcelableExtra("person_data") as Person
```

### 定制日志工具

```kotlin
//创建单例类
object LogUtil {

    private const val VERBOSE = 1//定义五个等级

    private const val DEBUG = 2

    private const val INFO = 3

    private const val WARN = 4

    private const val ERROR = 5

    private var level = VERBOSE

    fun v(tag: String, msg: String) {//使用v来打印小于等于v级的日志
        if (level <= VERBOSE) {
            Log.v(tag, msg)
        }
    }

    fun d(tag: String, msg: String) {
        if (level <= DEBUG) {
            Log.d(tag, msg)
        }
    }

    fun i(tag: String, msg: String) {
        if (level <= INFO) {
            Log.i(tag, msg)
        }
    }

    fun w(tag: String, msg: String) {
        if (level <= WARN) {
            Log.w(tag, msg)
        }
    }

    fun e(tag: String, msg: String) {
        if (level <= ERROR) {
            Log.e(tag, msg)
        }
    }

}
//调用
LogUtil.w("TAG","warn log")
开发阶段日志等级正常写，等上线了就把等级改为Error，就不会把调试信息也泄露出去
```

### 判断当前系统主题

```kotlin
fun isDarkTheme(context :Context):Boolean{
  val flag = context.resources.configuration.uiMode and Configuiation.UI_MODE_NIGHT_MASK
  return flag == Configuiation.UI_MODE_NIGHT_YES
}
kotlin里面没有&|^等与或的运算符，而是使用and or xor等关键字进行了替代
```

### java和kotlin之间相互转换

![java转kotlin](image/屏幕快照 2020-11-15 下午1.03.08.png)



先打开java文件就能直接转，或者复制java代码到kt文件里也会自动转，当然只是简单的转换，并不会自动简化，还需要人工来进行精简

kotlin转java，可以先把kotlin代码转换成kotlin字节码，再通过反编译还原成java，但大概率没法运行。

### Volley网络请求

和Retrofit功能类似

```
添加依赖
implementation 'com.android.volley:volley:1.1.1'
```

```java
//获取网络接口的返回
//先创建一个Volley中的RequestQueue
RequestQueue queue = Volley.newRequestQueue(this)
//再创建一个StringRequest,因为是返回json来解析，直接使用StringRequest
  String url = "https://www.jd.com"//定义个url
StringRequest stringRequest = new StringRequest(
	StringRequest.Method.Get,//声明请求
  url,//url地址
  new Response.Listener(String response){
    @Override
    public void onResponse(Strng response){//返回内容到response
      
    }
  },
  new Response.ErrorListener(){//发生错误
    @Override
    public void onErrorResponse(VolleyError error){
      
    }
  }
);
queue.add(stringRequest)//把请求加入队列
```

```java
//用Volley设置网络图片到本地
String url = "https://www.jd.com/xxx.jpg"//网络图片地址
RequestQueue queue = Volley.newRequestQueue(this)//请求队列还是需要的
ImageLoader imageLoader = new ImageLoader(queue,new ImageLoader.ImageCache(){
 // ImageLoader用于图片的加载和缓存
  private LruCache<String,Bitmap> cache = new Lrucache<>(50)//一次缓存50个
    @Override
    public Bitmap getBitmap(String url,){
    return cache.get(url)
  }
   @Override
  public void putBitmap(String url,Bitmap bitmap){
    cache.put(url,bitmap)
  }
});
	imageLoader.get(url,new ImageLoader.ImageListener(){//图片设置到view上
     @Override
     public void onResponse(Strng response){
      imageView.setImageBitmap(response.getBitmap)
    }
  },
  new Response.ErrorListener(){//发生错误
    @Override
    public void onErrorResponse(VolleyError error){
      Log.e(TAG,"onErrorResponse: ",error)
    }
  })
//使用NetworkImageView替代ImageView,那imageLoader.get部分就不需要了
networkImageView.setImageUrl(url,imageLoader)
```

### Glide图片加载

```
添加依赖
implementation 'com.github.bumptech.glide:glide:4.11.0'
annotationProcessor 'com.github.bumptech.glide:compiler:4.11.0'
```

```java
String url = "https://www.jd.com/xxx.jpg"//网络图片地址
RequestQueue queue = Volley.newRequestQueue(this)//请求队列还是需要的
Glide.with(this)
  	 .load(url)
  	 .placeholder(R.drawable.1.jpg)//设置占位图片，
  	 .listener(new RequestListener<Drawable>(){//设置监听器监听网络请求是否成功
       
     })
  	 .into(imageView);//指定显示控件
```

### ShimmerLayout

闪烁的布局，让视图进行闪烁。一般用于网络资源加载完成前的反馈。

```kotlin
shimmerLayout.apply {
            setShimmerColor(0x55ffffff)//55表示透明度,setShimmerColor设置闪动颜色
            setShimmerAngle(0)//闪动角度
            startShimmerAnimation()//启动
        }
```

### 全局获取Context

```kotlin
创建一个继承Application的类，再让所以的Activity继承它
class SunnyWeatherApplication :Application(){
    companion object{
        @SuppressWarnings("StaticFieldLeak")
        lateinit var context: Context
    }

    override fun onCreate() {
        super.onCreate()
        context=applicationContext
    }
}
然后在manifest的<application下添加android:name
<application
        android:name=".SunnyWeatherApplication"	
再使用SunnyWeatherApplication.context就能取得
```

### 沉浸式状态栏

```kotlin
在onCreate里添加
val decorView =window.decorView
decorView.systemUiVisibility=View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN or  View.SYSTEM_UI_FLAG_LAYOUT_STABLE
window.statusBarColor= Color.TRANSPARENT
这之后会使我们布局延伸到状态栏
再去最顶层中哪个不需要高到状态栏位置的布局里面加上，如标题文字就不应该显示到状态栏上
 android:fitsSystemWindows="true"
使其位置顶住状态栏
```

### 收起输入法

```kotlin
val manager = getSystemService(Context.INPUT_METHOD_SERVICE) as InputMethodManager           manager.hideSoftInputFromWindow(drawerView.windowToken,InputMethodManager.HIDE_NOT_ALWAYS)
```



### MPAndroidChart库绘制折线图等

```
官方 https://github.com/PhilJay/MPAndroidChart
添加依赖
项目Gradle的allprojects中
repositories {
    maven { url 'https://jitpack.io' }
}
AppGradle
dependencies {
    implementation 'com.github.PhilJay:MPAndroidChart:v3.1.0'
}
使用示例 https://www.jianshu.com/p/f1cfdf2dc98c
```

1. 折线图

   ```xml
   <com.github.mikephil.charting.charts.LineChart
           android:id="@+id/mLineChar"
           android:layout_width="match_parent"
           android:layout_height="0dp"
           android:layout_weight="1"/>
   ```

   ```java
   //基本设置
   mLineChar = (LineChart) findViewById(R.id.mLineChar);
           //设置手势滑动事件
           mLineChar.setOnChartGestureListener(this);
           //设置数值选择监听
           mLineChar.setOnChartValueSelectedListener(this);
           //后台绘制
           mLineChar.setDrawGridBackground(false);
           //设置描述文本
           mLineChar.getDescription().setEnabled(false);
           //设置支持触控手势
           mLineChar.setTouchEnabled(true);
           //设置缩放
           mLineChar.setDragEnabled(true);
           //设置推动
           mLineChar.setScaleEnabled(true);
           //如果禁用,扩展可以在x轴和y轴分别完成
           mLineChar.setPinchZoom(true);
   //填充数据
    ArrayList<Entry> values = new ArrayList<Entry>();
           values.add(new Entry(5, 50));
           values.add(new Entry(10, 66));
           values.add(new Entry(15, 120));
           values.add(new Entry(20, 30));
           values.add(new Entry(35, 10));
           values.add(new Entry(40, 110));
           values.add(new Entry(45, 30));
           values.add(new Entry(50, 160));
           values.add(new Entry(100, 30));
   
           //设置数据
           setData(values);
           //默认动画
           mLineChar.animateX(2500);
           //刷新
           mLineChar.invalidate();
           // 得到这个文字
           Legend l = mLineChar.getLegend();
           // 修改文字 ...
           l.setForm(Legend.LegendForm.LINE);
           //传递数据集
       private void setData(ArrayList<Entry> values) {
           if (mLineChar.getData() != null && mLineChar.getData().getDataSetCount() > 0) {
               set1 = (LineDataSet) mLineChar.getData().getDataSetByIndex(0);
               set1.setValues(values);
               mLineChar.getData().notifyDataChanged();
               mLineChar.notifyDataSetChanged();
           } else {
               // 创建一个数据集,并给它一个类型
               set1 = new LineDataSet(values, "年度总结报告");
   
               // 在这里设置线
               set1.enableDashedLine(10f, 5f, 0f);
               set1.enableDashedHighlightLine(10f, 5f, 0f);
               set1.setColor(Color.BLACK);
               set1.setCircleColor(Color.BLACK);
               set1.setLineWidth(1f);
               set1.setCircleRadius(3f);
               set1.setDrawCircleHole(false);
               set1.setValueTextSize(9f);
               set1.setDrawFilled(true);
               set1.setFormLineWidth(1f);
               set1.setFormLineDashEffect(new DashPathEffect(new float[]{10f, 5f}, 0f));
               set1.setFormSize(15.f);
   
               if (Utils.getSDKInt() >= 18) {
                   // 填充背景只支持18以上
                   //Drawable drawable = ContextCompat.getDrawable(this, R.mipmap.ic_launcher);
                   //set1.setFillDrawable(drawable);
                   set1.setFillColor(Color.YELLOW);
               } else {
                   set1.setFillColor(Color.BLACK);
               }
               ArrayList<ILineDataSet> dataSets = new ArrayList<ILineDataSet>();
               //添加数据集
               dataSets.add(set1);
   
               //创建一个数据集的数据对象
               LineData data = new LineData(dataSets);
   
               //设置数据
               mLineChar.setData(data);
           }
       }
   //填充效果
   List<ILineDataSet> setsFilled = mLineChar.getData().getDataSets();
       for (ILineDataSet iSet : setsFilled) {
           LineDataSet set = (LineDataSet) iSet;
           if (set.isDrawFilledEnabled())
               set.setDrawFilled(false);
           else
               set.setDrawFilled(true);
       }
       mLineChar.invalidate();
   //立方切换
   List<ILineDataSet> setsCubic = mLineChar.getData().getDataSets();
       for (ILineDataSet iSet : setsCubic) {
           LineDataSet set = (LineDataSet) iSet;
           set.setMode(set.getMode() == LineDataSet.Mode.CUBIC_BEZIER
                   ? LineDataSet.Mode.LINEAR
                   : LineDataSet.Mode.CUBIC_BEZIER);
       }
       mLineChar.invalidate();
   ```

   

2. 条形图线图

3. 饼状图

### ViewPager2

 ```kotlin
//配合TabLayout作为标题栏使用
//创建继承FragmentStateAdapter类的匿名类来作为Adapter
pager.adapter = object : FragmentStateAdapter(this) {
            
   override fun getItemCount() = 3//返还页面个数

   override fun createFragment(position: Int) = when (position) {
                0 -> ScaleFragment()
                1 -> RotateFragment()
                else -> TranslateFragment()
            }
        }//返还页面内容
TabLayoutMediator(tabLayout, pager) { tab, position ->
            when (position) {
                0 -> tab.text = "缩放"
                2 -> tab.text = "旋转"
                else -> tab.text = "移动"
            }
        }.attach()//实现标题栏和ViewPager同步

//跳转到指定vp，第一个参数为vp的序号，第二个表示是否需要跳过去的动画，缺省为ture
viewPager2.setCurrentItem(arguments?.getInt("PHOTO_POSITION")?:0, false)
//改为垂直滚动
viewPager2.orientation =ViewPager2.ORIENTATION_VERTICAL
 ```

### ObjectAnimator属性动画(缩放,旋转,移动)

```kotlin
//缩放
val  factor=if(Random.nextBoolean())0.1f else -0.1f//随机加减0.1
 //x和y同时缩放           
ObjectAnimator.ofFloat(it,"scaleX",it.scaleX+factor).start()
  //(动画的执行对象,需要执行的动画名称,动画所需的值)
ObjectAnimator.ofFloat(it,"scaleY",it.scaleY+factor).start()
//旋转，直接输入角度
ObjectAnimator.ofFloat(it,"rotation",it.rotation+30f).start()
//移动，最后面都需要加上.start()来执行	
ObjectAnimator.ofFloat(it,"translationX",it.translationX+ Random.nextInt(-100,100)).start()
       
```

### recyclerview.widget.ListAdapter的使用

```kotlin
//创建Adapter	
class PagerPhtotListAdapter: ListAdapter<PhotoItem, PagerPhtotViewHolder>(DiffCallBack) {
    //androidx.recyclerview.widget.ListAdapter需要传入一个比较器
    object DiffCallBack :DiffUtil.ItemCallback<PhotoItem>(){//创建比较器
        override fun areItemsTheSame(oldItem: PhotoItem, newItem: PhotoItem): Boolean {//判断是否是相同id
            return oldItem.photoId == newItem.photoId//通过photoId来判断id是否相同
        }

        override fun areContentsTheSame(oldItem: PhotoItem, newItem: PhotoItem): Boolean {//判断item的逐个属性是否相同
            return oldItem == newItem//kotlin中==对象时，会进行属性的逐项比较
        }

    }
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): PagerPhtotViewHolder {
        TODO("Not yet implemented")
    }

    override fun onBindViewHolder(holder: PagerPhtotViewHolder, position: Int) {
        TODO("Not yet implemented")
    }

}


//提交列表
adapter.submitList(pagerList)
//然后就按照列表数据生成viewholder
```

### Adapter创建顺序

```
1.先获取要创建的单元总数 ItemCount
2.要请求的每一个单元的类型 ItemType
3.创建ViewHolder onCreateViewHolder
4.绑定数据 bindView 
```

### ViewBinding

```java
功能和kotlin那个差不多，但他限制了作用域，其他视图的控件id不会出来，是一个轻量版的DataBinding
在app build.gradle android下添加
viewBinding{
    enabled = true
}
使用
private ActivityMainBinding binding;//声明对象，类名直接输入当前类名差不多的开头就能自动联想
binding =ActivityMainBinding.inflate(getLayoutInflater());//实例化，不需要指定layout文件，通过类名自动会找到队友的layout
用 setContentView(binding.getRoot());//getRoot()获取根节点
替代 setContentView(R.layout.activity_main);
然后使用 binding.button 这种形式使用控件
```



















































































### Git

```
工作流程
1.创建版本/快照，以一个四十位的序列号来区分版本，每次执行commit指令就会创建新的版本
2.以某一个快照为起点创建一个新的分支Branch
3.当一个分支，完成了它的分支目的，比如实现了某项功能，要合并回原有分支，Merge
4.如果两个分支都对一个地方进行了修改，就会产生冲突，这时候就要二者取其一
5.所有的代码修改都在分支上进行，完成后再合并到主分支 
```

| 名称       | 中文 | 功能   |
| ---------- | ---- | ------ |
| Repository | 仓库 |        |
| Clone      | 克隆 |        |
| Checkout   | 检出 | 核心： |
| Commit     | 提交 | 核心:  |
| Branch     | 分支 | 核心:  |
| Merge      | 合并 | 核心： |
| Push       | 推送 |        |
| Pull       | 拉取 |        |

#### GitHub

```
git config user.name yuyuyu
git config user.email 542148958@qq.com
修改用户信息
```



### 快捷键

#### java局部转全局变量工具

![](image/屏幕快照 2020-11-15 下午2.42.27.png)

#### 多段输入

shift+alt+鼠标左键单击，就能有多个光标

#### 同大小的数值修改

mac：ctrl+g  win：alt+J 会自动选中下一个同大小的值

#### kotlin-android-extensions开启

```kotlin
App gradel:
apply plugin: 'kotlin-android-extensions'或 id 'kotlin-android-extensions'
implementation  "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
Project gradel:
classpath  "org.jetbrains.kotlin:kotlin-android-extensions:$kotlin_version"
```

#### implementation依赖的本地路径

```
implementation依赖的本地路径 /Users/yuyuyu/.gradle/caches/modules-2/files-2.1
```

#### android项目改名

```
1.先改文件夹名
2.进去项目，改setting.gradle中的rootProject.name
3.重命名java下的包名
4.app build.gradle applicationId改为包名
5.string.xml中app_name来修改应用名
```

#### adbIdea插件

```
装好插件后顶部菜单的tools里面找到adbIdea就能进行应用管理
```

