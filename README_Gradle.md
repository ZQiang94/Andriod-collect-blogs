##### 1. [Android Plugin DSL Reference](http://google.github.io/android-gradle-dsl/current/)
##### 2. [Gradle详解](http://www.infoq.com/cn/articles/android-in-depth-gradle)
##### 3. [Android Gradle 语法](http://android.walfud.com/android-gradle-%E7%9C%8B%E8%BF%99%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86/)
##### 4. [给 ANDROID 初学者的 GRADLE 知识普及](http://stormzhang.com/android/2016/07/02/gradle-for-android-beginners/)
##### 5. [ANDROID 开发你需要了解的 GRADLE 配置](http://stormzhang.com/android/2016/07/15/android-gradle-config/)
##### 6. [GRADLE依赖的统一管理](http://stormzhang.com/android/2016/03/13/gradle-config/)
##### 7. [GRADLE自定义你的BUILDCONFIG](http://stormzhang.com/android/2015/01/25/gradle-build-field/)
##### 8. [ANDROID STUDIO系列教程五--GRADLE命令详解与导入第三方包](http://stormzhang.com/devtools/2015/01/05/android-studio-tutorial5/)
##### 9. [ANDROID STUDIO系列教程四--GRADLE基础](http://stormzhang.com/devtools/2014/12/18/android-studio-tutorial4/)
##### 11. [ANDROID GRADLE](http://stormzhang.com/android/2014/02/28/android-gradle/)
##### 12. [ANDROID STUDIO系列教程六--GRADLE多渠道打包](http://stormzhang.com/devtools/2015/01/15/android-studio-tutorial6/)
##### 13. [Gradle打包APK的一些小技巧和productFlavor配置](http://zheteng.me/android/2016/02/29/flavors-with-gradle/)
##### 14. [知道Android 中Gradle 的这些技巧，提升编译构建速度](http://tikitoo.github.io/2016/05/26/android-studio-gradle-build-run-faster/)
##### 15. [ Android应用开发编译框架流程与IDE及Gradle概要](http://blog.csdn.net/yanbober/article/details/49408489)
##### 16. [给 Android 初学者的 Gradle 知识普及](http://mp.weixin.qq.com/s?__biz=MzA4NTQwNDcyMA==&mid=2650661971&idx=1&sn=3fb69537bbc5fbb14d152ba6381c3b83#rd)
##### 17. [Android 开发你需要了解的Gradle配置](http://mp.weixin.qq.com/s?__biz=MzA4NTQwNDcyMA==&mid=2650662016&idx=1&sn=a3c338766b6ea9de654b1a011dcf5b3e#rd)
##### 18. [“替你”总结的Gradle配置](http://www.jianshu.com/p/642641dc7df3)
##### 19. [深入浅出聊聊Gradle三两事](http://crash.163.com/#news/!newsId=21)


#[Gradle配置最佳实践](http://gold.xitu.io/post/582d606767f3560063320b21)
本文会不定期更新，推荐watch下项目。如果喜欢请star，如果觉得有纰漏请提交issue，如果你有更好的点子可以提交pull request。本文意在分享作者在实践中掌握的关于gradle的一些技巧。

本文固定连接：https://github.com/tianzhijiexian/Android-Best-Practices

本文有部分关于加速配置的内容在Android打包提速实践已经有所涉及，如果有想了解打包加速的内容，可以移步去阅读。

需求

随着android的发展，新技术和新概念层出不穷。不同的测试环境、不同的分发渠道、不同的依赖方式，再加上各大厂家“优秀”的插件化方案，这些给我们的开发工作带来了新的需求。我希望可以通过gradle这个令人又爱又恨的东西来解决这些问题。

实现

调整gradle的编译参数

gradle.properties中允许我们进行各种配置：

配置大内存：

org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
守护进程

org.gradle.daemon=true
并行编译

org.gradle.parallel=true
开启缓存：

android.enableBuildCache=true
开启孵化模式：

