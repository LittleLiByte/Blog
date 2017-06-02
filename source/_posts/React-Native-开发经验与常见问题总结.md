---
title: React Native 开发经验与常见问题总结
date: 2017-06-02 17:09:06
tags:
	- react-native 
	- 经验
	- 常见问题
---
接触React Native才一个月有余，对这种“Learn once，Write anywhere”的移动端开发框架越来越喜爱。不敢说自己一个多月就掌握了React Native，这里只是将自己一个多月踩过的坑和一些经验心得以及教训分享记录于此，并且今后也会不断补充完善。

## 使用WebStorm开发React Native经验
**WebStorm**个人感觉真的是最好的ReactNative 开发工具之一，特别是当你有用过AndroidStudio的情况下，使用WebStorm绝对是最好的选择。
### 通过配置直接运行项目
<!-- more -->
![enter description here][1]
点击WebStorm右上角的小方框中的向下箭头，选中**Edit Configurations..**

![enter description here][2]
在弹出来的窗口中点击左上角的“**+**”按钮，下拉选项选择“**React Native**”

![enter description here][3]
右侧“name”项随意填写，自己能够区分就好，默认运行目标平台为Android，因此如果要运行的目标平台为iOS将Target Platform改为iOS即可。

![enter description here][4]

最后WebStorm可运行项目中如下

![enter description here][5]
选中Android，点击右侧的绿色三角形按钮就可在Android模拟器或真机运行ReactNative项目了，iOS同理。
注意，直接运行是在你的电脑上已经安装安卓模拟器或者Xcode的前提下,运行Android端时需提前启动好Android模拟器，否则会提示找不到设备，iOS则无需该操作。此外在WebStorm中运行最常见的问题是8081端口被占用

![enter description here][6]
此时需要在终端中通过lsof命令查找占用端口的进程PID 并通过kill 命令杀死进程。

![enter description here][7]
最好的避免方式是在需要停止运行，点击停止按钮来停止项目运行

![enter description here][8]

### 在WebStorm中调试
在编写的js文件中需要调试的地方打下断点，然后通过WebStorm运行项目，ReactNative运行在模拟器运行成功后中调出调试菜单，点击**Debug JS Remotely**（此处使用iOS作为示例，Android基本一致）

![enter description here][9]

此时就可以根据需要进行调试了。快捷键F8是单步调试，F7是步进调试
ps.在HTML标签块中打下的断点是无效的

![enter description here][10]

## ListView组件中的坑
如果你在ListView中renderHeader方法中加载图片，那么在Android端会无法显示，iOS端则不会有此问题。这是React Native中的一个Bug，还没有彻底解决，详见：
**[Render a ViewPagerAndroid in ListView.renderHeader() that the elements under ViewPagerAndroid will not display][11]**
目前暂时的解决方案如下：

``` vbscript-html
<ListView
  ...
  removeClippedSubviews={false}
/>
```
 removeClippedSubviews 属性，表示如果子 View 超出可视区域，是否自动移除，默认是 true,因此设置为false显然是牺牲了一定的性能。不过目前的做法是使用FlatList替代ListView，因为FlatList比ListView提升了很多（不过FlatList是否同样有这个Bug还未测试）

## TextInput组件中的坑
TextInput在iOS端必须设置**height**属性，否则无法获取焦点，Android端可不设置。

## 关于布局与屏幕适配的一点经验

 - 基本上所有组件都最好不指定宽高（图片除外，必须指定宽高否则显示图片原大小，不好控制），不指定宽高的组件将保持原大小，这点类似Android中的wrap_content.
 - React Native组件大小的数值单位是**dp**，不需要再进行转换

### flex
style中的**flex**是一个很有用的属性，有点类似Android中layout_weight属性

 - 在根布局中指定**flex：1**则表示占满全屏（view默认宽度为100%）
 - 如果一层布局中有多个组件其中只有一个组件指定了**flex:1**则表示该组件将占据其余组件外所剩余的空间
 - 如果一层布局中有多个组件指定了**flex:1**,则表示这些组件所占比相同

### 布局方向flexDirection
ReactNative默认布局方向为纵向column，此时justifyContent指定纵向对齐方式，alignItems指定横向对齐方式
横向需指定为row，justifyContent指定横向对齐方向，alignItems指定纵向对齐方式

### Image的调整模式resizeMode
resizeMode属性可调整图片相对于容器的展现方式，默认为cover模式。

## 页面卡顿的一个解决办法
InteractionManager.runAfterInteractions 方法可以让我们在页面的所有动画执行完后再获取数据
``` javascript
 componentDidMount() {
        InteractionManager.runAfterInteractions(() => {
            this._getListData().done()
        })
    }
```


> **cover模式**只求在显示比例不失真的情况下填充整个显示区域。可以对图片进行放大或者缩小，超出显示区域的部分不显示，
> 也就是说，图片可能部分会显示不了。
>  **contain模式**是要求显示整张图片, 可以对它进行等比缩小,
> 图片会显示完整,可能会露出Image控件的底色。 如果图片宽高都小于控件宽高，则不会对图片进行放大。
> **stretch模式**不考虑保持图片原来的宽,高比.填充整个Image定义的显示区域,这种模式显示的图片可能会畸形和失真。
>  **center模式,** 9月11号的0.33版本才支持，contain模式基础上支持等比放大。

## 其他，以后会继续补充完善...
  [1]: http://om2bpqram.bkt.clouddn.com/1496385059159.jpg "1496385059159.jpg"
  [2]: http://om2bpqram.bkt.clouddn.com/1496385172176.jpg "1496385172176.jpg"
  [3]: http://om2bpqram.bkt.clouddn.com/1496385351796.jpg "1496385351796.jpg"
  [4]: http://om2bpqram.bkt.clouddn.com/1496385541594.jpg "1496385541594.jpg"
  [5]: http://om2bpqram.bkt.clouddn.com/1496385604021.jpg "1496385604021.jpg"
  [6]: http://om2bpqram.bkt.clouddn.com/1496386478708.jpg "1496386478708.jpg"
  [7]: http://om2bpqram.bkt.clouddn.com/1496386543687.jpg "1496386543687.jpg"
  [8]: http://om2bpqram.bkt.clouddn.com/1496387018926.jpg "1496387018926.jpg"
  [9]: http://om2bpqram.bkt.clouddn.com/1496387265719.jpg "1496387265719.jpg"
  [10]: http://om2bpqram.bkt.clouddn.com/1496387859119.jpg "1496387859119.jpg"
  [11]: https://github.com/facebook/react-native/issues/4455