# Redis  Linux 混合学习

> 主线为学习Redis ，学习Linux为辅

## 安装

> 以下在linux下进行

 下载

 ```
wget https://download.redis.io/redis-stable.tar.gz
 ```

解压

需要提前准备c语言编译环境

[引用](https://www.runoob.com/linux/linux-comm-tar.html)

```
eg
压缩 a.c文件为test.tar.gz
# tar -czvf test.tar.gz a.c 
解压
# tar -xzvf test.tar.gz 
```

编译

`make`

安装

`make install`

后台启动

1. 修改配置文件使其支持后台启动

   复制到配置文件到etc下    

   cp redis.conf /etc/redis.conf

   修改 daemonize 为yes

```
linux命令：/（内容）   #linux搜索命令
linux命令：wq         #保存退出
```

2. 启动

cd /usr/local/bin

redis-server /etc/redis.conf



3. 查看redis情况

 ```
linux命令：ps [options] [--help]           #Linux ps （英文全拼：process status）命令用于显示当前进程的状态，类似于 windows 的任务管理器。

 eg：查找指定进程格式：
 ps -ef | grep 进程关键字
 ```

4. 用客户端访问：redis-cli

2.2.5.5.多个端口可以：redis-cli -p[端口号]

5. 测试验证

   ping

6. Redis 关闭

   单实例关闭：redis-cli shut也可以进入终端后再关闭

   多实例关闭，指定端口关闭：redis-cli -p[端口号]  shutdown

   