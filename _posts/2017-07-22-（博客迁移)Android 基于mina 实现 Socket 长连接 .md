一，什么是长连接

长连接顾名思义就是长时间持续的连接，想比较http，一次请求响应之后，连接就不在保持，即使当前比较流行的http请求框架，也只能尽量做到缓存这个层面。

二，应用场景

即时通讯、消息推送、实时位置上报、直播等等。。。

三，mina介绍

不知道的先百度百科一下

这是官网

四，用前准备

下载jar包：1、mina-core-2.0.16.jar 

                   2、slf4j-api-1.7.21.jar


五，实现思路。

长连接是耗时操作，所以要不能在程序主线程。要开service，在service 中来建立长连接。

下面我先来封装一些需要的类。

1）创建一个service，用来与远程服务器连接

2）封装一个ConnectionManager类用来提供与服务器连接、断开方法。

3）在service中启动线程，调用ConnectionManager完成连接的创建

4)   构造者模式来对参数进行配置


六，实现类代码

1.ConnectionConfig

package com.ij.minamanager;
import android.app.Service;
import android.content.Context;
import android.content.Intent;
import android.os.HandlerThread;
import android.os.IBinder;
import android.support.annotation.Nullable;

/**
 * ┏┓　　┏┓+ +
 * 　　　　　　　┏┛┻━━━┛┻┓ + +
 * 　　　　　　　┃　　　　　　　┃
 * 　　　　　　　┃　　　━　　　┃ ++ + + +
 * 　　　　　　 ████━████ ┃+
 * 　　　　　　　┃　　　　　　　┃ +
 * 　　　　　　　┃　　　┻　　　┃
 * iBoosJie.　 ┃　　　　　　　┃ + +
 * 　　　　　　　┗━┓　　　┏━┛
 * 　　　　　　　　　┃　　　┃
 * 　　　　　　　　　┃　　　┃ + + + +
 * 　　　　　　　　　┃　　　┃　　　　Code is far away from bug with the animal protecting
 * 　　　　　　　　　┃　　　┃ + 　　　　神兽保佑,代码无bug
 * 　　　　　　　　　┃　　　┃
 * 　　　　　　　　　┃　　　┃　　+
 * 　　　　　　　　　┃　 　　┗━━━┓ + +
 * 　　　　　　　　　┃ 　　　　　　　┣┓
 * 　　　　　　　　　┃ 　　　　　　　┏┛
 * 　　　　　　　　　┗┓┓┏━┳┓┏┛ + + + +
 * 　　　　　　　　　　┃┫┫　┃┫┫
 * 　　　　　　　　　　┗┻┛　┗┻┛+ + + +
 * 创建人：杰
 * 创建日期： 2017/6/6
 * 说明：
 * 修改：
 */
public class MinaService extends Service{

    private ConnectionThread thread;

    @Override
    public void onCreate() {
        super.onCreate();
        //全局context 避免内存泄漏，不多说
        thread = new ConnectionThread("mina",getApplicationContext());
        thread.start();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        return super.onStartCommand(intent, flags, startId);
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        thread.disConnection();
        thread = null;
    }


    //负责调用ConnectionManager类来完成与服务器连接
    class ConnectionThread extends HandlerThread{

        private Context context;

        boolean isConnection;

        ConnectionManager mManager;

        public ConnectionThread(String name,Context context) {
            super(name);
            this.context = context;

            ConnectionConfig config = new ConnectionConfig.Builder(context)
                    .setIp("127.0.0.1")
                    .setPort(9123)
                    .setReadBuilder(10240)
                    .setConnectionTimeout(10000).builder();
        }

