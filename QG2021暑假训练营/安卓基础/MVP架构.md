# MVP架构基本理解

## 零、对MVP架构的理解

* V和M层完全分离,互不关心
* P层同时持有V和M,负责业务处理
* V:  Acyivity Fragment layout  自定义View
* p:  V和M层的交互
* M:负责业务数据
* 心得: 理解三层不难  对于实现的理解才是关键(运用泛型 面向接口编程 接口回调等)

## 一、Model层

### 接口（回调接口）

里面定义回调成功或者失败的方法

```java
public interface ModuleInterface {
    //获取数据状态回调的接口
    void LoadSuccess(List<String> data);//获取成功要传入获取到的数据，在重写的时候就可以用这些数据了

    void LoadFailed();
}
```

### 类（实现逻辑）

持有回调接口 构造函数传入回调接口

方式一：构造函数传入

```java
public class DataModule {

    List data = new ArrayList();
     
     ModuleInterface mCallback;//传入接口才能进行覆写

    public DataModule(ModuleInterface moduleInterface){
        this.mCallback = moduleInterface;
    }
    ......
}
```

方式二：继承接口

MainContract 抽象出来的公共类  减少接口文件 一目了然

（把不同数据的处理搞成不同的接口，不同的View要用的话直接实现接口就可以了）

```java
public interface MainContract {
    interface IMainModel {
        void requestBaidu(Callback callback);
    }
 
    interface IMainView {
        void showDialog();
 
        void succes(String content);
    }
 
    interface IMainPresenter {
        void handlerData();
    }
}
```

okHttp的举例：返回一个callback给presenter

```java

public class DataModel implements MainContract.IMainModel {
 
    @Override
    public void requestBaidu(Callback callback) {
        OkHttpClient client = new OkHttpClient();
        Request request = new Request.Builder()
                .url("https://www.baidu.com/")
                .build();
        client.newCall(request).enqueue(callback);
    }
}

```

在DataModule里面可以实现任意请求数据的逻辑  网络请求等等

然后在实现过程的结尾回调函数 输入数据，直接调用即可

```java
//数据准备成功后，通过接口通知P：数据已经准备完毕，可以继续下面的逻辑操作
mCallback.LoadSuccess(strings);
```

## 二、Presenter层

### 类

* 继承了数据层的回调接口 实例化model的类

* 覆写其中的方法的时候操作view层的函数
* 构造函数要传入View层的类

```java
public class MvpPresenter implements ModuleInterface{

    private MvpView mvpView;

    //获取Module实例，执行数据处理方法。
    DataModule dataModule = new DataModule(this);;
    public MvpPresenter(MvpView mvpView){
        this.mvpView = mvpView;

    }

    //开始执行逻辑
    public void handledata() {
        mvpView.showLoading();
        dataModule.getdata();
    }

    public void onItemClick(int position){
        mvpView.showMessage("点击了item"+position);
    }

    //数据传过来之后，继续执行逻辑
    @Override
    public void LoadSuccess(List<String> data) {
        mvpView.hideLoading();
        mvpView.setListItem(data);
}

    @Override
    public void LoadFailed() {
     //处理失败后View的操作
        mvpView.failed();
    }
}
```

## 三、View层

### 接口

具体UI操作的函数

```java
public interface MvpViewInterface {
       void showLoading();
       void hideLoading();
       void setListItem(List<String> data);
       void failed();
       void showMessage(String message);
}
```

### 类（Activity/Fragment）

* 继承View的接口 声明Presenter的类

* 在onCreate中实例化Presenter 并调用开始执行的函数 

```java
@Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_mvp);
            mvpListView = (ListView)findViewById(R.id.mvp_listview);
            mvpListView.setOnItemClickListener(this);
            pb = (ProgressBar) findViewById(R.id.mvp_loading);
            //获取MvpPresenter实例
            mvpPresenter = new MvpPresenter(this);
            //开始执行，直接调用P中方法
            mvpPresenter.handledata();
        }
```

* 重写View接口中的方法