org.gradle.configureondemand=true
以上的配置需要针对自身进行选择，随意配置大内存可能会出现oom。如果想了解这样配置的原理，请移步官方文档。

写死库的版本

dependencies {
    compile 'com.google.code.gson:gson:2.+' // 不推荐的写法
}
这样的写法可以保证库每次都是最新的，但也带来了不少的问题：

每次build时会向网络进行检查，国内访问仓库速度很慢
库更新后可能会更改库的内部逻辑和带来bug，这样就无法通过git的diff来规避此问题
每个开发者可能会得到不同的最新版本，带来潜在的隐患
推荐写成固定的库版本：

dependencies {
    compile 'com.google.code.gson:gson:2.2.1'
}
即使是jar包和aar，我也期望可以写一个固定的版本号，这样每次升级就可以通过git找到历史记录了，而不是简单的看jar包的hash是否变了。

全局设定编码

allprojects {
    repositories {
        jcenter()
    }

    tasks.withType(JavaCompile){
        options.encoding = "UTF-8"
    }
}
支持groovy

在根目录的build.gradle中：

apply plugin: 'groovy'

allprojects {
    // ...
｝

dependencies {
    compile localGroovy()
}
设置java版本

如果是在某个module中设置，那么就在其build.gradle中配置：

android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_7
        targetCompatibility JavaVersion.VERSION_1_7
    }
}
如果想要做全局配置，那么就在根目录的build.gradle中配置：

allprojects {
    repositories {
        jcenter()
    }
    tasks.withType(JavaCompile) {
        sourceCompatibility = JavaVersion.VERSION_1_7
        targetCompatibility = JavaVersion.VERSION_1_7
    }
}
当我们在使用Gradle Retrolambda Plugin的时候，就会用到上述的配置（未来迁jack的时候也或许会用到）。

将密码等文件统一配置

密码和签名这类的敏感信息可以统一进行存放，不进行硬编码。在gradle.properies中，我们可以随意的定义key-value。
格式：

key value
例子：

STORE_FILE_PATH ../test_key.jks
STORE_PASSWORD test123
KEY_ALIAS kale
KEY_PASSWORD test123
PACKAGE_NAME_SUFFIX .test
TENCENT_AUTHID tencent123456
配置后，你就可以在build.gradle中随意使用了。

signingConfigs {
    release {
        storeFile file(STORE_FILE_PATH)
        storePassword STORE_PASSWORD
        keyAlias KEY_ALIAS
        keyPassword KEY_PASSWORD
    }
}
上述仅仅是应对于密码等信息的存放，其实你可以将这种方式用于插件化（组件化）等场景。

设置本地项目依赖

facebook的react native因为更新速度很快，jcenter的仓库已经无法达到实时的程度了（估计是官方懒得提交），所以我们需要做本地的库依赖。

先将库文件放入一个目录中：


接着配置maven的url为本地地址：

allprojects {
    repositories {
        maven {
            // All of React Native (JS, Obj-C sources, Android binaries) is installed from npm
            url "$rootDir/module_name/libs/android"
        }
    }
}
路径都是可以随意指定的，关键在于$rootDir这个参数。

设置第三方maven仓库

maven仓库的配置很简单，关键在于url这个参数，下面是一个例子：

allprojects {
    repositories {
        maven {
            url 'http://repo.xxxx.net/nexus/'
            name 'maven name'
            credentials {
                username = 'username'
                password = 'password'
            }
        }
    }
}
其中name和credentials是可选项，视具体情况而定。如果你用jitpack的库的话就需要用到上面的知识点了。

allprojects {
    repositories {
        jcenter()
        maven {
            url "https://jitpack.io"
        }
    }
}
删除unaligned apk

每次打包后都会有unaligned的apk文件，这个文件对开发来说没什么意义，所以可以配置一个task来删除它。

dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    // ...
}

