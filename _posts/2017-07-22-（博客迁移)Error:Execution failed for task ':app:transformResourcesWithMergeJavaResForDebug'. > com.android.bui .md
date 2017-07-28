So I'm continuously receiving a gradle build error upon trying to run my project. I have searched for other solutions and some say that adding:

packagingOptions {
    exclude 'META-INF/NOTICE'
}

to my app's gradle.build would fix the issue, however, it doesn't. Here is what my app's gradle.build currently looks like.

apply plugin: 'com.android.application'
apply plugin: 'com.google.gms.google-services'

dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    compile 'com.google.android.gms:play-services:8.3.0'
    compile 'com.google.android.gms:play-services-ads:8.3.0'
    compile 'com.google.android.gms:play-services-identity:8.3.0'
    compile 'com.firebase:firebase-client-android:2.3.1'
}

android {
    compileSdkVersion 23
    buildToolsVersion '23.0.1'
    useLibrary 'org.apache.http.legacy'

defaultConfig {
    applicationId "com.example.j.airportmeet"
    minSdkVersion 18
    targetSdkVersion 23
    versionCode 1
    versionName "1.0"
    multiDexEnabled true
}

packagingOptions {
    exclude 'META-INF/NOTICE'
}

buildTypes {
    release {
        minifyEnabled false
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
  }
}

EDIT: The full error is this:

Error:Execution failed for task ':app:transformResourcesWithMergeJavaResForDebug'.
> com.android.build.api.transform.TransformException: com.android.builder.packaging.DuplicateFileException: Duplicate files copied in APK META-INF/LICENSE
    File1: .gradle\caches\modules-2\files-2.1\com.fasterxml.jackson.core\jackson-databind\2.2.2\3c8f6018eaa72d43b261181e801e6f8676c16ef6\jackson-databind-2.2.2.jar
    File2: .gradle\caches\modules-2\files-2.1\com.fasterxml.jackson.core\jackson-core\2.2.2\d20be6a5ddd6f8cfd36ebf6dea329873a1c41f1b\jackson-core-2.2.2.jar

    File3: .gradle\caches\modules-2\files-2.1\com.fasterxml.jackson.core\jackson-annotations\


解决方法：

As the error message suggests you are not excluding the META-INF/LICENSE file which exists in more than one of your dependencies

    com.Android.build.api.transform.TransformException: com.android.builder.packaging.DuplicateFileException: Duplicate files copied in APK META-INF/LICENSE

packagingOptions {
    exclude 'META-INF/NOTICE' // will not include NOTICE file
    exclude 'META-INF/LICENSE' // will not include LICENSE file
}
