# 安卓四大组件及Fragment（实操总结）

## 一、Activity

### 简介

* Activity可为用户提供可视化界面

* 在app中只要看得见的几乎都要依赖于Activity

* Activity生命周期（Activity栈）

  ![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\Activity生命周期.png)

### 使用流程

#### 声明

在AndroidManifest.xml配置文件中声明（android studio会自动生成）

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.qg.rdcdictionary">

    <uses-permission android:name="android.permission.INTERNET" />
    ...

    <application
        ...
        <activity android:name=".taskword.WordActivity" />
        <activity android:name=".tasknewbook.NewBookActivity" />
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

#### 设计视图及加载

在res->layout的xml文件中设计视图布局

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    
    <!--可设置的布局-->
    
</RelativeLayout>
```

加载视图文件(一般在创建活动时AS会自动设置好Activity对应布局)

```java
public class MainActivity extends AppCompatActivity {
    ...

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
    ...
}
```

#### 初始化控件

初始化控件在onCreate函数里面（button textView  ImageView ViewPager TabLayout DataBaseHelper adapter onclickListener）

```java
public class MainActivity extends AppCompatActivity {
    private TabLayout myTab;
    private ViewPager2 myPager2;
    private ImageView bookIv;
    private DatabaseHelper mDatabaseHelper;
    List<String> titles=new ArrayList<>();
    List<Fragment> fragments=new ArrayList<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mDatabaseHelper = new DatabaseHelper(this,"Directory.db",null,1);

        myTab = findViewById(R.id.tab_layout);
        myPager2 = findViewById(R.id.view_pager);
        bookIv = findViewById(R.id.book_iv);

        //添加Fragment进去
        fragments.add(new SearchFragment());
        fragments.add(new DayFragment());
        fragments.add(new TranslateFragment());

        //实例化适配器
        MyViewPager2Adapter myAdapter=new MyViewPager2Adapter(getSupportFragmentManager(),getLifecycle(),fragments);
        //设置适配器
        myPager2.setAdapter(myAdapter);
        //TabLayout和Viewpager2进行关联
        new TabLayoutMediator(myTab, myPager2, new TabLayoutMediator.TabConfigurationStrategy() {
            @Override
            public void onConfigureTab(@NonNull TabLayout.Tab tab, int position) {
                tab.setText(titles.get(position));
            }
        }).attach();

        bookIv.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Intent intent = new Intent(MainActivity.this, NewBookActivity.class);
                startActivity(intent);
            }
        });
    }
}
```

#### 生命周期的应用

注意 进行解绑 在onstop里面 解除服务绑定，对其他类的持有等等

```java
@Override
    protected void onStop() {
        super.onStop();
        if (!executorService.isShutdown()) {
            executorService.shutdownNow();
        }
    }