        //run 开始连接我们的服务器
        @Override
        protected void onLooperPrepared() {
            super.onLooperPrepared();
            //死循环
            for (;;){
                isConnection = mManager.connection();
                if (isConnection){
                    break;
                }

                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }

        //断开连接
        public void disConnection(){

            mManager.disConnect();
        }
    }
}


2，ConnectionManager

package com.ij.minamanager;

import android.content.Context;
import android.content.Intent;
import android.support.v4.content.LocalBroadcastManager;

import org.apache.mina.core.future.ConnectFuture;
import org.apache.mina.core.service.IoHandlerAdapter;
import org.apache.mina.core.session.IoSession;
import org.apache.mina.filter.codec.ProtocolCodecFilter;
import org.apache.mina.filter.codec.serialization.ObjectSerializationCodecFactory;
import org.apache.mina.filter.logging.LoggingFilter;
import org.apache.mina.transport.socket.nio.NioSocketConnector;

import java.lang.ref.WeakReference;
import java.net.InetSocketAddress;

/**
 * ┏┓　　┏┓+ +
 * 　　　　　　　┏┛┻━━━┛┻┓ + +
 * 　　　　　　　┃　　　　　　　┃
 * 　　　　　　　┃　　　━　　　┃ ++ + + +
 * 　　　　　　 ████━████ ┃+
 * 　　　　　　　┃　　　　　　　┃ +
 * 　　　　　　　┃　　　┻　　　┃
 * iBoosJie.　 ┃　　　　　　　┃ + +
 * 　　　　　　　┗━┓　　　┏━┛
 * 　　　　　　　　　┃　　　┃
 * 　　　　　　　　　┃　　　┃ + + + +
 * 　　　　　　　　　┃　　　┃　　　　Code is far away from bug with the animal protecting
 * 　　　　　　　　　┃　　　┃ + 　　　　神兽保佑,代码无bug
 * 　　　　　　　　　┃　　　┃
 * 　　　　　　　　　┃　　　┃　　+
 * 　　　　　　　　　┃　 　　┗━━━┓ + +
 * 　　　　　　　　　┃ 　　　　　　　┣┓
 * 　　　　　　　　　┃ 　　　　　　　┏┛
 * 　　　　　　　　　┗┓┓┏━┳┓┏┛ + + + +
 * 　　　　　　　　　　┃┫┫　┃┫┫
 * 　　　　　　　　　　┗┻┛　┗┻┛+ + + +
 * 创建人：杰
 * 创建日期： 2017/6/6
 * 说明：连接管理类
 * 修改：
 */
public class ConnectionManager {


    public static final String BROADCAST_ACTION ="com.ij.minamanager";

    public static final String MESSAGE ="message";

    private ConnectionConfig mConfig;

    private WeakReference<Context> mContext; //避免内存泄漏

    private NioSocketConnector mConnection;

    private IoSession mSession;

    private InetSocketAddress mAddress;

    public ConnectionManager(ConnectionConfig config) {
        this.mConfig = mConfig;
        this.mContext =new WeakReference<Context>(config.getContext());
        
        init();
    }

    //通过构建者模式来进行初始化
    private void init() {


        mAddress = new InetSocketAddress(mConfig.getIp(),mConfig.getPort());

        mConnection = new NioSocketConnector();

        //设置读数据大小
        mConnection.getSessionConfig().setReadBufferSize(mConfig.getReadBufferSize());

        //添加日志过滤
        mConnection.getFilterChain().addLast("Logging",new LoggingFilter());

        //编码过滤
        mConnection.getFilterChain().addLast("codec",new ProtocolCodecFilter(
                new ObjectSerializationCodecFactory()));

        //事物处理
        mConnection.setHandler(new DeafultHandler(mContext.get()));
    }

    //连接方法（外部调用）
    public boolean connection(){
        try {
            ConnectFuture futrue = mConnection.connect();
            //一直连接，直至成功
            futrue.awaitUninterruptibly();
            mSession = futrue.getSession();
        }catch (Exception e){
            return false;
        }

        return mSession == null ? false : true;
    }

    //断开连接方法（外部调用）
    public void disConnect(){
        //关闭
        mConnection.dispose();
        //大对象置空
        mConnection = null;
        mSession = null;
        mAddress = null;
        mContext = null;

    }


    //内部类实现事物处理
    private class DeafultHandler extends IoHandlerAdapter {

        private Context mContext;

