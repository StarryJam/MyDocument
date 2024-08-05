# UIControlBase通用鼠标事件



## 描述

在UIControlBase的基础上拓展的一套处理**鼠标事件**的方法，相比于老的通过页面接收事件处理具体业务逻辑的流程能更好实现逻辑之间的**解耦合**和代码复用，从而优化代码结构，同时也**兼容老的流程**。另外一个好处是加新的鼠标行为基本上可以不用重新导出flash了(╯‵□′)╯︵┻━┻。

除了可以让外部订阅ui元件的鼠标事件之外，也加入了一些**内部事件回调**（onMouseOver等）方法使得元件可以直接响应自身的鼠标事件，将自身固有的逻辑可以封装在元件内部。



<u>设计思路在另一篇文档中：[UI通用事件机制](UI通用事件机制.md)</u> 



<u>**下面这段内容非常重要：**</u>

如果要使用这一套机制，需要flash中对应的控件的AS类型的**基类**指定为`chaos.UIControlBase`（如果需要有按钮行为则改为`chaos.ButtonBase`，美术做的时候一般会用`scaleform.clik.controls.Button`），并且在Lua中将元件声明为`UIControlBase`（或其子类）类型。

![image](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160110539.png)

<center>AS类型指定</center>



![image](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160139045.png)

<center>Lua中元件的类型声明</center>

## 方法`

| [UIControlBase:addUIEventListener(event_type, instance, call_back)](#addUIEventListener) | 为控件注册鼠标事件                           |
| ------------------------------------------------------------ | -------------------------------------------- |
| [UIControlBase:removeUIEventListener(event_type, instance, call_back)](#removeUIEventListener) | 移除注册的鼠标事件                           |
| [UIControlBase:reset()](#reset)                              | 重置元件，会移除所有已注册的鼠标事件回调函数 |

<span id = internal_callback><u>**以下为内部事件回调方法（一般情况下不要在外部调用，子类可以通过重写来改变行为）**</u></span>

| [UIControlBase:onMouseOver(event_param)](#onMouseOver) | 元件自身鼠标移入回调         |
| ------------------------------------------------------ | ---------------------------- |
| UIControlBase:onMouseOut(event_param)                  | 元件自身鼠标移出回调         |
| UIControlBase:onMouseClick(event_param)                | 元件自身鼠标点击回调         |
| UIControlBase:onMouseDown(event_param)                 | 元件自身鼠标按下回调         |
| UIControlBase:onMouseUp(event_param)                   | 元件自身鼠标抬起回调         |
| UIControlBase:onMouseWheelDown(event_param)            | 元件自身鼠标滚轮向下滚动回调 |
| UIControlBase:onMouseWheelUp(event_param)              | 元件自身鼠标滚轮向上滚动回调 |

### <span id = addUIEventListener>UIControlBase:addUIEventListener(event_type, instance, call_back)</span>

**参数：**

` event_type` 类型：`UIEventType` 

事件类型

`instance` 类型：`Object`

回调方法所属的实例，即回调方法中的**self所指的对象**。

`call_back_fun` 类型：`Function`

事件的回调函数，函数会接受到两个参数{ui_instance, event_param}，前者为触发事件的UI对象，后者为事件的相关参数结构体，类型为[`UIMouseEventParam`](UIMouseEventParam_UI鼠标事件参数.md)。

示例函数：

```Lua
function TestClass:testCallbackFunc(clicked_ui, event_param)
    clicked_ui:setVisible(k_false);
    local is_press_shift_key = event_param.m_shift_key;
    local screen_pos = event_param:getMouseScreenPosition();
    print("is_press_shift_key", is_press_shift_key, "screen_pos", screen_pos);
end
```

**返回值：** 无

**描述：**

为控件注册鼠标事件回调函数。



### <span id = removeUIEventListener>UIControlBase:removeUIEventListener(event_type, instance, call_back)</span>

**参数：**

`event_type` 类型：`UIEventType`

事件类型

`instance` 类型：`Object`

回调方法所属的实例，即回调方法中的**self所指的对象**。

`call_back_fun` 类型：`Function`

移除的事件回调函数。

**返回值：** 无

**描述：**

用于移除已注册的指定的鼠标事件回调函数。



### <span id = reset>UIControlBase:reset()</span>

**参数：** 无

**返回值：** 无

**描述：**

重置元件，会移除所有已注册的鼠标事件回调函数。**不会递归重置子元件**，以保证内部封装功能的完整性。



### <span id = onMouseOver>UIControlBase:onMouseOver(event_param)</span>

<u>[【内部事件回调】](#internal_callback)</u>

**参数：**

`event_param` 类型：[`UIMouseEventParam`](UIMouseEventParam_UI鼠标事件参数.md)

鼠标事件相关参数

**返回值：**无

**描述：**

鼠标进入时触发的事件回调，常与`onMouseOut`方法配合用于显示tooltips。

内部事件回调的触发会先于外部注册的事件回调。

![20240130165607_rec_-convert](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160110919.gif)



其他事件函数功能都差不多就不赘述了。

## 枚举类型

### UIEventType

**定义**

UI通用事件枚举

**值**

`UIEventType.MOUSE_OVER` 鼠标悬浮事件

`UIEventType.MOUSE_OUT` 鼠标移出事件

`UIEventType.MOUSE_CLICK` 鼠标点击事件

`UIEventType.MOUSE_UP` 鼠标抬起事件

`UIEventType.MOUSE_DOWN` 鼠标按下事件

`UIEventType.MOUSE_WHEEL_UP` 鼠标滚轮向上滚动事件

`UIEventType.MOUSE_WHEEL_DOWN` 鼠标滚轮向下滚动事件

## 示例

下面的示例实现文本框显示按钮被点击次数的功能。

```Lua
-- 按钮类，需要继承UIControlBase
TestButton = ClassDefinition(TestButton, UIControlBase)
function TestButton:ctor()
    self.m_click_count = 0;
end

-- 内部鼠标事件回调
function TestButton:onMouseClick(event_param)
    -- 计数+1
    self.m_click_count = self.m_click_count + 1;
end


-- 页面类
TestWidget = ClassDefinition(TestWidget, UIWidgetBase)
function TestWidget:ctor()
    self.m_ui_names =
    {
        -- 文本框
        m_text = "m_text"
    }
    
    local sub_ui_definition =
    {
        -- 按钮
        m_test_button = {class_definition = TestButton, path = "m_test_button"},
    }
    
    self:createDefaultValue();
end

-- 页面初始化函数
function TestWidget:registerImplImpl(root_ui)
    -- 注册鼠标事件回调
    self.m_test_button:addUIEventListener(UIEventType.MOUSE_OVER, self, self.onButtonClick);
end

-- 点击按钮的回调
function TestWidget:onButtonClick(button, event_param)
    -- 更新点击次数
    self.m_text:setText(button.m_click_count);
end
```
