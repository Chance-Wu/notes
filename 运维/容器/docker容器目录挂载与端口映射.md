#### 1. mysql为例

```shell
docker run -p 3306:3306 --name mysql \	#容器的3306端口映射到主机3306
-v /mydata/mysql/log:/var/log/mysql \	#日志挂载到主机/mydata/mysql/log
-v /mydata/mysql/data:/var/lib/mysql \	#mysql运行期间数据挂载到主机
-v /mydata/mysql/conf:/etc/mysql \		#mysql配置挂载到主机
-e MYSQL_ROOT_PASSWORD=root \			#初始化用户名密码
-d mysql:5.7							#作为一个守护进程在后台运行
```

<img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gkizen8p00j316a0jogp7.jpg" style="zoom:60%">























































