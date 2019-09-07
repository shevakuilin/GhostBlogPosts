# 前言

今年十月份，掘金开始拆分业务线，借此契机实施组件化，主要精力都消耗在业务拆分、解耦、组件之间的横纵向依赖关系梳理、组件依赖下沉上，前后耗时半个月，最终完成。

这篇文章记录了基于 [CocoaPods](https://cocoapods.org/) 实施组件化，创建私有 pod 库的基本流程，严格按照文中流程来操作，你的组件化之路将会一帆风顺。

# 私有库的创建

### 创建私有 repo 仓库
我们将所要创建的私有库拆分为两部分，spec 源仓库 & pod 源仓库。

- `spec 源仓库`：是私有 source 地址所指向的 repo 仓库，也是查找该私有库的主要索引。spec源仓库中可以存放多个不同私有库的 `.podpsec` 版本文件，也就是说，spec 源仓库和 pod 源仓库是一对多的关系。举个例子：A 工程准备进行组件化，计划拆分成 a, b, c, d 四个组件，那么这四个组件库就可以全部指向同一个 spec 源仓库的地址，这样一来，在主工程 `Podfile` 中只需要填写一个 source 地址即可完成对四个组件库的引用。

- `pod 源仓库`：用于存放本地创建的完整 pod 文件，包括 `.podspec`、组件源码、资源文件等，可以理解为一个组件代码仓库。实际上也主要是用来存放组件源码及资源文件。

下面我们就来创建这两个 repo，一个用来作为 spec 源仓库以下简称 A，另一个用来作为 pod 源仓库以下简称B：

### 创建本地spec：
```vim
$ pod repo add [A的仓库名] [A的repo地址]
```
举例：
```vim
pod repo add testModule-spec git@xxxxxx/testModule-spec.git
```
创建完成后，进入 `~/.cocoapods/repos` 路径下查看，你所创建的同名文件夹应该出现在这里面，除此之外还有有一个公有库自带的 `master` 文件夹

### 创建本地pod并自动拉取模板：
```vim
$ pod lib create [B的pod库名称]
```
举例：
```vim
$ pod lib create testModule
```
执行完成后，本地会多出一个同名文件夹，文件主要目录结构如下：
```ruby
|-- testModule
    |-- _Pods.xcodeproj
    |-- Example
        |-- testModule.xcodeproj
        |-- testModule.xcworkspace
        |-- Podfile
        |-- Podfile.lock
        |-- Pods
    |-- testModule
        |-- Assets
        |-- Classes
    |-- testModule.podspec
    |-- LICENSE
    |-- README.md
```
并且会自动打开 `testModule.xcworkspace`，这时候直接选择选择关闭。

`testModule` 目录下的 `Classes` 就是用来放置我们从工程中抽离出来的组件代码的，而 `Assets` 则是用来存放 `Classes` 中代码需要使用到的图片等资源文件，可以直接存放图片，也可以制作成 [Bundle 资源文件包](http://www.cnblogs.com/QianChia/p/6280435.html)。

## 编写 podspec 配置
前面的步骤都做完之后，就来到了最重要的一步，对 `.podspec` 文件的配置编写，打开 `testModule.podspec` 你会看到里面已经生成好了一个标准模板，我们首先要做的就是将这些标准参数填写完毕：
```ruby
Pod::Spec.new do |s|
  #组件名称，也是执行 pod search 时输入的名称
  s.name             = 'testModule'
  #版本号，通常和tag一致 
  s.version          = '0.1.0'
  #概要，一句话介绍
  s.summary          = '这是一个业务组件'
  #描述，比概要字多就可以
  s.description      = <<-DESC 
                         这是一个详细的描述，比上面的字多就可以了
                        DESC
  #B pod私有库的地址
  s.homepage         = 'http://xxxxxx/testModule.git'
  #遵循的开源协议类型，默认MIT
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  #作者及邮箱
  s.author           = { 'author name' => 'xxxxx@email.com' }
  #源码地址，B pod私有库的ssh地址，如果需要加入子模块，就在后面加一个
  s.source           = { :git => 'git@xxxx/testModule.git', :tag => s.version.to_s}
  #与Xcode中主工程的最低支持版本号一直即可
  s.ios.deployment_target = '8.0'
  #源码文件路径，如果是oc库可以像这样用一个头文件包含需要引用的本组件其他代码的头文件，便于拆分成各个独立的文件夹管理，参考AFNetworking的目录，swift库就不用了
  s.source_files = 'testModule/Classes/testModule.h'
  #模块名称，在工程中调用时 #import <TModule/xxxx.h>
  s.module_name  = 'TModule'
  #私有头文件路径，如果有不希望暴露在组件外的私有头文件路径可以设置
  s.private_header_files = 'testModule/Classes/*.h'
  #公共头文件路径
  s.public_header_files = 'testModule/Classes/testModule.h'
  #是否使用ARC，默认true
  s.requires_arc = true
  #如果有需要单独使用MRC的文件，将文件路径加入排除文件，并以,隔开
  s.exclude_files = 'testModule/Classes/Libraries/MRC/**/*.{h,m}','testModule/Classes/Categorys/MRC/**/*.{h,m}'
  #依赖的其他库，包括公开Pod库、私有Pod库、subspec等
  s.dependency 'Masonry', '~> 1.0.1'
```

在这里需要注意几点就是：

`s.version` 的版本号一定要与你私有代码仓库（B 库）的 tag 保持一致 『尤其是在 `pod spec lint` 远程验证或 `pod repo push` 发版时，如果不一致将会持续报错』

`s.description` 这个一定要填写，很多人忽略了这个，而且必须在标准模板范围内写上你的简介，也就是 <<-DESC 和 DESC 之间，并且这两个 DESC 都一定不能删除 『否则在进行 `pod lib lint` 本地验证时，会报错』

`s.source_files`『路径千万要正确』，如果没有使用 subspec 的话，那么这个路径就是你 Classes 文件夹下的所有需要纳入 pod 库中的内容，举例如果是纯 oc 库，那么可以写成 `s.source_files = 'testModule/Classes/**/*.{h,m}'` 这段的含义是指向 `Classes` 文件夹下所有子文件夹内的 `.h` 和 `.m` 文件，也可以将这些文件的头文件统一放在一个 `.h` 文件内，这样就可以直接写为 `s.source_files = 'testModule/Classes/xxxx.h'`，Swift 库的话同理 `s.source_files = 'testModule/Classes/**/*.swift`，Swift 和 oc 混编的话就是 `s.source_files = 'testModule/Classes/**/*.{h,m,swift}`

`s.public_header_files` 『Swift 库一定不要写这个，因为 Swift 没有 `.h` 文件』，混编库也尽量不要写这个，具体原因后面会说，一定要写的话，最好将 oc 和 swift 代码用 subspec 隔离，在 subspec 内部声明 `subspec.public_header_files`

`s.dependency` 这里的依赖不单单是指的其他第三方库像 AFNetworking 之类的公开 pod 库，也可以依赖你自己的私有 pod 库，公开/私有 Pod 库的依赖方式都是一样的， `s.dependency 'xxxxx'`，『注意一定不要加 ‘=’，否则会报错』，是否指定版本都可以，私有库建议不指定版本，否则被依赖的私有库一旦更新，那么你这个库就得重新发版一次了。关于依赖该库本身的其他 subspec 的话，是这么个书写原则 `s.dependency '库名/subspec名'` 如果是 subspec 内部的 subspec 之间的依赖的话，也是一样的，比如 A 这个 subspec 下有 A1 和 A2 两个 subspec，A1 需要依赖 A2，那么 A 的依赖就是 `s.dependency '库名/A/A2'`

可以说，常见的验证错误基本都与上面这几项配置参数的设置有关系。

## 发版验证命令

接下来就是关于私有库验证和发版了，假设上面的步骤的 podspec 文件我们顺利编写完毕了，那么接下来就要进入验证环节了，验证是为了保证你编写的私有库的可用性，比如是否可以顺利根据 `source_files` 找到源码文件，源码内的语法是否存在无法通过编译的错误等等。具体来说，验证分为本地验证和远程验证：

### 1.本地验证
本地验证，顾名思义也就是对私有库进行一个本地的验证，『即使你不联网也可以完成这个验证过程』，这个过程中主要对你的 podspec 文件的编写格式、参数设置(如 `source_files` 路径内是否可以正确的找到对应文件)、依赖关系的检查，而不会做任何和联网有关的操作，其基本命令是:
```vim
$ pod lib lint xxx.podspec
```
通过这行基本的命令我们可以看出，执行这条命令的前提条件之一就是你处于包含有 `.podspec` 文件的根路径中 (参考上文列出的文件目录结构)。其实该命令可以简化为:
```vim
$ pod lib lint
```
之所以列出上面相对完整的写法是为了阐明该命令的执行条件，我平时更习惯用简化版的命令。

如果你的库无法保证一条 `Warning` 都没有，那么当你按照上面的这行命令进行执行后，将会收到来自 CocoasPod 的第一条验证警告：
```vim
[!] testModule did not pass validation, due to 7 warnings (but you can use `--allow-warnings` to ignore them).
You can use the `--no-clean` option to inspect any issue.
```
CocoasPod 会告诉你，因为你有 x 条 warning 没有被处理掉，所以你的验证无法被通过。

如果这 x 条 warning 全部来自你自己的代码文件，将 warning 处修改即可。但如果 warning 是来自依赖的其他第三方库该怎么办呢？

回过头来仔细看看警告信息，CocoaPods 给你留了条后路 `you can use --allow-warnings to ignore them`，可以使用 `--allow-warnings` 参数来使 CocoaPods 在验证时忽略这些 Warning。

既然如此，我们继续尝试：
```vim
$ pod lib lint --allow-warnings
```
更推荐的方式是:
```vim
$ pod lib lint --allow-warnings --verbose
```
因为 `--verbose` 参数可以让你完整的看到验证过程与其中的细节。

假设一切以理想状态发生，这个时候你将会在验证结束后的输出 Log 的结尾处，看到 `Verification pass` 这句话，标志着本地验证通过，可以着手进行远程验证了。(如果验证失败了不要担心，本系列专门拿出了一篇（[iOS 组件化之路（二）—– 私有库podspec文件编写 & 验证的踩坑记录](http://47.94.212.60/ios-componentization-02/)）来讲如何解决验证无法通过的问题，本文主要介绍私有库完整的 创建 -> 发版 -> 使用 流程)

### 2.远程验证
远程验证不仅需要连接 CocoaPods 本身的服务器，也需要对你的远程源仓库的 Tag 版本进行校验，这就需要你首先保证本地编写的 `.podspec` 及源码没有问题，并将其推到远程源仓库 (也就是文章最开始所说的 B 库) 后才能进行验证，将远程验证放到本地验证之后进行也是出于这个目的。

所以，当本地验证通过后，我们需要将本地的源码与 podspec 一起推到远程源仓库中，在本文的例子中也就是将 testModule 提交到远程 B 库中，步骤很简单:

#### 步骤一，编写commit信息
```vim
$ git add --all
$ git commit -m "[UPDATE] 0.1.0"
```
注意：无论你是做了什么操作，都最好带上与 tag 和 podspec 中的 version 保持一致的版本信息，上面 commit 中的 0.1.0 就是遵循的这一原则

#### 步骤二，为你本地创建的 testModule（也就是本地 B 库）与远程 B 库建立连接
```vim
$ git remote add origin git@xxxx/testModule.git #远程 B 库的地址
```
如果没有建立远程连接，你的本地库是无法被推到远程库上的，道理我想你是明白的

#### 步骤三，准备将本地 B 库推到远程 B 库中去

这里需要注意的是，如果你在创建远程 B 库时有在网页上做过其他操作，比如手动创建了一个 `README.md` 文件，那么你需要先拉一下远程 B 库： (没有做过这类操作的可以直接跳到步骤四)
```vim
$ git pull origin master
```
如果在执行完上面的命令后提示你 `refusing to merge unrelated histories` 的话，不用担心，这是因为远程 B 库内此时还没有任何有效文件，空空如已，所以只需要加入 `--allow-unrelated-histories` 来忽略这一情况即可：
```vim
$ git pull origin master --allow-unrelated-histories
```
#### 步骤四，上传本地 B 仓库文件
```vim
$ git push origin master
```

#### 步骤五，为上传的 B 仓库文件打上标签
首先查看本地 tag，避免与本地 tag 冲突以及保持 tag 命名的连续性：
```vim
$ git tag
```
打上标签：
```vim
$ git tag 0.1.0
```

上传本地 tag 至远程 B 仓库：
```vim
$ git push --tags #上传本地所有 tag
```
至此，进行远程验证的前置步骤就已经完成了。

#### 步骤六，远程验证

远程验证实际上 = 本地验证 + 远程仓库验证，

这里有一点需要说明的是，在进行发版操作时，也会进行一次远程验证，也就是说，当本地验证通过之后，实际上就可以直接进行发版操作了。

当然也可以忽略本地验证而直接进行远程验证，但就笔者的实践经验来看，本地验证 + 发版 的效率要远高于 远程验证 + 发版，而且更重要的一点是，直接采取远程验证由于必须先 push 本地的仓库，一旦本地代码文件验证存在问题，就会出现污染远程仓库的问题。

远程验证的基本命令与本地验证大同小异：
```vim
$ pod spec lint --allow-warnings --verbose
```
如果严格按照上面的步骤顺序进行，这一步远程验证就不会出现问题。

#### 步骤七，组件库发版

组件库发版也就是将本地的 `.podspec` 推到远程 A 仓库，也就是我们文章开头所说的 spec 源仓库。

组件发版时必须指定 本地 spec 仓库（本地 A 仓库）和 `.podspec` 文件，基本命令如下：
```vim
$ pod repo push testModule-spec(A仓库名) testModule.podspec (.podspec文件名) --allow-warnings --verbose
```
需要注意的是，『如果在本地/远程验证时加入了 `--no-clean` 参数，在发版时需要去掉该参数，否则会报错。』

当执行发版命令后，会先进行远程验证，验证通过后会上传到 repo 仓库中去。

#### 步骤八，检查成功发版的组件

在这一步中主要就是检查我们已经发版的组件是否符合我们的预期。

1.首先是打开网页，查看 spec 远程仓库（A 仓库）的内容，此时 spec 仓库应该只有一个 0.1.0 （与版本号对应） 的文件夹，文件夹内是 `.podspec` 文件。

2.随后查看远程代码库（B 仓库），其内容和本地的 B 仓库应该是完全一致的。

3.进入 `~/.cocoapods/repos` 路径下查看与本地 spec 仓库同名的文件夹，其内容应与远程 spec 仓库内容一致。

以上三点检查无误后，便说明组件库已经成功被部署了。

# 私有库的使用
组件库的作者和其他使用者的使用方式还是略有不同的，需要考虑到本地缓存和 repo 缓存的问题。

1.作者本人

作者本人使用非常简单，首先执行搜索命令，查看本地是否可以正确索引到自己制作好的组件库
```vim
$ pod search xxxxx  #组件名
```
如果提示搜索不到，需要先删除 cocoaPods 的本地索引缓存，执行：
```vim
$ rm ~/Library/Caches/CocoaPods/search_index.json
```
执行完毕后重新搜索即可

随后打开需要引用的工程的 `Podfile` 文件，在顶部加入如下两行内容：
```ruby
source 'https://github.com/CocoaPods/Specs.git' # 公有库 repo
source 'xxx/xxx/xxxxx.git' # 私有库 repo
```
私有库 repo 地址就是 远程 spec (A 库) 的 git 地址，然后引入你的私有库：
```ruby
pod 'xxxx', '~> 0.1.0'
```
保存完毕后，执行：
```vim
$ pod install
```
现在私有库就可以直接使用了。

Objective-C
```objectivec
#import <moudleName/xxxxx.h> // 如果podspec中没有设置 moudleName，默认就是你的私有库名字
```

Swift
```swift
import module
```

2.其他使用者

由于 repo 缓存问题，作者在更新后，其他使用者很可能是搜索不到最新版本的，所以需要更新 repo 之后再使用，其余步骤完全一致
```vim
$ pod repo update
```

# 小结
至此，私有组件创建 -> 验证 -> 发版 -> 使用的全部流程就已经介绍完了。

私有库主要麻烦的地方在于 podspec 文件的编写，需要你清楚地知道源码中各个文件的依赖关系， podspec 验证时也需要你明确了解组件内是否依赖了静态库或其他资源，总的来说，你需要十分了解你所抽出的组件构成部分，组件化的过程其实也是一次熟悉项目结构的过程。
