# JAVA网络编程笔记

## 一、基础知识

* 特殊的IP地址 本机：127.0.0.1 == localhost
* TCP协议
  * 三次握手  发送端发送请求 接收端回复确认 发送端发送确认
  * 安全 可靠传输
  * 下载文件 浏览网页
* UDP协议
  * 只负责发送
  * 不安全的

## 二、stocket实现网络编程

* 基于TCP协议
* 可用于实现双向安全连接网络通信
* 需要借助数据流完成数据的传递工作
* Service
  * 创建 ServerSocket对象 绑定端口
  * 通过accept（）监听阻塞 等待对方连接
  * 连接后通过输入输出流传递信息
  * 关闭释放资源
* Client
  * 创建socket 指定服务器IP地址和端口号
  * 发送请求被accept侦听到后建立连接
  * 通过输出输入流传递信息
  * 关闭释放资源

## 三、聊天室实例

* 服务端Server
  * main函数 创建ServerSocket对象（要端口）accept侦听  放入客户端集合中
  * 重写run方法 接收到其中一个客户端要发送的消息给其他用户
  * Channel
    * 继承runnable接口
    * 构造函数传入client 并且获取输入输出流
    * 获得第一次输入的用户名
    * 发送给客户端 自己的和其他人的
    * 释放资源
  * receive（）输入流的读入
  * send（）输出流的发送
  * sendOthers（）
    * 规定私聊的格式 @  仅发送给某一位 遍历管理器找到后发送
    * 否则就群聊  给每个客户端都发一次
  * release() 某客户端退出 向其他客户端发送提示

```java
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.CopyOnWriteArrayList;

public class ChatServer {
    private static CopyOnWriteArrayList<Channel> all=new CopyOnWriteArrayList<Channel>();
    public static void main(String[] args) throws IOException {
        System.out.println("---服务端---");
        //指定端口 创建服务器
        ServerSocket server=new ServerSocket(7002);
        while (true){
            Socket client =server.accept();
            System.out.println("---一个客户已登录---");
            Channel c = new Channel(client);
            all.add(c);    //交给容器管理所有的成员
            new Thread(c).start();
        }
    }

    //一个客户代表一个Channel
    static class Channel implements Runnable{
        private DataInputStream dis;
        private DataOutputStream dos;
        public Socket client;
        private boolean isRunning ;
        private String name;

        public Channel(Socket client){
            this.client=client;
            try{
                dis = new DataInputStream(client.getInputStream());
                dos = new DataOutputStream(client.getOutputStream());
                isRunning = true;
                //获取名称
                this.name = receive();
                //欢迎
                this.send("欢迎你的到来");
                sendOthers(this.name+"来到了QG聊天室",true);
            }catch (IOException e){
                System.out.println("---1---");
                release();
            }
        }
        
        @Override
        public void run() {
            while (isRunning){
                String msg = receive();
                if(!msg.equals("")){
                   sendOthers(msg,false);
                }
            } 
        }

        //接收消息
        private String receive(){
            String msg="";
            try {
                msg =dis.readUTF();
            }catch (IOException e){
                System.out.println("---2---");
                release();
            }
            return msg;
        }

        //发送消息
        private void send(String msg){
            try {
                dos.writeUTF(msg);
                dos.flush();
            } catch (IOException e) {
                System.out.println("---3---");
                release();
            }
        }

        //群聊:获取自己的消息发给别人
        //私聊：约定格式：@xxx:msg
        private void sendOthers(String msg,boolean isSys){
            boolean isPrivate = msg.startsWith("@");
            if(isPrivate){//私聊
                int idx = msg.indexOf(":");
                //获取目标和资源
                String targetName = msg.substring(1,idx);
                msg = msg.substring(idx+1);
                for (Channel other : all) {
                    if (other.name.equals(targetName)) {//遍历找目标
                        other.send(this.name+"拍了拍你说："+msg);
                        break;
                    }
                }
            }else {//群聊
                for (Channel other : all) {
                    if (other == this) {
                        continue;
                    }
                    if (!isSys) {
                        other.send(this.name + "对所有人说" + msg);
                    } else {
                        other.send(msg);//系统消息
                    }
                }
            }
        }



        //释放资源
        private void release(){
            this.isRunning = false;
            QGUtils.close(dis,dos,client);
            //退出
            all.remove(this);
            sendOthers(this.name+"离开了大家庭...",true);
        }

       
    }
}

```



