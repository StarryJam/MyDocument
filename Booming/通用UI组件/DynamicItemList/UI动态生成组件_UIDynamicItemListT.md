# UI动态生成组件 UIDynamicItemList\<T>

## 描述

用于动态创建和排列同种类型的UI元素，例如商店的商品卡片，任务列表的任务描述等，该组件可以快速实现此类在游戏中非常常见的需求。

![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160113691.png)

<center>案例图</center>

## 属性

| **名字**                                          | **类型**                                                     | **描述**                                   |
| ------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------ |
| [m_panel](#m_panel)                               | UIValue                                                      | 生成动态元素的父节点                       |
| [m_arrangement_state](#m_arrangement_state)       | [UIDynamicItemListArrangementState](#UIDynamicItemListArrangementState) | 元素的排列方式（纵向或横向）               |
| [m_horizontal_diraction](#m_horizontal_diraction) | [UIDynamicItemListHorizontalDiraction](#UIDynamicItemListHorizontalDiraction) | 水平排列方向                               |
| [m_vertical_diraction](#m_vertical_diraction)     | [UIDynamicItemListVerticalDiraction](#UIDynamicItemListVerticalDiraction) | 垂直排列方向                               |
| [m_item_stacking_mode](#m_item_stacking_mode)     | [UIDynamicItemListStackingMode](#UIDynamicItemListStackingMode) | 主排列方向上的堆叠方式                     |
| m_column_limit                                    | int                                                          | 列数限制，超出自动换行（仅横向排列可用）   |
| m_raw_limit                                       | int                                                          | 行数限制，超出自动换行（仅纵向排列可用）   |
| m_pending_x                                       | int                                                          | 元素水平间隔                               |
| m_pending_y                                       | int                                                          | 元素垂直间隔                               |
| m_item_name                                       | string                                                       | 生成的元素名字（生成时会自动拼上序号后缀） |

### <span id = m_panel>m_panel</span>

**类型：** `UIValue`

**描述：**

生成动态元素的父节点，应当设置为flash中一个空的节点，否则可能在某些情况下影响排列的最终效果。



### <span id = m_arrangement_state>m_arrangement_state</span>

**类型：** `UIDynamicItemListArrangementState`

**描述：**

设置元素纵向或横向排列，默认值为`UIDynamicItemListArrangementState.horizontal`（横向排列）。



### <span id = m_horizontal_diraction>m_horizontal_diraction</span>

**类型：** `UIDynamicItemListHorizontalDiraction`

**描述：**

元素水平排列方向的方向，**默认值**为`UIDynamicItemListHorizontalDiraction.right` （从左往右）。

在**从左向右**和**从右向左**排列的情况下，排列的**起始位置**都为父节点的位置；在**居中排列**的情况下，会将所有元素对称排列到父节点**(0, 0)**位置的两侧。

下面三张图分别为三种设置的效果，红点为父节点的位置：

![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160113336.png)

<center>center</center>

![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160113357.png)

<center>right</center>

![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160113351.png)

<center>left</center>

<u>**注：** 在列表为纵向排列时该属性仍然会影响元素在排列时的**起始位置**和**多列时的排列效果**。</u>





### <span id = m_vertical_diraction>m_vertical_diraction</span>

**类型：** `UIDynamicItemListVerticalDiraction`

**描述：**

元素水平排列方向的方向，**默认值**为`UIDynamicItemListHorizontalDiraction.down` （自上而下）。

和[m_horizontal_diraction](#m_horizontal_diraction)是对称的。





### <span id = m_item_stacking_mode>m_item_stacking_mode</span>

**类型：** `UIDynamicItemListStackingMode`

**描述：**

主排列方向上的堆叠方式，**默认值**为`UIDynamicItemListStackingMode.overlap`。

下图示中前者堆叠方式为overlap，后者为pushing，pedding都是10：

![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160113403.png)

<center>overlap</center>

![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160113610.png)

<center>pushing</center>

通常情况，在默认的overlap模式下，将pedding设置为元素本身的尺寸+元素间距，和使用pushing模式并将pedding直接为想要的元素间距效果是相同的。但在某些特殊情况，比如元素大小可能是不确定的，或是下图展示的UI动画效果，使用pushing模式会是更好得选择。

**由于flash元件的尺寸经常会由于一些不可见的脏东西捉摸不定**，如果设计尺寸不会经常变动或者元件本身大小动态变化的话还是推荐用overlap+手动设置元素偏移，这样会更可控一些。

![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160113971.gif)

<center>UI动效</center>

<u>**注：** 目前的设计上 m _item_stacking_mode 仅生效于主要排列方向上，换行（列）时的间隔不会受到元素尺寸的影响。</u>









## 方法

| **方法名**                                                   | **描述**                       |
| ------------------------------------------------------------ | ------------------------------ |
| [ctor(item_class_define, action_script_class_name)](#ctor)   | 构造函数                       |
| [initListByInitFunctionAndDatas(init_func, data_array)](#initListByInitFunctionAndDatas) | 根据生成函数和数据生成若干元素 |
| [arrangeUI()](#arrangeUI)                                    | 重新排列UI                     |
| [iteratorUsed()](#iteratorUsed)                              | 正在使用的UI迭代器             |
| [iteratorAll()](#iteratorAll)                                | 所有UI元素的迭代器             |
| [reset()](#reset)                                            | 重置组件                       |
| [clear()](#clear)                                            | 清理函数                       |

### <span id = ctor>ctor(item_class_define, action_script_class_name)</span>

**参数：**

` item_class_define` 类型：`ClassDefinition` 

生成的UI类型

` action_script_class_name` 类型：`string` 

【可选参数】用于绑定或重绑定flash中的as类型

**返回值：**

无

**描述：**

构造函数需要绑定生成的UI类型，并且可以用可选参数绑定或重绑定指定的flash组件的as类，可选参数只在Lua类本身没有绑定as类或需要重新指定as类型的情况才需要传入。



### <span id = initListByInitFunctionAndDatas>initListByInitFunctionAndDatas(init_func, data_array)</span>

**参数：**

` init_func` 类型：`function` 

生成函数，方法将接受一个集合中的数据和生成的UI实例，通过该方法使用数据将生成的UI初始化。

` data_array` 类型：`DynamicArray` 

数据集合

**返回值：** 无

**描述：**

组件初始化完成后，可以使用此方法传入一个任意大小的数据集合（暂时就只支持` DynamicArray` 类型了，有需要后续可以优化成支持所有容器类），组件会生成对应数量的UI（[补充说明](https://boomingtech.feishu.cn/docx/TnQBdXh1cogyogx0hgXciJvAnsd#part-LskVdCjuco9CGIxTKFZckoqrnhd)），并且借助` init_func` 初始化每一个UI。

具体用法可见[示例](#example)。



### <span id = arrangeUI>arrangeUI()</span>

**参数：** 无

**返回值：** 无

**描述：**

重新排列UI，在生成UI时会自动调用一次，仅在排版动态变化的情况下需要手动调用。



### <span id = iteratorUsed>iteratorUsed()</span>

**参数：** 无

**返回值：**

迭代方法

**描述：**

遍历正在被使用的元素的迭代器。

类似` DynamicArray` 类的iterator方法，用`for..in..`语句使用。



### <span id = iteratorAll>iteratorAll()</span>

**参数：** 无

**返回值：**

迭代方法

**描述：**

遍历正在所有元素的迭代器，包括对象池（见[补充说明](#tips)）中的元素，一般情况下不会使用到这个方法。

类似` DynamicArray` 类的iterator方法，用`for..in..`语句使用。



### <span id = reset>reset()</span>

**参数：** 无

**返回值：** 无

**描述：**

清空元素列表（不会销毁元素）。



### <span id = clear>clear()</span>

**参数：** 无

**返回值：** 无

**描述：**

清理函数，销毁所有的元素，在持有该组件的UI销毁时需要调用该方法。

## 枚举类型

### <span id = UIDynamicItemListArrangementState>UIDynamicItemListArrangementState</span>

**定义：**

元素的排列方式（纵向或横向）

**值：**

`UIDynamicItemListArrangementState.horizontal` 横向排列

`UIDynamicItemListArrangementState.vertical` 纵向排列



### <span id = UIDynamicItemListHorizontalDiraction>UIDynamicItemListHorizontalDiraction</span>

**定义：**

水平排列方向

**值：**

`UIDynamicItemListHorizontalDiraction.left` 从右向左排列

`UIDynamicItemListHorizontalDiraction.right` 从左向右排列

`UIDynamicItemListHorizontalDiraction.center` 居中排列



### <span id = UIDynamicItemListVerticalDiraction>UIDynamicItemListVerticalDiraction</span>

**定义：**

水平排列方向

**值：**

`UIDynamicItemListVerticalDiraction.up`自下而上排列

`UIDynamicItemListVerticalDiraction.down` 从上往下排列

`UIDynamicItemListVerticalDiraction.center` 居中排列



### <span id = UIDynamicItemListStackingMode>UIDynamicItemListStackingMode</span>

**定义：**

排列方向上的堆叠方式

**值：**

` UIDynamicItemListStackingMode.overlap` 

不考虑元素的大小，下一个元素的位置为上一个元素的位置+pedding

` UIDynamicItemListStackingMode.pushing` 

考虑元素的大小，下一个元素的位置为上一个元素的位置+上一个元素的尺寸+pedding





## <span id = example>示例</span>

```Lua
-- 数据类
TestData = ClassDefinition(TestData)
function TestData:ctor(name, desc)
    self.m_name = name; -- string
    self.m_desc = desc; -- string
end

-- 动态生成的UI类
TestUI = ClassDefinition(TestUI, UIControlBase)
function TestUI:ctor()
    self.m_ui_names =
    {
        m_name_text = "m_name_text",
        m_desc_text = "m_desc_text"
    }
    
    self:createDefaultValue();
    
    self.m_action_script_class_name = "chaos.TestUI";
end

function TestUI:setData(data)
    self.m_name_text:setText(data.m_name);
    self.m_desc_text:setText(data.m_desc);
end



-- 页面类
TestWidget = ClassDefinition(TestWidget, UIWidgetBase)
function TestWidget:ctor()
    self.m_ui_names =
    {
        m_panel = "m_panel"
    }
    
    self:createDefaultValue();
    
    self.m_test_dynamic_ui_list = UIDynamicItemList:new(TestUI)
    
end

function TestWidget:registerImpl(root_ui)
    -- 设置父节点（需要在UI初始化完成后再设置panel）
    self.m_test_dynamic_ui_list.m_panel = self.m_panel;
    -- 设置排列方向
    self.m_test_dynamic_ui_list.m_arrangement_state = UIDynamicItemListArrangementState.horizontal;
    -- 设置堆叠模式
    self.m_test_dynamic_ui_list.m_item_stacking_mode = UIDynamicItemListStackingMode.pushing；
    -- 设置元素间隔
    self.m_test_dynamic_ui_list.m_pending_x = 10;
    self.m_test_dynamic_ui_list.m_pending_y = 110;
    -- 设置单行元素限制
    self.m_test_dynamic_ui_list.m_column_limit = 5;
    -- 设置生成UI的名字前缀
    self.m_item_name = "test_ui";
end

-- 生成UI的方法
function TestWidget:initTestUIList()
    local data_array = DynamicArray:new(TestData);
    local data_1 = TestData("ui_1", "this is ui No.1");
    local data_2 = TestData("ui_2", "this is ui No.2");
    data_array:add(data_1);
    data_array:add(data_2);
    
    local init_func = function(new_ui, data)
        new_ui:setData(data);
    end
    
    -- 将根据数据数组生成两个ui
    self.m_test_dynamic_ui_list:initListByInitFunctionAndDatas(init_func, data_array);
end

function TestWidget:clearImpl()
    -- 清理组件
    self.m_test_dynamic_ui_list:clear();
end
```









## <span id = tips>补充说明</span>

- 该类中的横纵轴相关属性含义以及实现方式是完全**对称**的，详情可见源码中 ` UIDynamicItemList:arrangeUI()`  方法。
- 动态生成的UI会以类似**对象池**的方式被保留，若下次生成的数量小于或等于已有的UI数，不会再生成新的UI，而只是用新的数据初始化UI，多余的UI将被隐藏，因此重复动态生成UI的开销并不大。
- 注意和flash**滚动列表元件`scroll_item_list`**的互动可能会引发的问题，因为`scroll_item_list`初始化时会将目标panel的位置设置到(0, 0)点位置
- 关于**老UI的适配**问题：动态生成UI需要调用UI的**`initializeFromParent`**方法，由于老UI的生命周期管理并不统一，可能会出现没有对应方法或者参数不一致的问题，如果在老UI上使用此组件可能需要对应做一定的改动。
