我的CSDN博客：http://blog.csdn.net/baidu_35255405
博客陆续会迁移到这里



接下来将全面的分析Android性能方面的问题。希望多多的Issues和Fork。


Android 性能优化之 布局优化

先广泛的说一下性能优化，如果是后台开发的话，内存溢出以及耗时算法可能引起性能上的问题。如果是移动端的开发的话，就多了这么一条，多了这么一层UI渲染。好吧，开头就是泛泛地说一下，那么，今天这篇博客的内容主要就是针对UI渲染上的优化。

2015年的时候，Google发布了Android性能优化典范专题 。都是以短视频的形式，来帮助我们开发人员开发出更好的android App。如果有兴趣的同学，可以自行搜索引擎。

Android的渲染机制：（PS：下面的介绍摘自开源中国：http://www.oschina.NET/news/60157/android-performance-patterns）

希望你们可以去看这篇文章。

下面主要谈一下这几个标签，在布局中的使用会提升渲染的性能。

1，include 标签

将外部布局文件引入到当前布局文件，适用于多界面频繁出现的相同布局，来实现布局的模块换。

2，viewstub 标签

同include标签相似，将外部布局引入到当前布局，与之不同之处，由该标签引入的布局默认不会扩张，即不占用位置，不展示，在UI渲染的时候后，节省CPU和内存。通常用于默认不展示的View ,当某个事件的发生，从而引起他的展示， 例如，网络连接断开，网络请求

在Java中通过(ViewStub)findViewById(id)找到ViewStub，通过stub.inflate()展开ViewStub，然后得到子View，如下：

private View networkErrorView;  

private void showNetError() {  
    // not repeated infalte  
    if (networkErrorView != null) {  
        networkErrorView.setVisibility(View.VISIBLE);  
        return;  
    }  

    ViewStub stub = (ViewStub)findViewById(R.id.network_error_layout);  
    networkErrorView = stub.inflate();  
    Button networkSetting = (Button)networkErrorView.findViewById(R.id.network_setting);  
    Button refresh = (Button)findViewById(R.id.network_refresh);  
}  

private void showNormal() {  
    if (networkErrorView != null) {  
        networkErrorView.setVisibility(View.GONE);  
    }  
}  

在上面showNetError()中展开了ViewStub，同时我们对networkErrorView进行了保存，这样下次不用继续inflate。这就是后面第三部分提到的减少不必要的infalte。

viewstub标签大部分属性同include标签类似。

ViewStub stub = (ViewStub)findViewById(R.id.network_error_layout);  
networkErrorView = stub.inflate();  

3，merge

merge标签可用于两种典型情况：
a. 布局顶结点是FrameLayout且不需要设置background或padding等属性，可以用merge代替，因为Activity内容试图的parent view就是个FrameLayout，所以可以用merge消除只剩一个。
b. 某布局作为子布局被其他布局include时，使用merge当作该布局的顶节点，这样在被引入时顶结点会自动被忽略，而将其子节点全部合并到主布局中。
