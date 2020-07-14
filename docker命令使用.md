
# docker 命令使用

## docker 日志查询

```sh
$ docker logs <容器ID>
# 根据时间查询 
$ docker logs --since 30m <容器ID>  # 查询30分钟内的日志
$ docker logs -f -t --since="2018-02-08" --tail=100 <容器ID> # 查看指定时间后的日志，只显示最后100行
$ docker logs -t --since="2018-02-08T13:23:37" <容器ID> # 查看某时间之后的日志
$ docker logs -t --since="2018-02-08T13:23:37" --until "2018-02-09T12:23:37" <容器ID> # 查看某时间段日志
```

## 查看docker容器使用的资源

```sh
$ docker stats # 命令会每隔 1 秒钟刷新一次输出的内容
$ $ docker stats --no-stream # 只输出当前的状态
$ docker stats --no-stream registry <容器ID> # 指定容器的系统资源状态
$ docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}" # 格式化输出
```

## 常用容器操作

```sh
$ docker restart <容器ID> # 指定容器的重启
```

## 拷贝操作

```sh
# 将容器96f7f14e99ab的/www目录拷贝到主机的/tmp目录中
$ docker cp  96f7f14e99ab:/www /tmp/

# 主机/www/runoob目录拷贝到容器96f7f14e99ab中，目录重命名为www。
$ docker cp /www/runoob 96f7f14e99ab:/www
```

## 查看容器数据卷信息

```sh
$ docker inspect <容器ID>
# 数据卷 信息在 "Mounts" Key 下面
```

## 构建镜像

```sh
# 不使用缓存重建镜像
$ docker build --no--cache .
```