android.applicationVariants.all { variant ->
    variant.outputs.each { output ->
        // 删除unaligned apk
        if (output.zipAlign != null) {
            output.zipAlign.doLast {
                output.zipAlign.inputFile.delete()
            }
        }
    }
}
更改生成文件的位置

如果你希望你库生成的aar文件都放在特定的目录，你可以采用下列配置：

android.libraryVariants.all { variant ->
    variant.outputs.each { output ->
        if (output.outputFile != null && output.outputFile.name.endsWith('.aar')) {
            def name = "${rootDir}/demo/libs/library.aar"
            output.outputFile = file(name)
        }
    }
}
apk等文件也可以进行类似的处理（这里再次出现了${rootDir}关键字）。

lint选项开关

lint默认会做严格检查，遇到包错误会终止构建过程。你可以用如下开关关掉这个选项，不过最好是重视下lint的输出，有问题及时修复掉。

android {
    lintOptions {
        disable 'InvalidPackage'
        checkReleaseBuilds false
        // Or, if you prefer, you can continue to check for errors in release builds,
        // but continue the build even when errors are found:
        abortOnError false
    }
}
引用本地aar

有时候我们有部分代码需要多个app共用，在不方便上传仓库的时候，可以做一个本地的aar依赖。

把aar文件放在某目录内，比如就放在某个module的libs目录内
在这个module的build.gradle文件中添加：
repositories {
 flatDir {
     dirs 'libs' //this way we can find the .aar file in libs folder
 }
}
之后在其他项目中添加下面的代码后就引用了该aar
dependencies {
 compile(name:'aar的名字（不用加后缀）', ext:'aar')
}
如果你希望把aar放在项目的根目录中，也可以参考上面的配置方案。在根目录的build.gradle中写上：

allprojects {
   repositories {
      jcenter()
      flatDir {
        dirs 'libs'
      }
   }
}
依赖项目中的module和jar

工程可以依赖自身的module和jar文件，依赖方式如下：

dependencies {
    compile project(':mylibraryModule')
    compile files('libs/sdk-1.1.jar')
}
这种的写法十分常用，语法格式不太好记，但一定要掌握。

根据buildType设置包名

