# 新手BP以及其他活动BP表格配置文档



## 总览

**该文档只涉及关于活动BP新增功能部分的表格配置说明，之前已有的任务或奖励兑换的配表流程会略过。**



挑战部分（获取经验值提升等级）：

![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160146748.png)

奖励部分（获取对应等级奖励、购买解锁进阶奖励）：

![img](https://cdn.jsdelivr.net/gh/StarryJam/PicDock@main/202404160146823.png)

## 配表

为了满足后续通用活动BP的配置需求，设计了一套用*<u>通行证名pass_name</u>*（下文均会用斜体+下划线做标记）为索引连通各个相关表格配置的数据结构，旨在后续新的活动BP策划可以不需要依赖程序介入来完成包括新的经验数值存取、关联物品读取、经验值奖励可视化、显示文本等内容的配置。

### BP配置

#### 基本信息配置

***activity_pass_setting_table.xlsx***（新增）：

该表配置活动BP的基本信息（部分内容对应在总览图中）

| STRING_ID            | DID                                        | DID                  | STRING_RES_ID           | STRING_RES_ID                        | STRING_RES_ID                       | RES:path         | RES:path         | RES:path         | RES:path           | RES:path               |
| -------------------- | ------------------------------------------ | -------------------- | ----------------------- | ------------------------------------ | ----------------------------------- | ---------------- | ---------------- | ---------------- | ------------------ | ---------------------- |
| **pass_name**        | **exp_display_item**                       | **unlock_item**      | **title**               | **desc_text_before_buy**             | **desc_text_after_buy**             | **bg_image_1**   | **bg_image_2**   | **bg_image_3**   | **free_pass_icon** | **advanced_pass_icon** |
| #通行证名            | 经验值物品item_did（用于在奖励列表中展示） | 解锁BP需要购买的物品 | BP标题                  | BP解锁广告词                         | BP解锁后广告词                      | BP奖励页面logo图 | BP奖励页面广告图 | BP挑战页面logo图 | 免费BP icon图      | 解锁后BP icon图        |
| #新手BP              |                                            |                      |                         |                                      |                                     |                  |                  |                  |                    |                        |
| *<u>newbie_pass</u>* | ’740410076                                 | '1160911339          | common.page_newbie_pass | guidance.newbie_pass_desc_before_buy | guidance.newbie_pass_desc_after_buy | image1           | image2           | image3           | image7             | image8                 |
| #活动BP示例          |                                            |                      |                         |                                      |                                     |                  |                  |                  |                    |                        |
| *<u>test_pass</u>*   | ’0123456789                                | '0123456789          | aa.bb                   | aa.bb                                | aa.bb                               | image4           | image5           | image6           | image9             | image10                |

#### 解锁物品配置

**0750表（*****member_item*****）或其他物品表**：

kind_of_part填入对应的*通行证名*即可

| ELEMENT_DID | ...     | STRING_ID            | ...     |
| ----------- | ------- | -------------------- | ------- |
| **id**      | **...** | **kind_of_part**     | **...** |
| #编号       | ...     | 部位类型             | ...     |
| #新手BP     |         |                      |         |
| '300001     | ...     | *<u>newbie_pass</u>* | ...     |
| #活动BP示例 |         |                      |         |
| '300002     | ...     | *<u>test_pass</u>*   | ...     |

#### 等级配置

***newbie_pass_level_table.xlsx***（沿用了做新手BP时候新建的表，名字暂时不改了）:

结构比较直观，顶级的升级所需经验配-1就可以了

| INT         | STRING_ID            | INT                |
| ----------- | -------------------- | ------------------ |
| **level**   | **pass_name**        | **level_up_exp**   |
| #等级       | #通行证名            | 升到下一级所需经验 |
| #新手BP     |                      |                    |
| 1           | *<u>newbie_pass</u>* | 10                 |
| 2           | <u>*newbie_pass*</u> | 10                 |
| ...         | ...                  | ...                |
| 100         | *<u>newbie_pass</u>* | -1                 |
| #活动BP示例 |                      |                    |
| 1           | <u>*test_pass*</u>   | 10                 |
| 2           | <u>*test_pass*</u>   | 10                 |
| ...         | ...                  | ...                |
| 100         | *<u>test_pass</u>*   | -1                 |

### 活动配置

**3401表（*****activity_formation*****）**:

**新手BP不需要在3401表里配活动**

活动BP需分奖励和挑战两个部分开启，奖励部分活动类型为*activity_pass*，挑战部分活动类型为*activity_pass_challenge*，两个活动子类型为对应的*pass_name*；挑战部分在activity_challenge_group_1列填上对应的挑战组

| ELEMENT_DID          | ...     | STRING_ID                 | STRING_ID          | ...     | DID                            |
| -------------------- | ------- | ------------------------- | ------------------ | ------- | ------------------------------ |
| **id**               | **...** | **type**                  | **sub_type**       | **...** | **activity_challenge_group_1** |
| #ID                  | ...     | 活动类型                  | 活动类型           | ...     | 活动挑战组1                    |
| # 活动BP奖励部分示例 |         |                           |                    |         |                                |
| ’100001              | ...     | activity_pass             | <u>*test_pass*</u> | ...     | '0000000000                    |
| # 活动BP挑战部分示例 |         |                           |                    |         |                                |
| ‘100002              | ...     | activity_pass_challeng*e* | *test_pass*        | ...     | '2856910002                    |
|                      |         |                           |                    |         |                                |

### 挑战配置

#### Challenge group

***season_legion_challenge_group_state_table.xlsx***：

challenge_type处填*通行证名*，group_did处填上对应的挑战组

| DID            | STRING_ID            | DID                            |
| -------------- | -------------------- | ------------------------------ |
| **season_did** | **challenge_type**   | **legion_challenge_group_did** |
| #新手BP        |                      |                                |
| '0000000000    | *<u>newbie_pass</u>* | '2856910001                    |
| #活动BP示例    |                      |                                |
| '0000000000    | *<u>test_pass</u>*   | '2856910002                    |

**2856表（*****season_legion_challenge_group_table*****）**：

group_type 中填 newbie_pass_group 或 activity_pass_group （这两种类型暂时没有功能上的区别）

| ELEMENT_DID | ...  | STRING_ID           | ...  |
| ----------- | ---- | ------------------- | ---- |
| id          | ...  | group_type          | ...  |
| #新手BP     |      |                     |      |
| '910001     | ...  | newbie_pass_group   | ...  |
| #活动BP示例 |      |                     |      |
| '910002     | ...  | activity_pass_group | ...  |

#### Challenge state

**2855表（*****season_legion_challenge_state_table*****）**：

challenge state表中增加了解锁条件功能，目前包括*player_level*（角色等级解锁）和 *activity_pass_day*（根据活动开启天数解锁，0则为随活动开启）两种条件，*activity_pass_day* 只能生效于由类型为 *activity_pass* 的活动开启的任务

| ELEMENT_DID       | ...     | STRING_ID               | INT                      |
| ----------------- | ------- | ----------------------- | ------------------------ |
| **id**            | **...** | **unlock_require_type** | **unlock_require_value** |
| #赛季兵团挑战阶段 | ...     | 解锁条件                | 解锁条件所需值           |
| #新手BP           |         |                         |                          |
| '910001           | ...     | player_level            | 21                       |
| #活动BP示例       |         |                         |                          |
| '910002           | ...     | activity_pass_day       | 1                        |
|                   |         |                         |                          |

#### Challenge quest

**1231表（*****hero_path_quest*****）或其他quest表**：

新增通用奖励配置，用stringkey可以进行树状分类和多参数配置，后续一些新的数值或者道具奖励需求都可以用这一列，配合程序开发功能来配置新的字段。

活动BP的经验奖励配置方法：*award_common_value_name* 列以 *activity_pass.**通行证名**.exp* 形式填入对应内容，并在*award_common_value_num*列配置奖励经验数量。

注：

1. 之前版本的*award_common_value_name* 字段类型为STRING_ID，这次改动后类型变更为STRING_KEY
2. 活动BP的任务需要关联活动DID来保证活动关闭时任务不再统计数据

| ELEMENT_DID | ...     | STRING_KEY                             | FLOAT                      | DID                         |
| ----------- | ------- | -------------------------------------- | -------------------------- | --------------------------- |
| **id**      | **...** | **award_common_value_name**            | **award_common_value_num** | **connect_activity**        |
| #id         | ...     | #奖励内容                              | #奖励数量                  | #关联活动关闭时任务暂停统计 |
| #新手BP     |         |                                        |                            |                             |
| ‘100007     | ...     | activity_pass.<u>*newbie_pass*</u>.exp | 5                          | ’0000000000                 |
| #活动BP示例 |         |                                        |                            |                             |
| ‘200007     | ...     | activity_pass.<u>*test_pass*</u>.exp   | 10                         | ‘3401910002                 |

### 奖励配置

***item_exchange_table.xlsx***:

type填对应的*通行证名*

| STRING_ID            | DID            | DID              |
| -------------------- | -------------- | ---------------- |
| **type**             | **season_did** | **exchange_did** |
| #兑换类型            | 赛季did        | 奖励did          |
| #新手BP              |                |                  |
| <u>*newbie_pass*</u> | '0000000000    | '2847990023      |
| *newbie_pass*        | '0000000000    | '2847990023      |
| ...                  | ...            | ...              |
| #活动BP示例          |                |                  |
| <u>*test_pass*</u>   | '0000000000    | '2847123451      |
| <u>*test_pass*</u>   | '0000000000    | '2847123452      |

**2847表（*****item_exchange_set*****）**:

通过等级和*通行证名*双条件组合的方式配置不同活动BP的奖励：

*free_activity_pass_level* 和 *activity_pass_level* 条件类型分别表示免费奖励和购买BP后的进阶奖励的BP等级需求；

*pass_name* 条件类型用来限定奖励所属的BP类型。

| ELEMENT_DID  | STRING_ID                | INT                     | ...     | STRING_ID            | STRING_ID              | ...     |
| ------------ | ------------------------ | ----------------------- | ------- | -------------------- | ---------------------- | ------- |
| **id**       | **condition_type_1**     | **condition_value_a_1** | **...** | **condition_type_2** | **condition_str_id_2** | **...** |
| **#编号**    | **条件1类型**            | **条件1数值A**          | **...** | **条件2类型**        | **条件1StringID**      | **...** |
| #新手BP 基础 |                          |                         |         |                      |                        |         |
| '980001      | free_activity_pass_level | 1                       | ...     | pass_name            | <u>*newbie_pass*</u>   | ...     |
| '980002      | free_activity_pass_level | 2                       | ...     | pass_name            | <u>*newbie_pass*</u>   | ...     |
| ...          | ...                      | ...                     | ...     | ...                  | ...                    | ...     |
| '980100      | free_activity_pass_level | 100                     | ...     | pass_name            | <u>*newbie_pass*</u>   | ...     |
| #新手BP 进阶 |                          |                         |         |                      |                        |         |
| '990001      | activity_pass_level      | 1                       | ...     | pass_name            | <u>*newbie_pass*</u>   | ...     |
| '990002      | activity_pass_level      | 2                       | ...     | pass_name            | <u>*newbie_pass*</u>   | ...     |
| ...          | ...                      | ...                     | ...     | ...                  | ...                    | ...     |
| '990100      | activity_pass_level      | 100                     | ...     | pass_name            | <u>*newbie_pass*</u>   | ...     |
|              |                          |                         |         |                      |                        |         |
|              |                          |                         |         |                      |                        |         |
| #活动BP 基础 |                          |                         |         |                      |                        |         |
| '981001      | free_activity_pass_level | 1                       | ...     | pass_name            | *<u>test_pass</u>*     | ...     |
| '981002      | free_activity_pass_level | 2                       | ...     | pass_name            | <u>*test_pass*</u>     | ...     |
| ...          | ...                      | ...                     | ...     | ...                  | ...                    | ...     |
| '980100      | free_activity_pass_level | 100                     | ...     | pass_name            | <u>*test_pass*</u>     | ...     |
| #活动BP 进阶 |                          |                         |         |                      |                        |         |
| '992001      | activity_pass_level      | 1                       | ...     | pass_name            | <u>*test_pass*</u>     | ...     |
| '992002      | activity_pass_level      | 2                       | ...     | pass_name            | <u>*test_pass*</u>     | ...     |
| ...          | ...                      | ...                     | ...     | ...                  | ...                    | ...     |
| '992100      | activity_pass_level      | 100                     | ...     | pass_name            | <u>*test_pass*</u>     | ...     |