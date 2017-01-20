因为国内应用市场太多，我不推荐我使用的方法。

所以先附上比较适合国内开发者的方法。


（美团老版多渠道打包）

http://tech.meituan.com/mt-apk-packaging.html

（美团新版多渠道打包）

http://tech.meituan.com/android-apk-v2-signature-scheme.html

新版本主要是针对Android 7.0(Nougat)推出了新的应用签名方案APK Signature Scheme v2。

附上Github地址

https://github.com/Meituan-Dianping/walle


------------------分割线-------------------

因为现在暂时只面向Google Play就行，所以只考虑最简单的build版本。

## 详细步骤

### 修改app/build.gradle文件

附上app/build.gradle文件，详细看注释

```
apply plugin: 'com.android.application'

def releaseTime() {
    return new Date().format("yyyyMMdd", TimeZone.getTimeZone("GMT+8"))
}
android {
    compileSdkVersion 25
    buildToolsVersion "25.0.2"
    defaultConfig {
        applicationId "com.foxmail.aroundme.labeltextview"
        minSdkVersion 14
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    //秘钥的位置，别名和密码，推荐保存到别处
    signingConfigs {
        release {
            storeFile file("../gzl.jks")
            storePassword KEYSTORE_PASSWORD
            keyAlias "gzl"
            keyPassword KEY_PASSWORD
        }
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
        }
        debug {
//            minifyEnabled true
//            shrinkResources true
//            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'

        }
    }
    //多渠道种类，只是一个示例
    productFlavors {
        //豌豆荚
        wandoujia {
            //AndroidManifest.xml中定义的meta-data标签,
            // 需要和meta-data中的android:value="${CHANNEL}对应
            manifestPlaceholders = [CHANNEL: "wandoujia"]
            //Build中生成一个字符用来区分不同渠道的不同策略
            buildConfigField "String", "CHANNEL", "\"wandoujia\""
        }
        //百度
        baidu {
            manifestPlaceholders = [CHANNEL: "baidu"]
            buildConfigField "String", "CHANNEL", "\"baidu\""
        }
    }
    //重命名apk文件
    def String apkName = "Alpha" + "_" + defaultConfig.versionName + "_" + defaultConfig.versionCode + "_dev_" + releaseTime() as String

    applicationVariants.all { variant ->

        if (variant.buildType.name.equals('release') || variant.buildType.name.equals('debug')) {
            variant.outputs.each { output ->
                def appName = 'Label'
                def oldFile = output.outputFile
                def buildName = ''

                variant.productFlavors.each { product ->
                    //循环取到的是productFlavors的每个名字，也是区分名字的关键
                    buildName = product.name
                }

                def releaseApkName = appName+ "_" + buildName + "_${apkName}.apk"
                output.outputFile = new File(oldFile.parent, releaseApkName)
            }
        }
    }


}

dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:25.0.1'
    testCompile 'junit:junit:4.12'
    compile project(':library')
}


```

### 修改
然后在AndroidManifest中添加

```

<meta-data
        android:name="UMENG_CHANNEL"
        android:value="${CHANNEL}" />

<application

    .......
/>
```


编译完后在BuildConfig中应该有一个字段为CHANNEL，
对应在productFlavors的wandoujia的语句：

buildConfigField "String", "CHANNEL", "\"wandoujia\""

(注意要有转义字符\,不然会认为是变量)


### 修改应用内代码

Java代码中添加

```

 if(BuildConfig.CHANNEL.equals("wandoujia")) {
            Log.d("msg", "wandoujia");
            Toast.makeText(MainActivity.this, "wandoujia", Toast.LENGTH_LONG).show();
        } else if (BuildConfig.CHANNEL.equals("baidu")) {
            Log.d("msg", "baidu");
            Toast.makeText(MainActivity.this, "baidu", Toast.LENGTH_LONG).show();
        }


```

不同渠道Apk可以采取这个策略来实现不同的操作

完美。
