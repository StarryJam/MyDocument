# UI通用事件机制



## 设计思路

### 设计目的

解耦页面和子控件的逻辑，减少页面的代码冗余，增加控件的复用性和通用性，可以以分治的方式处理页面逻辑，优化代码结构，提高项目可维护性。

### 现存问题分析

UI事件处理机制不完善，子控件的所有事件都发送到主页面的processUICallback函数处理，导致页面类代码量巨大，耦合严重，逻辑杂糅，另外还需要用传回的UI名映射来找到控件实例，这种依赖关系非常的不稳定且可能发生命名。更重要的问题是因为控件的行为都在页面类中实现，导致**控件难以复用**，例如鼠标悬浮在item_slot上时显示tooltips的功能，需要在每一个页面类中实现一次。

![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160106519.png)

<center>上千行的事件处理函数</center>



![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160106395.png)

分析下来比较重要的原因是触发UI事件缺少稳定的消息通路，首先**lua和flash中组件类的通信不是双向的**，暂时没有机制将flash组件类的事件直接转发到lua组件类中，所有事件都会被分发到页面层，所以最直接的做法就是直接在页面类中处理所有的事件；其次**组件上层（页面类）到组件也无法稳定通信**，没有一个好的机制让页面收到消息时可准确定位到对应的Lua控件，只能通过发送名字做暴力查找。

![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160107717.png)

### 设计方案

通过分配动态ID的方式关联Lua控件实例和Flash控件实例；定义一些通用事件字段，让一些通用事件可以从flash控件直接触发lua控件的行为，同时不影响原有的**flash控件->页面**的事件处理模式。

####  定义通用UI事件

 在UIControlBase中定义一些通用事件，例如OnMouseOver、OnMouseClick等，子组件只需要重写对应事件就轻松定义触发这些事件时的行为。

####  动态ID分配

 在UI组件注册的时候为组件分配动态ID，并在Dynamic-Instance Map中储存映射关系，并通过lua实例同步到关联的flash实例中；

![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160107652.png)

####  组件事件触发

 flash组件实例触发组件某些通用事件的时候，通过事件名和ID将事件发回lua，UIManager通过动态ID直接触发UIControlBase中的通用事件。

![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160107939.png)

## 相关API

该机制在原来的UIControlBase的基础上进行扩展，新增了一些接口，兼容了已有功能的使用。

基础功能文档：[UI管理系统](https://boomingtech.feishu.cn/wiki/wikcn5wjJIWscWq9B37NuiUIUed) 

新增API文档：[UIControlBase鼠标事件](https://boomingtech.feishu.cn/docx/PDXsdIL4VoZxryxrOPdc7tZHnjb) 

## 使用方法简介

使用方法的介绍会以新版商城中商品卡片的礼物赠送按钮做举例说明。

![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160107349.png)

有鼠标事件响应需求的Flash的控件需要继承类**chaos.UIControlBase**，或**chaos.ButtonBase**（如果是需要scaleform的按钮功能的组件）。

Lua中将控件类型声明为**UIControlBase**或其子类：

![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160107980.png)

然后在初始化时为其注册鼠标事件：

![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160107006.png)

详细使用方法见文档 [UIControlBase通用鼠标事件](https://boomingtech.feishu.cn/docx/PDXsdIL4VoZxryxrOPdc7tZHnjb) 