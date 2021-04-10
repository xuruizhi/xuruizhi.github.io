
## adb 启动失败，报端口被占用

```
C:\Users\xxx>adb devices
* daemon not running; starting now at tcp:5037
could not read ok from ADB Server
* failed to start daemon
adb.exe: failed to check server version: cannot connect to daemon
```

### 解决方法1：查看占用该端口的程序，kill

```
C:\Users\xxx>netstat -aon | findstr "5037"
  TCP    127.0.0.1:5037         0.0.0.0:0              LISTENING       4448

C:\Users\xxx>tasklist | findstr 4448
svchost.exe                   4448 Services                   0      6,416 K

C:\Users\xxx>TASKKILL /F /PID 4448
错误: 无法终止 PID 为 4448 的进程。
原因: 拒绝访问。

```

### 解决方法2：自定义 adb 使用的端口号

设置环境变量： `ANDROID_ADB_SERVER_PORT`。

