# UIMouseEventParam UI鼠标事件参数



## 描述

鼠标事件的参数类。

## 属性

| **名字**         | **类型**      | **描述**                            |
| ---------------- | ------------- | ----------------------------------- |
| m_ui_name        | string        | 触发事件的UI的名字                  |
| m_control        | UIControlBase | 触发事件的UI所属的UIControlBase对象 |
| m_ctrl_key       | bool          | 触发事件时是否按下了ctrl键          |
| m_alt_key        | bool          | 触发事件时是否按下了alt键           |
| m_shift_key      | bool          | 触发事件时是否按下了shift键         |
| m_local_position | Vecter2       | 触发事件的UI的local坐标             |

## 方法

| **方法名**                                                   | **描述**                 |
| ------------------------------------------------------------ | ------------------------ |
| [getMouseScreenPosition()](https://boomingtech.feishu.cn/docx/BS6ddWdmZoHm0nxhkrXcp32nnNM#part-I37ndER0toYwfLxLBrIcJsU0nFd) | 获取触发事件UI的屏幕坐标 |

### getMouseScreenPosition()

**参数：**无

**返回值：**

```
screen_position` 类型：`Vecter2
```

触发事件UI的屏幕坐标，常用于显示tooltips时调整位置。

**描述：**

该方法会返回触发事件UI的屏幕坐标，**开销较大**，不推荐在tick中频繁使用。