```java
@Override
    public void setListItem(List<String> data) {
        ArrayAdapter adapter = new ArrayAdapter(MvpActivity.this,android.R.layout.simple_list_item_1,data);
        mvpListView.setAdapter(adapter);
    }

    @Override
    public void failed() {
        Toast.makeText(MvpActivity.this,"加载失败",Toast.LENGTH_SHORT).show();
    }

    @Override
    public void showMessage(String message) {
        Toast.makeText(this,message,Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
        mvpPresenter.onItemClick(position);
    }
```

## 四、baseView的设置

### baseActivity

基类默认子类要做的事 都抽象出来

*  初始化布局  拿id
* 初始化控件
* 初始化监听器
* 初始化数据
* 创建
* 销毁

```java
public abstract class BaseActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable  Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(getContentViewId());

        initView();
        initData();
        initListener();
    }

    public abstract void initView();

    public abstract void initData();

    public abstract void initListener();

    //拿到布局的id
    public abstract int getContentViewId();

    @Override
    protected void onDestroy() {
        super.onDestroy();
        destroy();
    }

    public abstract void destroy();
}
```

* 持有P层 实例化P层  绑定建立联系

  ```java
  public abstract class BaseActivity<P extends BasePresenter> extends AppCompatActivity implements View.OnClickListener {
  
      public  P mPresenter;
      
      @Override
      protected void onCreate(@Nullable  Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          
          mPresenter = getPresenterInstance();
          mPresenter.bindView(this);
      }
      public abstract P getPresenterInstance();
  
  }
  ```

### baseModel

* 要持有P层  所以用泛型（传入限定是BasePresenter的子类）

  ```java
  public class BaseModel<P extends BasePresenter> {
  
      public P mPresenter;
  
      public BaseModel(P mPresenter) {
          this.mPresenter = mPresenter;
      }
  }
  ```

  

### basePresenter

* 拿到P层和V层的子类 定义泛型   

  ```java
  public abstract class BasePresenter<V extends BaseActivity,M extends BaseModel> {
  
      public V mView;
  
      public M mModel;
  }
  ```

* 初始化 建立关系（绑定及解绑）

  ```java
  public void bindView(V mView){
      this.mView = mView;
  }
  
  public void unBindView(){
      this.mView = null;
  }
  ```

* 在构造p时就绑定m  让P去实例化对应的model

  ```java
  public BasePresenter(){
      this.mModel = getModelInstance();
  }
  
  public abstract M getModelInstance();
  ```

## 五、面向方法实现

### 契约类接口

```java
public interface ILogin {
    public interface M{
        void requestLogin(String name, String pwd) throws Exception;
    }

    public interface VP{
        void requestLogin(String name, String pwd);

        void responseLoginResult(boolean loginStatusResult);
    }
}
```

### LoginActivity

* 继承BaseActivity

* 实现VP接口

```java
public class LoginActivity extends BaseActivity<LoginPresenter> implements ILogin.VP{

    private EditText etName;
    private EditText etPwd;
    private Button btLogin;
    private final static String TAG = "LoginActivity";

    @Override
    public void initView() {
        etName = findViewById(R.id.et_name);
        etPwd = findViewById(R.id.et_pwd);
        btLogin = findViewById(R.id.bt_login);
    }

    @Override
    public void initData() {

    }

    @Override
    public void initListener() {
        btLogin.setOnClickListener(this);
    }

    @Override
    public int getContentViewId() {
        return R.layout.activity_login;
    }

    @Override
    public LoginPresenter getPresenterInstance() {
        return new LoginPresenter();
    }

    @Override
    public void destroy() {

    }

    @Override
    public void onClick(View v) {
        String name = etName.getText().toString();
        String pwd = etPwd.getText().toString();
        Log.d(TAG,"点击"+name+pwd);

        requestLogin(name,pwd);
    }

    @Override
    public void requestLogin(String name, String pwd) {
        mPresenter.requestLogin(name,pwd);
    }

    @Override
    public void responseLoginResult(boolean loginStatusResult) {
        Log.d(TAG,"result" +String.valueOf(loginStatusResult));
        Toast.makeText(this,loginStatusResult?"登录成功":"登录失败",Toast.LENGTH_SHORT).show();
    }
}
```



### LoginModel

