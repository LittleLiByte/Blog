---
title: ReactNative学习路线
date: 2017-05-15 12:20:07
tags:
    - react-native
    - 学习路线
---


## 第一步 React.js基础学习
只推荐**[菜鸟教程][1]**，章节分明有条理，讲解得也非常清晰，并且附带可以在线运行的实例。学习时间：0.5～1天基本可以完成学习


## 第二步 ES6 语法特性
推荐搭配WebStorm 编写示例学习，可在WebStorm 直接运行ReactJs

推荐文章：
**[30分钟掌握ES6/ES2015核心内容][2]**
[**React/React Native 的ES5 ES6写法对照表**][3]
<!-- more -->
除了上面这两篇文章提到的内容，最好还了解一下比较有用的包括集合，proxy，promise。

> ps.ES6语法特性的学习是必不可少的，否则代码看得一知半解的对后续的学习会有很大阻碍。

## 第三步 React Native基础
比较推荐这个，
[**江清清的技术专栏：React Native专题**][4]

与React Native中文网的文档相比更加符合初学者，例子更加简单和完整，不会像React Native中文网的文档那样看得一知半解，看完了还不知道怎么用，用在哪。不是说React Native中文网的文档一无是处，在有一定基础的时候在看文档或者在编写项目的时候参考一下文档是大有益处的。

## 第四步 React Native开源项目学习
React Native基础学习完了，就是时候了解一下一个完整的React Native项目是怎么编写的，通过React Native开源项目是最好的途径。
这里推荐几个齐全的总结：
[**React-Native学习指南**][5]：Github ✨8700+,不过作者有几个月没更新了，一些收集的内容也比较旧了，选择性的学习较新的内容，比较推荐**开源APP**部分
[**React Native 学习资源精选仓库(汇聚知识，分享精华)**][6]：Github ✨500+，作者更新也很勤快。
[**ReactNativeMaterials**][7]：Github ✨160+,分类齐全
[**React Native 优秀开源项目大全**][8]：Github ✨380+,分类齐全

### 开源项目参考学习的选择
综合考虑Github Star数和最后更新时间。

### 开源项目运行常见问题

 - **npm install 错误**

``` coffeescript
npm ERR! Darwin 15.6.0
npm ERR! argv "/usr/local/bin/node" "/usr/local/bin/npm" "install"
npm ERR! node v4.2.1
npm ERR! npm v2.14.7
npm ERR! code EPEERINVALID

npm ERR! peerinvalid The package react@16.0.0-alpha.6 does not satisfy its siblings' peerDependencies requirements!
....
....
```
这种情况通常是依赖库版本不兼容造成的。
	 - 删除node_modules文件夹，重新运行npm install
	 - 仔细查看版本要求提示，修改为要求的版本即可，有时候版本号通常会有个“^”，某些情况下去掉该符号并指定确定的版本也可以解决问题。 
	 
 - **react-native run-android出错**
   - 运行`react-native link`生成必要的关联工程
   - 确保Android模拟器或真机已连接，同时也可以使用Android Studio打开React Native工程的android目录，让Android Studio检查哪里有问题

## React Native项目开发IDE的选择

 - 如果曾经开发过Android，那么[**WebStorm**][9]是不二选择。用过Android Studio可以无缝切换到WebStorm。WebStorm需要激活，激活方法见此：[传送门][10]
 - 如果是前端开发者，可以使用[**Atom**][11]+**Nuclide**，具体可参考： [React Native开发之IDE（Atom+Nuclide）安装，运行，调试][12]
 



  [1]: http://www.runoob.com/react/react-tutorial.html
  [2]: http://www.jianshu.com/p/ebfeb687eb70#
  [3]: http://bbs.reactnative.cn/topic/15/react-react-native-%E7%9A%84es5-es6%E5%86%99%E6%B3%95%E5%AF%B9%E7%85%A7%E8%A1%A8
  [4]: http://www.lcode.org/react-native/
  [5]: https://github.com/reactnativecn/react-native-guide
  [6]: https://github.com/crazycodeboy/react-native-awesome
  [7]: https://github.com/LeoMobileDeveloper/ReactNativeMaterials
  [8]: https://github.com/MarnoDev/react-native-open-project
  [9]: https://www.jetbrains.com/webstorm/
  [10]: http://idea.lanyus.com/
  [11]: https://atom.io/
  [12]: http://blog.csdn.net/hello_hwc/article/details/51612139