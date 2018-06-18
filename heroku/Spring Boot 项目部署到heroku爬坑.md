## Spring Boot 项目部署到heroku爬坑

​	背景：最近小组进行一个环境比较恶劣的项目，由于没有真实的测试环境，决定上云，最终选择国外的heroku，折腾半天，其中有一些坑在这里记录下来，方便网友及个人。

---

### 1.账号注册

​	heroku官网：https://www.heroku.com

​	heroku免费注册账号，heroku提供的功能已经可以满足大部分个人需求，有特殊需求的用户就需要进行付费了，比如heroku的数据库的免费空间只有5M，且项目在30分钟内无人访问就会休眠，下面是heroku对于休眠的说明：

```tex
By default, your app is deployed on a free dyno. Free dynos will sleep after a half hour of inactivity (if they don’t receive any traffic). This causes a delay of a few seconds for the first request upon waking. Subsequent requests will perform normally. Free dynos also consume from a monthly, account-level quota of free dyno hours - as long as the quota is not exhausted, all free apps can continue to run.
To avoid dyno sleeping, you can upgrade to a hobby or professional dyno type as described in the Dyno Types article. For example, if you migrate your app to a professional dyno, you can easily scale it by running a command telling Heroku to execute a specific number of dynos, each running your web process type.
```

​	heroku的注册界面：

![image-20180618142314263](https://raw.githubusercontent.com/ai-houzi/notes/master/picture/image-20180618142314263.png)



PS:

* heroku的网站需要翻墙才能访问，并且设置翻墙软件的模式为全局模式。
* heroku貌似不接受中国有限注册（Country可以选择中国区域），个人使用Gmail注册

### 2.安装CLI

​	简单注册完账号以后在官网登陆个人账号，点击Getting Started，选择一样自己需要的语言，然后选择合适自己系统的版本，下载安装CLI，本人为MAC系统。![image-20180618143840969](https://raw.githubusercontent.com/ai-houzi/notes/master/picture/image-20180618143840969.png)

![image-20180618143944050](https://raw.githubusercontent.com/ai-houzi/notes/master/picture/image-20180618143944050.png)

### 3.heroku基本操作

​	官网给了比较详细的操作说明，这里就不一一赘述，大家可以跟着官方教程一步一步操作，这里只说一下个人实践过程中遇到的问题，附送一些官网教程的截图。

![image-20180618144118752](https://raw.githubusercontent.com/ai-houzi/notes/master/picture/image-20180618144118752.png)

![image-20180618144432742](https://raw.githubusercontent.com/ai-houzi/notes/master/picture/image-20180618144432742.png)

​	详细教程请参见heroku官网

### 4.遇到的问题

​	上传项目到heroku时，一般系统会自动帮你打包并运行你的项目，这里我遇到两个问题：

* git的个人分支无法上传

* 项目无法启动

  下面是解决方法：

#### 1.git个人分支无法上传

​	官网上上传项目给了一条指令：

```shell
$ git push heroku master
```

​	然后会得到这样一个运行日志：

```shell
Initializing repository, done.
Counting objects: 110, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (87/87), done.
Writing objects: 100% (110/110), 212.71 KiB | 0 bytes/s, done.
Total 110 (delta 30), reused 0 (delta 0)

-----> Java app detected
-----> Installing OpenJDK 1.8... done
-----> Installing Maven 3.3.3... done
-----> Executing: mvn -B -DskipTests=true clean install
       [INFO] Scanning for projects...
...
       [INFO] ------------------------------------------------------------------------
       [INFO] BUILD SUCCESS
       [INFO] ------------------------------------------------------------------------
       [INFO] Total time: 11.417s
       [INFO] Finished at: Thu Sep 11 17:16:38 UTC 2014
       [INFO] Final Memory: 21M/649M
       [INFO] ------------------------------------------------------------------------
-----> Discovering process types
       Procfile declares types -> web
```

​	但是实际项目中，我是在自己的分支上开发，然后我用git上传自己的分支：

```shell
$ git push heroku XXX
```

​	运行结果：

```shell
Total 0 (delta 0), reused 0 (delta 0)
remote: Pushed to non-master branch, skipping build.
To https://git.heroku.com/certberus.git
   f2c01f2..40aa59d  xxx -> xxx
```

​	这样显然是不对的，最后发现上传分支需要这样输入：

```shell
$ git push heroku XXX:master
```

​	这样你的分支修改的内容就会合并到mater上进行上传，然后运行了。

#### 2.项目无法启动

​	通常maven项目在打包时，会被打成war包或者jar包，熟悉Spring boot的童鞋应该了解Spring boot的运行命令，其实heroku运行项目也非常简单。

​	首先说一下正常的一个文件的Spring boot部署到heroku，需要在根目录添加一个Procfile文件，告诉heroku你要打包哪个文件，文件内容如下：

```shell
web java -Dserver.port=$PORT $JAVA_OPTS -jar target/*.jar
```

​	但是本人的项目为多个子项目打包，启动类在子项目中，这样如何来让heroku启动呢，自己不停的修改Procfile中的文件路径仍然不起作用，后来发现heroku中有一个很爽的命令，如下：

```shell
$ heroku run bash
```

​	这样就相当于远程登录一台Linux服务器啦，我们可以使用Linux命令查看自己部署在heroku上的项目的目录结构啦，找到需要运行的jar包，将其在云端的路径修改到Procfile文件中，再次上传项目，就会发现项目跑起来了。