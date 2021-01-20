# 秀秀视频编辑组件化-Swift和Objective-C混编

------



## 1.背景

秀秀视频编辑组件化之路提及已久，为了将视频编辑接入美拍并替换其视频编辑模块，需要先将视频编辑组件化并以Pod的形式接入秀秀主工程。其中使用cocoapod管理的过程中，遇到部分Swift和Objective-C混编问题，包括主工程调用编辑Pod的代码(主工程oc调用编辑Pod内swift，主工程swift调用编辑Pod内oc)，编辑Pod内Swift和Objective-C的相互调用，编辑Pod调用其他Pod代码(编辑Pod内oc调用其他Pod内swift，编辑Pod内swift调用其他Pod内oc)。

## 2. Swift 和 Objective-C 混编场景

Swift 与 Objective-C 混编主要有两个使用场景。

* Swift 引用 Objective-C
  * 桥接方式。在Swift与Objective-C的桥接文件 ProductName-Bridging-Header.h 中，添加 #import "xxx.h" 即可实现 在 Swift 中 引用 xxx类的属性与方法。
  * Module 引入方式。

> 桥接方式中，需要在project里找到对应的 **[target](https://developer.apple.com/library/archive/featuredarticles/XcodeConcepts/Concept-Targets.html)** ，Build Setting 里找到 Objective-C Bridging Header，设置为 ProductName-Bridging-Header.h 所在路径（一般系统会创建 Objective-C 文件的时候提示创建 ProductName-Bridging-Header.h 并自动设置）。但是当在Framework中设置 Objective-C Bridging Header 的时候，编译报错"using bridging headers with framework targets is unsupported" ，Xcode 不支持在 Framework 中使用 Bridging Header。

* Objective-C 引用 Swift
  * Swift.h 引用方式。在 Objective-C 类中导入 ProductName-Swift.h，可使得 Objective-C 类拥有引用 swift 类的能力。
  * Module 引入方式。

> 桥接方式中，需要在project里找到对应的 **[target](https://developer.apple.com/library/archive/featuredarticles/XcodeConcepts/Concept-Targets.html)** ，Build Setting 里找到 Objective-C Generated Interface Header Name，设置为 ProductName-Swift.h 所在路径（一般系统会自动设置）。Swift的Framework中同样如此。
>
> ProductName-Swift.h只针对 ProductName 内部有效。否则 targetA 引用 targetB 中的 targetBName-Swift.h，提示fatal error: 'targetBName-Swift.h' file not found。

所有使用场景如下：

### 2.1 Objective-C 引用 Swift

参考Apple文档 [Importing Swift into Objective-C](https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/importing_swift_into_objective-c)

由于Swift的引用权限限制等问题。Swift类需要同时满足

* 被明确标记@objc，@IBAction，@OBOutlet 关键字。
* 访问权限

#### 2.1.1 主工程 Objective-C 引用 主工程 Swift

在 Objective-C 类中引入 MTXX-Swift.h，即可引用 Swift 中暴露给 Objective-C 的类和方法。

#### 2.1.2 主工程 Objective-C 引用Pod中 Swift

在 Objective-C 类中引入 PodName-Swift.h，提示fatal error: 'PodName-Swift.h' file not found。

#### 2.1.3 Pod 中 Objective-C 引用 Pod 中 Swift

在 Objective-C 类中引入 PodName-Swift.h，即可引用 Swift 中暴露给 Objective-C 的类和方法。

#### 2.1.4 PodA 中 Objective-C 引用 PodB 中 Swift

在 Objective-C 类中引入 其他 PodBName-Swift.h，提示fatal error: 'PodBName-Swift.h' file not found。

列表形式如下：

|                  描述                   |                           桥接方式                           |              Module 方式              |
| :-------------------------------------: | :----------------------------------------------------------: | :-----------------------------------: |
|   主工程 Objective-C 引用主工程 Swift   |           Objective-C 类中，#import "MTXX-Swift.h"           |                 缺省                  |
|   主工程 Objective-C 引用Pod中 Swift    | Objective-C 类 #import "PodName-Swift.h"，提示fatal error: 'PodName-Swift.h' file not found。 | 在 Objective-C 类中 #import PodName;  |
|  Pod 中 Objective-C 引用 Pod 中 Swift   |          Objective-C 类中 #import "PodName-Swift.h"          | 在 Objective-C 类中 #import PodName;  |
| PodA 中 Objective-C 引用  PodB 中 Swift | Objective-C 类 #import "PodNameB-Swift.h"，提示fatal error: 'PodBName-Swift.h' file not found。 | 在 Objective-C 类中 #import PodBName; |



### 2.2 Swift 引用 Objective-C

参考Apple文档 [Importing Objective-C into Swift](https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/importing_objective-c_into_swift)

#### 2.2.1 主工程 Swift 引用主工程 Objective-C

只需要在主工程桥接文件中（MTXX-Bridging-Header.h）中导入 Swift 模块要引用的 Objective-C 类头文件，即可在 Swift 中引用相应 Objective-C 的类和方法。

```c
#import "xxx.h"
```

#### 2.2.2 主工程 Swift 引用 Pod 中 Objective-C

桥接方式，在桥接文件中（MTXX-Bridging-Header.h）中导入 Swift 模块要引用Pod中的 Objective-C 类头文件，即可在 Swift 中引用相应 Objective-C 的类和方法。

```c
#import <PodName/xxx.h>
```

#### 2.2.3 Pod 中 Swift 引用 Pod 中 Objective-C

采用 Bridging-Header 方案，Xcode 编译报错 "using bridging headers with framework targets is unsupported"，编译器不支持在framework中使用 Bridging Header。

#### 2.2.4 PodA 中 Swift 引用 PodB 中 Objective-C

采用 Bridging-Header 方案，Xcode 编译报错 "using bridging headers with framework targets is unsupported"，编译器不支持在framework中使用 Bridging Header。

列表形式如下：

|                 描述                  |                           桥接方式                           |           Module 方式           |
| :-----------------------------------: | :----------------------------------------------------------: | :-----------------------------: |
|  主工程 Swift 引用主工程 Objective-C  |          MTXX-Bridging-Header.h 中，#import "xxx.h"          |              缺省               |
|  主工程 Swift 引用Pod中 Objective-C   |      MTXX-Bridging-Header.h 中 #import <PodName/xxx.h>       | 在 Swift 文件中 import PodName  |
| Pod 中 Swift 引用 PodB 中 Objective-C | PodName-Bridging-Header.h 中，#import "xxx.h"。<br /> error: using bridging headers with framework targets is unsupported | 在 Swift 文件中 import PodBName |
| PodA 中 Swift 引用PodB 中 Objective-C | PodAName-Bridging-Header.h 中 #import <PodBName/xxx.h>。<br />提示error: using bridging headers with framework targets is unsupported | 在 Swift 文件中 import PodBName |



由此可见，在 Pod 中如果存在 Swift 库引用 Objective-C，那么只能采用 Module 方式。

## 3. Module 系统

Module 定义: A module is a package describing a library

### 3.1 头文件引用

* #include，传统的 c语言 头文件引用方式，在编译过程中的编译预预处理阶段（Pre-Processor），预编译器 处理“#include”预编译指令，将被包含的文件插入到该预编译指令的位置。注意，这个过程是递归进行的，也就是说被包含的文件可能还包含其他文件。

* #import，Objective-C语言 引入的更加智能的头文件引用方式。同一个文件只会被引入一次。避免了头文件的递归引入问题。

> If you are working in Objective-C, you may use the #import directive instead of the #include directive. The two directives have the same basic results. but the #import directive guarantees that the same header file is never included more than once. [链接](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPFrameworks/Tasks/IncludingFrameworks.html)

### 3.2 LLVM Module

Apple 2012 年 11 月引入 LLVM 的 Module 系统（[链接](https://llvm.org/devmtg/2012-11/)），以解决宏污染以及include-ordering问题。# import Module 与 #include 有很大不同：当编译器遇到Module导入时，直接以二进制形式加载Module，并将其API提供给应用程序。import声明之前的预处理器定义不会对所提供的API产生任何影响，因为模块本身已被编译为单独，独立模块。

使用umbrella结构化的头文件描述，来取代以往的扁平化的头文件 。直接的变更表现形式就是传统的`#include <stdio.h>`  现在变成 `import std.io` 。Module[文档](http://clang.llvm.org/docs/Modules.html)

这样做的主要意义是：

- 提高编译时的可扩展性，同一 Module 只需编译，导入一次，避免了头文件的多次引用、解析。
- 减少碎片化，每个模块只处理一次，环境的变化不会导致不一致
- Module 描述了软件库的API，可以依靠 Module 定义来确保获取库的完整API，Module 可以指定使用的语言。语义上更加清晰地描述了一个Framework的作用。

### 3.3 Module 组织结构

##### 3.3.1 Module Maps

Module 与 Header 之间的关键联系在被描述在 module map 里，module map 描述了 Header 映射到 Module的逻辑结构形式。例如，可以想象一个`std`涵盖C标准库的模块。每个C标准库头的（`<stdio.h>`，`<stdlib.h>`，`<math.h>`等）将有助于`std`模块，通过将其各自的API为相应的子模块（`std.io`，`std.lib`，`std.math`等等）。拥有作为`std`模块一部分的标头列表，可使编译器将`std`模块构建为独立实体，并具有从标头名称到（子）模块的映射，从而可以将`#include`指令自动转换为模块导入。

模块映射在`module.modulemap`它们描述的标头旁边被指定为单独的文件（每个命名为），这使它们可以添加到现有软件库中，而不必更改库标头本身（在大多数情况下[[2\]](http://clang.llvm.org/docs/Modules.html#id6)）。[链接](http://clang.llvm.org/docs/Modules.html#id16)

```
// /usr/include/module.map
module std {
  module stdio { header “stdio.h” }
  module stdlib { header “stdlib.h” }
  module math { header “math.h” }
}
```

##### 3.3.2 Umbrella Headers

```
// clang/include/clang/module.map
module ClangAST {
  umbrella header “AST/AST.h”
	module * { } 
}
```

### 3.4 Apple Module 实现

Apple的设计中 Module系统被拆分成 .modulemap 和 umbrella.h 文件。其中 umbrella.h 类似于 include 方式中暴露出来的头文件。.modulemap 描述组织结构等feature。

TODO

## 4. Framework

Framework是分层目录，它在单个程序包中封装共享资源，例如动态共享库，nib文件，图像文件，本地化字符串，头文件和参考文档。多个应用程序可以同时使用所有这些资源。系统会根据需要将它们加载到内存中，并在所有可能的应用程序之间共享资源的一份副本。[链接](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPFrameworks/Concepts/WhatAreFrameworks.html#//apple_ref/doc/uid/20002303-BBCEIJFI)

### 4.1 Standard Frameworks

### 4.2 Umbrella Frameworks

开启了 Module 的Framework。

> The structure of an umbrella framework is similar to that of a standard framework, and applications do not distinguish between umbrella frameworks and standard frameworks when linking to them. 

> However, two factors distinguish umbrella frameworks from other frameworks. The first is the manner in which they include header files. The second is the fact that they encapsulate subframeworks

## 5. 实际 Pod 引入Module

### 5.1 Headers 与 Module 的 性能分析

> If you are worried that including a master header file may cause your program to bloat, don’t worry. Because OS X interfaces are implemented using frameworks, the code for those interfaces resides in a dynamic shared library and not in your executable. In addition, only the code used by your program is ever loaded into memory at runtime, so your in-memory footprint similarly stays small.

> As for including a large number of header files during compilation, once again, don’t worry. Xcode provides a precompiled header facility to speed up compile times. By compiling all the framework headers at once, there is no need to recompile the headers unless you add a new framework. In the meantime, you can use any interface from the included frameworks with little or no performance penalty.

综上，Module 替换 Headers 是可行的。

### 5.2 开启 Module

Pod 工程开启 module，在不同的Cocoapods版本之下需要不同的配置。Cocoapods 1.5 之前的版本不支持针对pod单独开启module且不支持swift static库开启 module，需要使用 use_frameworks!，cocoapods 会把所有库重新包装为 dynamic 库，这样会导致所有依赖库在编译成功打包之后会出现在 .App目录下的FrameWork目录下。Cocoapods 1.5 以及之后的版本，可以使用use_modular_headers! 来将所有显示依赖的 pod 开启 module，也可以在每个pod之后添加 “, :modular_headers => true”，对单个pod开启 module。（[链接](https://blog.cocoapods.org/CocoaPods-1.5.0/)）

那么在组件化场景中，MTVideoEdit 库是以源码形式引入的 swift 静态库，为了在 MTVideoEdit 内部实现 Swift 引用 Objective-C，需要对MTVideoEdit开启 module，即

```
pod 'MTVideoEdit', :path => '../../mtxx/MTXX/', :modular_headers => true
```

同时官方文档2.1中需要在Build Setting中将 Defines Module 设置为 YES，需要在MTVideoEdit.podspec中添加配置

```
spec.pod_target_xcconfig = { 'DEFINES_MODULE' => 'YES' }
```

设置完之后开始编译，但是出现编译问题。

```
spec.user_target_xcconfig // 可以修改主工程的Build Setting配置。
```

#### 5.2.1 问题1，MTVideoEdit中存在swift调用第三方库 MTPhotoLibrary

原本的方案是在 MTVideoEdit-umbrella.h 与 MTXX-Bridging-Header.h 中添加 #import <MTPhotoLibrary/MTPhotoLibrary.h>，黑魔法，在MTVideoEditDemo中编译正常，但是在MTXX工程中编译失败，部分swift代码找不到 MTPhotoLibrary 类

![image-20201225174514174](/Users/zj-db0449/Library/Application Support/typora-user-images/image-20201225174514174.png)

解决方案：针对 MTPhotoLibrary 开启 Module，整理MTXX工程中 直接/间接将 MTPhotoLibrary 引入在 MTXX-Bridging-Header.h 中的头文件，在头文件中用@class MTPhotoAsset等声明。

#### 5.2.2 问题2，MTXX中在添加了对于MTPhotoAsset的扩展A，swift 与 Objective-C都有调用A

![image-20201225175210178](/Users/zj-db0449/Library/Application Support/typora-user-images/image-20201225175210178.png)

解决方案：MTXX-Bridging-Header.h中移除 MTPhotoAsset 扩展A，针对扩展A添加一层包装，使得 MTXX-Bridging-Header.h 不引入MTPhotoLibrary内的Header。

#### 5.2.3 问题3，MTXX-Bridging-Header.h 没有引入 YYModel.h，YYModel也没有用 Module，但是编译没有报错

原因分析 MTMediaKit 的 MTMediaBaseDataModel.h 里引入了 YYModel.h，同时 MTMediaBaseDataModel.h 被引入到 MTMediaKit-umbrella.h 文件里，通知底层修改之后编译报错，在 MTXX-Bridging-Header.h 里引入 YYModel.h，问题解决。

![企业微信截图_b63e2127-a58c-49e5-ab51-1bcc8f9147fa](/Users/zj-db0449/Library/Containers/com.tencent.WeWorkMac/Data/Library/Application Support/WXWork/Data/1688852491345174/Cache/Image/2020-12/企业微信截图_b63e2127-a58c-49e5-ab51-1bcc8f9147fa.png)

结论：PodA 中的 Objective-C 文件被加入到 PodB-umbrella.h 文件里之后可以，可以在工程中和PodB的swift文件中访问。

#### 5.2.4 问题4，merge-module command failed due to signal 6

![企业微信截图_eaba13de-4778-4eae-8bf1-08a1c3b0cc49](/Users/zj-db0449/Library/Containers/com.tencent.WeWorkMac/Data/Library/Application Support/WXWork/Data/1688852491345174/Cache/Image/2020-12/企业微信截图_eaba13de-4778-4eae-8bf1-08a1c3b0cc49.png)

查看编译Error

```
1.	Apple Swift version 5.3.2 (swiftlang-1200.0.45 clang-1200.0.32.28)
2.	Contents of /var/folders/mp/0tkt1kvs7xs7v5qq620166hh0000gp/T/inputs-63cd6b:
3.	While evaluating request ExecuteSILPipelineRequest(Run pipelines { Mandatory Combines, Serialization, Rest of Onone } on SIL for MTXX.MTXX)
4.	While loading conformances for 'MTMusicModel' (in module 'MTXX')
```

问题分析：

Google之后将Build Setting中 Swift Compiler - Code Generation 里的 Compilation Mode 由 Increment改为 Whole Module，编译通过。但是会较大得影响编译时长。在 Compilation Mode = Whole Module时，依据conformances猜测是extension相关编译问题，移除YYModel与ListDiffable扩展之后，编译报错类变为MTMediaModel，此类也有YYModel与ListDiffable扩展，交叉验证之后确定是YYModel导致的问题，临时将YYModel开启 Module，这种情况只出现在验证 Compilation Mode = Increment 可行性的分支上出现，后来的版本未出现过。