---
title: React Native Build Apk
date: 2016-08-21 21:31:06
tags:
---
### 1 React Native安卓项目打包APK

#### 1.1 产生签名的key
先通过keytool生成key
```bash
keytool -genkey -v -keystore demo-release-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 20000
```
将生成的key启动到项目android/app目录下面
```bash
mv demo-release-key.keystore android/app/
```

#### 1.2 修改android/gradle.properties文件，增加如下
```bash
MYAPP_RELEASE_STORE_FILE=my-release-key.keystore
MYAPP_RELEASE_KEY_ALIAS=my-key-alias
MYAPP_RELEASE_STORE_PASSWORD=******
MYAPP_RELEASE_KEY_PASSWORD=******
```
其中******为Key设置的密钥和存储密码

#### 1.3 修改android/app/build.gradle文件中的签名配置
```bash
...
android {
  ...
  defaultConfig {
    ...
  }
  signingConfigs {
    release {
        storeFile file(MYAPP_RELEASE_STORE_FILE)
        storePassword MYAPP_RELEASE_STORE_PASSWORD
        keyAlias MYAPP_RELEASE_KEY_ALIAS
        keyPassword MYAPP_RELEASE_KEY_PASSWORD
    }
  }
  buildTypes {
    release {
      ...
      signingConfig signingConfigs.release
    }
  }
}
```

#### 1.4 然后进入android目录执行如下
```bash
./gradlew assembleRelease
```
结束后会生成apk文件在项目相关路径下面
```bash
android/app/build/outputs/apk/app-release.apk
```

> 每次执行前，注意将该apk文件删除

> 提示：如果你需要对apk进行混淆打包 编辑android/app/build.gradle：
```bash
/**     
 * Run Proguard to shrink the Java bytecode in release builds.  
 */  
def enableProguardInReleaseBuilds = true
```
