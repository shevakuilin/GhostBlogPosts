---
title: Fastlane 一键打包上传至 Bugly 的全自动化测试分发的搭建
date: 2018-08-17 15:22:51
categories: iOS Development
tags:
description: 项目每次进入上线前期的测试阶段，都要反复地利用Xcode打包，再上传到测试发布平台，再分发给测试人员，无论从哪个角度看，这都是一个毫无技术含量并且及其耗时的事情，那么能不能通过自动化部署来解放这部分的时间消耗呢？
---

无论如何，测试打包对于 iOS 开发者而言都是件繁琐枯燥的事情，项目规模小还好，几分钟不过是去喝杯咖啡的时间。但是对于稍微大一点的项目而言，少则十几分钟，多则半小时，在这么长的时间里，你得一直等待Xcode完成 Archive 任务，在此期间还需要你点击几个选项，既费时，又费心。尤其是遇到个别 UI 测试上的小问题，一个像素的调整、一处文案的修改，反复的执行打包操作，脾气再好的工程师都会感到一阵抓狂。

自从去年第一次使用过 Fastlane 之后，打包这项繁琐的工作，再也不会成为一项负担，你只需要在 iTerm 里敲两行命令，它就能全程自动化的帮你完成从打包到测试平台发布整套流程，并且更重要的是，这个过程不会影响你当前的任何操作，你仍然可以继续使用 Xcode，或者做任何你想做的事。


本文以下内容的运行环境：

- Xcode: 9.4.1

- macOS: High Sierra 10.13.4	

- Fastlane: 2.8.0

- Ruby: 2.4.0
	

## [Fastlane](https://fastlane.tools/)

<img src="http://ofg0p74ar.bkt.clouddn.com/fastlane_text.png">

