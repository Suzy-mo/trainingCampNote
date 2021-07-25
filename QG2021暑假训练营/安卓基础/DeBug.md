# DeBug

## Gosn

```java
E/AndroidRuntime: FATAL EXCEPTION: Thread-2
    Process: com.qg.summerglidetest, PID: 4973
    com.google.gson.JsonSyntaxException: java.lang.IllegalStateException: Expected BEGIN_ARRAY but was STRING at line 1 column 80 path $.exchange.word_third
        at com.google.gson.internal.bind.ReflectiveTypeAdapterFactory$Adapter.read(ReflectiveTypeAdapterFactory.java:226)
        at com.google.gson.internal.bind.ReflectiveTypeAdapterFactory$1.read(ReflectiveTypeAdapterFactory.java:131)
        at com.google.gson.internal.bind.ReflectiveTypeAdapterFactory$Adapter.read(ReflectiveTypeAdapterFactory.java:222)
        at com.google.gson.Gson.fromJson(Gson.java:932)
        at com.google.gson.Gson.fromJson(Gson.java:897)
        at com.google.gson.Gson.fromJson(Gson.java:846)
        at com.google.gson.Gson.fromJson(Gson.java:817)
        at com.qg.summerglidetest.NetWorkUtil.jsonExchange(NetWorkUtil.java:47)
        at com.qg.summerglidetest.GsonActivity$1.lambda$onClick$1$GsonActivity$1(GsonActivity.java:38)
        at com.qg.summerglidetest.-$$Lambda$GsonActivity$1$XZST3CU4OEl5GW5QckJTCF3EPOU.run(lambda)
        at java.lang.Thread.run(Thread.java:761)
     Caused by: java.lang.IllegalStateException: Expected BEGIN_ARRAY but was STRING at line 1 column 80 path $.exchange.word_third
        at com.google.gson.stream.JsonReader.beginArray(JsonReader.java:351)
        at com.google.gson.internal.bind.CollectionTypeAdapterFactory$Adapter.read(CollectionTypeAdapterFactory.java:80)
        at com.google.gson.internal.bind.CollectionTypeAdapterFactory$Adapter.read(CollectionTypeAdapterFactory.java:61)
        at com.google.gson.internal.bind.ReflectiveTypeAdapterFactory$1.read(ReflectiveTypeAdapterFactory.java:131)
        at com.google.gson.internal.bind.ReflectiveTypeAdapterFactory$Adapter.read(ReflectiveTypeAdapterFactory.java:222)
        at com.google.gson.internal.bind.ReflectiveTypeAdapterFactory$1.read(ReflectiveTypeAdapterFactory.java:131) 
        at com.google.gson.internal.bind.ReflectiveTypeAdapterFactory$Adapter.read(ReflectiveTypeAdapterFactory.java:222) 
        at com.google.gson.Gson.fromJson(Gson.java:932) 
        at com.google.gson.Gson.fromJson(Gson.java:897) 
        at com.google.gson.Gson.fromJson(Gson.java:846) 
        at com.google.gson.Gson.fromJson(Gson.java:817) 
        at com.qg.summerglidetest.NetWorkUtil.jsonExchange(NetWorkUtil.java:47) 
        at com.qg.summerglidetest.GsonActivity$1.lambda$onClick$1$GsonActivity$1(GsonActivity.java:38) 
        at com.qg.summerglidetest.-$$Lambda$GsonActivity$1$XZST3CU4OEl5GW5QckJTCF3EPOU.run(lambda) 
        at java.lang.Thread.run(Thread.java:761) 
D/OpenGLRenderer: endAllActiveAnimators on 0xb78a5b80 (RippleDrawable) with handle 0xb85b51d0

```

Expected BEGIN_ARRAY but was STRING at line 1 column 80 path $.exchange.word_third

预期是一个数组而不是一个字符串

问题所在：查单词由空的就返回了空格相当于字符串

暂时解决：不序列化可能为空的部分

>待解决？问师兄

```java
"word_third": [
            "words"
        ],
        "word_er": "",
        "word_est": ""
```

## 控件空指针

```java
'android.view.View android.view.View.findViewById(int)' on a null object reference
```

>超级低级的错误

就是把 = 写成了 .

就出现了控件一直没有被引用的问题    