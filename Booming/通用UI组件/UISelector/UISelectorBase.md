# UISelectorBase

## 描述

UI选择器的基类。

## 继承关系

**子类**

[UISingleSelector](https://boomingtech.feishu.cn/docx/Jqh1dhGMGoI00Gxip1WcetIbnyb)

UIMultiSelector（还没写😅）

## 方法

| **方法名**                                                   | **描述**                                 |
| ------------------------------------------------------------ | ---------------------------------------- |
| [addSelectable(selectable)](https://boomingtech.feishu.cn/docx/CPqJdMB59onxyyxpUFmcnB05nvg#part-RDj7dyLYWowVZjxTFGqczXuanDb) | 添加可选元素                             |
| [removeSelectable(selectable)](https://boomingtech.feishu.cn/docx/CPqJdMB59onxyyxpUFmcnB05nvg#part-VbNnd365Go4tDPxbaM4cwNwonrU) | 删除可选元素                             |
| [removeAllSelectable()](https://boomingtech.feishu.cn/docx/CPqJdMB59onxyyxpUFmcnB05nvg#part-Hot2dk1OFor5OXxOrttcAHaknad) | 清空可选元素                             |
| [addSelectCallback(instance, call_back)](https://boomingtech.feishu.cn/docx/CPqJdMB59onxyyxpUFmcnB05nvg#part-Be8DdvPLxo2d5mxt6iiclIWnnxh) | 注册选中UI事件回调                       |
| [removeSelecCallback(instance, call_back)](https://boomingtech.feishu.cn/docx/CPqJdMB59onxyyxpUFmcnB05nvg#part-I8GbdKsZjotdkyxccZtcsVyZnub) | 移除选中UI事件回调                       |
| [addDeselectCallback(instance, call_back)](https://boomingtech.feishu.cn/docx/CPqJdMB59onxyyxpUFmcnB05nvg#part-Xwredz1qbo7wGKxwt79cUpWknxq) | 注册取消选中UI事件回调                   |
| [removeDeselecCallback(instance, call_back)](https://boomingtech.feishu.cn/docx/CPqJdMB59onxyyxpUFmcnB05nvg#part-Q5iodd8kxo2Cjsxb7dhcoNnEnwd) | 移除取消选中UI事件回调                   |
| [selectDefault()](https://boomingtech.feishu.cn/docx/CPqJdMB59onxyyxpUFmcnB05nvg#part-TRgddFNZnoWVt2xJzjVcbfbKnlb) | 选中默认选项                             |
| [operateItem(item)](https://boomingtech.feishu.cn/docx/CPqJdMB59onxyyxpUFmcnB05nvg#part-X748d58OyoFsB6xNIeEcC9SQnNe) | 操作元素，可以理解为鼠标点击一次目标元素 |
| clear()                                                      | 清理函数                                 |

### addSelectable(selectable)

**参数：**

```
selectable` 类型：`ISelectable
```

添加的可选UI。

**返回值：**

无

**描述：**

向选择器中添加一个可选UI。**该UI必须有[ISelectable](https://boomingtech.feishu.cn/docx/Jqh1dhGMGoI00Gxip1WcetIbnyb#part-Og3Dd5P7XoRiiNxntiLcvK6cncd)定义的所有方法**。

### removeSelectable(selectable)

**参数：**

```
selectable` 类型：`ISelectable
```

删除的可选UI。

**返回值：**

无

**描述：**

向选择器中删除一个可选UI。

### removeAllSelectable()

**参数：**无

**返回值：**

无

**描述：**

删除选择器可选列表中所有可选UI。

### addSelectCallback(instance, call_back)

**参数：**

```
instance` 类型：`Object
```

回调方法所属的实例，即回调方法中的**self所指的对象**。

```
call_back_fun` 类型：`Function
```

选中元素事件的回调函数，函数会接受选中元素作为参数。

**返回值：**

无

**描述：**

注册选中UI事件的回调函数。

### addDeselectCallback(instance, call_back)

**参数：**

```
instance` 类型：`Object
```

回调方法所属的实例，即回调方法中的**self所指的对象**。

```
call_back_fun` 类型：`Function
```

取消选中元素事件的回调函数，函数会接受被取消选中的元素作为参数。

**返回值：**

无

**描述：**

注册反选UI事件的回调函数。

### removeSelectCallback(instance, call_back)

**参数：**

```
instance` 类型：`Object
```

回调方法所属的实例，即回调方法中的**self所指的对象**。

```
call_back_fun` 类型：`Function
```

移除的事件的回调函数。

**返回值：**

无

**描述：**

移除选中UI事件的回调函数。

### removeDeselectCallback(instance, call_back)

**参数：**

```
instance` 类型：`Object
```

回调方法所属的实例，即回调方法中的**self所指的对象**。

```
call_back_fun` 类型：`Function
```

移除的事件的回调函数。

**返回值：**

无

**描述：**

移除反选UI事件的回调函数。

### selectDefault()

**参数：**

无

**返回值：**

无

**描述：**

选中默认选项（列表不为空的话选中第一个元素）。

### operateItem(item)  

【virtual】

**参数：**

```
item` 类型：`ISelectable
```

目标UI，必须已经被添加进可选列表中。

**返回值：**

无

**描述：**

操作元素，可以理解为鼠标点击一次目标元素。该元素必须已经被添加进可选列表中。**具体行为在子类中定义**。

## 接口

项目中lua的面向对象框架并不支持接口继承，所以这里的接口定义**只是约定参数传入的对象必须要有某些属性和方法**，否则相关功能会报错，但在**代码中并没有定义接口类型**。

### ISelectable

**定义**

可被选择的对象

#### **接口方法**

##### getSelectHandler()

**参数：**无

**返回值：**

```
select_handler` 类型：`UISelectHandler
```

**描述：**

返回一个UISelectHandler类型的对象

##### onSelect()

**参数：**无

**返回值：**无

**描述：**

被选中时触发的行为。

##### onDeselect()

**参数：**无

**返回值：**无

**描述：**

被取消选中时触发的行为。