android {
    defaultConfig {
        applicationId "com" // 这里设置了com作为默认包名
    }

    buildTypes {
        release {
            applicationIdSuffix '.kale.gradle' // 设置release时的包名为com.kale.gradle
        }
        debug{
            applicationIdSuffix '.kale.debug' // 设置debug时的包名为com.kale.debug
        }
    }
这对于flavor也是同理：

android {
    productFlavors {
        dev {
            applicationIdSuffix '.kale.dev'
        }
    }
}
这种写法只能改包名后缀，目前没办法完全更改整个包名。

替换AndroidManifest中的占位符

我们在manifest中可以有类似{appName}这样的占位符，在module的build.gradle中可以将其进行赋值。

android{
    defaultConfig{
        manifestPlaceholders = [appName:"@string/app_name"]
    }
}
flavors或buildType也是同理：

debug{
    manifestPlaceholders = [
        appName: "123456",
    ]
}
ShareLoginLib中就大量用到了这个技巧，下面是一个例子：

[代码地址]

<!-- 腾讯的认证activity -->
<activity
    android:name="com.tencent.tauth.AuthActivity"
    android:launchMode="singleTask"
    android:noHistory="true"
    >
    <intent-filter>
        <!-- 这里需要换成:tencent+你的AppId -->
        <data android:scheme="${tencentAuthId}" />
    </intent-filter>
</activity>
我现在希望在build时动态改变tencentAuthId这个的值：

[代码地址]

release {
    minifyEnabled false
    shrinkResources false // 是否去除无效的资源文件
    proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    signingConfig signingConfigs.release
    applicationIdSuffix '.liulishuo.release'
    manifestPlaceholders = [
            // 这里的tencent123456是暂时测试用的appId
            "tencentAuthId": "tencent123456",
    ]
}
定义全局变量

先在project根目录下的build.gradle定义全局变量:

ext {
    minSdkVersion = 16
    targetSdkVersion = 24
}
然后在各module的build.gradle中可以通过rootProject.ext来引用：

android {
    defaultConfig {
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
    }
}
这里添加rootProject是因为这个变量定义在根目录中，如果是在当前文件中定义的话就不用加了（详见定义局部变量一节）。

动态设置额外信息

假如想把当前的编译时间、编译的机器、最新的commit版本添加到apk中，利用gradle该如何实现呢？此需求中有时间这样的动态参数，不能通过静态的配置文件做，动态化方案如下：

android {
    defaultConfig {
        resValue "string", "build_time", buildTime()
        resValue "string", "build_host", hostName()
        resValue "string", "build_revision", revision()
    }
}
def buildTime() {
    return new Date().format("yyyy-MM-dd HH:mm:ss")
}
def hostName() {
    return System.getProperty("user.name") + "@" + InetAddress.localHost.hostName
}
def revision() {
    def code = new ByteArrayOutputStream()
    exec {
        commandLine 'git', 'rev-parse', '--short', 'HEAD'
        standardOutput = code
    }
    return code.toString()
}
上述代码实现了动态添加了3个字符串资源: build_time、build_host、build_revision, 在其他地方可像引用字符串一样使用：

getString(R.string.build_time)  // 输出2015-11-07 17:01
getString(R.string.build_host)  // 输出jay@deepin，这是我的电脑的用户名和PC名
getString(R.string.build_revision) // 输出3dd5823, 这是最后一次commit的sha值
上面讲到的是植入资源文件，我们照样可以在BuildConfig.class中增加自己的静态变量。

defaultConfig {
    applicationId "kale.gradle.demo"
    minSdkVersion 14
    targetSdkVersion 20

    buildConfigField("boolean", "IS_KALE_TEST", "true") // 定义一个bool变量

    resValue "string", "build_time", "2016.11.17" // 上面讲到的植入资源文件
}
在sync后BuildConfig中就有你定义的这个变量了。

public final class BuildConfig {
  public static final boolean DEBUG = Boolean.parseBoolean("true");
  public static final String APPLICATION_ID = "kale.gradle.test";
  public static final String BUILD_TYPE = "debug";
  public static final String FLAVOR = "";
  public static final int VERSION_CODE = 1;
  public static final String VERSION_NAME = "1.0.0";

  // Fields from default config.
  public static final boolean IS_KALE_TEST = true;
}
如果有带引号的string，要记得转义：

 buildConfigField "String", "URL_ENDPOINT", "\"http://your.development.endpoint.com/\""
init.with

如果我们想要新增加一个buildType，又想要新的buildType继承之前配置好的参数，init.with()就很适合你了。

buildTypes {
        release {
            zipAlignEnabled true
            minifyEnabled true
            shrinkResources true // 是否去除无效的资源文件
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.txt'
            signingConfig signingConfigs.release
        }
        rtm.initWith(buildTypes.release) // 继承release的配置
        rtm {}
    }
多个flavor

flavor可以定义不同的产品场景，我们在之前的文章中已经多次讲到了这个属性，下面就是一个在dev的时候提升支持的android最低版本的做法。

productFlavors {
    // 自定义flavor
    dev { 
        minSdkVersion 21
    }
}
flavor的一大优点是可以通过as来动态的改变这个值，不用硬编码：


如果你定义了不同的flavor，可以在目录结构上针对不同的flavor定义不同的文件资源。

productFlavors{
     dev {}
     dev2 {}
     qihu360{}
     yingyongbao{}
 }

定义局部变量

有时候一个库会被引用多次，或者一个库有多个依赖，但这些依赖的版本都是统一的。我们通过ext来定义一些变量，这样在用到的时候就可以统一使用了。

ext {
    leakcanaryVersion = '1.3.1'
    scalpelVersion = "1.1.2" // other param
}
debugCompile "com.squareup.leakcanary:leakcanary-android:$leakcanaryVersion"
releaseCompile "com.squareup.leakcanary:leakcanary-android-no-op:$leakcanaryVersion"
exlude关键字

我们经常会遇到库冲突的问题，这个在多个部门协作的大公司会更常见到。将冲突的库通过exclude来做剔除是一个好方法。

剔除整个组织的库
compile ('com.facebook.fresco:animated-webp:0.13.0') {
 exclude group: 'com.android.support' // 仅仅写组织名称
}
剔除某个库
compile('com.android.support:appcompat-v7:23.2.0') {
    exclude group: 'com.android.support', module: 'support-annotations' // 写全称
    exclude group: 'com.android.support', module: 'support-compat'
    exclude group: 'com.android.support', module: 'support-v4'
    exclude group: 'com.android.support', module: 'support-vector-drawable'
}
聚合依赖多个库

有时候一些库是一并依赖的，剔除也是要一并剔除的，我们可以像下面这样进行统一引入：

compile([
        'com.github.tianzhijiexian:logger:2e5da00f0f',
        'com.jakewharton.timber:timber:4.1.2'
])
这样别的开发者就知道哪些库是有相关性的，在下掉库的时候也比较方便。

剔除task

Gradle每次构建时都执行了许多的task，其中或许有一些task是我们不需要的，可以把它们都屏蔽掉，方法如下：

tasks.whenTaskAdded { task ->
    if (task.name.contains('AndroidTest') || task.name.contains('Test')) {
         task.enabled = false
    }
}
这样我们就会在build时跳过包含AndroidTest和Test关键字的task了。

ps：有时候我们自己也会写一些task或者引入一些gradle插件和task，通过这种方式可以简单的进行选择性的执行（下文会将如何写逻辑判断）。

通过逻辑判断来跳过task

我们上面有提到动态获得字段的技巧，但有些东西是在打包发版的时候用，有些则是在调试时用，我们需要区分不同的场景，定义不同的task。我下面以通过“用git的commit号做版本号”这个需求做例子。

def cmd = 'git rev-list HEAD --first-parent --count'
def gitVersion = cmd.execute().text.trim().toInteger()

android {
  defaultConfig {
    versionCode gitVersion
  }
}
因为上面的操作可能比较慢，或者在debug时没必要，所以我们就做了如下判断：

def gitVersion() {
  if (!System.getenv('CI_BUILD')) { // 不通过CI进行build的时候返回01
    // don't care
    return 1
  }
  def cmd = 'git rev-list HEAD --first-parent --count'
  cmd.execute().text.trim().toInteger()
}

android {
  defaultConfig {
    versionCode gitVersion()
  }
}
这里用到了System.getenv()方法，你可以参考java中System下的getenv()来理解，就是得到当前的环境。

引用全局的配置文件

在根目录中建立一个config.gradle文件：

ext {
    android = [
            compileSdkVersion: 23,
            applicationId    : "com.kale.gradle",
    ]

    dependencies = [
            "support-v4": "com.android.support:appcompat-v7:24.2.1",
    ]
}
然后在根目录的build.gradle中引入apply from: "config.gradle"，即：

// Top-level build file where you can add configuration options common to all sub-projects/modules.
apply from: "config.gradle" // 引入该文件

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.2'
    }
    // ...
}
之后就可以在其余的gradle中读取变量了：

