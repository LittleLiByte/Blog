---
title: ReactNative-CodePush实践小结
date: 2017-06-01 16:51:47
tags: 
    - react-native
    - codePush
description:
 "最近在React Native调研中使用了Code Push作为热更新实现方案，因此记录了一下实践过程和经验分享于此"
---
## 前言
**CodePush**是微软提供的一套可用于React Native和Cordova的热更新服务，国内也有类似的服务**Pushy**，从关注度和使用人数来说，CodePush完胜于Pushy（截至目前，CodePush在Github上Star数2900+，Pushy Star数600+，毕竟大公司的产品更让开发者心里有底，但CodePush是否真的绝对比Pushy要好不做评价）

## CodePush 安装与注册

### 1.安装 CodePush CLI
使用命令`npm install -g code-push-cli`安装CodePush终端

ps.都在开发React Native了，npm安装就无需赘言了吧。

### 2.注册CodePush 账号
CodePush终端安装完成后就可以使用`code-push`命令了。
在终端输入`code-push register`，会跳转授权网页。在这个网页可以选择Github。或者微软作为授权提供者，不过我觉得90%的开发者都会选择Github。
<!-- more -->
![enter description here][1]

授权完成后，CodePush会显示你的**Access Key**，复制输入到终端即可完成注册并登陆。
ps.只要不主动退出(通过`code-push logout`命令)，登陆状态会一直有效。

![enter description here][2]

![enter description here][3]

## 在CodePush服务器中创建App
在终端输入`code-push app add <appName>`即可完成创建，注册完成之后会返回一套deployment key，包括Staging和Production。该key在后面步骤中会用到。

![enter description here][4]

> 心得：如果你的应用分为Android和iOS版，那么在向CodePush注册应用的时候需要注册两个App获取两套deployment key，如：
 code-push app add AppDemo-Android
code-push app add AppDemo-iOS
因为发布的时候使用的打包命令是有所不同的，因此需要做区分。

code-push相关常见命令如下：

``` stata
Usage: code-push app <command>
命令：
  add       创建一个新的App
  remove    删除App
  rm        删除App
  rename    重命名已经存在App
  list      列出与你账户关联的所有App
  ls        列出与你账户关联的所有App
  transfer  将一个App的所有权转让给另一个帐户
```

## CodePush集成
这里跟进使用端分**Android集成**和**iOS集成**，但二者都有共通的部分

 - 在React Native项目中安装codePush依赖：`npm install --save react-native-code-push`
 - 通过react-native link命令自动构建关联，这里会要求输入 deployment key，直接Enter跳过即可，因为我们会在后续步骤中通过更加灵活的方式配置
 
 接下来的步骤，AndroidiOS有所不同，因此分开来说。

### Android端集成

 1. 打开android/app/build.gradle文件，修改android-buildTypes节点成如下

``` scilab
 buildTypes {
        debug{
			//省略了其他配置
            buildConfigField "String", "CODEPUSH_KEY", '""'
        }
        releaseStaging {
            buildConfigField "String", "CODEPUSH_KEY", '"此处填写Staging key"'
        }
        release {
			//省略了其他配置
            buildConfigField "String", "CODEPUSH_KEY", '"此处填写Production key"'
        }
    }
```
 2. 在android/app/build.gradle设置好deployment-key之后呢,我们就可以使用了:
修改MainApplication.java文件中的getPackages()方法为如下：

``` java
 protected List<ReactPackage> getPackages() {
            return Arrays.<ReactPackage>asList(
                    new MainReactPackage(),
                    new RCTSwipeRefreshLayoutPackage(),
                    new VectorIconsPackage(),
                    new CodePush(BuildConfig.CODEPUSH_KEY, MainApplication.this, BuildConfig.DEBUG)// Add/change this line.
            );
        }
```
3.修改VersionName
在 android/app/build.gradle中有个 android.defaultConfig.versionName属性，我们需要把 应用版本改成 三位，比如**1.0**，需要修改成**1.0.0**

至此，CodePush在Android端的集成工作已经完成了。

### iOS端集成

 1. 使用Xcode打开项目，Xcode的项目导航视图中的PROJECT下选择你的项目， 选择Info页签 ，在Configurations节点下单击 + 按钮 ，选择Duplicate "Release Configaration ， 输入Staging。

![enter description here][5]

 2. 在build Settings页签中单击 + 按钮然后选择添加User-Defined Setting，然后输入CODEPUSH_KEY(名称随意)，然后填入deployment key。

