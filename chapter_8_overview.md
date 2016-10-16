# ***第八章 Android安全测试*** 
=====================

&emsp;&emsp;本章将从安全测试的角度为大家讲解安全内容，作为一名Android安全测试人员需要掌握哪些基础知识，从哪些角度上去做安全测试，什么样的安全结果会被认为安全测试通过，如何对安全测试的结果进行相关的统计，衡量安全的整体趋势。

## **8.1 安全测试的基础**

&emsp;&emsp;本书的1-7章的内容都需要安全测试同学熟读，作为一个Android端的安全测试同学，需要掌握Android系统的底层架构，了解逆向工程，汇编语言，静态与动态的安全分析手段，除此以外，作为一名Android安全测试人员还需要掌握以下的基础能力。


*   基础的Android开发能力
*   数据库操作能力
*   模糊测试的思维能力
*   多语言代码编写的能力

### **8.1.1 基础的Android开发能力**

&emsp;&emsp;如下图为Android安全测试同学需要掌握的基础开发能力的图谱，然后我们逐一进行阐述。

&emsp;&emsp;
![基础的Android开发能力](https://github.com/wirelesscollege/androidSecurity/blob/dev/image/8-1-1基础的Android开发能力.png "基础的Android开发能力")

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;[8-1-1 基础的Android开发能力]

#### **8.1.1.1 了解Android开发工具**

&emsp;&emsp;AndroidStudio是当前最流程的开发工具，了解常用Android开发工具，可以将一些安全隐患解决在开发同学编码阶段，例如，我们之前章节讲解过的Lint工具，就已经被集成进AndroidStudio IDE工具中，我们可以将一些其他第三方的工具，或者制作相关的插件，防止开发阶段代码安全缺陷，提早发现解决掉这些缺陷。下面以Lint为例子讲解，下图为AndroidStudio内Lint静态分析工具的入口以及Lintcheck的流程。

![AndroidStudioLint位置](https://github.com/wirelesscollege/androidSecurity/blob/dev/image/8-1-1AndroidStudioLint位置.png "AndroidStudioLint位置")

  &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;[8-1-1 AndroidStudioLint位置]

&emsp;&emsp;知道位置后，在右键run之前需要检查你的功能内是否含有Lint的配置，只有配置了Lint的相关参数，才可以控制Lint的输出。

![Lint的配置](https://github.com/wirelesscollege/androidSecurity/blob/dev/image/8-1-1Lint的配置.png "Lint的配置")

  &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;[8-1-1 Lint的配置]

&emsp;&emsp;如上参数只是一个简单的例子，如果想自己控制Lint的各种输出和检查项，可以使用如下图内的所有可用参数。

![Lint配置属性](https://github.com/wirelesscollege/androidSecurity/blob/dev/image/8-1-1Lint配置属性.png "Lint配置属性")

  &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;[8-1-1 Lint的配置属性]

&emsp;&emsp;如下是LintCheck代码的流程图。

![Lintcheck流程](https://github.com/wirelesscollege/androidSecurity/blob/dev/image/8-1-1Lintcheck流程.png "Lintcheck流程")

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;[8-1-1 Lintcheck流程]

&emsp;&emsp;如下说明Lint检查哪些安全检测项目。

| 项目        | 检查内容          | 说明  |
| ------------- |:-------------:| -----:|
| PackagedPrivateKey      | 查找私钥文件 | 一般情况下，私钥文件不应该存在于工程之中 |
| GrantAllUris      | 检查<grant-uri-permission>项是否共享了所有内容      |   通常情况下<grant-uri-permission>路径不应为‘/’（共享所有内容），应该制定确切的子集 |
| SetJavaScriptEnabled | 检查是否包含了SetJavaScriptEnabled     |    JavaScript交互会出很多问题，一些常见的webview问题前提就是该项为true |
| ExportedContentProvider | 检查是否未设置权限就共享了content provider     |    默认情况下content provider是共享的，任何应用程序都可以通过它来读写数据。如果content provider提供的是敏感数据，应该禁止共享或增加权限控制 |
| ExportedService | 检查是否公开了服务 | 所有公开的服务都应该设置权限，否则所有应用程序都可以绑定在这个服务上 |
| ExportedReceive | 检查是否没设置权限就共享了receiver | 如果receiver的exported属性设置为true，或intent-filter没有指明exported属性为false，那么该receiver是共享的，任何程序都可以使用这个receiver，这是很不安全的。 |
| HardcodedDebugMode | 检查在mainifest中是否为android：debuggable设置了固定值 | 默认情况下，程序调试时编译器会自动将android：debuggable设置为true，在发布的是否将其设置为false。如果在发布的时候manifest中包含该属性为true的语句，可能会泄露一些敏感的调试信息。 |
| WorldReadableFiles | 检查调用openFileOutput()和getSharedPreferences()函数时是否使用了MODE_WORLD_READABLE参数 | 建议在将文件设置成全局可读属性前确保文件内不会有敏感信息。 |
| WorldWriteableFiles | 检查调用openFileOutput()和getSharedPreferences()函数时是否使用了MODE_WORLD_WRITEABLE参数 | 建议在将文件设置成全局可写属性前确保文件内不会有敏感信息。如果有恶意软件修改了该文件，会导致程序严重出错。 |
| AllowBackup | 检查manifest中是否包含了allowBackup的设置 | allowBackup默认为真，如果允许备份，用户可以讲程序中所有数据备份出来，以便修改后可以恢复回去。开发者应该仔细考虑自己的程序是否允许用户备份，这个在manifest中可以设置android：allowBackup |
| ExportedActivity | 检查是否没有设置权限就共享了Activity | 如果没对Activity设置权限控制，任何程序都可以使用它。 |

&emsp;&emsp;刚刚我们运行Lint的报告会生成HTML与xml2份报告，其中xml报告可以持续集成到Jenkins平台，HTML报告我们这里打开为读者展示一下。这里我们发现了一个允许备份的安全问题。如图。

![Lint检查结果](https://github.com/wirelesscollege/androidSecurity/blob/dev/image/8-1-1lint检查结果.png "Lint检查结果")

  &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;[8-1-1 Lint检查结果]


&emsp;&emsp;Google官方维护的工具就是AndroidStudio，我们要了解AndroidStudio插件的安装方式。这样可以让我们安全插件集成到开发工具上。如下图为AndroidStudio内插件的安装方式。其中Jet Brains为Intelli J IDEA的官方插件，然后是在线安装方式和本地安装方式。在线安装方式找到插件的updatesite地址即可安装，本地安装方式需要将插件下载到本地然后选择进行安装。

![AndroidStudio插件安装](https://github.com/wirelesscollege/androidSecurity/blob/dev/image/8-1-1AndroidStudio插件安装.png "AndroidStudio插件安装")

  &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;[8-1-1 AndroidStudio插件安装]


#### **8.1.1.2 了解Android开发语言**

&emsp;&emsp;以上介绍仅为你讲述了AndroidStudio插件方面的能力，你可以选择其他安全插件集成到开发工具中，下面为你介绍Android开发语言，Android主要开发语言为Java，如果是NDK开发者使用的语言为C，这里我们需要掌握Java语言的基本开发能力，了解[JavaAPI][javaapi]以及[AndroidAPI][androidapi]。

[javaapi]: https://docs.oracle.com/javase/7/docs/api/  "javaapi"
[androidapi]: https://developer.android.com/reference/packages.html  "androidapi"

&emsp;&emsp;JavaAPI地址以及相关截图，如下。

![JavaApi](https://github.com/wirelesscollege/androidSecurity/blob/dev/image/8-1-1JavaApi.png "JavaApi")

&emsp;&emsp;AndroidAPI地址以及相关截图，如下。

![AndroidApi](https://github.com/wirelesscollege/androidSecurity/blob/dev/image/8-1-1AndroidApi.png "AndroidApi")

&emsp;&emsp;因本书重点是讲解Android安全，Java编码以及相关Api建议你可以参考专业的编程书籍。但作为安全测试人员，你需要了解JavaApi与AndroidApi在Android版本更新中，常见一些安全风险，例如：压缩文件目录遍历漏洞。

&emsp;&emsp;例：8-1-1例 压缩文件目录遍历漏洞

&emsp;&emsp;在Linux/Unix系统中“../”代表的是向上级目录跳转，有些程序在当前工作目录中处理到诸如用“../../../../../../../../../../../etc/hosts”表示的文件，会跳转出当前工作目录，跳转到到其他目录中。 

&emsp;&emsp;Java代码在解压ZIP文件时，会使用到ZipEntry类的getName()方法，如果ZIP文件中包含“../”的字符串，该方法返回值里面原样返回，如果没有过滤掉getName()返回值中的“../”字符串，继续解压缩操作，就会在其他目录中创建解压的文件。

&emsp;&emsp;利用此方式就可以替换，或者在任意目录下植入木马文件，或者恶意软件。你可以参考海豚浏览器远程代码执行漏洞案例：http://www.cnblogs.com/alisecurity/p/5610654.html。（参考地址）
 
&emsp;&emsp;海豚浏览器的主题设置中允许用户通过网络下载新的主题进行更换，主题文件其实是一个ZIP压缩文件。通过中间人攻击的方法可以替换掉这个ZIP文件。替换后的ZIP文件中有重新编译过的libdolphin.so。此so文件重写了JNI_OnLoad()函数：


![海豚主题加载代码](https://github.com/wirelesscollege/androidSecurity/blob/dev/image/8-1-1海豚主题加载代码.png "海豚主题加载代码")

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;[8-1-1 海豚主题加载代码]
 

&emsp;&emsp;此so文件以“../../../../../../../../../../data/data/mobi.mgeek.TunnyBrowser/files/libdolphin.so”的形式存在恶意ZIP文件中。海豚浏览器解压恶意ZIP文件后，重新的libdolphin.so就会覆盖掉原有的so文件，重新运行海豚浏览器会弹出Toast提示框：

![海豚浏览器加载恶意代码执行](https://github.com/wirelesscollege/androidSecurity/blob/dev/image/8-1-1海豚浏览器加载恶意代码执行.png "海豚浏览器加载恶意代码执行")

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;[8-1-1 海豚浏览器加载恶意代码执行]

&emsp;&emsp;能弹出Toast说明也就可以执行其他代码。

&emsp;&emsp;这里分析下此漏洞产生的原因是： 

&emsp;&emsp;1、主题文件其实是一个ZIP压缩包，从服务器下载后进行解压，但是解压时没有过滤getName()返回的字符串中是否有“../”：

![漏洞代码分析](https://github.com/wirelesscollege/androidSecurity/blob/dev/image/8-1-1漏洞代码分析.png "漏洞代码分析")

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;[8-1-1 漏洞代码分析]


&emsp;&emsp;2、动态链接库文件libdolphin.so，并没有放在应用数据的lib目录下，而是放在了files目录中：

![文件被替换](https://github.com/wirelesscollege/androidSecurity/blob/dev/image/8-1-1文件被替换.png "文件被替换")

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;[8-1-1 文件被替换]

&emsp;&emsp;加载使用的地方是com.dolphin.browser.search.redirect包中的SearchRedirector：

![加载代码分析](https://github.com/wirelesscollege/androidSecurity/blob/dev/image/8-1-1加载代码分析.png "加载代码分析")

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;[8-1-1 加载代码分析]

#### **8.1.1.3 了解Android项目结构**

&emsp;&emsp;了解Android项目的目录结构，知晓项目中每个部分的存储内容以及重要性对于安全测试工程师是必不可缺的一部，下图是一个最简单的Demo项目的目录。

![Android项目目录结构](https://github.com/wirelesscollege/androidSecurity/blob/dev/image/8-1-1Android项目目录结构.png "Android项目目录结构")

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;[8-1-1 Android项目目录结构]

*   iml文件为整体工程的配置文件，与eclipse的.project文件功能类似
*   App目录项目最主要的目录，包含源码，依赖包，编译后文件，以及主工程配置文件
	*   App.iml App目录配置文件
	*   build App编译后所产生的中间以及结果性文件目录
	*   build.gradle 项目子工程的Gradle配置文件
	*   libs 项目所依赖的相关库文件
	*   proguard-rules.pro 源码混淆配置文件
	*   src 项目源码文件目录
*   build 项目构建过程中文件目录，例如lint-cache
*   gradle Gradle wrapper的依赖库
*   gradle.properties Gradle属性配置文件
*   gradlew Gradle wrapper命令文件，Linux环境
*   gradlew.bat Gradle wrapper命令文件，windows环境
*   local.properties Android本地属性key-value配置文件
*   settings.gradle Gradle主子工程关系配置文件

![App-src](https://github.com/wirelesscollege/androidSecurity/blob/dev/image/8-1-1App-src.png "App-src")

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;[8-1-1 App-src目录]

*   androidTest 项目Android应用测试代码
*   main 项目源码资源目录
	*   AndroidManifest.xml 项目资源、权限、组件配置文件
	*   java 项目源码目录
	*   res 项目资源目录   
*   test 项目单元测试代码


&emsp;&emsp;作为安全测试人员需要重点掌握如下几个目录：progurad-rules.pro混淆文件，build.gradle配置文件，local.properties配置文件，AndroidManifest.xml配置文件。

&emsp;&emsp;如上文件会告知你以下信息：项目的依赖关系，项目的依赖库，项目打包的配置与参数，项目混淆的配置，项目的权限配置，项目的组件安全配置，项目的第三方SDK配置。这些配置信息对于我们来说都很重要。

![Gradle混淆配置](https://github.com/wirelesscollege/androidSecurity/blob/dev/image/8-1-1Gradle混淆配置.png "Gradle混淆配置")

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;[8-1-1 Gradle混淆配置]

&emsp;&emsp;例如：在上图例子中我们就可以知道Android的混淆是否被配置，混淆可以降低逆向工程后代码的可读性，当前的配置就是未开启混淆，那么应用上线后就是有风险的。其他维度我们在8.2小节内在做讲解。

#### **8.1.1.3 了解Android项目编译过程**

![Android项目编译过程](https://github.com/wirelesscollege/androidSecurity/blob/dev/image/8-1-1Android项目编译过程.png "Android项目编译过程")

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;[8-1-1 Android项目编译过程]

&emsp;&emsp;上图就是一个执行task为assembleRelease的Android单项目的编译过程，而项目的线上环境上线都是走这个task，如果你的项目工程为多样化工程，编译过程会比上述截图更加复杂，会包含所有子项目的编译过程，我们就拿上面单项目的编译过程来说明，了解了单项目的编译过程，多样化项目的编译过程就会更好理解。

*   :app:preBuild 
*   :app:preReleaseBuild
*	:app:preBuild 
*	:app:preReleaseBuild 
*	:app:checkReleaseManifest
*	:app:preDebugBuild UP-TO-DATE
*	:app:prepareComAndroidSupportAnimatedVectorDrawable2340Library UP-TO-DATE
*	:app:prepareComAndroidSupportAppcompatV72340Library UP-TO-DATE
*	:app:prepareComAndroidSupportSupportV42340Library UP-TO-DATE
*	:app:prepareComAndroidSupportSupportVectorDrawable2340Library UP-TO-DATE
*	:app:prepareReleaseDependencies
*	:app:compileReleaseAidl UP-TO-DATE
*	:app:compileReleaseRenderscript UP-TO-DATE
*	:app:generateReleaseBuildConfig UP-TO-DATE
*	:app:mergeReleaseShaders UP-TO-DATE
*	:app:compileReleaseShaders UP-TO-DATE
*	:app:generateReleaseAssets UP-TO-DATE
*	:app:mergeReleaseAssets UP-TO-DATE
*	:app:generateReleaseResValues UP-TO-DATE
*	:app:generateReleaseResources UP-TO-DATE
*	:app:mergeReleaseResources UP-TO-DATE
*	:app:processReleaseManifest UP-TO-DATE
*	:app:processReleaseResources UP-TO-DATE
*	:app:generateReleaseSources UP-TO-DATE
*	:app:incrementalReleaseJavaCompilationSafeguard UP-TO-DATE
*	:app:compileReleaseJavaWithJavac UP-TO-DATE
*	:app:compileReleaseNdk UP-TO-DATE
*	:app:compileReleaseSources UP-TO-DATE
*	:app:lintVitalRelease
*	:app:prePackageMarkerForRelease
*	:app:transformClassesWithDexForRelease
*	:app:mergeReleaseJniLibFolders
*	:app:transformNative_libsWithMergeJniLibsForRelease
*	:app:processReleaseJavaRes UP-TO-DATE
*	:app:transformResourcesWithMergeJavaResForRelease
*	:app:packageRelease
*	:app:assembleRelease






## **8.2 安全测试的维度**
## **8.3 安全测试的标准**
## **8.4 安全测试统计**
