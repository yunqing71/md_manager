**关注我，精彩文章第一时间推送给你**
![公众号.jpg](https://upload-images.jianshu.io/upload_images/8020666-bca6c792f9a1488a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#docker官方centos安装教程：
[https://docs.docker.com/install/linux/docker-ce/centos/](https://docs.docker.com/install/linux/docker-ce/centos/)
- 在新主机上首次安装Docker Engine-Community之前，需要设置Docker存储库。之后，您可以从存储库安装和更新Docker。
- 设置存储库
1. 安装所需的软件包。yum-utils提供了yum-config-manager 效用，并device-mapper-persistent-data和lvm2由需要 devicemapper存储驱动程序。
```
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```
2.使用以下命令来设置稳定的存储库。
```
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
3.安装docker
```
sudo yum install docker-ce -y
```
4.启动docker
```
sudo systemctl start docker
```
5.通过运行hello-world 映像来验证是否正确安装了Docker
```
sudo docker run hello-world
```
- 此命令下载测试图像并在容器中运行。容器运行时，它会打印参考消息并退出
![image.png](https://upload-images.jianshu.io/upload_images/8020666-bbe7f696019b66e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#下面就开始搭建gitlab仓库
1.拉取gitlab镜像
```
docker pull gitlab/gitlab-ce
```
2.下载速度太慢，设置国内的阿里镜像加速
参考地址 [https://blog.csdn.net/Funny_Ma/article/details/96478395](https://blog.csdn.net/Funny_Ma/article/details/96478395)


3.启动gitlab
```
sudo docker run --detach \
  --hostname gitlab.example.com \
  --publish 10443:443 --publish 1080:80 --publish 1022:22 \
  --name gitlab \
  --restart always \
  --volume /usr/local/docker/gitlab/config:/etc/gitlab \
  --volume /usr/local/docker/gitlab/logs:/var/log/gitlab \
  --volume /usr/local/docker/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```
- 解释一下上面的一条长命令
>--hostname gitlab.example.com \   # 设置主机名或域名
 --publish 10443:443 --publish 1080:80 --publish 1022:22 \ #本地端口的映射
--name gitlab \     # gitlab-ce 的镜像运行成功的容器命名为gitlab
--restart always \  # 设置重启方式，always 代表一直开启，即开机自启
--volume 分别将 gitlab 的配置文件、日志文件、数据文件目录映射到 /usr/local/docker/gitlab的相应目录中

4.通过命令查看所有容器
```
docker ps -a
```
![image.png](https://upload-images.jianshu.io/upload_images/8020666-d7c84f5d0e481736.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
5.因为我们把gitlab映射到1080端口，所以防火墙开启1080端口
```
# 开启1080端口
firewall-cmd --zone=public --add-port=1080/tcp --permanent 
# 重启防火墙才能生效
systemctl restart firewalld
# 查看已经开放的端口
firewall-cmd --list-ports
```
5.访问http://192.168.8.128:1080
- 注意改成你服务器的ip
- 第一次访问让你设置密码，默认账户是root

![image.png](https://upload-images.jianshu.io/upload_images/8020666-75f437c3634c2c0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 设置好密码，root账户名登录即可
![image.png](https://upload-images.jianshu.io/upload_images/8020666-24887aad8f0517dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/8020666-264314c6e1e61c9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 接下来怎么使用，就不是本文要讲的范围了哦

6. 新建一个项目，clone的时候发现地址http://gitlab.example.com/root/test.git
![image.png](https://upload-images.jianshu.io/upload_images/8020666-8b1d9647820aa734.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>上面设置的时候设置主机名或者域名的时候设置了gitlab.example.com,这时候怎么解决？

- 去上面那条长命令找到配置文件地址，我的是这个
```
cd /usr/local/docker/gitlab/config

```
- vim打开进入这个文件
![image.png](https://upload-images.jianshu.io/upload_images/8020666-8f7bd9c254ec4c63.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 搜索这行# external_url 'GENERATED_EXTERNAL_URL'
- 去掉注释并改成你的ip或域名，保存退出。
![image.png](https://upload-images.jianshu.io/upload_images/8020666-5344161121e62b92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 使配置生效
```
docker exec gitlab gitlab-ctl reconfigure
```
- 重启gitlab
```
docker restart gitlab
```
![image.png](https://upload-images.jianshu.io/upload_images/8020666-ae1754382920d262.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 启动gitlab较慢，遇到502多等一会。
- 启动成功，看到clone的url已经修改成了ip
![image.png](https://upload-images.jianshu.io/upload_images/8020666-f779d9f1cbe0a03c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
