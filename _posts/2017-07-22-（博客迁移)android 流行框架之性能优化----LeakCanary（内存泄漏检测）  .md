 内存泄漏，内存溢出。图片太大，或者变量没有回收，都会导致内存泄漏，这种东西有的时候神不知鬼不觉，我们可以通过ES自带的MAT或者AS的内存监控来生成一个文件来进行分析，但这并不是简单粗暴的，今天就给大家推荐一个简单粗暴的检测框架LeakCanary。如何简单粗暴呢，下面请我细细说来。


1.首先要去下载一下这个库，或者gradle 依赖一下，自己百度最新的进行下载。


dependencies {
    debugCompile 'com.squareup.leakcanary:leakcanary-android:1.3'
    releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.3'
}



2.自己写一个Application来集成Application  并在里面对LeakCanary进行初始化操作。

class MyApplication extends Application {  
  
  @Override public void onCreate() {  
    super.onCreate();  
    LeakCanary.install(this);  
  }  
}  

运行程序，会在内存泄漏的地方出现



LeakCanary的内存泄露提示一般会包含三个部分:
第一部分(LeakSingle类的sInstance变量)引用第二部分(LeakSingle类的mContext变量), 导致第三部分(MainActivity类的实例instance)泄露.

应用最常见的泄露位置就是Activity的实例, 要手动或使用shell命令, 启动所有的Activity. LeakCanary判断时机是Activity启动到结束, 检测这一过程是否会发生内存泄露.





