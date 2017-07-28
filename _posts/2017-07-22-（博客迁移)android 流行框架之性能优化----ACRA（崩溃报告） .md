在实际开发中，不论是debug或者release 的时候。初期来说，程序的崩溃是在所难免的，这些崩溃是因为我们的逻辑判断上没有考虑周全，亦或是因为我们的粗心大意，在某个关键位置多写或者少写了一行代码。作为一个合格的程序员，仅仅为了实现功能就像是不想当一个将军的士兵一样不是一个好的程序员，所以，我们做出来的产品既要实现需求，更要让他运行流畅，程序员+测试+用户，才是一个真正的合格的程序员。

扯多了，下面来说一个在开发过程中，一个便于我们检测程序崩溃的框架 ACRA。又叫做 Application CrashReport for Android。

接下来我们看看如何将他应用到我们的项目中，可以参考：https://github.com/ACRA/acra/wiki/BasicSetup。

首 先     1.下 载jar包： .新版本v4.9.0 - 4-jun-2016 acrA 添加到工程中。

              2. compile 'ch.acra:acra:4.9.0'

然后新建一个自己的Application 继承 Application 并在AndroidMainfest 中修改成自己的Application。

添加一个@ reportscrashes注释到你的应用程序类，然后覆盖attachbasecontext()方法添加AcrA。init（this）；进行初始化，代码如下：

   import org.acra.*;
    import org.acra.annotation.*;

    @ReportsCrashes(
        formUri = "http://www.backendofyourchoice.com/reportpath"
    )
    public class MyApplication extends Application {
        @Override
        protected void attachBaseContext(Context base) {
            super.attachBaseContext(base);

            // The following line triggers the initialization of ACRA
            ACRA.init(this);
        }
    }

或者你也可以构建 ACRA programmatically, 然后再进行初始化，代码如下：

  import org.acra.ACRA;
    import org.acra.configuration.*;

    public class MyApplication extends Application {
        @Override
        protected void attachBaseContext(Context base) {
            super.attachBaseContext(base);

            // Create an ConfigurationBuilder. It is prepopulated with values specified via annotation.
            // Set any additional value of the builder and then use it to construct an ACRAConfiguration.
            final ACRAConfiguration config = new ConfigurationBuilder(this)
                .setFoo(foo)
                .setBar(bar)
                .build();

            // Initialise ACRA
            ACRA.init(this, config);
        }
    }

最后要注意,因为我们会将崩溃信息通过网络反馈给我们，所以，在这里，要声明网络权限：

<uses-permission android:name="android.permission.INTERNET"/>

由于时间原因，这里对于崩溃的处理以及收集。请大家参考一下链接：

出现异常怎么显示呢？参考https://github.com/ACRA/acra/wiki/AdvancedUsage#user-interaction

反馈错误的方式有很多，参考https://github.com/ACRA/acra/wiki/AdvancedUsage#reports-destination

怎么错误报告的内容呢？参考：https://github.com/ACRA/acra/wiki/AdvancedUsage#reports-content


