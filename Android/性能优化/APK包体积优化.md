### 包体积优化

1. 使用`webp`格式图片代替`png`、或者`.jgp`,`webp基本是无损压缩`
2. 减少无用的资源文件，使用 Lint 检查无用的资源文件
3. 去除不必要的 so 库。例如只支持 64 位平台，只保留库的 arm64-v8a 和 x86_64 的 so 库
4. 使用`ProGuard`进行代码混淆和资源压缩，减少代码和资源的体积
5. 使用`R8`进行代码压缩，减少代码的体积
6. 使用`shrinkResources`进行资源压缩，减少资源的体积
7. 使用`minifyEnabled`进行代码混淆，减少代码的体积
8. 使用 AndGuard 进行资源文件的压缩
9. 合理的设计避免重复的类和不必要的资源文件

### Apk 的打包流程

```bash
//aidl 转换aidl文件为java文件
> Task :app:compileDebugAidl

//生成BuildConfig文件
> Task :app:generateDebugBuildConfig

//获取gradle中配置的资源文件
> Task :app:generateDebugResValues

// merge资源文件，AAPT2 编译阶段
> Task :app:mergeDebugResources

// merge assets文件
> Task :app:mergeDebugAssets
> Task :app:compressDebugAssets

// merge所有的manifest文件
> Task :app:processDebugManifest

//生成R文件 AAPT2 链接阶段
> Task :app:processDebugResources

//编译kotlin文件
> Task :app:compileDebugKotlin

//javac 编译java文件
> Task :app:compileDebugJavaWithJavac

//转换class文件为dex文件
> Task :app:dexBuilderDebug

//打包成apk并签名
> Task :app:packageDebug
```

<img src="https://mmbiz.qpic.cn/mmbiz_png/v1LbPPWiaSt5ibH61k2aSyMGBcayEqf6GD69cxYhPvt9V1JoqeXHhiaoqGxU6TictAHCD9UNM7qJ1icw76dBwrICMnw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1">

### Apk 的构成

1. resources.arsc：包含了所有资源文件的映射，可以理解为索引，通过该文件能找到对应的资源文件信息

2. AndroidManifest.xml：Project 中 AndroidManifest.xml 编译后得到的二进制 xml 文件

3. META-INF：主要保存各个资源文件的 SHA1 hash 值，用于校验资源文件是否被篡改，防止二次打包时资源文件被替换，该目录下主要包括下面三个文件：

- CERT.RSA：保存签名和公钥证书
- MANIFEST.MF：保存版本号以及对每个文件（包括资源文件）整体的 SHA1 hash
- CERT.SF：保存对每个文件头 3 行的 SHA1 hash
- res：Project 中 res 目录下资源文件编译后得到的二进制 xml 文件

3. classes.dex：Dex 是 DalvikVM executes 的缩写，即 Android Dalvik 执行程序

4. lib：对应 Project 中的 libs 目录，包含.so 文件。
