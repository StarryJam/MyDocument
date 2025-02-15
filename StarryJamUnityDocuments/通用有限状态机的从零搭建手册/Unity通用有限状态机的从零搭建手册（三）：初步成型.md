## Unity通用有限状态机的从零搭建手册（三）：初步成型

### 状态机的创建和初始化

在上一章我们完成了框架的搭建，但是并没有提及如何创建一个完整的状态机以及如何初始化它，这一章我们就来尝试解决这一问题。

上一章中提到状态机如果脱离主体将失去意义，所以我们状态机的构造函数需要一个主体的参数，并且我们不希望外部可以改动状态机的主体，我们将set访问器改为private：

```c#
/// <summary>
/// 状态机
/// </summary>
public class FiniteStateMachine<T> : IFSM<T>, IUpdater
{
    public T Subject { get; private set; }
    
    public FiniteStateMachine(T subject)
    {
        Subject = subject;
    }
    
    /*
    ...
    */
}
```

结构上我们认为状态类从属于状态机，并且使用时也是无法脱离状态机而单独发挥作用的，所以在构造方法中传入状态机实例，主体就可以直接读取状态机的主体而不需要额外存储了：

```c#
/// <summary>
/// 状态类
/// </summary>
/// <typeparam name="T">主体</typeparam>
public abstract class State<T>
{
    public FiniteStateMachine<T> StateMachine { get; private set; }

    public State(FiniteStateMachine<T> stateMachine)
    {
        StateMachine = stateMachine;
    }

    //状态主体
    public T Subject => StateMachine.Subject;
}
```

初始化方法我们效仿Unity的Awake()和Start()方法来定义状态和状态机初始化和开始运行的行为

```c#
/// <summary>
/// 状态类
/// </summary>
/// <typeparam name="T">主体</typeparam>
public abstract class State<T>
{
 	/*
 	...
 	*/
    
    //状态初始化
    public virtual void Awake() { }
    
 	/*
 	...
 	*/
}
```

```c#
/// <summary>
/// 状态机
/// </summary>
public class FiniteStateMachine<T> : IFSM<T>, IUpdater
{
    
    /*
    ...
    */
    
    //默认状态
    public State<T> DefaultState { get; set; }

    /// <summary>
    /// 状态机初始化
    /// </summary>
    public void Awake()
    {
        foreach (var state in _states) 
        {
            //遍历状态列表初始化
            state.Awake();
        }
    }

    /// <summary>
    /// 状态机开始运行
    /// </summary>
    public void Start()
    {
        if (DefaultState == null)
        {
            Debug.LogError("状态加没有设置默认状态");
            return;
        }

        _currentState = DefaultState;
        _currentState.Enter();
    }

    /*
    ...
    */
}
```

设计上我们不允许状态为空的情况出现，所以需要在状态机开始运行之前设置一下默认状态（DefaultState）以确保状态机一旦运行就有当前状态，同时也便于实现状态机的重置功能。

条件类之前介绍过通过传参的方式免去了初始化的步骤，不需要做改动。OK到此我们已经完成了状态机的基本功能搭建，开始尝试做一个小Demo吧！



### 第一个实例

我们测试状态机功能的第一个Demo内容很简单，创建一个小方块响应鼠标的悬浮、点击等操作改变自己的状态，状态的转移通过颜色的变化来体现，下面是状态关系示意图：