>  ps.可以通过`code-push deployment ls Flow800-Android -k`查看deployment key

![enter description here][6]

![enter description here][7]

3.打开 Info.plist文件，在CodePushDeploymentKey中输入$(CODEPUSH_KEY)，并修改Bundle versions为三位，如下图

![enter description here][8]

至此，iOS端集成也完成了。

## 使用react-native-code-psuh进行热更新

该配置的都已经配置完了，接下来就是使用了。
在使用之前需要考虑的是检查更新时机，更新是否强制，更新是否要求即时等等。
### 更新时机
一般常见的应用内更新时机分为两种，一种是打开APP就检查更新，一种是放在设置界面让用户主动检查更新并安装。

 - **打开APP就检查更新**
	最为简单的使用方式在React Natvie的根组件的componentDidMount方法中通过
	codePush.sync()（需要先导入codePush包：import codePush from 'react-native-code-push'）方法检查并安装更新，如果有更新包可供下载则会在重启后生效。不过这种下载和安装都是静默的，即用户不可见。如果需要用户可见则需要额外的配置。具体可以参考codePush官方API文档，下面是个人的一些实践过的配置：
	
``` javascript
codePush.sync({
      updateDialog: {
        appendReleaseDescription: true,
        descriptionPrefix:'\n\n更新内容：\n',
        title:'更新',
        mandatoryUpdateMessage:'',
        mandatoryContinueButtonLabel:'更新',
      },
      mandatoryInstallMode:codePush.InstallMode.IMMEDIATE,
      deploymentKey: CODE_PUSH_PRODUCTION_KEY,
    });
```
上面的配置在检查更新时会弹出提示对话框， mandatoryxxx表示强制更新，appendReleaseDescription表示在发布更新时的描述会显示到更新对话框上让用户可见

来个更加接近实际应用的：在用户点击**检查更新**按钮后进行检查，如果有更新则弹出提示框让用户选择是否更新，如果用户点击**立即更新**按钮，则会进行安装包的下载（实际上这时候应该显示下载进度，这里省略了）下载完成后会立即重启并生效（也可配置稍后重启）
ps.这里面还有个神坑，下文再说。

``` javascript
codePush.checkForUpdate(deploymentKey).then((update) => {
                            if (!update) {
                                Alert.alert("提示", "已是最新版本--", [
                                    {
                                        text: "Ok", onPress: () => {
                                        console.log("点了OK");
                                    }
                                    }
                                ]);
                            } else {
                                codePush.sync({
                                        deploymentKey: deploymentKey,
                                        updateDialog: {
                                            optionalIgnoreButtonLabel: '稍后',
                                            optionalInstallButtonLabel: '立即更新',
                                            optionalUpdateMessage: '有新版本了，是否更新？',
                                            title: '更新提示'
                                        },
                                        installMode: codePush.InstallMode.IMMEDIATE,

                                    },
                                    (status) => {
                                        switch (status) {
                                            case codePush.SyncStatus.DOWNLOADING_PACKAGE:
                                                console.log("DOWNLOADING_PACKAGE");
                                                break;
                                            case codePush.SyncStatus.INSTALLING_UPDATE:
                                                console.log(" INSTALLING_UPDATE");
                                                break;
                                        }
                                    },
                                    (progress) => {
                                        console.log(progress.receivedBytes + " of " + progress.totalBytes + " received.");
                                    }
                                );
                            }
                    }
```
### 更新是否强制
如果是强制更新需要在发布的时候指定，发布命令中配置`--m true`，下文在细说

### 更新是否要求即时
在更新配置中通过指定installMode来决定安装完成的重启时机，亦即更新生效时机
 - **codePush.InstallMode.IMMEDIATE**：表示安装完成立即重启更新
 - **codePush.InstallMode.ON_NEXT_RESTART**：表示安装完成后会在下次重启后进行更新
 - **codePush.InstallMode.ON_NEXT_RESUME**：表示安装完成后会在应用进入后台后重启更新

## 发布CodePush更新包
codepush的更新包发布其实很简单。在终端输入命令

``` vbscript-html
code-push release-react <Appname> <Platform> --t <本更新包面向的旧版本号> --des <本次更新说明>
```
CodePush默认是更新 Staging 环境的，如果发布生产环境的更新包，需要指定--d参数：`--d Production` ，如果发布的是强制更新包，需要加上 `--m true`强制更新