```java
public class LoginModel extends BaseModel<LoginPresenter> implements ILogin.M{

    public LoginModel(LoginPresenter mPresenter) {
        super(mPresenter);
    }

    @Override
    public void requestLogin(String name, String pwd) throws Exception {
        //请求服务器登录接口，然后拿到服务器返回JSON数据

        if("abc".equals(name)&&"123".equals(pwd)){
            mPresenter.responseLoginResult(true);
        }else {
            mPresenter.responseLoginResult(false);
        }
    }
}
```



### LoginPresenter

```java
public class LoginPresenter extends BasePresenter<LoginModel,LoginActivity> implements ILogin.VP{

    private final String TAG = "LoginPresenter";

    @Override
    public LoginModel getModelInstance() {
        return new LoginModel(this);
    }

    @Override
    public void requestLogin(String name, String pwd) {
        //接收请求的信息进行处理
        //...
        try {
            mModel.requestLogin(name, pwd);
        } catch (Exception e) {
            e.printStackTrace();
            //做异常处理
            //保存日记等等
        }
    }

    @Override
    public void responseLoginResult(boolean loginStatusResult) {
        //真实开发过程中是 解析数据  传给View
        mView.responseLoginResult(loginStatusResult);
    }

}
```

### 升级  加入错误信息的处理

BaseActivity

```java
public abstract P getPresenterInstance();

//处理响应错误信息
public abstract <ERROR extends Object> void respondsError (ERROR error, Throwable throwable);

```

LpoginActivity

```java
@Override
public <ERROR> void respondsError(ERROR error, Throwable throwable) {
    Toast.makeText(this,":" +error,Toast.LENGTH_SHORT).show();
}
```

## 六、面向接口编程

> 1:30:00

### 增加SuperBase

不用用户主动去创建接口，只需要传入接口就可以自动创建了

CONTRACT 契约接口泛型

```java
public abstract class SuperBase<CONTRACT> {
    public abstract CONTRACT getContract();
}
```

### baseModel

BaseModel子类拿到泛型接口后给父类SuperBase

```java
public abstract class BaseModel<P extends BasePresenter,CONTRACT> extends SuperBase<CONTRACT> {

    public P mPresenter;

    public BaseModel(P mPresenter) {
        this.mPresenter = mPresenter;
    }
}
```

### baseActivity

让子类把CONTRACT 传给自己

然后让子类去重写契约接口的实现类   public abstract CONTRACT getContract();

```java
public abstract class BaseActivity<P extends BasePresenter,CONTRACT> extends AppCompatActivity implements View.OnClickListener {

    public abstract CONTRACT getContract();

    public  P mPresenter;
    @Override
    protected void onCreate(@Nullable  Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(getContentViewId());

        initView();
        initData();
        initListener();

        mPresenter = getPresenterInstance();
        mPresenter.bindView(this);
    }

    public abstract void initView();

    public abstract void initData();

    public abstract void initListener();

    public abstract int getContentViewId();

    public abstract P getPresenterInstance();

    //处理错误信息
    public abstract <ERROR extends Object> void responseError(ERROR error,Throwable throwable );

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mPresenter.unBindView();
        destroy();
    }

    public abstract void destroy();
}
```

### basePresenter

同理 子类拿到泛型接口后给父类

```java
public abstract class BasePresenter<M extends BaseModel,V extends BaseActivity,CONTRACT> extends SuperBase<CONTRACT>{

    public V mView;

    public M mModel;

    public BasePresenter(){
        this.mModel = getModelInstance();
    }

    public void bindView(V mView){
        this.mView = mView;
    }

    public void unBindView(){
        this.mView = null;
    }

    public abstract M getModelInstance();
}
```

### 接口类 ILogin

M层接口需要找数据并抛出异常

VP层接口发送请求及返回结果

```java
public interface ILogin {
    public interface M{
        void requestLogin(String name, String pwd) throws Exception;
    }

    public interface VP{
        void requestLogin(String name, String pwd);

        void responseLoginResult(boolean loginStatusResult);
    }
}
```

### LoginModel

* 继承BaseModel 要求传入 P层和契约接口（根据提示传即可，基类已经规范好了）
* 构造函数  调用父类构造函数
* 覆写要写的函数  按要求返回接口(因为没有就new一个)
* 在接口覆写的函数里面书写逻辑  进行数据业务处理 根据结果返回给P层