![image-20220302115528266](https://raw.githubusercontent.com/StarryJam/PicDock/main/imgimage-20220302115528266.png)

三个颜色表示三种状态，箭头上的文字表示转移条件，箭头则指向转移的目标状态。根据图示，我们需要创建三个具体状态类和四个具体条件类，当然主体Cube类也是不能少的：

![image-20220301000736071](https://raw.githubusercontent.com/StarryJam/PicDock/main/imgimage-20220301000736071.png)

**Cube类：**

我们的小方块需要响应鼠标事件，我们写一下Unity自带的几个鼠标事件方法，用两个布尔值储存一下鼠标和物体的交互关系，使用之前记得检查一下创建的方块是否带有碰撞体。

```c#
public class Cube : MonoBehaviour
{
    //获取材质
    public Material material => GetComponent<MeshRenderer>().material;
    //鼠标是否悬浮
    public bool mouseOver = false;
    //鼠标是否按下
    public bool mouseDown = false;

    private void OnMouseEnter()
    {
        mouseOver = true;
        Debug.Log("MouseOver");
    }

    private void OnMouseExit()
    {
        mouseOver = false;
        Debug.Log("MouseExit");
    }

    private void OnMouseDown()
    {
        mouseDown = true;
        Debug.Log("MouseDown");
    }

    private void OnMouseUp()
    {
        mouseDown = false;
        Debug.Log("MouseUp");
    }
    
    //TODO 状态机的初始化
}
```

后面几个状态类内容比较简单直接贴代码了

**状态类：**

```c#
//默认状态
public class Cube_State_Default : State<Cube>
{
    public Cube_State_Default(FiniteStateMachine<Cube> stateMachine) : base(stateMachine) { }
    
    public override void Enter()
    {
        base.Enter();
        Subject.material.color = Color.white;
    }
}
```

```c#
//鼠标悬浮状态
public class Cube_State_MouseDown : State<Cube>
{
    public Cube_State_MouseDown(FiniteStateMachine<Cube> stateMachine) : base(stateMachine) { }

    public override void Enter()
    {
        base.Enter();
        Subject.material.color = Color.green;
    }
}
```

```c#
//鼠标按下状态
public class Cube_State_MouseDown : State<Cube>
{
    public Cube_State_MouseDown(FiniteStateMachine<Cube> stateMachine) : base(stateMachine) { }

    public override void Enter()
    {
        base.Enter();
        Subject.material.color = Color.green;
    }
}
```



**条件类：**

```c#
//鼠标是否悬浮
public class Cube_Condition_MouseOver : Condition<Cube>
{
    public override bool ConditionCheck(Cube subject)
    {
        return subject.mouseOver;
    }
}
```

```c#
//鼠标是否移开
public class Cube_Condition_MouseExit : Condition<Cube>
{
    public override bool ConditionCheck(Cube subject)
    {
        return !subject.mouseOver;
    }
}
```

```c#
//鼠标是否按下
public class Cube_Condition_MouseDown : Condition<Cube>
{
    public override bool ConditionCheck(Cube subject)
    {
        return subject.mouseDown;
    }
}
```

```c#
//鼠标是否松开
public class Cube_Condition_MouseUp : Condition<Cube>
{
    public override bool ConditionCheck(Cube subject)
    {
        return !subject.mouseDown;
    }
}
```

**状态机配置：**

状态和条件都准备好之后，我们需要在主体类当中把状态机配置起来，先在类中新建一个私有变量存储状态机实例。

```c#
public class Cube : MonoBehaviour
{
    private FiniteStateMachine<Cube> _stateMachine;
    
    //...
}
```

在Awake方法中配置并初始化状态机：

```c#
public class Cube : MonoBehaviour
{
    /*
    ...
    */
    
    private void Awake()
    {
        _stateMachine = new FiniteStateMachine<Cube>(this);

        //状态创建
        var stateDefault = new Cube_State_Default(_stateMachine);
        var stateMouseDown = new Cube_State_MouseDown(_stateMachine);
        var stateMouseOver = new Cube_State_MouseOver(_stateMachine);

        //条件创建
        var conditionMouseOver = new Cube_Condition_MouseOver();
        var conditionMouseExit = new Cube_Condition_MouseExit();
        var conditionMouseDown = new Cube_Condition_MouseDown();
        var conditionMouseUp = new Cube_Condition_MouseUp();

        //条件添加
        stateDefault.AddCondition(conditionMouseOver, stateMouseOver);
        stateMouseOver.AddCondition(conditionMouseExit, stateDefault).AddCondition(conditionMouseDown, stateMouseDown);
        stateMouseDown.AddCondition(conditionMouseUp, stateMouseOver);
        
        //状态添加
        _stateMachine.AddState(stateDefault).AddState(stateMouseDown).AddState(stateMouseOver).DefaultState = stateDefault;
        
        _stateMachine.Awake();
    }
    
    /*
    ...
    */
}
```

为了编写方便，状态和条件添加的时候我们使用了链式编程格式，相应地我们需要在AddState方法中将this作为返回值：

![image-20220301154539959](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220301154539959.png)

最后在Start中启动状态机，Update中更新状态机：

```c#
public class Cube : MonoBehaviour
{
    /*
    ...
    */
    
    private void Start()
    {
        _stateMachine.Start();
    }

    private void Update()
    {
        _stateMachine.Update();
    }
    
    /*
    ...
    */
}
```

运行，效果达成！

![cubeTest](https://raw.githubusercontent.com/StarryJam/PicDock/main/imgcubeTest.gif)



欸这时有人就会说了，哎呀你这这么简单的逻辑又是配置又是初始化的，还新建了一堆的类，new了一堆实例，也太麻烦了。啊确实，为一个简单的逻辑去做一个状态机有时的确是吃力不讨好的行为，不过要注意到的是，我们有两个每一个状态下只需要判断一个bool值就可以确定状态的转化，而单纯使用if-else语句需要在任何情况下同时判断两个bool值：

```c#
public class Cube : MonoBehaviour
{
	/*
	...
	*/
    
    public void StateUpdate()
    {
        if (!mouseOver && !mouseDown)
        {
            material.color = Color.white;
        }
        else if (mouseOver && !mouseDown)
        {
            material.color = Color.yellow;
        }
        else if(mouseOver && mouseDown)
        {
             material.color = Color.red;
        }
    }

    private void Update()
    {
        StateUpdate();
    }
}

```

这样的写法显然也能达成一样的效果，但是如果我希望在没有点击的情况下移入移出鼠标打印一条信息呢，这一串判断语句可能有需要进行破坏式的改动；如果状态的转化与更多属性有关，分支语句很有可能会指数增长；随着项目的迭代，状态的数量将变得庞大，状态之间的转换关系也会变得更为复杂，这时候你就会发现离开状态机将寸步难行。当然我们的状态机也有不少的优化空间，可以让配置的过程变得更为简洁，之后的章节我们来尝试用C#的反射特性对状态机的配置和繁杂的实例创建过程进行优化，实现一定程度的自动化，同时也会有更加复杂的案例让我们感受到状态机的优势所在。