示例：
``` brainfuck
code-push release-react Flow800-Android android --t 2.0.0 --dev false --d Production --des "1.全新页面\n2.上线流量商城\n3.已知bug修复" --m ture
```
常用部署命令如下：

``` stata
Usage: code-push deployment <command>
命令：
  add      在已存在的App中创建一个部署
  clear    清除与部署相关的发布历史记录
  remove   在App中删除一个部署
  rm       在App中删除一个部署
  rename   重命名一个已存在的部署
  list     列出App中的所有部署
  ls       列出App中的所有部署
  history  列出一个部署的发布历史记录
  h        列出一个部署的发布历史记录
```

至此，一个完整的发布，检查，安装流程已经基本描述完了。下面来看下更新的效果：

![enter description here][9]

## 实践经验小结

 1. 在Android端进行调试时，在更新之前需要修改Debug Server地址和端口为任意字符串，让其访问不到真正Debug Server，否则更新重启后，就直接访问了Debug Server导致新的安装包没有安装上。
![enter description here][10]
也可以直接打Android离线包，这样子就不需要理会Debug Server的影响

 2. 在iOS端调试时，需要打离线包并拖拽到Xcode工程中。
 打包命令：

``` brainfuck
react-native bundle --entry-file index.ios.js --bundle-output ./bundle/ios/main.jsbundle --platform ios --assets-dest ./bundle/ios --dev false
```

![enter description here][11]
拖拽完成后目录结构如上。
此外，还需要对AppDelegate.m文件进行修改：

![enter description here][12]
否则更新重启后还是旧版的App。

3.发布更新包时，切记--t参数指定的是本次更新包的目标版本号，而不是本次更新包的版本号

4.如果不是在根组件的componentDidMount()方法通过codePush.sync()方法进行更新检查与安装，比如像我在设置中通过一个**检查更新**按钮，这时候就需要十分小心了。必须在根组件的componentDidMount()方法中添加 `codePush.notifyAppReady()`，否则应用会出现第一次重启是更新版，第二次重启又回滚到旧版的现象。

5.一个App通常不会是一个人开发，因此codePush添加合作者是很常见的。通过 `code-push collaborator`来操作合作者相关命令。

``` avrasm
Usage: code-push collaborator <command>
命令：
  add     对指定的项目添加一个新的合作者
  remove  删除指定的项目中的合作者
  rm      删除指定的项目中的合作者
  list    列出指定项目中的所有合作者
  ls      列出指定项目中的所有合作者
示例:
    code-push collaborator add AppDemo foo@bar.com 添加一个合作者foo@bar.com到AppDemo这个App中
```
6.最后说一下CodePush的缺陷，因为是国外服务器，所以有时候下载速度并不是很理想。此外CodePush现在是免费服务，以后会不会收费还是个未知数。因此已经有人提出了自建codePush服务器，可见于此：[**code-push-server**][13]

更多常见问题及API使用方法请参见以下资料
## 参考资料
[微软官方ReactNative CodePush文档][14]
[React Native热更新部署/热更新-CodePush最新集成总结(新)][15]
[react-native-code-push Github][16]
基本上所有问题都可以在官方文档或者github的issue中找到答案。


  [1]: http://om2bpqram.bkt.clouddn.com/1496284115079.jpg
  [2]: http://om2bpqram.bkt.clouddn.com/1496284247053.jpg
  [3]: http://om2bpqram.bkt.clouddn.com/1496284287922.jpg
  [4]: http://om2bpqram.bkt.clouddn.com/1496284777374.jpg
  [5]: http://om2bpqram.bkt.clouddn.com/1496288208995.jpg
  [6]: http://om2bpqram.bkt.clouddn.com/1496288348109.jpg
  [7]: http://om2bpqram.bkt.clouddn.com/1496288455133.jpg
  [8]: http://om2bpqram.bkt.clouddn.com/1496288738755.jpg
  [9]: http://om2bpqram.bkt.clouddn.com/update.gif
  [10]: http://om2bpqram.bkt.clouddn.com/1496299813860.jpg
  [11]: http://om2bpqram.bkt.clouddn.com/1496301251075.jpg
  [12]: http://om2bpqram.bkt.clouddn.com/1496301458140.jpg
  [13]: https://github.com/lisong/code-push-server
  [14]: http://microsoft.github.io/code-push/docs/react-native.html
  [15]: http://www.jianshu.com/p/9e3b4a133bcc
  [16]: https://github.com/Microsoft/react-native-code-push
