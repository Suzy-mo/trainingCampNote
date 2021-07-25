# 第一行代码第三版AS

## 第三章 Activity

#### 初始化控件 省去fid  

没有出现自动导包的原因   没有插件kotlin-android-extensions(有些AS是自动导的，没有的话就手动加)

```java
plugins {
    id 'com.android.application'
    id 'kotlin-android'
    id 'kotlin-android-extensions'
}
```

#### button使用

```kotlin
val button1: Button = findViewById(R.id.button1);
```

#### Toast的使用

无差别

```kotlin
button1.setOnClickListener { 
            Toast.makeText(this,"Toast",Toast.LENGTH_SHORT).show();
        }
```

#### Menu

步骤：

1. 新建 menu 文件夹
2. 新建对应Activity的menu文件  里面可以有多个item标签
3. 重写onCreateOptionsMenu方法  在方法里面加载menuInflter  
4. 重写onOptionsItemSeledted方法   里面可以加操作逻辑

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">

    <item
        android:id="@+id/add_item"
        android:title="add"/>
    <item
        android:id="@+id/remove_item"
        android:title="remove"/>
</menu>
```



```kotlin
    override fun onCreateOptionsMenu(menu: Menu?): Boolean {
        menuInflater.inflate(R.menu.main,menu)
        return true
    }

    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        when(item.itemId){
            R.id.add_item -> Toast.makeText(this,"Add",Toast.LENGTH_SHORT).show();
            R.id.remove_item -> Toast.makeText(this,"remove",Toast.LENGTH_SHORT).show();
        }
        return true
    }
```

#### 销毁活动

```kotlin
finish()
```

#### Intent的使用

显式

```kotlin
val intent = Intent(this,SecondActivity::class.java)
startActivity(intent);
```

隐式

* 主活动

```xml
<intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
```

* 自定义

```xml
<activity android:name=".SecondActivity">
    <action android:name="com.qg.threeoneapp.ACTION_START" />

    <category android:name="com.qg.threeoneapp.DEFAULT" />
</activity>
```

```Kotlin
val intent = Intent("com.qg.threeoneapp.ACTION_START")
intent.addCategory("com.qg.threeoneapp.DEFAULT")
startActivity(intent);
```

* 隐式其他用法

```kotlin
val intent = Intent("com.qg.threeoneapp.ACTION_START")
intent.data = Uri.parse("http://www.baidu.com")
startActivity(intent);
```

基本跟java一样

* 传递数据

  其他都一样

  tips 字符串里面可以加入变量   $

#### 活动的样式设定

* Dialog对话框

  在activity前标签内设定

  ```xml
  <activity android:name=".SecondActivity"
      android:theme="@style/Theme.AppCompat.Dialog">
  </activity>
  ```

#### 临时数据的保存

重写 保存在Bundle中    借用 savedInstancdState  再创建就取出就行

#### 选择启动模式

在activity 前标签里面指定

```xml
android:launchMode:standard dingleTask singleIntance
```

#### 知晓在哪个活动

```Kotlin
open class BaseActivity : AppCompatActivity(){
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        Log.d("BaseActivity",javaClass.simpleName)
    }
}
```

#### 随时退出

```Kotlin
object ActivityCollector {
    private val activities = ArrayList<Activity>()
    
    fun addActivity(activity: Activity){
        activities.add(activity)
    }

    fun removeActivity(activity: Activity){
        activities.remove(activity)
    }

    fun finish(){
        for (activity in activities){
            if (!activity.isFinishing){
                activity.finish()
            }
        }
        activities.clear()
    }
}
```

在onCreate或者onDestory调用相关的函数

## 第四章 UI

### 常用控件

* TextView

  在资源设置字符串格式  style

* 监听器

  ```kotlin
      override fun onClick(v: View?) {
          when(v?.id){
              R.id.button1 -> {
                  
              }
              R.id.button2 -> {
                  
              }
          }
      }
  
  ```

* EditText

  ```kotlin
  val inputText = editText.text.toString()
  ```

* ImageView

  主流手机分辨率 xxhdpi  

* PasserBar

  设置可见与不可见  visibility 

  修改为水平  style = “？android : attr/processbBarStyleHorizonal”

  设置最大值   android:max="100"

  动态修改 processBar.process = processBar.process + 10

* AlterDialog

  ```kotlin
      override fun onClick(v: View?) {
          when(v?.id){
              R.id.button1 -> {
                  AlertDialog.Builder(this).apply {
                      setTitle("This is a alertDialog")//标题
                      setMessage("Something importance")//内容
                      setCancelable(false)//是否可以用Back键关闭对话框
                      setPositiveButton("OK"){
                          dialog, which ->
                      }
                      setNegativeButton("CANCEL"){
                          dialog, which ->
                      }//设置按件的相关点击事件
                      show()//展示出来
                  }
              }
              R.id.button2 -> {
  
              }
          }
      }
  
  ```

* 隐藏标题栏

  ```kotlin
  supportActionnBar?.hide()
  ```

  