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


## **8.2 安全测试的维度**
## **8.3 安全测试的标准**
## **8.4 安全测试统计**
