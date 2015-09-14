title: Apk 加入sdk的流程
---

大概的流程是这样：

[http://www.webtag123.com/android/41453.html](http://www.webtag123.com/android/41453.html)

[![总流程图](http://images.cnitblog.com/blog/231632/201412/211153140304884.png "总流程图")](http://images.cnitblog.com/blog/231632/201412/211153140304884.png "总流程图")

## APK
用[apktool](https://bitbucket.org/iBotPeaches/apktool/downloads)反编译：   

	1.打开cmd命令行，进入apktool.jar和xxx.apk所在的文件夹（置于同一个文件夹，方便操作）
	
	2.输入java -jar apktool.jar，可以看到相关的使用命令的提示，d是反编译指令
	
	3.再输入java -jar apktool.jar d xxx.apk，即可完成反编译


反编译后，产生的目录：

	D:\pojo\09月10日\pojo\demo\HiSdk\HiSdk>ls
	AndroidManifest.xml  apktool.yml  build  dist  original  res  smali

## SDK
参考：[http://blog.csdn.net/lucherr/article/details/39896549](http://blog.csdn.net/lucherr/article/details/39896549)


在流程图中是用 jar 文件变成 dex 文件，再变成 smali 文件，最后放入 apk 反编译后的smali 文件夹内：

Jar->dex->smali->apk 反编译后的smali 文件夹

但因为SDK 还可以导出为 APK 文件，所以在操作上是这样的：

	1、SDK 导出为 APK 文件，解压apk文件，获取classes.dex并拷贝到资源根目录（使用zip或其他解压工具即可）
	
	2、使用 baksmali工具 将classes.dex转为smali文件，在命令行定位到资源根目录并执行：
	   java -jar baksmali-2.0.3.jar -x classes.dex
	这时会在当前文件夹生成out文件夹，里面就是将classes.dex转为后的smali文件
	
	3、将产生的smali文件加入到反编译产生的smali文件内
	
	4、根据“sdk使用说明.txt”文档，修改反编文件夹中的文件：AndroidManifest.xml


重打包与签名

	1、将apktool.jar放在与反编译文件夹同目录下，执行：
	java -jar apktool.jar b HiSdk（反编产生的文件夹名字）
	apk文件产生在反编译文件夹中的 dist 文件夹下
	
	2、使用360签名工具进行签名，下载地址：
	http://jiagu.360.cn/1101144936.php?dtid=1101144931&did=1101151286


工具：
[baksmali工具]((https://bitbucket.org/JesusFreke/smali/downloads))

[android反编译工具的合集](https://github.com/Juude/droidReverse)

[安卓APK反编译、重编译和签名工具，基于Apktool和Dex2Jar开发集两大利器功能于一体，图形化界面操作，简单易用，是安卓程序反编译、破解、汉化的得力助手。](https://code.google.com/p/anti-droid/)
[downloads](https://code.google.com/p/anti-droid/downloads/list)

[360签名工具](http://jiagu.360.cn/1101144936.php?dtid=1101144931&did=1101151286)