```

### 活动间的通信Intent

活动间的切换及通信 ：Intent的应用

* 切换 一般用显式Intent

  * 上一级的活动

  参数：context  跳转到下一个活动的class

  ```java
  Intent intent = new Intent(getApplicationContext(), MainActivity.class);
               startActivity(intent);
  ```

  * 下一级的活动

  ```java
  Intent intent = getIntent();
  ```

* 结束或者返回上一个活动（back键）

  ```java
  finish();
  ```

* 数据传递

  * 用Extra

    ```java
    //发送端
    Intent intent = new Intent(getContext(), WordActivity.class);
    intent.putExtra("word", data);//参数：暗号 要传递的数据
    intent.putExtra("isSave", 0);
    //接收端
    word = intent.getStringExtra("word"); //参数：暗号
    isSaved = intent.getIntExtra("isSave", 0);//参数：暗号 没有接收到数据的预赋值
    ```

  * 返回数据给上一个活动

    ```java
    //第一个活动
    Intent intent = new Intent(FirstActivity.this,SecondActivity.class);
    startActivityForResult(intent,1);
    //重写onActivityResult方法
    @Override
        protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
            super.onActivityResult(requestCode, resultCode, data);
            if(requestCode == RESULT_OK){
                String returnedData = data.getStringExtra("data_return");
            }
        }
    
    //第二个活动
    Intent intent = new Intent();
    intent.putExtra("data_return","Hello");
    setResult(RESULT_OK,intent);//参数 返回码：RESULT_OK / RESULT_CANCELED
    finish();
    
    //如果用户是按返回键返回的
    @Override
        public void onBackPressed() {
            Intent intent = new Intent();
            intent.putExtra("data_return","Hello");
            setResult(RESULT_OK,intent);
            finish();
        }
    ```

    * 用Bundle传递类（序列化和反序列化）

      序列化类要实现Serializable接口

      ```java
      public class User implements Serializable { 
      }
      ```

      在intnet的使用时加入

      ```java
      //发送端
      Intent intent = new Intent(getApplicationContext(), MainActivity.class);
      Bundle bundle = new Bundle();
      bundle.putSerializable("data",user);
      intent.putExtras(bundle);
      startActivity(intent);
      //接收端
      Intent intent = this.getIntent();  
      User user = (User) intent.getSerializableExtra("user");  
      ```

### 补充IntentFilter

* 使用隐式启动时需要加上IntentFilter进行过滤

* | 属性     | 作用                       |
  | -------- | -------------------------- |
  | action   | 表示Intent对象要完成的动作 |
  | data     | 表示Intent对象要完成的动作 |
  | category | 表示为action添加的额外信息 |

* 匹配规则

  * ##### action属性

    标签中间可以有多个action属性，但是当使用隐式Intent启动时，只要Intnt携带的action与其中一个< intent-filter >标签中action的声明相同即可

    ```xml
    <intent-filter>
    	<action android:name="android.intent.action.EDIT">
    </intent-filter>
    ```

    ```java
    Intent intent = new Intent("android.intent.action.EDIT");
    startIntent(intent);
    ```

    

  * ##### data属性

    标签中间可以有多个data属性，每个data属性可以指定数据MIME类型和URI。MIME类型可以表示image/ipeg、video/*等媒体类型。
    隐式Intent携带的data数据只要与IntentFilter中的任意一个data声明相同即可

    ```xml
    <intent-filter>
    	<data android:mimeType="video/mpeg" android:scheme="http">
    </intent-filter>
    ```

    data标签中可以设置一下标签

    ![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\data标签中.png)

    隐式Intent的应用

    ```java
    Intent intent = new Intent(Intent.ACTION_VIEW);
    //Intent.ACTION_VIEW是系统内置动作
    intent.setData(Uri.parse"http://www.baidu.com");
    //Uri.parse将网址解析为Uri对象
    startActivity(intent);
    
    Intent intent = new Intent(Intent.ACTION_DIAL);
    //Intent.ACTION_DIAL是系统内置动作
    intent.setData(Uri.parse"tel:10086");
    startActivity(intent);
    ```

  * ##### category属性

    隐式Intent中声明的category全部能够与某一个IntentFilter中的category匹配才算匹配成功。IntentFilter中罗列的category属性数量必须大于或者等于隐式Intent携带的category属性数量时，category属性才能匹配成功

    "android.intent.category.DEFAULT"是默认的category

    ```xml
    <intent-filter>
    	<category android:name="android.intent.category.DEFAULT">
    </intent-filter>
    ```

    可动态添加category

    ```java
    Intent intent = new Intent("android.intent.action.EDIT");
    intent.addCategory("com.example.activity.MY_CATEGORY");
    startIntent(intent);
    ```

## 二、Server

### 简介

* 一般用作后台 进行耗时操作的处理 如下载或者进行后台播放等等

* 生命周期

  ![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\服务生命周期.png)

### 基本使用流程

#### 定义

像新建Activity那样新建Service AS会自动在AndroidManifest.xml中注册

#### 启动

* 启动有两种方式 startService和bindService

  >**startService**只是启动Service，启动它的组件（如Activity）和Service并没有关联，只有当Service调用stopSelf或者其他组件调用stopService服务才会终止。
  >**bindService**方法启动Service，其他组件可以通过回调获取Service的代理对象和Service交互，而这两方也进行了绑定，当启动方销毁时，Service也会自动进行unBind操作，当发现所有绑定都进行了unBind时才会销毁Service。

* startService

  ```java
  //启动
  Intent startIntent = new Intent(this,MusicService.class);
  startService(startIntent);
  //停止
  Intent stopIntent = new Intent(this,MusicService.class);
  stopService(stopIntent);
  ```

* bindService（常用）

  ```java
  //ServiceConnection connection;
  //绑定服务
  Intent bindIntent = new Intent(SearchActivity.this, MusicService.class);
  bindService(bindIntent, connection, BIND_AUTO_CREATE);
  //解除绑定
  unbindService(connection);
  ```

#### 绑定数据及操作

* 在Myservice类里面新建Binder类并覆写onBinder

  （在MusicBinder类里面的函数可以在Activity中调用）

  ```java
  public class MusicService extends Service {
      ...
      private MusicBinder mMusicBinder = new MusicBinder();
  
      @Nullable
      @Override
      public IBinder onBind(Intent intent) {
          return mMusicBinder;
      }
  
      public class MusicBinder extends Binder {
  
          public MusicService getService(){
              return MusicService.this;
          }
  
          public void playClickMusic(List<MusicBean> data, int position){
              new Thread(new Runnable() {
                  @Override
                  public void run() {
                      MusicBean musicBean = data.get(position);
                      setMusicPlayer(musicBean);
                      Log.d(TAG,"searchMusicPlay path =  "+ musicBean.getPath());
                  }
              }).start();
          }
  
          ....
      }
  ```

* 在Activity中可以调用

  ```java
  public class SearchActivity extends AppCompatActivity {
      //声明binder类
      private MusicService.MusicBinder musicBinder;
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.searching_activity);
      	...
          Intent startIntent = new Intent(SearchActivity.this, MusicService.class);
          startService(startIntent);
          playingBean = Data.get(position);
          //实例化ServiceConnection类
          ServiceConnection connection = new ServiceConnection() {
               @Override
               public void onServiceConnected(ComponentName name, IBinder service) {
                   musicBinder = (MusicService.MusicBinder) service;
                   //与服务绑定时建立联系 调用服务binder里面的方法
                   musicBinder.playClickMusic(Data, position);
                }
  
                @Override
                public void onServiceDisconnected(ComponentName name) {
                    //解绑时也可以调用
                    }
                };
           Intent bindIntent = new Intent(SearchActivity.this, MusicService.class);
           bindService(bindIntent, connection, BIND_AUTO_CREATE);//把connection
      }
  }
  ```

### IntentService

内置开启线程和自动关闭功能

* 标准的service写法 执行完某个耗时操作后自己停止

  ```java
  public class MusicService extends Service {
      ...
      public int onStartCommand(Intent intent, int flags, int startId) {//启动后台服务
          super.onStartCommand(intent, flags, startId);
          new Thread(()->{
              //具体操作逻辑
              stopSelf();//任务执行完后需要停止服务
          }).start();
          return super.onStartCommand(intent, flags, startId);
  }
  ```

  要回到主线程更新UI

  ```java
  //在Activity中
  runOnUiThread(new Runnable() {
              @Override
              public void run() {
                  //更新UI
              }
          });
  //在Fragment中
  getActivity().runOnUiThread(new Runnable() {
              @Override
              public void run() {
                  //更新UI
              }
          });
  ```

* 使用IntentService

  * 新建MyIntentService类继承自IntentService

    * 构造方法 一定要有 如果是无参则调用父类构造器 名字为子线程的名字，用于区分线程
    * onHandleIntent方法里面可以加入耗时操作等具体逻辑
    
    ```java
    public class MyIntentService extends IntentService {
     
        public MyIntentService() {
            super("MyIntentService");
        }
        
        public MyIntentService(String name) {
            super(name);
        }
     
        @Override
        public int onStartCommand(@Nullable Intent intent, int flags, int startId) {
            return super.onStartCommand(intent, flags, startId);
        }
     
        @Override
        protected void onHandleIntent(@Nullable Intent intent) {
            if (intent != null) {
                //进行耗时操作
                }
            }
        }
    }
    ```
    
  * 启动服务
  
    与显式启动service只有跳转对象的不同
    
    ```java
    intent.setClass(this,MyIntentService.class);
    ```

### 线程池的使用

#### 简介

* 便于线程管理
* 复用线程减少消耗

#### 一般步骤

（可以结合下面的常用例子结合）

* 创建线程池

  * java.util.concurrent.Executors 线程池的工厂类 里面的方法可以用来创建线程，不同线程池的方法不一样

  * java.util.concurrent.ExecutorService接口，用接口接收创建的线程

* 提交子线程

  * 实现Runnable接口，重写run方法，设置线程任务
  * 调用ExecutorService中的方法进行提交

* 回到主线程更新UI

  用Handler

#### 几种线程池的使用

* FixedThreadPool (可重用固定线程数)

  ```java
  //创建
  executorService = Executors.newFixedThreadPool(3);
  //使用
  executorService.excute(()->{
              // 执行耗时操作代码
              Handler uiThread = new Handler(Looper.getMainLooper());
              uiThread.post(()->{
                  //
              });
          });
  ```

* CachedThreadPool (按需创建)

  ```java
  //创建
  executorService = Executors.newCachedThreadPool();
  //使用同上
  ```

* SingleThreadPool(单个核线的fixed)

  ```java
  //创建
  executorService = Executors.newSingleThreadExecutor();
  //使用同上
  ```

* ScheduledThreadPool(定时延时执行)

  参数 

  * 第一个是runnable子线程任务
  * 第二个是延长的时间
  * 第三个是提交周期  TimeUnit.SECONDS/DAYS等等

  ```java
  //创建
  ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(3);
                  scheduledThreadPool.schedule(()->{
                      // 执行耗时操作代码
                      Handler uiThread = new Handler(Looper.getMainLooper());
                      uiThread.post(()->{
  						//更新UI
                      });
                  },10, TimeUnit.SECONDS);
  ```

#### 常用例子（**替代AsyncTask**）

```java
public class SearchActivity extends AppCompatActivity{
    
