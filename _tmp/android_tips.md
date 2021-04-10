
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

1. 需要设置自动启动(test.apk)

04-09 17:25:55.987 I/ConsoleReporter: [4/4 arm64-v8a app 127.0.0.1:18888] com.example.demo.ExampleInstrumentedTest#useAppContext fail: java.lang.AssertionError: Activity never becomes requested state "[CREATED, DESTROYED, STARTED, RESUMED]" (last lifecycle transition = "PRE_ON_CREATE")

2. 不能用单用例调试生成的apk安装，会报 INSTALL_FAILED_TEST_ONLY


2. AndroidJUnitRunner
		架构
		a. Espresso / UI Automator
		b. Android Test Orchestrator		adb shell am instrument
		应用上下文


AndroidX Test -> AndroidJUnitRunner
							-> UI Automator
							-> Espresso

    dependencies {
        androidTestImplementation 'androidx.test:runner:1.1.0'
        androidTestImplementation 'androidx.test:rules:1.1.0'
        // Optional -- Hamcrest library
        androidTestImplementation 'org.hamcrest:hamcrest-library:1.3'
        // Optional -- UI testing with Espresso
        androidTestImplementation 'androidx.test.espresso:espresso-core:3.1.0'
        // Optional -- UI testing with UI Automator
        androidTestImplementation 'androidx.test.uiautomator:uiautomator:2.2.0'
    }




adb shell am instrument -w -m -e numShards 3 -e debug false -e class 'com.example.android.testing.androidjunitrunnersample.CalculatorInstrumentationTest' com.example.android.testing.androidjunitrunnersample.test/androidx.test.runner.AndroidJUnitRunner




  instrument [-r] [-e <NAME> <VALUE>] [-p <FILE>] [-w]
          [--user <USER_ID> | current] [--no-hidden-api-checks]
          [--no-isolated-storage]
          [--no-window-animation] [--abi <ABI>] <COMPONENT>
      Start an Instrumentation.  Typically this target <COMPONENT> is in the
      form <TEST_PACKAGE>/<RUNNER_CLASS> or only <TEST_PACKAGE> if there
      is only one instrumentation.  Options are:
      -r: print raw results (otherwise decode REPORT_KEY_STREAMRESULT).  Use with
          [-e perf true] to generate raw output for performance measurements.
      -e <NAME> <VALUE>: set argument <NAME> to <VALUE>.  For test runners a
          common form is [-e <testrunner_flag> <value>[,<value>...]].
      -p <FILE>: write profiling data to <FILE>
      -m: Write output as protobuf to stdout (machine readable)
      -f <Optional PATH/TO/FILE>: Write output as protobuf to a file (machine
          readable). If path is not specified, default directory and file name will
          be used: /sdcard/instrument-logs/log-yyyyMMdd-hhmmss-SSS.instrumentation_data_proto
      -w: wait for instrumentation to finish before returning.  Required for
          test runners.
      --user <USER_ID> | current: Specify user instrumentation runs in;
          current user if not specified.
      --no-hidden-api-checks: disable restrictions on use of hidden API.
      --no-isolated-storage: don't use isolated storage sandbox and
          mount full external storage
      --no-window-animation: turn off window animations while running.
      --abi <ABI>: Launch the instrumented process with the selected ABI.
          This assumes that the process supports the selected ABI.