defaultConfig {
    applicationId rootProject.ext.android.applicationId // 引用applicationId
    minSdkVersion 14
    targetSdkVersion 20
}

dependencies {
    compile rootProject.ext.dependencide["support-v7"] // 引用dependencide
}
区分不同环境下的不同依赖

我们除了可以通过buildtype来定义不同的依赖外，我们还可以通过写逻辑判断来做：

dependencies {
    //根据是不同情形进行判断
    if (!needMultidex) {
        provided fileTree(dir: 'libs', include: ['*.jar'])
    } else {
        compile 'com.android.support:multidex:1.0.0'
    }
    // ...
}
动态改变module种类

插件化有可能会要根据环境更改当前module是app还是lib，gradle的出现让其成为了可能。

if (isDebug.toBoolean()) {
    apply plugin: 'com.android.application'
} else {
    apply plugin: 'com.android.library'
}
接下来只需要在gradle.properties中写上：

isDebug = false
需要说明的是：根据公司和插件化技术的不同，此方法因人而异。

定义库的私有混淆

有很多库是需要进行混淆配置的，但让使用者配置混淆文件的方式总是不太友好，consumerProguardFiles的出现可以让库作者在库中定义混淆参数，让混淆配置对使用者屏蔽。
ShareLoginLib中的例子：

