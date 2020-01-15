#会用docker之后才发现，这玩意真香哈哈
##首先linux下yum、docker全部配置国内源，为了拉取快速。
- 参考这篇文章
[https://www.jianshu.com/p/c717940f9230](https://www.jianshu.com/p/c717940f9230)


#拉取rocketmq镜像
- 你可以通过docker search rocketmq查询一下可拉取的镜像
![image.png](https://upload-images.jianshu.io/upload_images/8020666-ae69f8fc3028ac35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 额，有点多，反正选stars最多的准没错，我们拉取第一个的最新版latest版
```
docker pull rocketmqinc/rocketmq:latest
```
##安装namesrv
- 执行下面一条长指令启动容器，**注意，长指令不要用我的，修改成你自己的，详解长指令看下面**
```
docker run -d -p 9876:9876 -v /usr/local/docker/rocketmq/data/namesrv/logs:/root/logs -v /usr/local/docker/rocketmq/data/namesrv/store:/root/store --name rmqnamesrv -e "MAX_POSSIBLE_HEAP=100000000" rocketmqinc/rocketmq:latest sh mqnamesrv

```
- 详解
```
-d   # 后台运行
-p   #设置默认端口，这里rocketmq默认9876端口
-v  #设置映射本地目录到容器内的目录，这个注意我都是把本地的/usr/local/docker/rocketmq/**映射到容器内的对应目录的，这个可以改成你本地的linux目录，当然也可以和我一样。我理解的就是MQ的数据和日志什么的不能放在容器中啊，因为容器毕竟占用的空间有限，就映射一下放在本地目录中。
```
##安装 broker 
- 安装完namesrv就是安装broker了。
- 首先需要创建broker.conf配置文件，我的路径是/usr/local/docker/rocketmq/conf/broker.conf
```
/usr/local/docker/rocketmq/conf/broker.conf
```
- 其中填入如下内容，注意最后一项ip改成你的linux的ip
```
brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
brokerIP1 = 192.168.8.128
```
- 用broker.conf配置启动容器
```
docker run -d -p 10911:10911 -p 10909:10909 -v  /usr/local/docker/rocketmq/data/broker/logs:/root/logs -v  /usr/local/docker/rocketmq/data/broker/store:/root/store -v  /usr/local/docker/rocketmq/conf/broker.conf:/opt/rocketmq-latest/conf/broker.conf --name rmqbroker --link rmqnamesrv:namesrv -e "NAMESRV_ADDR=namesrv:9876" -e "MAX_POSSIBLE_HEAP=200000000" rocketmqinc/rocketmq:latest sh mqbroker -c /opt/rocketmq-latest/conf/broker.conf
```
- 注意的地方还是上面需要映射的目录，还有就是linux要开启相应的防火墙端口

##安装 rocketmq 控制台
- 拉取rocketmq控制台的镜像
```
docker pull pangliang/rocketmq-console-ng
```
- 后台启动rocketmq的控制台镜像，映射到18080端口
```
docker run -d -e "JAVA_OPTS=-Drocketmq.namesrv.addr=192.168.8.128:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false" -p 18080:8080 -t pangliang/rocketmq-console-ng
```
###全部启动这3个容器后可通过下面命令查看容器运行状态
```
docker ps -a
```
![image.png](https://upload-images.jianshu.io/upload_images/8020666-397a311fea048181.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###注意注意：一定要开启防火墙的相应端口
- 通过看上面的配置需要开启9876、10911、10909、18080四个端口号
```
firewall-cmd --zone=public --add-port=9876/tcp --permanent  #开启9876端口
#10911
#10909
#18080
```
- 重新载入防火墙配置
```
firewall-cmd --reload
```
###检验成果的时候到了
- 打开浏览器访问192.168.8.128:18080   注意访问你linux的ip和上面rocketmq控制台映射的18080端口

![image.png](https://upload-images.jianshu.io/upload_images/8020666-bc461e850a2d4278.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/8020666-e05cfc2ebbb18ddc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
