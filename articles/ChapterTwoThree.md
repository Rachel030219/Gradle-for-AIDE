2. Gradle构造的使用
-------
### 3: 在Gradle构造中使用依赖  
  
这一节，是Gradle构造学习的重点，也是Gradle构造与Eclipse构造最大的不同之处－依赖库的引入。在原来我们使用的Eclipse构造中，依赖库都是在 `project.properties` 文件中声明，而依赖的jar则放在 `libs` 文件夹。但Gradle构造完全改变了这些，你需要更复杂地引入依赖库和jar，但它减少了依赖库冲突的可能。而且，AndroidStudio使你能够把工程打包为aar，可以在引入工程时同时引入资源文件，且在向各个网站上传自己的依赖库时，会大大减少工程量。
> 当然在AIDE上这并没有什么~~卵~~用  

#### <导入本地的依赖库>
本地的依赖分两种，工程库和jar库。  
jar库可以使用两种方法导入，单文件导入和文件夹导入。单文件导入适合于调试的场景，如果没有这种需求建议使用文件夹导入。  
- 单文件导入

需要在Module的 `build.gradle` 文件的 `dependencies{...}` 中进行添加。  
例如，我现在想引用一个在 `libs` 的文件夹内的名为 `mylib.jar` 的库文件，则可以输入以下代码：
```groovy
...
dependencies{
    compile files('libs/mylib.jar')
}
...
```
仅仅一句代码。 `libs` 是文件夹， `mylib.jar` 是文件名。如果你想在一个文件夹内嵌套一个文件夹，也可以很简单的实现。    
这一句代码，指定了这个 `mylib.jar` 文件的引用。然后一切都不用你操心，AIDE会自动到Module根目录下的 `libs` 文件夹里找到这个库文件并将其导入到工程。非常简单，不是吗？    
- 文件夹导入

可能有些人会说，单文件导入的效率实在是太低了，每一个jar库都要一个一个的来输入，这时就需要用到文件夹导入了。  
例如，我在 `somelibs` 文件夹中，同时放了好几个库文件，比如放了 `Nine Old Androids` ， `Support V4` 等等很多，这时一个一个导入的方法有点太过死板，就需要用到文件夹导入了：
```groovy
...
dependencies{
    compile fileTree(dir: 'somelibs', include: '*.jar')
}
...
```
就像上面的一样，简简单单的一句代码。 `lib` 是文件夹名，而后面则指定了文件的种类。AIDE不会管你到底想导入哪一个库，它只管老老实实的把你放进文件夹的所有库都导入。  
> 另外，文件夹名字是可以自定义的，你想取什么都可以，"leibusi"，"wuyaowang"什么的都没关系啦，你甚至还可以分类管理，比如微信SDK的就放进"wechat"，友盟的SDK就放进"umeng"等等
******
而工程库的导入，就只有一种方法了：作为一个Module，在应用的主Module中导入。这种导入方法，要求工程库必须要使用 `com.android.library` 插件。  
> 顺带一提，不管是Eclipse构造还是Gradle构造都是不能够直接导入aar文件的，只能先用压缩软件解压之后才能作为工程导入。  

这种导入方法，需要保证主Module和库在同一个根目录下，例如，如果有一个主Module叫 `main` ，一个库叫 `lib` ，那么就应该是这样的： 
- main
  - Module内容
  - build.gradle
- lib
  - Module内容
  - build.gradle
- build.gradle
- settings.gradle
- 其它文件

这里注意一定要把库在settings.gradle中声明。  
好了，现在我们已经有了一个库Module和主Module。接下来的问题就是如何引用这个库了。其实也很简单，在主Module的 `build.gradle` 内，就这么写：
```groovy
...
dependencies{
    compile project(':lib')
}
```
对，你没看错，就是这么简单的一句代码。但是有些操蛋的问题就是，AIDE有时会提示说依赖库不存在。我在这里给出一点点方法，不一定可行。  
这种情况首先需要检查插件，一定要保证是 `com.android.library` ，然后需要保证 `compileSdkVersion` 相同，库的 `minSdkVersion` 一定要低于主Module的 `minSdkVersion` ，大概就是这么点问题吧。  
> 这一点非常重要！在之后的远程库依赖的教程中，也需要保证这几点，所以在导入远程库的时候，一定要仔细看清楚开发者给的这些看上去没有什么用的东西，往往就是这些决定了是否可以兼容更低版本的Android设备。

******
#### <导入远程的依赖库>
远程的依赖，不管是jar还是工程库，都是一样的方法：直接 `compile` 。  
现在我们来导入一个~~操蛋的~~远程库。以上一章的 `material-ripple` 为例，来解析解析这一个库的具体导入过程。  
假设现在你想在你的应用中加入 `Ripple Effect` ，但是受够了那个需要自己单独设置的可以兼容到2.3的库，所以我们选择了这一个库。现在我们仍在使用 `mavenCentral()` ，所以还有很多地方需要更改。
- 第一步：修改根目录的 `build.gradle` 文件。

如果你想愉快地享用Gradle的话，一定要记得自己改 `build.gradle` 。如果网络依赖库找不到的话，就看看这个吧。虽然一开始默认的已经是 `jcenter()` ，但还是强烈推荐自己检查一遍。在 `repositories{...}` 中，把 `mavenCentral()` 改为 `jcenter()` ，这个仓库有着更多的依赖库和更低的门槛，所以就用这个吧。
```groovy
...
buildScripts{
    ...
    repositories{
        jcenter()
    }
    ...
}
...
allprojects{
...
    repositories{
        jcenter()
    }
...
}
```
大体就是这样，上一章也提到过。当然，如果你是企业级的开发者，也会有一个企业自己的仓库，但是一般都会提供教程，这里不再赘述。   
> 话说，企业级的开发者会看这个嘛。。。 

- 第二步：修改Module的 `build.gradle` 
很简单的一件事情。假设我们现在还在用17的SDK来 `compile` ，最低是Android 2.3(API Level 9)，但是这个依赖库却要求用21的SDK和最低14，那么我们的 `build.gradle` 就必须是这样：
```groovy
...
android{
    ...
    compileSdkVersion 21
    ...
    defaultConfig{
        ...
        minSdkVersion 14(或更高)
        ...
    }
    ...
}
...
dependencies{
    compile 'com.balysv:material-ripple:1.0.2'
}
```
这样，一个远程库就导入进工程了。保存后，AIDE会自动下载这个远程库，这样你就可以使用这个远程库了。而且，不管是jar还是工程，都能够顺利导入，只要作者发布在了这些仓库中。
******
好了，关于库的依赖差不多就是这样。 

 `Rachel暂时不在家哦，期待回家更新吧` 
