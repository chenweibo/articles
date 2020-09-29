### jenkins安装

#### 官方安装方式

##### 添加yum

```shell
 sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
 sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
```



##### 安装

```shell
  yum install jenkins
```



> 这种方式下载方式很慢，建议使用下述方法



#### 推荐方式

从以下下载所需rpm文件，上传到指定位置

[Jenkins华为镜像源](https://mirrors.huaweicloud.com/jenkins/redhat-stable/)
[Jenkins清华大学镜像源](https://mirror.tuna.tsinghua.edu.cn/jenkins/redhat-stable/)
[Jenkins开源软件镜像源](https://mirrors.cnnic.cn/jenkins/redhat-stable/)
[Jenkins北京外国语大学镜像源](https://mirrors.bfsu.edu.cn/jenkins/redhat-stable/)



##### 安装

```shell
sudo yum -y localinstall jenkins-2.235.5-1.1.noarch.rpm
```



##### 查询安装所在位置

```shell
rpm -qa | grep jenkins

rpm -ql jenkins-2.235.5-1.1.noarch //文件名为上面查询出来的名字
```



##### 修改端口

> 打开配置文件 /etc/sysconfig/jenkins ，找到配置项: JENKINS_PORT 修改为自定义端口



```shell
JENKINS_PORT="6080"
```



##### 启动服务

```shell
systemctl start jenkins
```

> 1.通过ip:6080访问jenkins
>
> 2.注意需要放行6080端口