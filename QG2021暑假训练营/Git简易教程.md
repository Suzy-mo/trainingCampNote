# Git简易教程

## 一、本地提交

### 忽略提交

根目录下有**.gitignore**文件（IDEA会自动生成）

也可以自己再修改减轻项目负担

### 基础设置

右键加鼠标滚动，可以调字体大小；需要固定的话就在右键 options text 设置字体大小和编码U-8

小技巧：在终端中选中就是复制  ；粘贴：右键-> paste

清屏    输入 clear

### 新建本地仓库

* 在工作目录下右键 选择 Git Bush Here打开窗口  输入下面代码初始化

```git
git init
```

* 或者全程命令 

```git
mkdir 文件名

cd 文件名

git init 
```

### 本地仓库添加内容

* 查看有无可以提交的内容

  ```java
  git status
  ```

  nothing to commit 缓存区为空

  Untracked files（没有被管理）

* 提交到暂存区

  提交某个文件

  ```git
  git add (工作目录下的)名称
  ```

  提交全部文件（两种方式）

  ```git
  git add .
  git add --all
  ```

* 提交到本地区

  * 一句话备注提交内容 -m是表示一句话概括说明（注意提交规范）

  ```git
  git commit -m "提交说明，如第一次提交等等"
  ```

  * commit注释详细说明

    ```git
    git commit
    ```

    * 输入会进入进入vim编辑界面

      ![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\Git commit.png)

    * 输入 c 进入编辑状态

    * 然后输入信息  注意输入的规范

      ![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\Git 输入.png)

    * 按 Esc  退出编辑状态 （光标不能到处移动 不能随便输入）

    * 完成编辑：直接按下两个 ZZ（刚退出时光标不能移动） 或者在最底下输入   :wq / :wq!(分号不能漏)

## 二、日志查看及应用

* 详细显示最近到最远的日志

  ```git
  git log
  ```

  ![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\git log.png)

  * 下一页：空格 
  * 上一页：b
  * 最后一页的页尾有（END）
  * 退出：q

* 简单显示（常用）

  会有指针(HEAD)走到当前需要多少步

  ```git
  git reflog
  ```

  ![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\gitreflog.png)

* 前进或者后退历史版本（撤销删除）== 本地库移动

  索引：复制粘贴前面reflog显示的开头那串数字

  * 本地库移动的同时，重置工作区和暂存区

  ```git
  git reset --hard 索引
  git reset --hard 529a11
  ```

  * 重置暂存区但工作区不动

  ```git
  git reset  --mixed 索引
  ```

  * 暂存区和工作区都不动

  ```git
  git reset --soft 索引
  ```

* 删除和找回（常用上一种方法）

  删除工作区中的数据

  ```git
  git rm
  ```

  然后同步操作到暂存区跟工作区 add commit 

* 比对工作区和缓存区中的文件的不一致

  * 比较某一分支与工作区的不一样

    ```git
    git diff 分支名
    ```

  * 比较某一文件

    ```git
    git diff [file name]
    ```

  * 比较所有文件

    ```git
    git diff
    ```

  * 比较某一版本 比较暂存区与工作区的差异

    ```git
    git diff[历史版本] [文件名]
    ```

## 三、分支管理

### 创建分支

* 创建后切换

  ```git
  git checkout 分支名
  git checkout dev
  ```

* 创建同时切换

  ```git
  git switch -c dev
  ```

![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\git 分支切换.png)

### 查看分支

查看分支的最新版本      *表示当前在哪个分支上

```git
git branch -v
```

![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\git查看分支.png)

### 合并分支

* 先进入主分支（一般是把其他分支的合并到主分支上）

* 输入合并命令（两种形式都可以）

  ```git
  git merge 要合并的分支名
  git merge --no--off -m "备注" 要合并的分支名
  ```


### 解决冲突

* 只要合并的地方同时修改的是不一样的就会冲突，如果是直接修改了的话就直接合并了

* 解决方法：打开文件在冲突的地方保留想要的，删除不要的

* 然后add commit就可以了

  ![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\测试文件冲突.png)

  ```java
  ImageView imageView;//测试devtest上的冲突
  TextView textView1;//测试devtest上的冲突
  ```


### 删除分支

合并分支后或者不需要了就可以删除分支

```git
git branch -D 分支名
```

## 四、远端操作

### 推送到远端

* 创建本地库 在gitee或者github上创建远程库

