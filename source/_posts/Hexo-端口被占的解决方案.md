---
title: Hexo 端口被占的解决方案
date: 2018-01-18 16:21:20
tags: [hexo, amber, port, 4000, lsof, pid, blog, github]
---
# Hexo server 命令提示端口被占


```
➜ hexo server

# FATAL Port 4000 has been used. Try other port instead.
# FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: listen EADDRINUSE 0.0.0.0:4000
    at Object.exports._errnoException (util.js:1016:11)
    at exports._exceptionWithHostPort (util.js:1039:20)
    at Server.setupListenHandle [as _listen2] (net.js:1307:14)
    at listenInCluster (net.js:1355:12)
    at doListen (net.js:1481:7)
    at _combinedTickCallback (internal/process/next_tick.js:105:11)
    at process._tickCallback (internal/process/next_tick.js:161:9)
```

## 解决方案一

查找占用端口的进程（以上面4000为例）


```
➜  lsof -i:4000

COMMAND   PID  USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
node    49336 amber   32u  IPv4 0x3651d91f8ada5fa9      0t0  TCP *:terabase (LISTEN)
```

根据上面的返回内容，可以看到占用端口4000的进程pid 是49336，接下来就是 kill 进程。


```
➜  kill -9 `lsof -i TCP:40001 | awk '/LISTEN/{print $2}'`
```

## 解决方案二

修改端口
       
```
➜   hexo server -p 40001
```