    private ExecutorService executorService;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.searching_activity);
        //创建固定线程数量的线程池 参数为线程数量
        executorService = Executors.newFixedThreadPool(3);
        
        executorService.submit(new Runnable() {
            @Override
            public void run() {
            	// 执行耗时操作代码
                
                //执行完耗时代码回到主线程更新UI
            	Handler uiThread = new Handler(Looper.getMainLooper());
            	uiThread.post(new Runnable() {
                @Override
                public void run() {
                    //
                    }
                });
            }
        });
    }
}
```

lanmda表达式简易写法

```java
public class SearchActivity extends AppCompatActivity{
    
    private ExecutorService executorService;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.searching_activity);
        
        executorService = Executors.newFixedThreadPool(3);
        
        executorService.submit(()->{
            // 执行耗时操作代码
            Handler uiThread = new Handler(Looper.getMainLooper());
            uiThread.post(()->{
                //
            });
        });
    }
}
```



## Broadcast Receiver

### 简介

* 广播是一种广泛运用的在应用程序之间传输信息的机制，广播接收器是对发送出来的广播进行过滤接受并响应的一类组件
* 可以使用广播接收器来让应用对一个外部时间做出响应。例如，当电话呼入这个外部事件到来时或当下载一个程序成功完成时，仍然可以利用广播接收器进行处理。
* 快捷创建广播 ：Exported表示是否允许该广播接受去接收本程序之外的广播，Enabled表示是否启用这个广播接收器

### 系统全局广播使用流程

#### 广播注册

* 广播注册有静态和动态

* 静态在AndroidManifest中注册(AS快捷方式创建new->other->Broadcast Receiver会自动生成）

  ```xml
  <receiver
        android:name=".MyReceiver"
        android:enabled="true"
        android:exported="true">
  </receiver>
  ```

* 动态在Avtivity中注册，是否关闭与Activity相关联（现在的安卓都要动态注册）

  在用到广播的Activity中动态注册

  ```java
  protected void onCreate(Bundle savedInstanceState) {
     super.onCreate(savedInstanceState);
     setContentView(R.layout.activity_main);
      
     registerReceiver(new MyBroadcastReceiver(), new IntentFilter("com.example.broadcasttest.MY_BROADCAST"));//加入过滤条件
  }
  ```

#### 发送广播

发送广播用Intent

* 自定义广播

```java
Intent intent=new Intent("com.example.broadcasttest.MY_BROADCAST");
sendBroadcast(intent);
```

* 发送有序广播

在注册时给广播接收器设置优先级

在Activity中只改发送那个行代码，多了一个与权限相关的参数

```java
Intent intent=new Intent("com.example.broadcast.MY_BROADCAST");
sendBroadcast(intent,null);
```

```xml
<receiver
	android:enabled="true"
    android:exported="true">
    <intent-filter android:priority="100">
           <action android:name="com.example.broadcast.MY_BROADCAST"/>
    </intent-filter>