apply plugin: 'com.android.library'

android {
    compileSdkVersion 24
    buildToolsVersion '24.0.2'

    defaultConfig {
        minSdkVersion 9
        targetSdkVersion 24
        consumerProguardFiles 'consumer-proguard-rules.pro' // 自定义混淆配置
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

realm也用到了这样的配置：


打包工具会将*.pro文件打包进入aar中，库混淆时候会自动使用此混淆配置文件。

以consumerProguardFiles方式加入的混淆具有以下特性：

*.pro文件会包含在aar文件中
这些pro配置会在混淆的时候被使用
此配置针对此aar进行混淆配置
此配置只对库文件有效，对应用程序无效
如果你对于consumerProguardFiles有疑问，可以去ConsumerProGuardFilesTest这个项目了解更多。

指定资源目录

android {
    sourceSets {
        main {
            manifest.srcFile 'AndroidManifest.xml'
            java.srcDirs = ['src']
            resources.srcDirs = ['src']
            aidl.srcDirs = ['src']
            renderscript.srcDirs = ['src']
            assets.srcDirs = ['assets']
            if (!IS_USE_DATABINDING) { // 如果用了databinding
                jniLibs.srcDirs = ['libs']
                res.srcDirs = ['res', 'res-vm'] // 多加了databinding的资源目录
            } else {
                res.srcDirs = ['res']
            }
        }

        test {
            java.srcDirs = ['test']
        }

        androidTest {
            java.srcDirs = ['androidTest']
        }
    }
}
通过上面的配置，我们可以自定义java代码和res资源的目录，一个和多个都没有问题，更加灵活（layout文件分包也是利用了这个知识点）。

定义多个Manifest

sourceSets {
    main {
        if (isDebug.toBoolean()) {
            manifest.srcFile 'src/debug/AndroidManifest.xml'
        } else {
            manifest.srcFile 'src/release/AndroidManifest.xml'
        }
    }
}
根据flavor也可以进行定义：

productFlavors {
    hip {
        manifest.srcFile 'hip/AndroidManifest.xml'
    }

    main {
        manifest.srcFile '<where you put the other one>/AndroidManifest.xml'
    }
}
总结

gradle的最佳实践是最好写也是相当难写的。好写之处在于都是些约定俗成的配置项，而且写法固定；难写之处在于很难系统性的解释和说明它在实际中的意义。因为它太灵活了，可以做的事情太多了，用法还是交给开发者来扩展吧。
当年从eclipse切到android studio时，gradle没少给我添麻烦，也正是因为这些麻烦和不断的填坑积累，给我了上述的多个实践经验。
从写demo到正式项目，从正式项目做到开发库，从开发库做到组件化，这一步步的走来都少不了gradle这个魔鬼。今天我将我一年内学到的和真正使用过的东西分享在此，希望大家除了获益以外，还能真的将gradle视为敌人和友人，去多多了解这个家伙。


developer-kale@foxmail.com

参考自：

GRADLE构建最佳实践
Gradle依赖统一管理
深入理解Android（一）：Gradle详解
生成带混淆配置的aar
Making Gradle builds faster
Gradle Plugin User Guide
