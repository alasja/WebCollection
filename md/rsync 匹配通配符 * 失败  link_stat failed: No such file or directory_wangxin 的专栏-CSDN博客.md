\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[blog.csdn.net\](https://blog.csdn.net/wangxin6722513/article/details/43489131) 

rsync -avP /home/map/mongodb2.4.6/data/road140403\* map@hz12:/home/map/users/wangxin/script/tmp  

上面的命令执行的时候不会报任何错误，并正常的同步数据，此时会弹出交互界面，并提示要输入

hz12服务器的密码，但如果这种操作放在脚本里就要expect来支持，如果放在expect里面的话会报

如下错误：

rsync: **link\_stat** "/home/map/mongodb/data/road140403\*" **failed**: No such file or directory (2)

具体情况如下：

![](https://img-blog.csdn.net/20150204175708824?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2FuZ3hpbjY3MjI1MTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

刚开始的时候我以为是rsync无法匹配通配符\*才导致的。

后来发现不是该问题，是由于expect里面无法匹配 \* 才导致的。

解决办法：

在spawn 后面加上 bash -c "command"

这样expect就认出了通配符\*。

**脚本**如下：  

```
#!/bin/bash
function func_expect {
ExpEnv=`which expect`
$ExpEnv -c "
set timeout -1;
spawn bash -c \"$1\";
expect {
\"(yes/no)?\" {send \"yes\n\";expect \"assword:\";send \"$2\n\"}
\"assword:\" {send \"$2\n\"}
eof {exit 0;}
}
expect eof"
}
{
/home/map/mongodb2.4.6/bin/mongod --shutdown --dbpath=/home/map/mongodb2.4.6/data
pwd="******"
cmd1="rsync -avP /home/map/mongodb2.4.6/data/road140403* map@hz12:/home/map/users/wangxin/script/tmp"
func_expect "$cmd1" "$pwd"
```

  

PS:为什么在spawn处用\\" $1\\" 而不是"$1" ？

      因为在上面已经有"了，此处用  \\  转义。