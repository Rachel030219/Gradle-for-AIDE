2. Gradle构造的使用
-------
### 2: *.gradle配置文件  
关于这个配置文件，有三个：

- /Project/build.gradle
- /Project/settings.gradle
- /Project/Module/build.gradle

我们一个一个以上文的material-ripple为例子来解析。 
> 另附：在这些配置文件中，字符串需要用 `' '` 也就是单引号括起来而不是双引号，这点要注意。

**** 
#### /Project/build.gradle

这是一个作用于整个Project的配置文件。完全内容如下：
```groovy
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.0.0'
    }
}

allprojects {
    repositories {
        jcenter()
    }
}
```
第一句是官方的注释，意思大概是这是一个可以对所有的子工程/Module生效的顶部配置文件(英语渣，将就看看吧)。  
 `buildscipt{...}` 内，是工程编译时使用的配置， `repositories{...}` 内放有编译使用的依赖仓库，以前的版本是mavenCentral()，新的版本中Google弃用了它转而使用更人性化，上传更容易的jcenter()，如果在mavenCentral()上没有但又写清楚能够下载的，一般在jcenter()上都能找到，除非是作者自己搭建的仓库，届时在这里加上就可以了，记得要与下面的一起改。 `dependencies{...}` 内指定了编译的版本，当然在AIDE上更改没有影响。  
 `allprojects{...}` 中，是对所有子工程/Module的设置， `repositories{...}` 就是所有工程使用的依赖仓库了，要和上面的一起改，不然打包时也许会出现问题。  
****
#### /Project/settings.gradle

这个就非常简单了，见下面代码：
```groovy
include ':library', ':demo'
```
只有一句， `include ':Module'` ，指定打包时需要打包的Module。  
  
****
#### /Project/Module/build.gradle

这个文件内，装着每个Module自己的配置，具体内容如下。  
```groovy
apply plugin: 'com.android.application'

android {
    compileSdkVersion 21
    buildToolsVersion "21.1.2"

    defaultConfig {
        applicationId "com.balysv.materialripple.demo"
        minSdkVersion 14
        targetSdkVersion 21
        versionCode 1
        versionName "1.0"
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_7
        targetCompatibility JavaVersion.VERSION_1_7
    }
}

dependencies {
    compile project(':library')
    compile 'com.android.support:appcompat-v7:22.0.0'
    compile 'com.android.support:recyclerview-v7:22.0.0'
}
```
我们来分析分析。  
第一行的 `apply plugin` 指定了Module的类型。 `com.android.application` 指的是可以安装的应用， `com.android.library` 指的是依赖的库。这两者必须严格区分开来，应用不能作为库导入，库不能被编译运行，且如果同时指定两个，会不能编译且报错。  
当 `apply plugin` 指定为 `com.android.*` 时， `android{...}` 节点就可以添加到 `build.gradle` 中，它指定了应用的基本信息，和Manifest中指定的是一样的，只不过现在需要指定两个。 `compileSdkVersion` 右侧指明了编译时使用的SDK版本，在AIDE上没有要求，但是还有一些特殊的要求，这里暂且不提。 `buildToolsVersion` 右侧是字符串，就是编译使用的具体版本，前面的大版本一定要和 `compileSdkVersion` 相同。另外记住，需要使用单引号，当然你用双引号也没有什么问题。。。  
 `defaultConfig{...}` 中，就是关于低版本的适配了。 `applicationId` 包含了应用的包名， `minSdkVersion` 是最低适配的API版本， `targetSdkVersion` 是调试的API版本，当然也可以添加 `maxSdkVersion` 。 `versionCode` 和 `versionName` 不用多说了。
> 特别注明：对于 `compileSdkVersion` 、 `targetSdkVersion` 、 `minSdkVersion` ，建议的大小关系是： `minSdkVersion < targetSdkVersion <= compileSdkVersion` 。另外，在6.0 Marshmallow上，应用需要请求运行时权限，如果还没有做好适配，建议不要把 `targetSdkVersion` 设置到23。知名博主 技术小黑屋 有过关于这方面的文章，这里由于篇幅问题，不能全部说明，那篇文章几乎是想要适配6.0的程序猿的必读了。链接将在文末放出。  

 `compileOptions{...}` 中，大概是Java的版本，AIDE不用多考虑，一开始新建工程就可以自动指明。  
那么对于 `android{...}` 的分析就到这里。下面是 `dependencies{...}` 依赖，将在下一节讲明。  

> [ 聊一聊Android 6.0的运行时权限 ](http://droidyue.com/blog/2016/01/17/understanding-marshmallow-runtime-permission/)
