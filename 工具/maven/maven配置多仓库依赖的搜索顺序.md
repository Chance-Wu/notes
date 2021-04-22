> maven多仓库查找依赖的顺序大致如下：
>
> 1. 在==本地仓库==中寻找，如果没有则进入下一步。
> 2. 在==全局配置的私服仓库==（settings.xml中配置的并有激活）中寻找，如果没有则进入下一步。
> 3. 在==项目自身配置的私服仓库==（pom.xml）中寻找，如果没有则进入下一步。
> 4. 在==中央仓库==中寻找，如果没有则终止寻找。

<img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gjz85ncn2lj30zz0nn0up.jpg" style="zoom:80%">

> 注：
>
> 1. 在找寻的过程中，如果发现该仓库有镜像设置，则用镜像的地址代替，例如现在进行到要在respository A仓库中查找某个依赖，但仓库配置了mirror，则会转到从A的mirror中查找该依赖，不会在从A中查找。
> 2. settings.xml中配置的profile（激活的）下的respository优先级高于项目中pom文件配置的respository。
> 3. ==如果仓库的id设置成“central”，则该仓库会覆盖maven默认的中央仓库配置==。