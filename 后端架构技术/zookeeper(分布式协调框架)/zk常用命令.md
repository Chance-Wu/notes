#### 启动命令

---

```shell
./zkServer.sh start 		//启动zookeeper服务端

./zkServer.sh status 		//查看zookeeper状态（model显示状态）	

./zkCli.sh 					//启动zookeeper客户端

quit 						//退出客户端

./zkServer.sh stop			//退出zookeeper服务端
```



#### 客户端命令

---

```shell
# 客户端
help   						#显示所有操作指令

ls /						#查看当前znode中所包含的内容

ls2 /						#查看当前节点的详细数据
```



#### 创建节点

---

```shell
create /sanguo	#在此client下创建名称为sanguo的节点，后面可指定节点内容  例子：create /sanguo "liubei"

create /sanguo/shuguo "liubei"	#创建多级目录
```

```shell
get /sanguo/shuguo	#获取创建节点内的内容
```

```shell
set /sanguo/shuguo "diaochan"	#修改节点内的值zhouyu->diaochan
```



#### 设置监听

---

```shell
get -w /sanguo/shuguo	#获得节点内容，并且设置监听（设置一次监听一次）(当此节点下内容变化有输出)
```



#### 创建临时节点（client重启后消失）

---

```shell
create -e /sanguo/weiguo "caocao"	#-e创建短暂的节点(client重启后节点消失)	
```

```shell
ls -w /sanguo	#监听子节点的变化
```



#### 创建有序节点

---

```shell
create -s /sanguo/weiguo "caocao"	#-s创建带有序号的节点（序号从总共节点数开始往后排）
```



#### 删除节点

---

```shell
delete /sanguo/wuguo2	#删除/sanguo下的wuguo2

deleteall /sanguo/wuguo	#递归删除
```