* 为远程库创建别名(origin 为常用默认的远程库别名)

  在远端库界面有 clone&downlone  复制地址

  ```git
  git remote add 别名 远程库地址
  git remote add origin http...
  ```

* 推送

  ```git
  git push 远程库地址 远程库的分支
  git push origin master
  ```

  解决冲突

  推送出现错误

  ```git
   suzy@LAPTOP-QV66ISNV MINGW64 /e/大一下工作室/QG/homework/linkList (master)
  $  git push qg master
  To https://gitee.com/suzymo/qgcamp.git
   ! [rejected]        master -> master (fetch first)
  error: failed to push some refs to 'https://gitee.com/suzymo/qgcamp.git'
  hint: Updates were rejected because the remote contains work that you do
  hint: not have locally. This is usually caused by another repository pushing
  hint: to the same ref. You may want to first integrate the remote changes
  hint: (e.g., 'git pull ...') before pushing again.
  hint: See the 'Note about fast-forwards' in 'git push --help' for details
  ```

  ![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\推送错误.png)

解决方案：强制推送（会覆盖远程库中原有的不同文件的）

```git
git push -u 远程库地址 分支 -f
git push -u qg master -f
```

### 从远端拉取

* 直接下载

```git
git clone 远程库地址
```

done表示下载完成

* 拉取同步一步到位

```git
git pull 远程库地址 分支名
git pull origin master
```

* 拉取同步分布操作

  * 拉取远程库（只是下载到本地了，但是工作区中的文件还没更新）

  ```git
  git fetch otigin master
  ```

  * 会自动下载到 origin/master  分支中，检查改了什么，确认一下

    直接查看全部

  ```git
  git diff origin/master
  ```

  ​		先切换分支 看看有啥  然后分别看文件的内容

  ![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\抓取后查看.png)

  ```git
  git checkout origin/master
  ll
  cat 文件名
  ```

  * 合并分支（先切换回master分支）

  ```git
  git merge origin/master
  ```


## 五、标签管理

1. **git tag 标签名**     新建标签 默认HEAD 也可指定一个commit id
2. **git tag -a 标签名 -m “标签信息”**     可指定标签信息
3. **git tag**     查看所有标签
4. **git push 远端地址 标签名**      可以推送标签
5. **git push 远端地址 --tags **     可以推送所有未推送的标签
6. **git tag -d 标签名 **    删除本地标签
7. **git push 远端地址 :refs/tags/标签名 **    删除远端标签

## 六、团队协作

### 邀请项目成员

* 进入设置

  ![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\gutHub 邀请用户.png)

* 找到邀请入口 点击 invite a collabortor  然后输入查找用户

  ![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\邀请用户2.png)

* 复制邀请链接

  ![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\邀请用户3.png)

​	

* 把邀请链接发送给团队成员（点开链接同意加入即可）

### 推送冲突失败

* 先拉取远程库 git pull origin
* 人为解决冲突
* 再推送到远端

### 跨团队合作（或借鉴别人的项目）

* 找到合作团队项目的链接

* 然后在自己的主页进行搜索

* 在项目页面进行 fork 操作（点击右上角的 fork）

* 自己团队会新增这个项目且有新的地址

* 克隆到本地（git clone 新地址）

* 自己可以进行修改

* 提交到本团队的仓库

* 发送请求推送到合作团队的仓库（pull requests）

  * 点击顶部第二个 Pull requests 然后点右上角绿色按钮 New Pull Requests

    ![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\跨团队合作.png)

  * 进入后写好题目和说明   点右下角Create pull request

  * 等待合作团队审核（可以在pull request那发回信和相互交流）和进行合并等待操作

## 七、实际开发中的SSH免密登录

windows系统验证账号一次即可，若不是windows系统，采用SSH免密登录

* 在任意位置打开git 窗口

* 进入用户的主目录

  ```git
  cd ~
  ```

* 执行命令生成一个.ssh目录

  ```git
  ssh-keygen -t rsa -C github账号绑定的邮箱
  ```

  ![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\ssh1.png)

* 然后按三次回车（相当于选择默认的方式）

* 在C盘->用户->suzy(用户名)->.ssh->id_rsa.pub

  打开文件复制里面的内容

* 打开github的setting里面左边目录栏的 SSH and GPG keys

* 再点右边的 new SSH key

  * 第一个框的名字随便起
  * 第二个框的用粘贴前面复制的内容
  * 点击下方的添加绿色按钮

* 然后就可以用了（远程库的地址选择SSH形式的 其他使用方式不变）