</receiver>
```

获得接收广播的优先权后的接收器可以选择是否继续传播该广播

```java
public class MyReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        Toast.makeText(context,"received in MyReceiver",Toast.LENGTH_SHORT).show();
        abortBroadcast();//表示拦截 不允许其他接收器接收
    }
}
```

#### 接收广播

* 创建一个类继承自BroadcastReceiver（快捷方式自动生成）
* 重写onReceive（）

```java
public class MyBroadcastReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        //具体逻辑
        Toast.makeText(context,"receive in MyBroadcastReceiver",Toast.LENGTH_SHORT).show();
    }
}
```

### 本地广播使用流程

解决安全性问题，只允许在程序应用内传递，广播接收器也只能接收本程序发出的广播

与全局广布不同之处主要在于用LocalBroadcastManager对广播进行管理

```java
//声明LocalBroadcastManager
private LocalBroadcastManager localBroadcastManager;

//获取实例
localBroadcastManager = LocalBroadcastManager.getInstance(this);

//注册广播  参数的receiver和intentFilter可以分开实例化，也可以直接实例化
localBroadcastManager.registerReceiver(MyReceiver,intentFilter3);
localBroadcastManager.registerReceiver(new MyBroadcastReceiver(), new IntentFilter("com.example.broadcasttest.MY_BROADCAST"));