        DeafultHandler(Context context){

            this.mContext = context;

        }

        @Override
        public void sessionCreated(IoSession session) throws Exception {


        }

        @Override
        public void sessionOpened(IoSession session) throws Exception {
            //将我们的session 保存到我们sessionManager 中，从而可以发送消息到服务器
        }

        @Override
        public void sessionClosed(IoSession session) throws Exception {
            super.sessionClosed(session);
        }

        @Override
        public void messageReceived(IoSession session, Object message) throws Exception {
            //1,EventBus 来进行事件通知
            //2,广播
            if (mContext!=null){
                Intent  intent = new Intent(BROADCAST_ACTION);
                intent .putExtra(MESSAGE,message.toString());
                //使用局部广播，保证安全性
                LocalBroadcastManager.getInstance(mContext).sendBroadcast(intent);
            }
        }

        @Override
        public void messageSent(IoSession session, Object message) throws Exception {
            super.messageSent(session, message);
        }
    }
}


3，service

package com.ij.minamanager;
import android.app.Service;
import android.content.Context;
import android.content.Intent;
import android.os.HandlerThread;
import android.os.IBinder;
import android.support.annotation.Nullable;

/**
 * ┏┓　　┏┓+ +
 * 　　　　　　　┏┛┻━━━┛┻┓ + +
 * 　　　　　　　┃　　　　　　　┃
 * 　　　　　　　┃　　　━　　　┃ ++ + + +
 * 　　　　　　 ████━████ ┃+
 * 　　　　　　　┃　　　　　　　┃ +
 * 　　　　　　　┃　　　┻　　　┃
 * iBoosJie.　 ┃　　　　　　　┃ + +
 * 　　　　　　　┗━┓　　　┏━┛
 * 　　　　　　　　　┃　　　┃
 * 　　　　　　　　　┃　　　┃ + + + +
 * 　　　　　　　　　┃　　　┃　　　　Code is far away from bug with the animal protecting
 * 　　　　　　　　　┃　　　┃ + 　　　　神兽保佑,代码无bug
 * 　　　　　　　　　┃　　　┃
 * 　　　　　　　　　┃　　　┃　　+
 * 　　　　　　　　　┃　 　　┗━━━┓ + +
 * 　　　　　　　　　┃ 　　　　　　　┣┓
 * 　　　　　　　　　┃ 　　　　　　　┏┛
 * 　　　　　　　　　┗┓┓┏━┳┓┏┛ + + + +
 * 　　　　　　　　　　┃┫┫　┃┫┫
 * 　　　　　　　　　　┗┻┛　┗┻┛+ + + +
 * 创建人：杰
 * 创建日期： 2017/6/6
 * 说明：
 * 修改：
 */
public class MinaService extends Service{

    private ConnectionThread thread;

    @Override
    public void onCreate() {
        super.onCreate();
        //全局context 避免内存泄漏，不多说
        thread = new ConnectionThread("mina",getApplicationContext());
        thread.start();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        return super.onStartCommand(intent, flags, startId);
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        thread.disConnection();
        thread = null;
    }


    //负责调用ConnectionManager类来完成与服务器连接
    class ConnectionThread extends HandlerThread{

        private Context context;

        boolean isConnection;

        ConnectionManager mManager;

        public ConnectionThread(String name,Context context) {
            super(name);
            this.context = context;

            ConnectionConfig config = new ConnectionConfig.Builder(context)
                    .setIp("127.0.0.1")
                    .setPort(9123)
                    .setReadBuilder(10240)
                    .setConnectionTimeout(10000).builder();
        }

        //run 开始连接我们的服务器
        @Override
        protected void onLooperPrepared() {
            super.onLooperPrepared();
            //死循环
            for (;;){
                isConnection = mManager.connection();
                if (isConnection){
                    break;
                }

                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }

        //断开连接
        public void disConnection(){

            mManager.disConnect();
        }
    }
}

代码已经写的很详细了，讲道理注释也是没有毛病。

源码连接：https://github.com/jiezongnewstar/MinaManager



