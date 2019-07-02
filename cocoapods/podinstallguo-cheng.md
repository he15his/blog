## 运行`pod install`命令 {#-pod-install-}

当运行`pod install`命令时会引发许多操作。要想深入了解这个命令执行的详细内容，可以在这个命令后面加上`--verbose`。

### 读取 Podfile 文件 {#-podfile-}

你是否对 Podfile 的语法格式感到奇怪过，那是因为这是用 Ruby 语言写的。相较而言，这要比现有的其他格式更加简单好用一些。

在安装期间，第一步是要弄清楚显示或隐式的声明了哪些第三方库。在加载 podspecs 过程中，CocoaPods 就建立了包括版本信息在内的所有的第三方库的列表。Podspecs 被存储在本地路径`~/.cocoapods`中。

#### 版本控制和冲突 {#-}

CocoaPods 使用[语义版本控制 - Semantic Versioning](http://semver.org/)命名约定来解决对版本的依赖。由于冲突解决系统建立在非重大变更的补丁版本之间，这使得解决依赖关系变得容易很多。例如，两个不同的 pods 依赖于 CocoaLumberjack 的两个版本，假设一个依赖于`2.3.1`，另一个依赖于`2.3.3`，此时冲突解决系统可以使用最新的版本`2.3.3`，因为这个可以向后与`2.3.1`兼容。

但这并不总是有效。有许多第三方库并不使用这样的约定，这让解决方案变得非常复杂。

当然，总会有一些冲突需要手动解决。如果一个库依赖于 CocoaLumberjack 的`1.2.5`，另外一个库则依赖于`2.3.1`，那么只有最终用户通过明确指定使用某个版本来解决冲突。

### 加载源文件 {#-}

CocoaPods 执行的下一步是加载源码。每个`.podspec`文件都包含一个源代码的索引，这些索引一般包裹一个 git 地址和 git tag。它们以 commit SHAs 的方式存储在`~/Library/Caches/CocoaPods`中。这个路径中文件的创建是由 Core gem 负责的。

CocoaPods 将依照`Podfile`、`.podspec`和缓存文件的信息将源文件下载到`Pods`目录中。

### 生成 Pods.xcodeproj {#-pods-xcodeproj}

每次`pod install`执行，如果检测到改动时，CocoaPods 会利用 Xcodeproj gem 组件对`Pods.xcodeproj`进行更新。如果该文件不存在，则用默认配置生成。否则，会将已有的配置项加载至内存中。

### 安装第三方库 {#-}

当 CocoaPods 往工程中添加一个第三方库时，不仅仅是添加代码这么简单，还会添加很多内容。由于每个第三方库有不同的 target，因此对于每个库，都会有几个文件需要添加，每个 target 都需要：

* 一个包含编译选项的
  `.xcconfig`
  文件
* 一个同时包含编译设置和 CocoaPods 默认配置的私有
  `.xcconfig`
  文件
* 一个编译所必须的
  `prefix.pch`
  文件
* 另一个编译必须的文件
  `dummy.m`

一旦每个 pod 的 target 完成了上面的内容，整个`Pods`target 就会被创建。这增加了相同文件的同时，还增加了另外几个文件。如果源码中包含有资源 bundle，将这个 bundle 添加至程序 target 的指令将被添加到`Pods-Resources.sh`文件中。还有一个名为`Pods-environment.h`的文件，文件中包含了一些宏，这些宏可以用来检查某个组件是否来自 pod。最后，将生成两个认可文件，一个是`plist`，另一个是`markdown`，这两个文件用于给最终用户查阅相关许可信息。

### 写入至磁盘 {#-}

直到现在，许多工作都是在内存中进行的。为了让这些成果能被重复利用，我们需要将所有的结果保存到一个文件中。所以`Pods.xcodeproj`文件被写入磁盘，另外两个非常重要的文件：`Podfile.lock`和`Manifest.lock`都将被写入磁盘。

#### Podfile.lock {#podfile-lock}

这是 CocoaPods 创建的最重要的文件之一。它记录了需要被安装的 pod 的每个已安装的版本。如果你想知道已安装的 pod 是哪个版本，可以查看这个文件。推荐将 Podfile.lock 文件加入到版本控制中，这有助于整个团队的一致性。

#### Manifest.lock {#manifest-lock}

这是每次运行`pod install`命令时创建的`Podfile.lock`文件的副本。如果你遇见过这样的错误`沙盒文件与 Podfile.lock 文件不同步 (The sandbox is not in sync with the Podfile.lock)`，这是因为 Manifest.lock 文件和`Podfile.lock`文件不一致所引起。由于`Pods`所在的目录并不总在版本控制之下，这样可以保证开发者运行 app 之前都能更新他们的 pods，否则 app 可能会 crash，或者在一些不太明显的地方编译失败。

### xcproj {#xcproj}

如果你已经依照我们的建议在系统上安装了[xcproj](https://github.com/0xced/xcproj)，它会对`Pods.xcodeproj`文件执行一下`touch`以将其转换成为旧的 ASCII plist 格式的文件。为什么要这么做呢？虽然在很久以前就不被其它软件支持了，但是 Xcode 仍然依赖于这种格式。如果没有 xcproj，你的`Pods.xcodeproj`文件将会以 XML 格式的 plist 文件存储，当你用 Xcode 打开它时，它会被改写，并造成大量的文件改动。

## 结果 {#-}

运行`pod install`命令的最终结果是许多文件被添加到你的工程和系统中。这个过程通常只需要几秒钟。当然没有 Cocoapods 这些事也都可以完成。只不过所花的时间就不仅仅是几秒而已了。

## 补充：持续集成 {#-}

CocoaPods 和持续集成在一起非常融洽。虽然持续集成很大程度上取决于你的项目配置，但 Cocoapods 依然能很容易地对项目进行编译。

### Pods 文件夹的版本控制 {#pods-}

如果 Pods 文件夹和里面的所有内容都在版本控制之中，那么你不需要做什么特别的工作，就能够持续集成。我们只需要给`.xcworkspace`选择一个正确的 scheme 即可。

### 不受版本控制的 Pods 文件夹 {#-pods-}

如果你的`Pods`文件夹不受版本控制，那么你需要做一些额外的步骤来保证持续集成的顺利进行。最起码，`Podfile`文件要放入版本控制之中。另外强烈建议将生成的`.xcworkspace`和`Podfile.lock`文件纳入版本控制，这样不仅简单方便，也能保证所使用 Pod 的版本是正确的。

一旦配置完毕，在持续集成中运行 CocoaPods 的关键就是确保每次编译之前都执行了`pod install`命令。在大多数系统中，例如 Jenkins 或 Travis，只需要定义一个编译步骤即可 \(实际上，Travis 会自动执行`pod install`命令\)。对于[Xcode Bots，在书写这篇文章时我们还没能找到非常流畅的方式](https://groups.google.com/d/msg/cocoapods/eYL8QB3XjyQ/10nmCRN8YxoJ)，不过我们正朝着解决方案努力，一旦成功，我们将会立即分享。

## 结束语 {#-}

CocoaPods 简化了 Objective-C 的开发流程，我们的目标是让第三方库更容易被发现和添加。了解 CocoaPods 的原理能让你做出更好的应用程序。我们沿着 CocoaPods 的整个执行过程，从载入 specs 文件和源代码、创建`.xcodeproj`文件和所有组件，到将所有文件写入磁盘。所以接下来，我们运行`pod install --verbose`，静静观察 CocoaPods 的魔力如何显现。

