React Native 打包发布

    http://localhost:8081/index.android.bundle?platform=android;当应用启动运行的时候，会自动拉取这个bundle文件，该文件里存放的是全部的逻辑代码，在目录中并不存在这个文件，事实上，这个地址只是一个请求地址，而并非真正的静态资源文件，是通过包服务器packager通过动态分析index.android.js中的依赖，并对其进行合并得到的，而且该服务器允许代码实时渲染。

生成签名秘钥

    通过工具生成，例如Android studio
    keytool生成
    在控制台输入：keytool -genkey -v -keystore "自定义key名字（英文）".keystore -alias "自定义别名(英文)" -keyalg RSA -keysize 2048 -validity 10000
    剩下配置一下基本的信息即可，注：签名文件输出路径在当前操作路径

找到路径/android/app/src/main,并在该目录下新建assets文件夹
下载 crul 工具 (windows用户需要下载，mac不需要)

    这里针对windows可以分两种情况进行配置
        创建全局的环境变量
        将下载好的crul.exe文件复制到工程目录中

命令行执行 curl -k "http://localhost:8081/index.android.bundle" > android/app/src/main/assets/index.android.bundle
添加gradle的android keystore 配置

    在build.gradle中进行如下配置：
    signingConfigs{
    release{
    storeFile file("keystore 的绝对路径" 这里要将“\”替换成"/")
    storePassword "密码"
    keyAlias "别名
    keyPassword “密码”
    }

}


buidTypes{
release{
minifyEnabled false
proguardFiles getDefaultProguardFile('proguard-android.txt'),'proguard-rules.pro'
signingConfig signingConfigs.release
}
}
启用Proguard代码混淆来缩小APK文件的大小

Proguard是一个Java字节码混淆压缩工具，它可以移除掉React Native Java（和它的依赖库）中没有被使用到的部分，最终有效的减少APK的大小。
·  重要：启用Proguard之后，你必须再次全面的测试你的应用，Proguard有时候需要为你引入每个原生库做一些额外的配置。参见app/proguard-rules.pro文件。
·def enableProguardInReleaseBuilds = true
在/android/目录中执行gradle assembleRelease命令，打包后的文件在android/app/bulid/outputs/apk目录中，例如app-release.apk。如果打包碰到问题可以先执行gradle clean 清理一下。

    安装grade工具(版本与android\gradle\wrapper下的一致)，并配置环境变量，配置GRADLE_HOMEDAO 到你的gradle根目录当中，然后把%GRADLE_HOME%/bin(linux或mac的是$GRADLE_HOME/bin)加到PATH的环境变量。 
    配置完成之后，运行gradle -v,检查一下是否安装成功 ### 将apk发布到各大应用市





