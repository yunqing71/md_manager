前提：jdk1.8,git,maven

1.RocketMQ被阿里贡献给Apache基金会了，所以官网[http://rocketmq.apache.org/](http://rocketmq.apache.org/)

2.下载最新版二进制文件目前是4.6.0 [https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.6.0/rocketmq-all-4.6.0-bin-release.zip](https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.6.0/rocketmq-all-4.6.0-bin-release.zip)
![image.png](https://upload-images.jianshu.io/upload_images/8020666-16f69360282699b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.环境变量配置，变量名ROCKETMQ_HOME
变量值，二进制文件解压后地址

![image.png](https://upload-images.jianshu.io/upload_images/8020666-5336fe76d610a7f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4.注意：RocketMQ NAMESERVER默认分配的jvm参数需要占用较大内存，但是一般来说我们自己用不需要占用这么大内存，测试而已，没那么大的需求
>更改bin目录下的这两个文件的相应部分，直接拖到vscode里即可打开。

![image.png](https://upload-images.jianshu.io/upload_images/8020666-57c0841db7784c6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>改成如下这样256m即可如下：改runbroker.cmd的时候注意给%classpath%加上双引号，不然启动会报错。

![image.png](https://upload-images.jianshu.io/upload_images/8020666-1150ca6386b6b676.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


5.启动nameserver
>进入bin目录，cmd执行

```
start mqnamesrv.cmd
```
>启动成功如下所示：********注意cmd窗口不要关闭*******

![image.png](https://upload-images.jianshu.io/upload_images/8020666-491e14ea143734f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

6.启动broker，bin目录下执行
```
start mqbroker.cmd -n 127.0.0.1:9876 autoCreateTopicEnable=true
```
>启动成功如下：*******注意也不要关闭这个cmd窗口***********

![image.png](https://upload-images.jianshu.io/upload_images/8020666-b4e709c32fa7f5c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

7.RocketMQ插件安装

git克隆下这个项目,或者直接去[https://github.com/apache/rocketmq-externals](https://github.com/apache/rocketmq-externals)下载zip压缩包
```
git clone https://github.com/apache/rocketmq-externals
```
>解压之后进入rocketmq-externals\rocketmq-console\src\main\resources文件夹，打开application.properties进行配置。

![image.png](https://upload-images.jianshu.io/upload_images/8020666-933b0d12edd2f25a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

8.进入\rocketmq-externals\rocketmq-console文件夹，maven编译打包,maven安装配置不再赘述
>注意要进cmd,不要使用win10的powerShell

```
mvn clean package -Dmaven.test.skip=true
```
9.启动插件

>编译成功之后cmd进入target文件夹，执行target文件夹下的jar文件

```
 java -jar rocketmq-console-ng-1.0.1.jar
```

10.打开浏览器，访问127.0.0.1:8080

![image.png](https://upload-images.jianshu.io/upload_images/8020666-bd41f5ed1c7123ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


更多精彩文章请关注本人公众号
更多精彩文章请关注本人公众号
更多精彩文章请关注本人公众号
![yunqing.jpg](https://upload-images.jianshu.io/upload_images/8020666-7e97ce57ac7a2143.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)