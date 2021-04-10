重点:
- 区分 项目级 build.gradle 和 模块级 build.gradle 的不同作用；
- 区分 Android Gradle 插件版本 与 Gradle 版本的[关系](https://developer.android.google.cn/studio/releases/gradle-plugin.html#updating-gradle)；

http://www.imooc.com/wiki/gradlebase

```
Gradle is an open-source build automation tool focused on flexibility and performance. Gradle build scripts are written using a Groovy or Kotlin DSL.
Gradle 是专注于灵活性和性能的开源构建自动化工具。Gradle 构建脚本是使用 Groovy 或 Kotlin DSL 编写的。
```
## Android项目根目录下的 build.gradle

项目的顶级构建文件，配置 Gradle 版本和 Maven 依赖，您可以在其中添加所有 `子项目/模块` 共有的配置选项。

- buildscript 闭包：配置 App `构建`所需要的的依赖，分别是对应 Maven 仓库和构建工具 Gradle 的版本
    - repositories 闭包：配置远程的 Maven 仓库地址，国外 Maven 访问太慢，我们可以加入国内阿里云的 Maven 库
    - dependencies 闭包：配置项目构建所使用的 `Android Gradle 插件版本`，而不是项目构建工具 Gradle 的版本（[二者有配套关系](https://developer.android.google.cn/studio/releases/gradle-plugin.html#updating-gradle)）
- allprojects 闭包：配置 App `运行`所需要的的依赖
    - repositories 闭包：配置远程的 Maven 仓库地址，国外 Maven 访问太慢，我们可以加入国内阿里云的 Maven 库

```
buildscript {
    repositories {
        maven { url "http://maven.aliyun.com/nexus/content/groups/public/" }
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.5.1'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
allprojects {
    repositories {
        maven { url "http://maven.aliyun.com/nexus/content/groups/public/" }
        google()
        jcenter()
    }
}
```

## app 目录下的 build.gralde文件

app 打包和签名配置，及模块的依赖。

- apply plugin 语句
    - com.android.application：代表这个模块是一个 `Android application` ，可以独立运行，项目构建后生成 apk 文件。
    - com.android.library：代表这个模块是一个 `Android library` ，不能够独立运行，必须依附于 application 才能运行，项目构建后生成 aar 文件。
- android 闭包，配置打包的一些信息，包括`包名、版本号、版本名称；打包类型、混淆配置、签名信息`等
- dependencies 闭包：配置项目运行所需要的依赖。可以配置需要引用的`本地` libs 目录下的第三方的 jar 包或是 aar 包，还可以是`maven 库`里面的第三方的开源库。

```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 30
    buildToolsVersion rootProject.buildToolsVersion
    defaultConfig {
        applicationId "com.example.android.testing.uiautomator.BasicSample" // APK 包名
        minSdkVersion 18
        targetSdkVersion 30
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner" // 单元测试
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        debug {
            minifyEnabled false     // 是否混淆
            shrinkResources false       // 是否启用未使用资源的收缩
            signingConfig signingConfigs.release    // 设置签名信息
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'  // 指定混淆的规则文件
            zipAlignEnabled true    // 是否对 APK 包执行 ZIP 对齐优化，减小 zip 体积
            renderscriptOptimLevel 5
        }
    }

    // 签名
    signingConfigs {
        // 你自己的 keystore 信息
        release {
            storeFile file(rootProject.ext.store_file)
            storePassword rootProject.ext.store_password
            keyAlias rootProject.ext.key_alias
            keyPassword rootProject.ext.key_password
            v1SigningEnabled true
            v2SigningEnabled true
        }
    }

}

dependencies {
    implementation 'com.google.guava:guava:' + rootProject.guavaVersion
    // Testing-only dependencies
    androidTestImplementation 'androidx.test:core:' + rootProject.coreVersion;
    androidTestImplementation 'androidx.test.ext:junit:' + rootProject.extJUnitVersion;
    androidTestImplementation 'androidx.test:runner:' + rootProject.runnerVersion;
    // UiAutomator Testing
    androidTestImplementation 'androidx.test.uiautomator:uiautomator:' + rootProject.uiAutomatorVersion;
    androidTestImplementation 'org.hamcrest:hamcrest-integration:1.3'
}
```

## setting.gradle 文件
添加本项目参与编译的所有模块。一个 Android 工程可以包含多个模块，每个模块会编译出一个 apk 或者 aar。

```
include ':app'
// 如果，我们的项目中有 person/common/home 等模块时，可以这样引入
include ':app', ':person', ':common', ':home'
```

## gradle 文件夹
配置 gradel-wrapper。
官方建议任何 Gradle 构建方法在 Gradle Wrapper 帮助下运行。
```
Gradle Wrapper 它是一个脚本，调用了已经声明的 Gradle 版本，并且我们编译时需要事先下载它。所以，开发者能够快速的启动并且运行 Gradle 项目，不用再手动安装，从而节省了时间成本。
```

gradle-wrapper.properties 中指定了 Gradle 的版本号，和 Gradle 运行时的行为属性文件。
```
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-6.5.1-all.zip
```

### 如何切换 Gradle 版本

修改 `distributionUrl` 中 gradle 版本号，然后在 Android Studio 中 Sync Project 即可。

----

## Maven
[maven使用指南](https://www.ibofine.com/mavenbook/index.html)

• maven是一个项目构建工具，能帮我们自动化构建过程，从清理、编译、测试到生成报告，再到打包和部署
• maven不仅是构建工具，还是一个依赖管理和项目信息管理工具，还提供了中央仓库，可以帮我们自动下载依赖包。