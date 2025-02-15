# Unity通用有限状态机的从零搭建手册（四）：优化之路（1）

### 核心痛点

计算机的发明是为了替人类完成无聊的复杂劳作，解放生产力，软件工程的发展更是给软件提出了可维护性强易使用等要求，如果一个程序在使用时过于繁琐，那他一定有可以改进的空间。上一章的Demo我们已经尝试了应用我们写好的状态机，但是不难发现其实有很多的问题，比如配装前需要的用new去新建许多实例，并且所有的状态和条件都散落在各个文件中，后期维护的时候很难一下子看到这个类所拥有的所有状态，这些问题会阻碍我们状态机应用在一个稍有规模的项目当中。本章就将会循着我在工作中摸索出的道路来优化我们的状态机。

![image-20220302165654542](https://raw.githubusercontent.com/StarryJam/PicDock/main/imgimage-20220302165654542.png)



### 解决方案

我们知道状态机中不可能出现重复的状态，那么是不是表示我们只需要用状态的名字就可以用工厂类生产实例代替new的使用，同时避免直接用实例作为参数来使用状态机了呢？我们可以先尝试一下写出我们理想的配装方法，然后看一下会不会遇到新的问题。为了可以有代码补全和编译器检查防止手打出错，我们用枚举类来给状态和条件一个ID，所以我们的第一步是给状态和条件取个名字（ID）。

创建一个Config文件夹集中存放未来所有的State和Condition的枚举：

![image-20220302175451100](https://raw.githubusercontent.com/StarryJam/PicDock/main/imgimage-20220302175451100.png)

```c#
public enum CubeState
{
    /// <summary>
    /// 默认状态
    /// </summary>
    Default,
    /// <summary>
    /// 鼠标悬浮状态
    /// </summary>
    MouseOver,
    /// <summary>
    /// 鼠标按下状态
    /// </summary>
    MouseDown
}
```

```c#
public enum CubeCondition
{
    /// <summary>
    /// 鼠标是否悬浮
    /// </summary>
    MouseOver,
    /// <summary>
    /// 鼠标是否移除
    /// </summary>
    MouseExit,
    /// <summary>
    /// 鼠标是否按下
    /// </summary>
    MouseDown,
    /// <summary>
    /// 鼠标是否弹起
    /// </summary>
    MouseUp
}
```

做完这一步之后先不急着改状态机的代码，先想想我们期望怎样去使用它，在Cube类的状态机装配过程中尝试用我们期望的方式去书写代码：

```c#
public class Cube : MonoBehaviour
{
    private void Awake()
    {
        _stateMachine = new FiniteStateMachine<Cube>(this);

        // var stateDefault = new Cube_State_Default(_stateMachine);
        // var stateMouseDown = new Cube_State_MouseDown(_stateMachine);
        // var stateMouseOver = new Cube_State_MouseOver(_stateMachine);
        //
        // var conditionMouseOver = new Cube_Condition_MouseOver();
        // var conditionMouseExit = new Cube_Condition_MouseExit();
        // var conditionMouseDown = new Cube_Condition_MouseDown();
        // var conditionMouseUp = new Cube_Condition_MouseUp();

        stateDefault.AddCondition(conditionMouseOver, stateMouseOver);
        stateMouseOver.AddCondition(conditionMouseExit, stateDefault).AddCondition(conditionMouseDown, stateMouseDown);
        stateMouseDown.AddCondition(conditionMouseUp, stateMouseOver);
        
        _stateMachine.AddState(CubeState.Default).AddState(CubeState.MouseDown).AddState(CubeState.MouseOver).DefaultStateID = CubeState.Default;
        
        _stateMachine.Awake();
    }
}
```

把new的过程全部去掉一下子就为一次装配省去了一半的代码量，不过同时也会发现，由于没有拿到状态类的实例，没法调用添加条件的方法了。如何解决这个问题呢，我们的迭代目的是为了简化代码，用名字或者说枚举去代替直接对实例进行操作从而简化代码，那么我们之前对状态类的一些接口应该转移到状态机类中去，让我们的状态机真正成为整个系统的入口和管理者，彻底隔离开发过程中和状态类以及条件类的直接接触。由此为方向，我们来修改之前的代码。

首先给状态机类加一个AddTransition的方法，参数是转换两头的状态和条件：

```c#
public class FiniteStateMachine<T>
{
    /*
    ...
    */

    /// <summary>
    /// 添加条件转换，如果所加转换中有尚未添加的状态会自动添加
    /// </summary>
    /// <returns>current FSM</returns>
    public FiniteStateMachine<T> AddTransition(Enum fromState, Enum toState, Enum condition)
    {
        //TODO
        
        return this;
    }
    
    
    /*
    ...
    */
}
```

加了这个方法之后，因为在添加转换关系的时候就会自动把没有添加的状态加入，所以就不需要添加状态的语句了，状态机的配置过程会简化成下面这样，依然是链式编程格式：

```c#
private void Awake()
{
    _stateMachine = new FiniteStateMachine<Cube>(this);
    
    _stateMachine
        .AddTransition(CubeState.Default, CubeState.MouseOver, CubeCondition.MouseOver)
        .AddTransition(CubeState.MouseOver, CubeState.Default, CubeCondition.MouseExit)
        .AddTransition(CubeState.MouseOver, CubeState.MouseDown, CubeCondition.MouseDown)
        .AddTransition(CubeState.MouseDown, CubeState.MouseOver, CubeCondition.MouseUp)
        .DefaultStateID = Cube_State.Default;
    
    _stateMachine.Awake();
}
```

怎么样，状态条件一步到位，是不是变得非常简洁！

确定我们的方案可行之后，只要给State和Condition类加上ID，再完成工厂类，我们优化的第一步就大功告成了：

![image-20220303175524473](https://raw.githubusercontent.com/StarryJam/PicDock/main/imgimage-20220303175526893.png)

```c#
//状态工厂
public class StateFactory
{
    public static object GetState(object stateMachine, Enum stateID)
    {
        switch (stateID)
        {
            case CubeState.Default:
                return new Cube_State_Default((FiniteStateMachine<Cube>)stateMachine, stateID);
            case CubeState.MouseDown:
                return new Cube_State_MouseDown((FiniteStateMachine<Cube>)stateMachine, stateID);
            case CubeState.MouseOver:
                return new Cube_State_MouseOver((FiniteStateMachine<Cube>)stateMachine, stateID);
        }
        
        Debug.LogError($"[{stateID}]没有找到对应的状态");
        return null;
    }
}
```

```c#
//条件工厂
public class ConditionFactory
{
    public static object GetCondition(Enum stateID)
    {
        switch (stateID)
        {
            case CubeCondition.MouseDown:
                return new Cube_Condition_MouseDown(stateID);
            case CubeCondition.MouseExit:
                return new Cube_Condition_MouseExit(stateID);
            case CubeCondition.MouseOver:
                return new Cube_Condition_MouseOver(stateID);
            case CubeCondition.MouseUp:
                return new Cube_Condition_MouseUp(stateID);
        }
        
        Debug.LogError($"[{stateID}]没有找到对应的条件");
        return null;
    }
}
```

由于我们使用的是模板类，编译时无法通过由State<Cube>像State<T>的转换，要返回具体类的实例只能把返回值设为object然后在接收的时候强转为模板类：

```c#
public class FiniteStateMachine<T>
{
    /*
    ...
    */

    /// <summary>
    /// 添加状态
    /// </summary>
    public FiniteStateMachine<T> AddState(Enum stateID)
    {
        if (_states.Find(state => state.StateID == stateID) != null)
            Debug.LogWarning($"尝试重复添加状态{stateID}");
        else
        {
            //工厂生成状态实例
            var addedState = (State<T>)StateFactory.GetState(this, stateID);
            _states.Add(addedState);   
        }

        return this;
    }
    
    /// <summary>
    /// 添加条件转换，如果所加转换中有尚未添加的状态会自动添加
    /// </summary>
    /// <returns>current FSM</returns>
    public FiniteStateMachine<T> AddTransition(Enum fromState, Enum toState, Enum condition)
    {
        var fState = _GetState(fromState);
        var tState = _GetState(toState);
        if (fState == null)
        {
            AddState(fromState);
            fState = _GetState(fromState);
        }

        if (tState == null)
        {
            AddState(toState);
            tState = _GetState(toState);
        }

        //工厂生成条件实例
        var addCondition = (Condition<T>)ConditionFactory.GetCondition(condition);
        fState.AddCondition(addCondition, toState);

        return this;
    }
    
    
    /*
    ...
    */
}
```

由于我们隔离了对实例的操作，装配的过程中并不会直接调用工厂，所以返回类型为object不会对日后使用造成麻烦。

运行之后依旧达成了之前的效果。

可以看到经过我们这一步的优化，装配的过程被压缩了近三分之二的代码，省去了麻烦的new实例过程，简单地通过ID就完成了状态机的配置，这为以后给主体的不同子类去配置不同的状态省下了非常多的时间；但是相应的，我们付出的代价是每次添加新的状态或条件类都要在ID配置和工厂当中分别添加一个条目，虽然相比于之前的代码已经有了不少提升，但是在这个地方还有很大的优化空间，或者不如说是如果不优化这个系统还完全称不上好用。

先休息一下，基于反射的进一步的优化我们留给下一章。
