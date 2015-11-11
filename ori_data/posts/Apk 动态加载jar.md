title: Apk 动态加载jar
---

动态加载jar参考网址：
[http://www.cnblogs.com/over140/archive/2011/11/23/2259367.html](http://www.cnblogs.com/over140/archive/2011/11/23/2259367.html)

	6.1.2　　导出jar时不能带接口文件，否则会报以下错：

	 java.lang.IllegalAccessError: Class ref in pre-verified class resolved to unexpected implementation

	6.1.3　　将jar优化时应该重新成jar(jar->dex->jar)，如果如下命令：

	E:\Android\adt-bundle-windows-x86-20140702\sdk\build-tools\23.0.1>dx.bat --dex --output=test.jar  hi_jar.jar
	E:\Android\adt-bundle-windows-x86-20140702\sdk\build-tools\23.0.1>


### 注！不要混淆引用的接口

	-keep class com.ffkj.** { *;}