```java
public class LoginModel extends BaseModel<LoginPresenter,ILogin.M>{

    public LoginModel(LoginPresenter mPresenter) {
        super(mPresenter);
    }

    @Override
    public ILogin.M getContract() {
        return new ILogin.M() {
            @Override
            public void requestLogin(String name, String pwd) throws Exception {
                //请求服务器登录接口，然后拿到服务器返回JSON数据

                if("abc".equals(name)&&"123".equals(pwd)){
                    mPresenter.getContract().responseLoginResult(true);
                }else {
                    mPresenter.getContract().responseLoginResult(false);
                }
            }
        };
    }
}
```

### LoginPresenter

* 继承基类

* 按要求传入了类和契约类接口 
* 重写方法 返回接口new ILogin.VP()（再实现接口里面的方法）
* 按覆写的方法需要创建M层的实例 并传入P的实例进行绑定（本身就是 this）

* 在接到V层请求后在函数中调用M去处理数据
* 收到M层结果后返回给V层

```java
public class LoginPresenter extends BasePresenter<LoginModel,LoginActivity,ILogin.VP>{

    private final String TAG = "LoginPresenter";

    @Override
    public LoginModel getModelInstance() {
        return new LoginModel(this);
    }

    @Override
    public ILogin.VP getContract() {
        return new ILogin.VP() {
            @Override
            public void requestLogin(String name, String pwd) {
                //接收请求的信息进行处理
                //...
                try {
                    mModel.getContract().requestLogin(name, pwd);
                } catch (Exception e) {
                    e.printStackTrace();
                    //做异常处理
                    //保存日记等等
                }
            }

            @Override
            public void responseLoginResult(boolean loginStatusResult) {
                //真实开发过程中是 解析数据  传给View
                mView.getContract().responseLoginResult(loginStatusResult);
            }
        };
    }
}
```

### LoginActivity

* 继承基类
* 初始化控件
* 设置监听（V层与用户交互 接收用户的请求然后调用P层的方法）

* 按要求传入了类和契约类接口 

* 重写方法 返回接口new ILogin.VP()（再实现接口里面的方法： 里面是让P层去找数据  接收P层传回的数据然后更新UI）

```java
public class LoginActivity extends BaseActivity<LoginPresenter,ILogin.VP>{

    private EditText etName;
    private EditText etPwd;
    private Button btLogin;
    private final static String TAG = "LoginActivity";

    @Override
    public ILogin.VP getContract() {
        return new ILogin.VP() {
            @Override
            public void requestLogin(String name, String pwd) {
                mPresenter.getContract().requestLogin(name,pwd);
            }

            @Override
            public void responseLoginResult(boolean loginStatusResult) {
                Toast.makeText(LoginActivity.this,loginStatusResult? "成功":"失败",Toast.LENGTH_SHORT).show();
            }
        };
    }

    @Override
    public void initView() {
        etName = findViewById(R.id.et_name);
        etPwd = findViewById(R.id.et_pwd);
        btLogin = findViewById(R.id.bt_login);
    }

    @Override
    public void initData() {

    }

    @Override
    public void initListener() {
        btLogin.setOnClickListener(this);
    }

    @Override
    public int getContentViewId() {
        return R.layout.activity_login;
    }

    @Override
    public LoginPresenter getPresenterInstance() {
        return new LoginPresenter();
    }

    @Override
    public <ERROR> void respondsError(ERROR error, Throwable throwable) {
        Toast.makeText(this,":" +error,Toast.LENGTH_SHORT).show();
    }

    @Override
    public void destroy() {

    }

    @Override
    public void onClick(View v) {
        String name = etName.getText().toString();
        String pwd = etPwd.getText().toString();
        Log.d(TAG,"点击"+name+pwd);

        getContract().requestLogin(name,pwd);
    }
}
```

### 实操心得

#### 步骤

* 先导入包  声明权限
* 创建各层的类并继承基类
* 声明Activity
* 写好布局文件
* 写好接口的契约类
* 写M层基类要传入的参数 生成覆写方法
* 写P层基类要传入的参数 生成覆写方法
* 写V层基类要传入的参数 生成覆写方法
* 写V层的具体实现
* 写P层的具体实现
* 写M层的具体实现