# 多页动态生成UI管理组件 UIDynamicItemListMultipageHandler



## 描述

用于实现需要**翻页**展示的UI需求，通过持有一个`UIDynamicItemList`来切换显示当前页面的UI元素。

<u>使用前建议先阅读`UIDynamicItemList`类的说明文档。</u>

![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160123897.png)

<center>案例图</center>

## 属性

| **名字**                                                     | **类型**                                                     | **描述**                 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------ |
| [m_dynamic_item_list](https://boomingtech.feishu.cn/docx/W9AVdQTBIomvLixBw8JcM8rTnnd#part-K7asdygm5ogjaBx42fIcAeaqnFb) | [UIDynamicItemList](https://boomingtech.feishu.cn/docx/TnQBdXh1cogyogx0hgXciJvAnsd?from=from_copylink) | UI动态生成组件           |
| m_page_capacity                                              | int                                                          | 每页的元素数量           |
| m_cur_page_index                                             | int                                                          | 当前所在页数**（只读）** |

### m_dynamic_item_list

**类型：**`UIDynamicItemList`

**描述：**

UI生成需要借助一个UI动态生成组件来实现，使用时需要先初始化一个`UIDynamicItemList`组件，并赋值给该变量。

## 方法

| **方法名**                                                   | **描述**                       |
| ------------------------------------------------------------ | ------------------------------ |
| [setInitFuncAndDataArray(init_func, datas)](https://boomingtech.feishu.cn/docx/W9AVdQTBIomvLixBw8JcM8rTnnd#part-AcWvdBF0LoSlRBxIZ2acUSgenWb) | 设置子UI的初始化方法和数据集合 |
| [setPageIndex(idx)](https://boomingtech.feishu.cn/docx/W9AVdQTBIomvLixBw8JcM8rTnnd#part-Fgk4d96KgoVrC6xxbLXcWYEHngg) | 翻页至目标页                   |
| [updatePage()](https://boomingtech.feishu.cn/docx/W9AVdQTBIomvLixBw8JcM8rTnnd#part-A2U7dvSeOovUVVxEprYcYzTynld) | 刷新当前页                     |
| [nextPage()](https://boomingtech.feishu.cn/docx/W9AVdQTBIomvLixBw8JcM8rTnnd#part-S1crdoNfqoZTRCxINj0cScjvnrf) | 翻到下一页                     |
| [prePage()](https://boomingtech.feishu.cn/docx/W9AVdQTBIomvLixBw8JcM8rTnnd#part-QWCSdVQPJoAjCWxvFnCcA67hn5c) | 翻到上一页                     |
| [getMaxPage()](https://boomingtech.feishu.cn/docx/W9AVdQTBIomvLixBw8JcM8rTnnd#part-WElUdnjE7ostsTxjWE6cACEhnKh) | 获取最大页数                   |
| [iteratorCurrentPage()](https://boomingtech.feishu.cn/docx/W9AVdQTBIomvLixBw8JcM8rTnnd#part-BCI5dgFeTo3qtdxJsCJc1KE3nbh) | 当前页面元素迭代器             |
| [clear()](https://boomingtech.feishu.cn/docx/W9AVdQTBIomvLixBw8JcM8rTnnd#part-DK6cdtTDyoxh61xUtv1cgfVQnZg) | 清理方法                       |

### setInitFuncAndDataArray(init_func, datas)

**参数：**

```
init_func` 类型：`function
```

**返回值：**无

**描述：**

设置动态生成组件的初始化方法和数据集合。更新页面显示时组件会根据数据集合生成当前页面的UI，并且借助init_func初始化当前页面生成的UI。

### setPageIndex(idx)

**参数：**

```
idx` 类型：`int
```

**返回值：**无

**描述：**

翻页至目标页，会自动修正idx的值至**[1, 最大页数]**区间内。

### updatePage()

**参数：**无

**返回值：**无

**描述：**

刷新当前页显示，一般只在数据有更新时才需要调用。翻页时会自动刷新一次显示。

### nextPage()

**参数：**无

**返回值：**无

**描述：**

翻到下一页，在最后一页时不翻页。

### prePage()

**参数：**无

**返回值：**无

**描述：**

翻到上一页，在第一页时不翻页。

### getMaxPage()

**参数：**无

**返回值：**

`max_page_idx` 类型：`int`

**描述：**

获取最大页数。

### iteratorCurrentPage()

**参数：**无

**返回值：**

迭代方法

**描述：**

遍历当前页UI的迭代器。

类似DynamicArray类的iterator方法，用`for..in..`语句使用。

### clear()

**参数：**无

**返回值：**无

**描述：**

清理函数，清空设置，解除引用，**<u>并不会调用[m_dynamic_item_list](https://boomingtech.feishu.cn/docx/W9AVdQTBIomvLixBw8JcM8rTnnd#part-K7asdygm5ogjaBx42fIcAeaqnFb)的clear方法，需要手动清理</u>**。

## 示例

本示例重点在于展示`UIDynamicItemListMultipageHandler`的用法，其他相关组件和功能的使用方法请参考`DynamicItemList`和[UI通用鼠标事件](https://boomingtech.feishu.cn/docx/PDXsdIL4VoZxryxrOPdc7tZHnjb)的文档。

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
    
    -- 动态生成组件
    self.m_test_dynamic_ui_list = UIDynamicItemList:new(TestUI)
    -- 多页管理组件
    self.m_test_multy_page_handler = UIDynamicItemListMultipageHandler:new();
end


function TestWidget:registerImpl(root_ui)
    -- ============================== 动态生成组件配置 ==============================
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
    -- ============================== 动态生成组件配置 ==============================
    
    
    -- ============================== 多页管理组件配置 ==============================
    -- 设置动态生成组件引用
    self.m_test_multy_page_handler.m_dynamic_item_list = self.m_test_dynamic_ui_list;
    -- 设置一页显示的元素数量
    self.m_test_multy_page_handler.m_page_capacity = 10;
    -- ============================== 多页管理组件配置 ==============================
end


-- 根据传入的数据数组生成UI
-- data_info: DynamicArray<TestData >
function TestWidget:initTestMultyPageUIList(data_array)
    local data_array = DynamicArray:new(TestData);
    
    local init_func = function(new_ui, data)
        new_ui:setData(data);
    end
    
    -- 设置UI生成方法和数组集
    self.m_test_multy_page_handler:setInitFuncAndDataArray(init_func, data_array);
    -- 显示第一页UI
    self.m_test_multy_page_handler:setPageIndex(1);
end


-- 翻到上一页（可以通过点击某个按钮或按键触发）
function TestWidget:toPrePage()
    self.m_test_multy_page_handler:prePage();
end


-- 翻到下一页（可以通过点击某个按钮或按键触发）
function TestWidget:toNextPage()
    self.m_test_multy_page_handler:nextPage();
end


-- 直接翻到指定页数
function TestWidget:setPage(page_idx)
    self.m_test_multy_page_handler:setPageIndex(page_idx);
end


function TestWidget:clearImpl()
    -- 清理组件
    self.m_test_dynamic_ui_list:clear();
    self.m_test_multy_page_handler:clear();
end
```