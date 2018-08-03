---
layout: post
title:  "Android Xposed HOOK的编写案例"
date: 2018-08-3

description: "Android Xposed HOOK的编写案例"

tag: Android 逆向
---   

  ### 前言

最近对Android的Xposed有点兴趣。所以就准备记录下自己使用Xposed的一些笔记与诸位分享~

### Xposed扫盲

​     Xposed框架是一款可以在不修改APK的情况下影响程序运行(修改系统)的框架服务，基于它可以制作出许多功能强大的模块，且在功能不冲突的情况下同时运作。没错，以上介绍摘抄自百度~，通俗点，Xposed是一个框架，基于这个框架，可以让我们自己写代码，在不对APK文件修改的情况下，实现我们自己想要的功能。再通俗点，比如你手上有个可以提供文本阅读、编辑的APK，功能强大，但是美中不足的是，他要收费！得付费给作者，然后作者根据你的机器码或者其他特征给你生成个注册码，填上这个注册码，你才能继续使用。这下该怎么办呢？我们可以通过反编译这个APK，找到关键的地方，然后我们编写Xposed代码，Hook(这个不知道的就去百度吧~)这个地方，让APK执行到这个地方的时候，调用我们的代码，这下我们不就可以为所欲为了么？有人说，我可以直接修改smail文件(APK反编译后的文件，可以理解为Java的.class文件)啊。不可否认，那样确实可以，但是那种方式太粗暴了。我们需要的是优雅的和谐~~ 好了。扫盲结束。或许还是有一脸懵逼的。那么，百度吧~~ 我扯不下去了 QAQ~~

### 玩Xposed的前置条件

1. 一台Android手机
2. 手机必须root(如果手机不想root，可以下载个VirtualXposed软件，可以免root使用Xposed框架)
3. Android SDK
4. AndroidStudio或Eclipse

### 开整

安装SDK什么的，我就不说了，百度上有大把的教程。

1. 用Android Studio 新建一个项目

   ![](http://p9sepa44i.bkt.clouddn.com/18-8-3/77823384.jpg)

2. 下一步

   ![](http://p9sepa44i.bkt.clouddn.com/18-8-3/37576981.jpg)

3. 选择Add No activity

   ![](http://p9sepa44i.bkt.clouddn.com/18-8-3/51893591.jpg)

4. 把视图切换到project

   ![](http://p9sepa44i.bkt.clouddn.com/18-8-3/85571920.jpg)

5. 在app下面新建个lib文件夹，放入Xposed API的jar包(api-82.jar)。然后选中jar包，右键->add as Library

6. 打开app/build.gradle.把`dependencies` 修改成如下：

   ```groovy
   dependencies {
       implementation fileTree(include: ['*.jar'], dir: 'libs')
       implementation 'com.android.support:appcompat-v7:26.1.0'
       testImplementation 'junit:junit:4.12'
       androidTestImplementation 'com.android.support.test:runner:1.0.2'
       androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
       //其实就是把下面的 implementation 改成 compileOnly即可
       compileOnly files('lib/api-82.jar')  //这是Xposed API
       compileOnly files('lib/api-82-sources.jar') //这是Xposed API的源码。不导入也关系
   }
   ```

7. 打开app/src/main下的AndroidManifest.xml，修改成如下：

   ```xml
   <manifest xmlns:android="http://schemas.android.com/apk/res/android"
       package="zyzl.in.xposedhookdemo">
   	<!-- 以下是自动生成的 -->
       <application
           android:allowBackup="true"
           android:icon="@mipmap/ic_launcher"
           android:label="@string/app_name"
           android:roundIcon="@mipmap/ic_launcher_round"
           android:supportsRtl="true"
           android:theme="@style/AppTheme">
     	<!-- 以上是自动生成的 -->
       
           <!-- 声明为Xposed模块 -->
           <meta-data android:name="xposedmodule"  android:value="true" />
           <!-- 模块描述 -->
           <meta-data android:name="xposeddescription"  android:value="HookDemo" />
           <!-- Xposed的最低版本，这里固定53就好 -->
           <meta-data android:name="xposedminversion"  android:value="53" />
       </application>
   </manifest>
   ```

8. 接下来就是coding阶段了~

   在app/src/main/java/包名下新建一个类，实现`IXposedHookLoadPackage` 我的代码如下：

   ```java
   public class Hook implements IXposedHookLoadPackage {
       @Override
       public void handleLoadPackage(XC_LoadPackage.LoadPackageParam lpparam) throws Throwable {
           //因为是模版，所以我们这里就在加载包的时候，把包名打印出来
           XposedBridge.log(lpparam.packageName);
       }
   }
   ```

9. 万事具备了，只差一个文件了。该文件的目的就是告诉Xposed框架，你的启动类或者入口是哪个。

   1. 在/app/src/main下面建立一个assets文件夹。并在下面建一个`xposed_init` 文件，在其中写上你自己编写的Hook类的全类名。比如我是`zyzl.in.xposedhookdemo.Hook` 所以文件的内容为：

      `zyzl.in.xposedhookdemo.Hook`

10. 到此，整个的目录结构如下（修改过的地方使用红框框起来了）：

    ![](http://p9sepa44i.bkt.clouddn.com/18-8-3/80173122.jpg)

11. 接下来就是进行打包签名了。然后在手机上安装。注意：新模块安装后，需要在Xposed框架中勾选，并且重启手机~

12. 使用`XposedBridge.log()`打印出来的日志，可以在Xposed框架的日志Tab中看到~

    ​

好了~这篇到此也结束了。下次有空带来个Xposed实战破解APK。