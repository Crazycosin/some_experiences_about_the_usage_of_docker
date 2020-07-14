# docker安装部署spark集群

## 系统环境，软件配置信息

```txt
主机系统：windows10
虚拟机系统：linux Ubuntu-20.04LTS
docker：19.03.11
```

## 懒人方法安装

```sh
# 为了方便管理各docker镜像，建议创建一个docker文件夹，在此下面根据不同镜像创建不同文件夹
$ cd /root
$ mkdir docker
$ cd docker
$ mkdir py-spark
$ cd py-spark

# 编写Dockerfile
# 这里即将体现懒的地方了。
# 糅合了基于jdk8的spark的dockerfile与基于alpine3.12的python3.6.10的dockerfile
# 实际构建jdk8的基础镜像是alpine3.9，但是并不影响与python3的构建

# 构建jdk8的spark教程参考：https://juejin.im/post/5ca997176fb9a05e59588f13#heading-26
# 这篇文章讲的非常简洁清晰，以及创建master， worker的脚本也一一俱全基本测试通过，
# 唯一不足是一些软件下载软速度是在感人，做了一些调整

# 构建python3的dockerfile 参考： https://github.com/docker-library/python/blob/ece154e2849e78c383419d0be591cfd332a471d3/3.6/alpine3.12/Dockerfile

# 这种思路其实可以复制的，就是你不怎么会docker，但是你想找一个混合两种环境的docker，不如先
# 找一个成熟的方案，然后在其dockerfile中构建另一个环境，比如这里的python3，但是一定要都是基于某一个最基础的
# 版本，比如这里的alpine。

# python3 的dockerfile 同样为了避免感人的下载源，也进行了修改操作

最后得到该目录下的Dockerfile，缝合怪的胜利！！！

```

## 测试

```txt
这里测试pyshark
使用spark-submit 执行
使用的案例来自：https://tellyouwhat.cn/p/docker-build-spark-wordcount-app/
```