* 客户端Client
  * 创建socket对象建立连接，需要IP地址和端口号
  * 提示输入用户名
  * 获取键盘输入得用户名
  * 创建线程对象进行发送消息，需要用户名（用以区分子线程）
  * 创建线程对象进行接收消息

```java
import java.io.*;
import java.net.Socket;
import java.net.UnknownHostException;

public class ChatClient {
    public static void main(String[] args) throws UnknownHostException,IOException {
        System.out.println("---客户端---");
        //建立连接 
        Socket client = new Socket("localhost",7002);
        //客户端发送消息
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        System.out.println("请输入用户名：");
        String name =br.readLine();
        new Thread(new Send(client,name)).start();
        new Thread(new Receive(client)).start();
    }
}
```



* 接收封装Receive
  * 继承runnable接口
  * 设置构造函数，传入客户端的socket，用输入流获取接收到的数据
  * 重写runnable方法：把接收到的消息显示出来
    * receive（）将输入流转化
    * release() 释放资源

```java
public class Receive implements Runnable{
    private DataInputStream dis;
    public Socket client;
    public boolean isRunning;
    public Receive(Socket client){
        this.client=client;
        try {
            dis =new DataInputStream(client.getInputStream());
        } catch (IOException e) {
            System.out.println("===2===");
            release();
        }
    }
    
    @Override
    public void run() {
    	boolean isRunning =true;
        while (isRunning){
            String msg =receive();
            if(!msg.equals("")){
                System.out.println(msg);
            }

        }

    }
    
    //接收消息
    private String receive(){
        String msg="";
        try {
            msg =dis.readUTF();
        }catch (IOException e){
            System.out.println("===4===");
            release();
        }
        return msg;
    }

    //释放资源
    private void release(){
        this.isRunning = false;
        QGUtils.close(dis,client);
    }
}
```



* 发送封装Send

  * 继承runnable接口

  * 设置构造函数

    * 传入客户端的socket及名称
    * 用console获取要控制台要发送的数据（键盘输入）
    * 用输出流获取客户端要发出的数据
    * 发送名称（第一次建立）

  * 重写runnable方法：获取控制台消息并把消息发送出去

    * getStrFromConsole()

    * send（）输出流传输

```java
public class Send implements Runnable{
    private BufferedReader console;
    private DataOutputStream dos;
    private Socket client;
    private boolean isRunning;
    private String name;
    public Send(Socket client,String name){
        this.client=client;
        console =new BufferedReader(new InputStreamReader(System.in));
        this.name=name;
        try {
            dos = new DataOutputStream(client.getOutputStream());
            //发送名称
            send(name);
        } catch (IOException e) {
            System.out.println("===1===");
            this.release();
        }
    }

    @Override
    public void run() {
    	boolean isRunning =true;
        while (isRunning){
            String msg = getStrFromConsole();
            if(!msg.equals("")){
                send(msg);
            }

        }
    }
    //发送消息
    private void send(String msg){
        try {
            dos.writeUTF(msg);
            dos.flush();
        } catch (IOException e) {
            System.out.println("===3===");
            release();
        }
    }


    //从控制台获取消息
    private String getStrFromConsole(){
        try {
            return console.readLine();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return "";
    }
    //释放资源
    private void release(){
        this.isRunning = false;
        QGUtils.close(dos,client);
    }

}
```



* 关闭工具类封装QGtils

把关闭流 关闭socket等关闭的封装起来，实现代码复用

```java
public class QGUtils {
    /**
     * 释放资源
     */
    public static void close(Closeable... targets){//采用不定参数提高类的适用性
        for(Closeable target:targets){
            try{
                if (null!=target){
                    target.close();
                }
            }catch (Exception e){

            }
        }
    }
}
```

