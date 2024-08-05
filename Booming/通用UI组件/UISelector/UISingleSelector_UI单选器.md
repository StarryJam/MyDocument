# UISingleSelector UI单选器

## 描述

用于游戏中常见的在一个UI集合中单选某一元素的需求，例如商城的页签切换、服装商城展示选中时装等。实际场景大多可以配合`UIControlBase`类的鼠标事件（触发选中）和`UIDynamicItemList`动态生成组件（动态生成可选元素）一起构建UI。

![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160134581.png)

<center>三块红框中的部分均使用到了单选器</center>

## 继承关系

<u>UISingleSelector</u> -> [UISelectorBase](UISelectorBase.md)

## 属性

| **名字**                                                     | **类型** | **描述**               |
| ------------------------------------------------------------ | -------- | ---------------------- |
| [m_allow_duplicate_select](UISelectorBase.md/#target-anchor) | bool     | 是否允许元素被重复选择 |

### m_allow_duplicate_select

<span id="m_allow_duplicate_select"></span>

**类型：** `bool`

**描述：**

是否允许元素被重复选择，如果为true则重复选中相同的元素也会触发选中事件。默认值为false。

## 方法

| **方法名**                                                   | **描述**     |
| ------------------------------------------------------------ | ------------ |
| [operateItem(item)](https://boomingtech.feishu.cn/docx/Jqh1dhGMGoI00Gxip1WcetIbnyb#part-HZkrd6OOEoYsiWxY0dhcKT5GnXe) | 操作元素     |
| clearSelect()                                                | 清除当前选则 |

**继承的方法**

| **方法名**                                                   | **描述**               |
| ------------------------------------------------------------ | ---------------------- |
| [addSelectable(selectable)](https://boomingtech.feishu.cn/docx/CPqJdMB59onxyyxpUFmcnB05nvg#part-RDj7dyLYWowVZjxTFGqczXuanDb) | 添加可选元素           |
| [removeSelectable(selectable)](https://boomingtech.feishu.cn/docx/CPqJdMB59onxyyxpUFmcnB05nvg#part-VbNnd365Go4tDPxbaM4cwNwonrU) | 删除可选元素           |
| [removeAllSelectable()](https://boomingtech.feishu.cn/docx/CPqJdMB59onxyyxpUFmcnB05nvg#part-Hot2dk1OFor5OXxOrttcAHaknad) | 清空可选元素           |
| [addSelectCallback(instance, call_back)](https://boomingtech.feishu.cn/docx/CPqJdMB59onxyyxpUFmcnB05nvg#part-Be8DdvPLxo2d5mxt6iiclIWnnxh) | 注册选中UI事件回调     |
| [removeSelecCallback(instance, call_back)](https://boomingtech.feishu.cn/docx/CPqJdMB59onxyyxpUFmcnB05nvg#part-I8GbdKsZjotdkyxccZtcsVyZnub) | 移除选中UI事件回调     |
| [addDeselectCallback(instance, call_back)](https://boomingtech.feishu.cn/docx/CPqJdMB59onxyyxpUFmcnB05nvg#part-Xwredz1qbo7wGKxwt79cUpWknxq) | 注册取消选中UI事件回调 |
| [removeDeselecCallback(instance, call_back)](https://boomingtech.feishu.cn/docx/CPqJdMB59onxyyxpUFmcnB05nvg#part-Q5iodd8kxo2Cjsxb7dhcoNnEnwd) | 移除取消选中UI事件回调 |
| [selectDefault()](https://boomingtech.feishu.cn/docx/CPqJdMB59onxyyxpUFmcnB05nvg#part-TRgddFNZnoWVt2xJzjVcbfbKnlb) | 选中默认选项           |
| clear()                                                      | 清理函数               |

### operateItem(item) 

**参数：**

```
item` 类型：`ISelectable
```

目标UI，必须已经被添加进可选列表中。

**返回值：**

无

**描述：**

操作元素，可以理解为鼠标点击一次目标元素。该元素必须已经被添加进可选列表中。

## 示例

示例展示了如何用元素和选择器创建一个可选列表，并且**可选元素自身和页面都可以响应选中事件**。

```Lua
-- 可选对象继承UISelectableButton，实现了可选类接口，且会切换目标的select状态帧（如果在flash中制作了的话）
TestSelectableItem = ClassDefinition(TestButton, UISelectableButton)
function TestSelectableItem:ctor()

end

-- 选中事件
function TestButton:onSelectImpl()
    print("TestButton:onDeselectImpl" .. self.m_name .. " selected");
end

-- 取消选中事件
function TestButton:onDeselectImpl()
    print("TestButton:onDeselectImpl" .. self.m_name .. " deselected");
end


-- 页面类
TestWidget = ClassDefinition(TestWidget, UIWidgetBase)
function TestWidget:ctor()
    self.m_ui_names =
    {
    
    }
    
    local sub_ui_definition =
    {
        m_item_1 = {class_definition = TestSelectableItem , path = "item_1"},
        m_item_2 = {class_definition = TestSelectableItem , path = "item_2"},
        m_item_3 = {class_definition = TestSelectableItem , path = "item_3"},
    }
    
    self:createDefaultValue();
    
    -- 单选器
    self.m_item_selector = UISingleSelector:new();
end

-- 页面初始化函数
function TestWidget:registerImplImpl(root_ui)
    -- 将元素添加进选择器
    self.m_item_selector:addSelectable(self.m_item_1);
    self.m_item_selector:addSelectable(self.m_item_2);
    self.m_item_selector:addSelectable(self.m_item_3);
    
    -- 通过UI鼠标事件触发选择器的选中元素
    self.m_item_1:addUIEventListenerUIEvent(Type.MOUSE_CLICK, self.m_item_selector, self.m_item_selector.operateItem);
    self.m_item_2:addUIEventListenerUIEvent(Type.MOUSE_CLICK, self.m_item_selector, self.m_item_selector.operateItem);
    self.m_item_3:addUIEventListenerUIEvent(Type.MOUSE_CLICK, self.m_item_selector, self.m_item_selector.operateItem);

    -- 为页面注册选中（反选）元素事件回调
    self.m_item_selector:addSelectCallback(self, self.onItemSelected);
    self.m_item_selector:addDeselectCallback(self, self.onItemDeselected);

    self.m_item_selector:selectDefault();
end

-- 页面的选中回调
function TestWidget:onItemSelected(selected_item)
    print("TestWidget:onItemSelected" .. selected_item.m_name .. " selected");
end

-- 页面的取消选中回调
function TestWidget:onItemDeselected(deselected_item)
    print("TestWidget:onItemDeselected" .. deselected_item.m_name .. " deselected");
end
```