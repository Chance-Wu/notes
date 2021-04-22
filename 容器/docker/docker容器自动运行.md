>创建容器时没有添加参数`--restart=always`，当Docker重启时，容器未能自动启动。
>
>解决办法：docker命令修改
>
>```shell
>$ sudo docker update redis --restart=always
>$ sudo docker update mysql --restart=always
>```