//注销广播
localBroadcastManager.unregisterReceiver(localReceiver);
```

## 四、Content Provider

### 简介

* 主要用于在不同的应用程序之间实现数据共享
* 统一数据访问方式
* ContentProvider用于保存和获取数据，并使其对所有应用程序可见，这是不同应用程序间共享数据的唯一方式

### 使用流程

#### 访问其他程序中的数据

主要基于ContentResolver（内容解析者）

##### 要用到的知识

* Uri的解析（内容URI）

  由authority、path、id组成：authority一般采用程序包命名；path用于对同一个程序中不同表的区分，id区分不同数据

  ```java
  content://com.example.app.provider/tabel1/999
  content://com.example.app.provider/tabel2/100
  ```

  ```java
  Uri uri = Uri.parse("content://com.example.app.provider/tabel1");
  ```

* courser的使用

  ```java
  //查询
  Cursor cursor = getContentResolver().query(
  	uri,
  	projection,
  	selection,
  	selectionArgs,
  	sortOrder);
  
  //读取
  if(cursor!=null){
      while(cursor.moveToNext()){
          String column = cursor.getString(cursor.getColumnIndex("colum1"));
      }
      cursor.close();
  }
  ```

![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\内容提供器参数.png)

* 插入

  ```java
  ContentValues values = new ContentValues();
  values.put("column1","text");
  values.put("column2",1);
  getContentResolver().insert(uri,values); 
  ```

* 更新(清空)

  ```java
  ContentValues values = new ContentValues();
          values.put("column1","");
          getContentResolver().update(uri,values,"column1 = ? and column2 = ?",new String[]{"text","1"});
  ```

* 删除

  ```java
  getContentResolver().delete(uri,values,"column2 = ?",{"1"});
  ```

##### 读取联系人实例

* 申请权限

  ```xml
  <uses-permission android:name="android.permission.READ_CONTACTS"/>
  ```

* 向用户获取授权 （判断+重写onRequestPermissionsResult方法）

  ```java
  if(ContextCompat.checkSelfPermission(this, Manifest.permission.READ_CONTACTS)!= PackageManager.PERMISSION_GRANTED){
              ActivityCompat.requestPermissions(this,new String[]{Manifest.permission.READ_CONTACTS},1);
          }else {
              //查询联系人
      		readContacts();
          }
  
  @Override
      public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
          switch (requestCode){
              case 1:
                  if(grantResults.length > 0 && grantResults[0]==PackageManager.PERMISSION_GRANTED){
                      //查询联系人
                      readContacts();
                  }else {
                      Toast.makeText(this,"You denied the permission",Toast.LENGTH_SHORT).show();
                  }
                  break;
              default:
          }
      }
  ```

* 查询

  联系人的查询有固定的uri字符串

  ```java
  private void readContacts(){
          Cursor cursor = null;
          try {
              cursor = getContentResolver().query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI,null,null,null,null);
              if (cursor !=null){
                  while (cursor.moveToNext()){
                      String name = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));
                      String number = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
                      //对数据的操作 可加入集合等等
                  }
                  //列表适配器更新
              }
          }catch (Exception e){
              e.printStackTrace();
          }finally {
              if (cursor!=null){
                  cursor.close();
              }
          }
      }
  ```

##### 小结

访问其他程序中数据的应用

还可以访问本地音乐 本地文件 本地照片 本地视频 等等本地的数据，都是差不多的流程，只是uri字符串不一样

#### 创建自己的内容提供器

定义匹配的规则，重写六个方法，与数据库使用相结合

* 快捷方式创建 new->other->contect proider（自动注册）

  URI Authorities ：包名.provider

  ```java
  com.example.darkCloudApp.provider
  ```

* 在MyContectProvider类中定义匹配的常量

  内容URI格式有两种 以路径结尾表示访问表中所有数据，以id结尾表示访问该表相应id的数据，用通配符分别匹配

  * /*  表示匹配任意长度的任意字符   content://com.example.darkCloudApp.provider/*(匹配任意表)
  * /#  表示匹配任意长度的数字content://com.example.darkCloudApp.provider/tabel/#（匹配任意一行数据）

  借助uriMatcher进行匹配

  *  addURI 把authorties path id 传进去
  * 调用match（）可以将一个uri对象传入，返回值是某个能够匹配这个uri对象所对应的自定义代码，利用这个代码可以判断用户想访问什么数据

  ```java
  public class MyContentProvider extends ContentProvider {
  
      //这里的AUTHORITY就是我们在AndroidManifest.xml中配置的authorities
      private static final String AUTHORITY = "com.example.darkCloudApp.provider";
      
      //匹配成功后的匹配码 不同类型对应不同的码 
      private static final int TABLE_DIR = 1;
      private static final int TABLE_ITEM = 2;
  
      private static final Uri NOTIFY_URI = Uri.parse("content://" + AUTHORITY + "/tabel");
  
  	//声明uriMatch
      private static final UriMatcher uriMatcher;
  
      //在静态代码块中添加要匹配的 Uri
      static {
          //匹配不成功返回NO_MATCH(-1)
          uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
         
          uriMatcher.addURI(AUTHORITY, "tabel", TABLE_DIR);// 匹配tabel表中所有数据
          uriMatcher.addURI(AUTHORITY, "tabel/#", TABLE_ITEM);// 匹配tabel中单条数据
      }
  ```

* onCreate

  ```java
  @Override
  public boolean onCreate() {
      //创建数据库
      return false;
  }
  ```

* query

  ```java
  @Override
  public Cursor query(Uri uri, String[] projection, String selection,
                      String[] selectionArgs, String sortOrder) {
       switch (uriMatch.match(uri)){
           case TABLE_DIR:
               //数据库操作查询table表中所有数据
               break;
           case TABLE_ITEM:
               //数据库操作查询table表中所有数据
               break;
           default:
               break;
       }
       ...
  }
  ```

* delete

  ```java
  @Override
      public int delete(Uri uri, String selection, String[] selectionArgs) {
          //打开数据库
          int deletedRows = 0;
          switch (uriMatch.match(uri)){
              case TABLE_DIR:
                  //数据库操作删除table表中所有数据
                  deletedRows = db.delete("表名",selection,selectionArgs);
                  break;
              case TABLE_ITEM:
                  String bookId = uri.getPathSegments().get(1);
                  //数据库操作查询table表中某条数据
                  break;
              default:
                  break;
          }
          return deletedRows;
      }
  ```

* insert

  ```java
  @Override
  public Uri insert(Uri uri, ContentValues values) {
      //打开数据库
      Uri uriReturn = null;
      switch (uriMatch.match(uri)){
          case TABLE_DIR:
          case TABLE_ITEM:
              long newBookId = db.insert("表名",null,values);
              uriReturn = Uri.parse(AUTHORITY+newBookId);
              break;
          default:
              break;
      }
      return uriReturn;
  }
  ```

* updata

  与delete类似 把delete换成updata

  ```java
  @Override
  public int update(Uri uri, ContentValues values, String selection,
                    String[] selectionArgs) {
      //打开数据库
      int upDataRows = 0;
      switch (uriMatch.match(uri)){
          case TABLE_DIR:
              //数据库操作更新table表中所有数据
              upDataRows = db.update("表名",selection,selectionArgs);
              break;
          case TABLE_ITEM:
              String bookId = uri.getPathSegments().get(1);
              //数据库操作更新table表中某条数据
              break;
          default:
              break;
      }
      return upDataRows;
  }
  ```

* getType

  每个uri对象对应一个MIME类型的格式

  ![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\内容提供器MINI类型.png)

  ```java
  @Override
  public String getType(Uri uri) {
      //按格式拼接 return对应MIME类型的字符串
  }
  ```

## 五、Fragment

### 简介

* 可以嵌入在活动中的UI片段

* 好处：可以兼顾平板

* 与活动的通信

  * 在活动中获取碎片

    ```java
    RightFragment rightFragment=（RightFragment）getFragmentManager（）.findFragmentById(R.id.right_fragment)
    ```

  * 在碎片中获取活动

    ```java
    MainActivity activity=（MainActivity）getActivity（）;
    ```

    

### 使用流程

#### 基础使用

* 快捷创建（AS自动注册 生成对应xml布局文件和类）

* Activity的布局文件

  像添加控件一样添加Fragment

  ```xml
  <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@+id/main_out_layout"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:background="@mipmap/bg3"
      >
  
      <fragment
          android:layout_width="0dp"
          android:layout_height="match_parent"
          android:id="@+id/left_fragment"
          android:name="com.suzy.fragmenttestone.LeftFragment"
          android:layout_weight="1"/>
      <fragment
          android:layout_width="0dp"
          android:layout_height="match_parent"
          android:id="@+id/right_fragment"
          android:name="com.suzy.fragmenttestone.RightFragment"
          android:layout_weight="1"/>
  
  </RelativeLayout>
  ```

* Fragment布局文件

  跟其他布局文件一样 可以设置碎片中的控件

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:tools="http://schemas.android.com/tools"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      tools:context=".city_main.CityWeatherFragment"
      android:layout_gravity="center"
      >
  
      <RelativeLayout
          android:layout_width="match_parent"
          android:layout_height="wrap_content">
  
          <TextView
              android:id="@+id/frag_tv_currenttemp"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:layout_centerHorizontal="true"
              android:text="3℃"
              android:textSize="70sp"
              android:textStyle="bold" />
          ...
  
      </RelativeLayout>
  </FrameLayout>
  ```

* Fragment类中加载Fragment

  ```java
  @Override
      public View onCreateView(LayoutInflater inflater, ViewGroup container,
                               Bundle savedInstanceState) {
          View view = inflater.inflate(R.layout.fragment_city_weather, container, false);
          //进行fragment内控件的初始化 设置监听等的操作
          return view;
      }
  ```

#### 其他应用 

> 与viewPager2、TabLayout实现联动

![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\ViewPager联动.png)

* MainActivity布局

  像加入控件一样加入TabLayout和Viewpager2

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto"
      xmlns:tools="http://schemas.android.com/tools"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      tools:context=".MainActivity">
  
      <com.google.android.material.tabs.TabLayout
          android:id="@+id/tab_layout"
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          app:layout_constraintTop_toTopOf="parent"
          app:layout_constraintStart_toStartOf="parent"
          />
      <androidx.viewpager2.widget.ViewPager2
          android:id="@+id/view_pager"
          android:layout_width="match_parent"
          android:layout_height="match_parent"
          app:layout_constraintTop_toBottomOf="@+id/tab_layout"
          android:layout_below="@+id/tab_layout"
          />
          <ImageView
              android:id="@+id/book_iv"
              android:layout_width="40dp"
              android:layout_height="40dp"
              android:layout_centerHorizontal="true"
              android:layout_alignParentBottom="true"
              android:src="@drawable/ic_book"/>
  </RelativeLayout>
  ```

* Fragment的布局文件（自由发挥）

  其中一个Fragement

  顶部相对布局中含一张图和文本

  中间是一张图

  底部是一张小图

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:tools="http://schemas.android.com/tools"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      tools:context=".taskday.DayFragment">
  
      <LinearLayout
          android:layout_width="match_parent"
          android:layout_height="match_parent"
          android:orientation="vertical">
          <RelativeLayout
              android:id="@+id/day_top_layout"
              android:layout_width="match_parent"
              android:layout_height="wrap_content">
  
              <ImageView
                  android:id="@+id/day_play_iv"
                  android:layout_width="30dp"
                  android:layout_height="30dp"
                  android:src="@drawable/ic_play"
                  android:layout_centerVertical="true"/>
  
              <TextView
                  android:id="@+id/day_sentence_tv"
                  style="@style/TitleText"
                  android:layout_width="match_parent"
                  android:layout_height="wrap_content"
                  android:layout_toRightOf="@+id/day_play_iv" />
          </RelativeLayout>
          <ImageView
              android:id="@+id/day_picture_iv"
              android:layout_width="440dp"
              android:layout_height="560dp"
              android:layout_gravity="center_horizontal"
              />
      </LinearLayout>
  
  </FrameLayout>
  ```

* Fragment类

  跟普通的Activity类差不多

  ```java
  public class DayFragment extends Fragment {
  
      private View view;
      private TextView sentenceTv;
      private ImageView playIv, pictureIv;
      private String playPath;
  
      MediaPlayer player = new MediaPlayer();
  
      public DayFragment() {
          // Required empty public constructor
      }
  
      @Override
      public View onCreateView(LayoutInflater inflater, ViewGroup container,
                               Bundle savedInstanceState) {
          //加载视图
          view = inflater.inflate(R.layout.fragment_day, container, false);
          //初始化
          initView();
          //设置播放的监听
          playIv.setOnClickListener(new View.OnClickListener() {
              @Override
              public void onClick(View view) {
                  try {
                      player.reset();
                      player.setDataSource(playPath);
                      player.prepare();
                      player.start();
                  } catch (IOException e) {
                      e.printStackTrace();
                  }
              }
          });
          return view;
      }
  
      private void initView() {
          //加载控件
          playIv = view.findViewById(R.id.day_play_iv);
          pictureIv = view.findViewById(R.id.day_picture_iv);
          sentenceTv = view.findViewById(R.id.day_sentence_tv);
          //初始化控件
          sentenceTv.setText("I feel I stand in a desert with my hands outstretched, and you are raining down upon me.");
          playPath = "https://staticedu-wps.cache.iciba.com/audio/349d814dccb823f3263a8be842b6ac71.mp3";
          pictureIv.setImageResource(R.mipmap.picture);
      }
  }
  ```

* MainActivity类

  ```java
  public class MainActivity extends AppCompatActivity {
      private TabLayout myTab;
      private ViewPager2 myPager2;
      private ImageView bookIv;
      private DatabaseHelper mDatabaseHelper;
      List<String> titles=new ArrayList<>();
      List<Fragment> fragments=new ArrayList<>();
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_main);
  
          //加载控件
          myTab = findViewById(R.id.tab_layout);
          myPager2 = findViewById(R.id.view_pager);
          bookIv = findViewById(R.id.book_iv);
  
          //添加标题
          titles.add("查词");
          titles.add("每日一句");
          titles.add("翻译");
  
          //添加Fragment进去
          fragments.add(new SearchFragment());
          fragments.add(new DayFragment());
          fragments.add(new TranslateFragment());
  
          //实例化适配器
          MyViewPager2Adapter myAdapter=new MyViewPager2Adapter(getSupportFragmentManager(),getLifecycle(),fragments);
          //设置适配器
          myPager2.setAdapter(myAdapter);
          //TabLayout和Viewpager2进行关联
          new TabLayoutMediator(myTab, myPager2, new TabLayoutMediator.TabConfigurationStrategy() {
              @Override
              public void onConfigureTab(@NonNull TabLayout.Tab tab, int position) {
                  tab.setText(titles.get(position));
              }
          }).attach();
  
  
      }
  }
  ```

* ViewPager2适配器

  与frament集合绑定  让位置与fragment对应上

  ```java
  public class MyViewPager2Adapter extends FragmentStateAdapter {
      List<Fragment> fragments;
      public MyViewPager2Adapter(@NonNull FragmentManager fragmentManager, @NonNull Lifecycle lifecycle, List<Fragment> fragments) {
          super(fragmentManager, lifecycle);
          this.fragments = fragments;
      }
  
      @NonNull
      @Override
      public Fragment createFragment(int position) {
          return fragments.get(position);
      }
  
      @Override
      public int getItemCount() {
          return fragments.size();
      }
  }
  ```