Fastlane 是 Google 的大神 [Felix Krause](https://github.com/KrauseFx) 基于 Ruby 写的一套自动化工具集，能够帮助iOS 和 Android 开发者实现自动化部署，持续集成等工作。Fastlane 的组件包括：

|    名称     | 作用 |
| ---------- | --- |
| deliver |  自动上传截图，APP的元数据，二进制(ipa)文件到iTunes Connect |
| snapshot       |  自动截图（基于Xcode的UI test） |
| frameit       |  可以把截图自动加上边框 |
| pem       |  自动生成、更新推送配置文件 |
| sigh       |  用来创建、更新、下载、修复Provisioning Profile的工具|
| produce       |  自动在iTunes Connect或Apple Developer Center中建立你的产品 |
| cert       |  自动创建管理iOS代码签名证书 |
| pilot       |  管理TestFlight的测试用户，上传二进制文件 |
| boarding       |  建立一个添加测试用户界面，发给测试者，可自行添加邮件地址，并同步到iTunes Connect |
| gym       |  自动化编译打包工具 |
| match       |  证书和配置文件管理工具 |
| scan       |  自动运行测试工具，并生成HTML报告 |


## 安装 & 初始化

Fastlane 的安装非常简单，网上的教程很多，这里就不赘述了，推荐下面两篇教程：

1. [小团队的自动化发布－Fastlane带来的全自动化发布](https://whlsxl.github.io/fastlane1/)

2. [Fastlane实战（一）：移动开发自动化之道](https://www.jianshu.com/p/1aebb0854c78)



## Fastfile 脚本的编写

初始化完成后，Fastlane 就可以投入使用了。Fastlane 整个自动化的核心就是 Fastfile 这个脚本执行文件和控制自动化链条的 Action，所以，如果想要完成自动化测试发布，就需要对这个文件进行自定义编写，并处理其相应的 Action。

我们先将自动化测试发布的步骤，按照一个个 Action 进行独立拆分，最后在合并成为完整的一键自动化流程。

我们想要达到的效果是，在终端执行一行 lane 指令后，可以自由选择是否对上传到 Bugly 的测试包展示内容进行自定义修改，如果需要，则允许使用者在终端中输入自定义内容并保存到最终的上传参数中，不需要则直接自动执行打包+上传的操作。

基于这些诉求，我们拆分后的步骤大致如下：

1.lane指令的配置

2.gym Action的自动打包脚本编写

3.upload_app_to_bugly Action的自动上传 ipa 到 Bugly 的脚本编写

4.手动设置参数任务的编写，用于自定义 Bugly 上传参数

5.合并编写好的脚本到统一的lane指令


#### lane指令的配置

打开 Fastlane 文件夹下的 Fastfile，注意，一定不能直接使用文本编辑器打开，会引起自动引号等报错，这里推荐使用 [Visual Studio Code](https://code.visualstudio.com/) 选择 Ruby 文本样式来打开编写。

lane指令的配置非常简单，直接用以下内容覆盖 fastfile 默认的模板：

```ruby
#设置平台参数
default_platform(:ios)
platform :ios do

#lane执行指令，使用时直接在终端输入fastlane adhoc，这里的adhoc可以换成你喜欢的任何名字
lane :adhoc do

end
end

```


#### 自动化打包

我们使用 Fastlane 提供的 gym, 这个专门的打包 Action 来对脚本进行编写：

在上一步设置好的lane指令作用域内（也就是lane :adhoc do - end 中间这部分区域），输入如下内容：

```ruby
#指定scheme, 可通过Xcode->Product->Scheme->ManageSchemes查看
scheme = "Your project scheme"
#当前时间字符串，用于输出文件夹的后缀，遵循 Xcode 的输出规则
currentTime = Time.new.strftime("%Y-%m-%d %H-%M-%S")
#指定输出文件夹路径, 将ipa_name换成你想要的ipa名称
output_path = "../ipa_name #{currentTime}"
#指定输出ipa名称
ipa_name = "Your ipa name"

#自动打ad_hoc测试包
gym(
  #指定workspace, 这个参数很重要，否则你还需要手动选择一次打包对象
  workspace:"xxxx.xcworkspace",
  #指定scheme
  scheme:"#{scheme}",
  #打包后的ipa名称
  output_name:"#{ipa_name}",
  #是否在运行时自动清理上一次的执行
  clean:true,
  #指定要打包的配置名
  configuration:"Adhoc",
  #指定打包所使用的输出方式，目前支持app-store, package, ad-hoc, enterprise, development, 和developer-id，即xcodebuild的method参数
  export_method:"ad-hoc",
  #最后输出的文件夹路径
  output_directory:"#{output_path}",
)

```


写好之后，你的 fastfile 应该是这个样子:

```ruby
default_platform(:ios)
platform :ios do

#一键打包上传到 Bugly
lane :adhoc do
	scheme = "Your project scheme"
	currentTime = Time.new.strftime("%Y-%m-%d %H-%M-%S")
	output_path = "../ipa_name #{currentTime}"
	ipa_name = "Your ipa name"

	#自动打ad_hoc测试包
	gym(
  		workspace:"xxxx.xcworkspace",
  		scheme:"#{scheme}",
  		output_name:"#{ipa_name}",
  		clean:true,
  		configuration:"Adhoc",
  		export_method:"ad-hoc",
  		output_directory:"#{output_path}",
	)
end
end
```

到这里，一个基本的 gym Action 自动打包脚本就编写完毕了。但是仅仅依靠上面这些内容还不足以完成我们想要的打包全自动化流程，现在，我们在执行 fastlane adhoc 打包之前，还需要手动对每个target中的 build 版本号进行更改，这非常麻烦，而且如果在某次打包之前忘记了更改，还得打断任务重新执行。

不过不用担心，Fastlane 早已替我们考虑周到。我们只需要在 gym Action 执行之前，也就是在它的前一行加上一行代码即可：

```ruby
default_platform(:ios)
platform :ios do

#一键打包上传到 Bugly
lane :adhoc do
	scheme = "Your project scheme"
	currentTime = Time.new.strftime("%Y-%m-%d %H-%M-%S")
	output_path = "../ipa_name #{currentTime}"
	ipa_name = "Your ipa name"
	
	#build自动增加, 会自动的再当前build version的基础上+1, 需要对工程进行额外配置，下文会说明
	increment_build_number
	
	#自动打ad_hoc测试包
	gym(
  		workspace:"xxxx.xcworkspace",
  		scheme:"#{scheme}",
  		output_name:"#{ipa_name}",
  		clean:true,
  		configuration:"Adhoc",
  		export_method:"ad-hoc",
  		output_directory:"#{output_path}",
	)
end
end
```

#### Xcode 工程配置

[increment_build_number](https://docs.fastlane.tools/actions/increment_build_number/) 是 Fastlane 自带的脚本命令，作用是在当前 build version 的基础上自动 +1，省去了每次打测试包都要手动修改 Build 的工作。

其原理是代替我们调用了 Xcode 自带的自动增加版本号的命令行工具 `agvtool`。有对 `agvtool` 感兴趣的可以了解一下[具体如何使用](https://segmentfault.com/a/1190000004678950)。

在 Fastfile 脚本文件中添加了 increment_build_number 之后，还需要对 Xcode 进行额外的工程配置：

注意：如果有多个 target，需要对每个 target 都进行下面的设置

1. 设置 Current Project Version & Versioning System
	- TARGETS -> Build Settings

	- Current Project Version 填写当前版本号，最好从1开始

	- Versioning System 选择 Apple Generic
	
<img src="http://ofg0p74ar.bkt.clouddn.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_ab587b3b-42fb-4cb1-9d6c-bda906a187eb.png">

2. 设置 Bundle version & Bundle versions string, short

	- TARGETS -> Info

	- Bundle version 和 第1步设置的Current Project Version保持一致即可

	- Bundle versions string, short 是当前应用的 Version 版本号，如果需要自增 Version 再设置，其对应的脚本参数是[increment_version_number](https://docs.fastlane.tools/actions/increment_version_number/)，仅自增 Build 无需设置


<img src="http://ofg0p74ar.bkt.clouddn.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_94af5e95-4614-4c26-81b3-2ceca99f009d.png">

至此，我们的1、2两步就全部完成了。


#### 将 ipa 包自动上传至 Bugly

接下来，进行我们自动化流程的第3步，将ipa包自动上传至 Bugly。

Bugly 官方为我们提供了两个 Fastlane 的扩展插件：上传文件和更新文件，由于目前腾讯已经关闭了 Bugly 官网的文档入口，可以直接查看他们的 Github repo 的[使用文档](https://github.com/BuglyDevTeam/Bugly-FastlanePlugin)

首先点击下载插件，下载完成后，本地会多出两个 ruby 文件，更新文件插件`update_app_to_bugly` 和 上传文件插件`upload_app_to_bugly`。然后将下载好的上传文件插件的文件名 `upload_app_to_bugly` 复制下来，在下一步中会用到。

这时，就需要我们开始编写 upload_app_to_bugly Action 了。

首先，自定义一个 Action，在 cd 到工程根目录（有 fastlane 文件夹的那个）终端中执行如下命令

```ruby
fastlane new_action
```

输入完之后，Fastlane 会让你输入自定义的 Action 名字，直接粘贴刚才复制好的名字即可。

随后，Fastlane 就会在 fastelane 文件目录下创建一个名为 actions 的文件夹，而文件夹下也会默认有一个名为 `upload_app_to_bugly.rb` 的 ruby 文件。然后我们把刚才下载好的同名文件直接进行替换就可以了。

upload_app_to_bugly Action 创建成功后，我们需要回到 fastfile 里面对其 Action 内容进行编写，因为 fastfile 的 Action 执行优先级是自上而下的，所以我们需要将 upload_app_to_bugly Action 的内容写在 gym Action 下面:

```ruby
#指定输出ipa文件路径 
ipa_path = "#{output_path}/#{ipa_name}.ipa"

#上传Bugly
upload_app_to_bugly(
	#ipa的文件绝对路径
	file_path:"#{ipa_path}",
	#应用编号, 用于识别产品的唯一ID
	app_key:"xxxxx",
	#用于识别API调用者身份, 创建应用时自动获得
	app_id:"xxxx",
	#应用平台标识, 用于区分产品平台android:1 iOS:2
	pid:"2",
	#版本名称, 如有中文必须UTF-8格式
	title:"xxxxx",
	#版本介绍, 如有中文必须UTF-8格式
	desc:"xxxxx",
	#公开范围（1:所有人, 2:密码, 4:管理员, 5:QQ群, 6:白名单, 默认所有人）
	secret:"1",
	#如果公开范围是"QQ群"填写QQ群号, 如果是"白名单"填写QQ号码, 并使用;切分开, 5000个以内. 其他场景无需设置
	users:"",
	#如果公开范围是"密码"需设置
	password:"",
	#下载上限（大于0, 默认1000）
	download_limit:999
)
```

这里只列举了必填参数和部分可选参数，腾讯在去年将插件的API文档入口也一并从 Bugly 移除了，如果你想了解更多相关参数的含义，可以在我前搭档写的[这篇文章](https://juejin.im/post/58c237cd44d9040068e80be1)里找到其文档的网页快照。

此时你的 fastfile 应该是这个样子:

```ruby
default_platform(:ios)
platform :ios do

#一键打包上传到 Bugly
lane :adhoc do
	scheme = "Your project scheme"
	currentTime = Time.new.strftime("%Y-%m-%d %H-%M-%S")
	output_path = "../ipa_name #{currentTime}"
	ipa_name = "Your ipa name"
	ipa_path = "#{output_path}/#{ipa_name}.ipa"
	
	#build自动增加
	increment_build_number
	
	#自动打ad_hoc测试包
	gym(
  		workspace:"xxxx.xcworkspace",
  		scheme:"#{scheme}",
  		output_name:"#{ipa_name}",
  		clean:true,
  		configuration:"Adhoc",
  		export_method:"ad-hoc",
  		output_directory:"#{output_path}",
	)
	
	#上传Bugly
	upload_app_to_bugly(
		file_path:"#{ipa_path}",
		app_key:"xxxxx",
		app_id:"xxxx",
		pid:"2",
		title:"xxxxx",
		desc:"xxxxx",
		secret:"1",
		users:"",
		password:"",
		download_limit:999
	)

end
end

```

现在，我们将第3步也完成了。

#### 通过控制脚本管理上传参数

每次发布测试包后，都需要在 Bugly 测试分发平台上手动修改，测试包名称和本次更新内容，也就是上一步中，upload_app_to_bugly Action 中的 title、desc 参数。

如果你完全没有修改 title、decs 的需求，到第3步结束，这个 fastfile 实际就可以直接投入使用了。但是我们为了更好的把控测试分发和规范流程，还是需要在每次测试包更新后，对所更新的内容进行一个简要的说明。

下面我们就开始第4个步骤，对手动设置参数任务的编写，在 gym Action 之前，添加 Ruby 代码：

```ruby
#设置测试包默认title
version_title = "xxx"
#设置测试包默认desc
version_desc = "xxxxxx"

begin
	#选择是否手动设置测试包的标题和描述
	puts "Do you want to manually set the title of the test package & description? (y/n)" 
	res = STDIN.gets.chomp

	if res != "n"
		#用于 Bugly 显示的测试包标题
		print Input title:  "
		version_title = STDIN.gets.chomp
		puts "---Title set successfully!---"

		#用于 Bugly 显示的测试包更新描述
		print "Input description:  "
		version_desc = STDIN.gets.chomp
		puts "---Description set successfully!---"
	end
end
```

上面这段代码是一段选择任务，会在自动打包执行之前进行一次询问，是否需要手动设置测试包的标题和描述，如果选择y，则会要求使用者输入测试包的标题和更新描述，如果选择n则自动跳过这一步，直接开始打包。

这就避免了在分发新的测试包的时候，还需要修改 upload_app_to_bugly Action 脚本参数 或者 打开 Bugly 网页 -> 点击内测分发 -> 选择上传的测试包 -> 点击修改，这样繁琐的步骤。

最后，你的 fastfile 应该是这个样子:

```ruby
default_platform(:ios)
platform :ios do

#一键打包上传到 Bugly
lane :adhoc do
	scheme = "Your project scheme"
	currentTime = Time.new.strftime("%Y-%m-%d %H-%M-%S")
	output_path = "../ipa_name #{currentTime}"
	ipa_name = "Your ipa name"
	ipa_path = "#{output_path}/#{ipa_name}.ipa"
	version_title = "xxx"
	version_desc = "xxxxxx"
	
	begin
		#选择是否手动设置测试包的标题和描述
		puts "Do you want to manually set the title of the test package & description? (y/n)" 
		res = STDIN.gets.chomp

		if res != "n"
			#用于 Bugly 显示的测试包标题
			print "Input title:  "
			version_title = STDIN.gets.chomp
			puts "---Title set successfully!---"

			#用于 Bugly 显示的测试包更新描述
			print "Input description:  "
			version_desc = STDIN.gets.chomp
			puts "---Description set successfully!---"
		end
	end
	
	#build自动增加
	increment_build_number
	
	#自动打ad_hoc测试包
	gym(
  		workspace:"xxxx.xcworkspace",
  		scheme:"#{scheme}",
  		output_name:"#{ipa_name}",
  		clean:true,
  		configuration:"Adhoc",
  		export_method:"ad-hoc",
  		output_directory:"#{output_path}",
	)
	
	#上传Bugly
	upload_app_to_bugly(
		file_path:"#{ipa_path}",
		app_key:"xxxxx",
		app_id:"xxxx",
		pid:"2",
		title:"xxxxx",
		desc:"xxxxx",
		secret:"1",
		users:"",
		password:"",
		download_limit:999
	)

end
end
```

至此，这个自动打包上传至 Bugly 的 整套脚本就已经基本搭建完毕了。


#### 完善脚本

最后一步，我们需要将这个 “裸奔” 的脚本进行一些必要的点缀，在关键步骤上做一些处理：

1.添加输出 log

我们在一些关键步骤上，比如更新 build_number 之后、打包完成开始上传到 Bugly 的执行节点上，加上一些提示性的 log, 对查看打包上传进度是很有帮助的:

你可以选择使用 puts 进行打印，它会在打印后自动切换光标到下一行，如：

```ruby
puts "---Build number updated successfully !---"
```

也可以使用 print 进行打印，它在打印后会将光标停留在当前这行，如：

```ruby
print "Input title:  "
```


2.让关键 log 以颜色进行区分

如果不进行颜色区分，脚本在执行中密密麻麻的 log 全部都是同一个颜色，让人眼花缭乱，那么我们添加的输出 log 就是白费功夫了，所以我们需要对关键 log 加以颜色进行渲染，如：

```ruby
#设置文本颜色
def colorize(text, color_code)
	"\e[#{color_code}m#{text}\e[0m"
end

#绿色
def green(text); colorize(text, 32); end
#深紫色
def magenta(text); colorize(text, 35); end
#青蓝色
def cyan(text); colorize(text, 36); end
``` 


这些都是 ruby 当中常用的颜色值设置方法，更多颜色的设置方法可以参考 [这里](https://blog.csdn.net/dazhi_100/article/details/39528527)


使用方法，以上面的 log 为例：


```ruby
puts ""+ green("---Build number updated successfully !---") +""
```


最终这行 log 在终端中就会以绿色的文本进行显示了。

通过这些简单的点缀，我们来最后完善一下这个自动化打包发布脚本：

```ruby
default_platform(:ios)
platform :ios do

#一键打包上传到 Bugly
lane :adhoc do
	scheme = "Your project scheme"
	currentTime = Time.new.strftime("%Y-%m-%d %H-%M-%S")
	output_path = "../ipa_name #{currentTime}"
	ipa_name = "Your ipa name"
	ipa_path = "#{output_path}/#{ipa_name}.ipa"
	version_title = "xxx"
	version_desc = "xxxxxx"
	
	def colorize(text, color_code)
    	"\e[#{color_code}m#{text}\e[0m"
  	end

  	def green(text); colorize(text, 32); end
  	def magenta(text); colorize(text, 35); end
  	def cyan(text); colorize(text, 36); end
	
	begin
		#选择是否手动设置测试包的标题和描述
		puts ""+ magenta("Do you want to manually set the title of the test package & description? (y/n)") +"" 
		res = STDIN.gets.chomp

		if res != "n"
			#用于 Bugly 显示的测试包标题
			print ""+ magenta("Input title:  ") +""
			version_title = STDIN.gets.chomp
			puts ""+ green("---Title set successfully!---") +""

			#用于 Bugly 显示的测试包更新描述
			print ""+ magenta("Input description:  ") +""
			version_desc = STDIN.gets.chomp
			puts ""+ green("---Description set successfully!---") +""
		end
	end
	
	#build自动增加
	increment_build_number
	puts ""+ green("---Build number updated successfully !---") +""
	
	#自动打ad_hoc测试包
	gym(
  		workspace:"xxxx.xcworkspace",
  		scheme:"#{scheme}",
  		output_name:"#{ipa_name}",
  		clean:true,
  		configuration:"Adhoc",
  		export_method:"ad-hoc",
  		output_directory:"#{output_path}",
	)
	
	#上传Bugly
	puts ""+ cyan("---Start uploading ipa to Bugly---") +""
	upload_app_to_bugly(
		file_path:"#{ipa_path}",
		app_key:"xxxxx",
		app_id:"xxxx",
		pid:"2",
		title:"xxxxx",
		desc:"xxxxx",
		secret:"1",
		users:"",
		password:"",
		download_limit:999
	)
	puts ""+ green("---ipa to upload successfully !---") +""

end
end
```

现在，这个 fastfile 才算正式宣告完成了，让我们来看看使用效果。


## 自动化打包测试分发的使用

首先在终端中 cd 到你项目的根目录，也就是有 fastlane 文件的那一层，然后在终端中输入:

```ruby
fastlane adhoc
```

当 Fastlane 标志性的小火箭开始出现，就表明 lane 指令开始执行了

<img src="http://ofg0p74ar.bkt.clouddn.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_0da97211-8b49-43c6-801f-c0cf9574a0cd.png">

接下来脚本开始向你询问，是否需要手动修改上传参数，选择y进行修改，选择n直接跳过

<img src="http://ofg0p74ar.bkt.clouddn.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_9678db7b-6993-49b6-a9df-bb133bb3a4ec.png">

为了演示效果，我们在这里输入y，然后脚本就会要求你输入本次上传测试包的标题，这里我们随便输入一个 test0002

<img src="http://ofg0p74ar.bkt.clouddn.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_37647e08-1fc5-4e92-b106-f12604131944.png">

接下来是更新描述，我们也随便输入一个 decr0002，输入完毕后，就开始执行打包操作，也就是 gym 这个 Action 了:

<img src="http://ofg0p74ar.bkt.clouddn.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_da3ed7ff-5e7a-425a-9cbc-15fcbee13e91.png">

接下来我们只要等着 Fastlane 执行完毕即可，这期间我们可以继续使用 Xcode 或着做其他任何事，都不会受到影响。

当 gym 的打包执行完毕后，就开始执行 upload_app_to_bugly Action 了，此时终端会显示这段分水岭的 log:

<img src="http://ofg0p74ar.bkt.clouddn.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_078d1dc1-2af2-4234-b3c8-a376eed059a7.png">

当 upload_app_to_bugly 执行完毕，也就宣告 ipa 已经被成功上传到 Bugly，这段自动打包分发测试脚本也会跟随结束执行，并在终端上生成一段漂亮的摘要报告，上面会显示出 fastfile 的所有执行步骤，以及每个 Action 的执行时间：

<img src="http://ofg0p74ar.bkt.clouddn.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_69ebbf23-3717-456f-baad-a49af92e5a6d.png">

最终，我们这次自动打包上传总共耗时15分钟，其中打包过程用时大约12分半，上传到 Bugly 用时1分半。我用手动打包 + 上传对比了一下，单单使用 Archive 的打包耗时就达到了18分钟，后面的上传算上找到文件夹，打开网站，点击上传这期间耗费的时间，大约3分半，Fastlane 足足为我节省了6分钟的时间，效率可以说是非常可观了。


进入 Bugly 的分发测试页面，你就可以看到已经上传好的测试包了，直接复制链接地址或者 QRCode 给测试人员就可以了。

上传成功后会在你的项目根目录下生成一个无用的json文件`upload_app_to_bugly_result.json`，这个文件在上传成功后直接删除就可以了。

当然最后的分发以及删除操作你也可以写在脚本中一起帮你执行。

至此，Fastlane + Bugly 全自动化部署测试发布就全部搭建完毕了，同理，上传至 AppStore 一样可以使用 Fastlane 完成自动化部署，有了上面这些经验，就可以很轻松的配置完成了，亲手去尝试和体验 Fastlane 的便捷吧。


## 后记

除了 gym ，也可以直接通过 shell 脚本来执行 Fastlane，想了解更多使用上的配置，可以参考 [官方文档](https://docs.fastlane.tools/)


希望 Fastlane 能够帮助更多的工程师解放双手和时间。

