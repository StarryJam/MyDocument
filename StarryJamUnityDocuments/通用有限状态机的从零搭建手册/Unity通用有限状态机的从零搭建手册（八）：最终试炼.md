# Unity通用有限状态机的从零搭建手册（八）：最终试炼

本章作为整篇文章的的最后一节，将使用我们搭建的状态机系统完成一个相对复杂的综合案例，以测试该系统在工程中的可行性，以及记录一些在实际应用当中的问题作为我们未来改进方向的参考。案例中与状态机系统无关的一些细节会尽量较少着墨，文章末尾我会把项目地址贴上感兴趣的读者可以下载下来运行和阅读代码。

### 需求分析

该案例中玩家和电脑AI会各自通过指令的方式控制一个单位进行移动和射击，玩家通过WASD控制移动，鼠标控制射击方向，AI则会在玩家与一定范围内时追击玩家，并在足够靠近时进行射击。我们的状态机系统将会被应用在单位的行为控制和AI的行为决策中。

**状态机图：**

![image-20220312205355296](https://raw.githubusercontent.com/StarryJam/PicDock/main/imgimage-20220312205355296.png)

有必要解释几点：

1、单位行为相比上一章复合状态中所举的例子有所不同的是存活状态中再次做了一个复合状态的嵌套，给了一个攻击和移动状态的父状态，执行指令状态，之所以这么做是考虑到真实项目中单位的指令会更为多样，每一种指令对应的状态可能都需要在执行完成之后回到待机状态，用一个复合状态去统一有利于简化状态之间的关系；

2、在状态机运行的过程中可能会出现某种条件下，A->B->C和A->C的转换同时成立的情况，例如上图AI行为状态机中，若满足玩家进入射击距离（射击距离<追击距离），既可以先进入追击状态然后再次判断满足射击条件转到射击状态，也可以直接判断满足射击条件而进入射击状态，这种情况我们尽可能通过调整条件的优先级的方法去避免第一种情况的出现，否则会进行不必要的状态转移，并且在我们的系统中还会导致射击动作延后一个逻辑帧执行。



**类图：**

根据Demo的内容，我们需要实现的一些类分别为：

Unit：可以被玩家或AI控制的单位类；

Bullet：子弹类；

IUnitController：用于控制单位的控制器接口，向单位发送单位指令；

UnitInstructions：用于控制单位的指令集。

![image-20220313221634505](https://raw.githubusercontent.com/StarryJam/PicDock/main/imgimage-20220313221634505.png)





### 案例实现

**控制器和一些工具方法**

抽象出控制器接口是为了让玩家和AI可以以统一的方式控制单位

```c#
/// <summary>
/// 单位控制器接口，可以获取控制指令
/// </summary>
public interface IUnitController
{
    /// <summary>
    /// 单位控制指令
    /// </summary>
    UnitInstructions Instructions { get; }

}

/// <summary>
/// 默认的控制器类，提供空指令
/// </summary>
 class UnitDefaultController : IUnitController
 {
     public UnitInstructions Instructions => null;
 }
```



创建一个工具方法的静态类，方便以后将各种小工具方法汇总在一个文件中。

下面两个拓展方法目的是将空间向量和平面向量相互转换，因为我们的游戏逻辑实际上只是2D的但是表现层是3D的。

```c#
/// <summary>
/// 工具方法集
/// </summary>
public static class Utils
{
    public static Vector3 ToVector3(this Vector2 vector2)
    {
        return new Vector3(vector2.x, 0, vector2.y);
    }

    public static Vector2 ToVector2(this Vector3 vector3)
    {
        return new Vector2(vector3.x, vector3.z);
    }
}
```



**Unit预制体**

先创建一个单位的预制体，我们还是用最简单的立方体来作为模型；测试的过程中我们希望可以更加直观地看到单位的血量和当前状态，所以我们在预制体上绑定一个Canvas和一个Text组件用来显示信息，将Canvas的Render Mode设置为World Space，调整至合适的位置和大小：

![image-20220313223558488](https://raw.githubusercontent.com/StarryJam/PicDock/main/imgimage-20220313223558488.png)

为预制体创建并添加一个Unit脚本，根据需求先大致填充一下Unit类中的内容，状态机的配置我们先空着：

```c#
/// <summary>
/// 单位类
/// </summary>
public partial class Unit : MonoBehaviour
{
    /// <summary>
    /// 最大血量
    /// </summary>
    public static readonly int maxLife = 10;
    
    /// <summary>
    /// 单位移动速度
    /// </summary>
    public float moveSpeed = 3;

    //当前血量
    public int curLife;

    //状态文本显示组件
    [SerializeField]
    private Text _stateDisplauy;

    //当前状态
    private string _stateTxt = "";

    //状态机
    private FiniteStateMachine<Unit> _stateMachine;

    /// <summary>
    /// 单位控制器
    /// </summary>
    public IUnitController Controller { get; set; } = new UnitDefaultController(); //创建一个默认不会发出任何指令的控制器

    private UnitInstructions _curInstructions = null;

    private void Awake()
    {
        //获取单位身上的控制器
        var controller = GetComponent<IUnitController>();
        if (controller != null)
            Controller = controller;
        
        curLife = maxLife;
        _stateMachine = new FiniteStateMachine<Unit>(this);
        _ConfigStateMachine();
        _stateMachine.Awake();
    }

    private void Start()
    {
        _stateMachine.Start();
    }
    

    //状态机配置
    private void _ConfigStateMachine()
    {
        //TODO
    }

    private void Update()
    {
        _stateMachine.Update();
        
        //状态文本更新
        _stateDisplauy.text = $"Life:{curLife}\nState:{_stateTxt}";
    }
}
```

单位指令：

```c#
/// <summary>
/// 单位指令
/// </summary>
public class UnitInstructions
{
    /// <summary>
    /// 移动方向（归一化的）
    /// </summary>
    public Vector2 moveDirection
    {
        get => _moveDirection;
        set => _moveDirection = value.normalized;
    }

    private Vector2 _moveDirection;

    /// <summary>
    /// 攻击方向（归一化的）
    /// </summary>
    public Vector2 attackDirection
    {
        get => _attackDirection;
        set => _attackDirection = value.normalized;
    }

    private Vector2 _attackDirection;
}
```



**状态和条件类**

首先去状态机配置文件中加入对应的条目：

![image-20220313222605317](https://raw.githubusercontent.com/StarryJam/PicDock/main/imgimage-20220313222605317.png)

![image-20220313222703877](https://raw.githubusercontent.com/StarryJam/PicDock/main/imgimage-20220313222703877.png)

![image-20220313222730671](https://raw.githubusercontent.com/StarryJam/PicDock/main/imgimage-20220313222730671.png)



分别完成每个状态和条件的编写。这里代码有点多，主要看备注的部分就可以了，读者也可以先跳过这一段阅读尝试自己实现一下。

存活状态：

```c#
partial class Unit
{
    //存活（复合）状态
    private class Unit_State_Alive : CompositeState<Unit>
    {
        public Unit_State_Alive(IStateMachine<Unit> stateMachine, Enum stateID) : base(stateMachine, stateID) { }
        
        public override void Update()
        {
            base.Update();
            //每帧从控制器获取命令
            Subject._curInstructions = Subject.Controller.Instructions;
        }
        
    }
}
```



执行指令状态，这个状态有些特殊，因为它作为复合方法但是需要在进入的时候判断指令的内容决定切入哪一个子状态中，所以写法上稍微变通了一下：

```c#
partial class Unit
{
    //执行指令（复合）状态
    private class Unit_State_Executing : CompositeState<Unit>
    {
        private Unit_State_Executing(IStateMachine<Unit> stateMachine, Enum stateID) : base(stateMachine, stateID) { }

        public override void Awake()
        {
            base.Awake();
            
            //考虑到未来很可能会加入新的指令状态，将子状态的配置封装在当前类当中了；并且因为这个状态比较特殊也确实封装在内部比较合适。
            AddTransition(Unit_State.Attacking, Unit_State.Moving, Unit_Condition.Attacking2Moving);
            AddTransition(Unit_State.Moving, Unit_State.Attacking, Unit_Condition.Moving2Attacking);
            
            //只是为了可以成功初始化所以设置一个初始状态
            DefaultStateID = Unit_State.Moving;
        }

        public override void Enter()
        {
            //根据指令优先级决定默认状态
            if(Subject._curInstructions.attackDirection != Vector2.zero)
                DefaultStateID = Unit_State.Attacking;
            else
                DefaultStateID = Unit_State.Moving;
            
            base.Enter();
        }
    }
}
```



移动状态：

```c#
partial class Unit
{
    //移动状态
    private class Unit_State_Moving : State<Unit>
    {
        private Unit_State_Moving(IStateMachine<Unit> stateMachine, Enum stateID) : base(stateMachine, stateID) { }

        public override void Enter()
        {
            base.Enter();
            //改变状态文本
            Subject._stateTxt = "Moving";
        }

        public override void Update()
        {
            base.Update();
            //根据移动指令的方向进行移动
            Subject.transform.position += 
                Subject._curInstructions.moveDirection.ToVector3().normalized * (Subject.moveSpeed * Time.deltaTime);
        }
    }
}
```



攻击状态：

```c#
partial class Unit
{
    //攻击状态
    private class Unit_State_Attacking : State<Unit>
    {
        private Unit_State_Attacking(IStateMachine<Unit> stateMachine, Enum stateID) : base(stateMachine, stateID) { }

        public override void Enter()
        {
            base.Enter();
            //改变状态文本
            Subject._stateTxt = "Attacking";
        }

        public override void Update()
        {
            base.Update();
            //TODO 发射子弹
        }
    }
}
```



待机状态：

```c#
partial class Unit
{
    //待机状态，不需要做什么
    private class Unit_State_Idle : State<Unit>
    {
        private Unit_State_Idle(IStateMachine<Unit> stateMachine, Enum stateID) : base(stateMachine, stateID) {}

        public override void Enter()
        {
            base.Enter();
            //改变状态文本
            Subject._stateTxt = "Idle";
        }
    }
}
```



死亡状态：

```c#
partial class Unit
{
    //死亡状态
    private class Unit_State_Dead : State<Unit>
    {
        private Unit_State_Dead(IStateMachine<Unit> stateMachine, Enum stateID) : base(stateMachine, stateID) { }

        public override void Enter()
        {
            base.Enter();
            //为了更明显一些将材质变为红色
            Subject.GetComponent<MeshRenderer>().material.color = Color.red;
            //改变状态文本
            Subject._stateTxt = "Dead";
        }
    }
}
```



存活->死亡的条件：

```c#
partial class Unit
{
    //生命值小于等于零
    private class Unit_Condition_ZeroLife : Condition<Unit>
    {
        private Unit_Condition_ZeroLife(Enum conditionID) : base(conditionID) { }

        public override bool ConditionCheck(Unit subject)
        {
            return subject.curLife <= 0;
        }
    }
}
```



待机->执行指令的条件：

```c#
partial class Unit
{
    //收到新的指令
    private class Unit_Condition_GetInstruction : Condition<Unit>
    {
        private Unit_Condition_GetInstruction(Enum conditionID) : base(conditionID) { }

        public override bool ConditionCheck(Unit subject)
        {
            return subject._curInstructions != null;
        }
    }
}
```



执行指令->待机的条件：

```c#
partial class Unit
{
    //没有指令
    private class Unit_Condition_NoInstruction : Condition<Unit>
    {
        private Unit_Condition_NoInstruction(Enum conditionID) : base(conditionID) { }

        public override bool ConditionCheck(Unit subject)
        {
            return subject._curInstructions == null;
        }
    }
}
```



攻击->移动的条件：

```c#
partial class Unit
{
    //攻击->移动条件
    private class Unit_Condition_Attacking2Moving : Condition<Unit>
    {
        private Unit_Condition_Attacking2Moving(Enum conditionID) : base(conditionID) { }

        public override bool ConditionCheck(Unit subject)
        {
            return
                subject._curInstructions.attackDirection == Vector2.zero &&
                subject._curInstructions.moveDirection != Vector2.zero;
        }
    }
}
```



移动->攻击的条件：

```c#
partial class Unit
{
    //移动->攻击
    private class Unit_Condition_Moving2Attacking : Condition<Unit>
    {
        private Unit_Condition_Moving2Attacking(Enum conditionID) : base(conditionID) { }

        public override bool ConditionCheck(Unit subject)
        {
            //一旦有攻击指令优先执行攻击指令
            return subject._curInstructions.attackDirection != Vector2.zero;
        }
    }
}
```



**玩家控制器**

玩家控制器允许玩家用WASD控制方向，用鼠标控制射击。为了确定射击的方向，我们需要在鼠标点击的时候发射一条射线检测与地面的碰撞，然后根据碰撞点和玩家位置确定射击方向。

我们在场景中创建一个地面，用Plane或Cube都可以，将其放大到足够大，为他的碰撞层级设置为Floor，并确保有一个勾选了trigger的碰撞器。

![image-20220314115102229](https://raw.githubusercontent.com/StarryJam/PicDock/main/imgimage-20220314115102229.png)

```c#
/// <summary>
/// 玩家控制器
/// </summary>
public class PlayerController : MonoBehaviour, IUnitController
{
    public UnitInstructions Instructions { get; private set; }

    private void Update()
    {
        //移动控制
        Vector2 moveDir = Vector2.zero;

        if (Input.GetKey(KeyCode.W))
        {
            moveDir += Vector2.up;
        }
        if (Input.GetKey(KeyCode.A))
        {
            moveDir += Vector2.left;
        }
        if (Input.GetKey(KeyCode.S))
        {
            moveDir += Vector2.down;
        }
        if (Input.GetKey(KeyCode.D))
        {
            moveDir += Vector2.right;
        }

        
        //攻击控制
        Vector2 attackDir = Vector2.zero;
        
        if (Input.GetMouseButton(0))
        {
            //创建射线
            Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
            RaycastHit hit;
            //与地面进行射线碰撞检测
            Physics.Raycast(ray, out hit, 1000, 1 << LayerMask.NameToLayer("Floor"));
            attackDir = (hit.point - transform.position).ToVector2().normalized;
        }

        if (attackDir == Vector2.zero && moveDir == Vector2.zero)
            Instructions = null;
        else
        {
            Instructions = new UnitInstructions();
            Instructions.attackDirection = attackDir;
            Instructions.moveDirection = moveDir;
        }
        
        //在调试模式显示一下攻击移动方向
        Debug.DrawRay(transform.position, attackDir.ToVector3(), Color.green);
        Debug.DrawRay(transform.position, moveDir.ToVector3(), Color.red);
    }
}
```

之后将一个Unit预制体拖到场景中，添加上玩家控制器组件测试一下基本功能：

![UnitTest4](https://raw.githubusercontent.com/StarryJam/PicDock/main/imgUnitTest4.gif)

**子弹**

为了不喧宾夺主我们子弹就用最简单的方法实现，为了不会在发射的时候撞到发射者一会儿写发射逻辑的时候让子弹在离发射者一定距离的地方生成。

```c#
public class Bullet : MonoBehaviour
{
    //飞行速度
    public float speed = 10;

    //生命周期时长
    public float lifeCycle = 1;

    //飞行方向
    public Vector2 direction;

    private void Start()
    {
        //生命周期结束后自我销毁
        Destroy(gameObject, lifeCycle);
    }

    private void Update()
    {
        transform.position += direction.normalized.ToVector3() * (speed * Time.deltaTime);
    }

    private void OnTriggerEnter(Collider other)
    {
        //撞到单位扣血
        var unitCmp = other.GetComponent<Unit>();
        if (unitCmp)
        {
            unitCmp.curLife--;
            Destroy(gameObject);
        }
    }
}
```

和单位一样创建一个预制体并绑定脚本，加上刚体和碰撞体组件。

给Unit类中加入一个Bullet成员变量并在单位预制体编辑器界面中把子弹预制体拖进去。

![UnitTest2](https://raw.githubusercontent.com/StarryJam/PicDock/main/imgUnitTest2.gif)

玩家部分的功能完整了，最后来解决AI的部分。



**Enemy AI**

敌人AI的逻辑比较简单，也没有复合状态，这里就贴一下控制器的代码以及条件和类的文件名，想看源码的话可以下载工程查看。

![image-20220315163520476](https://raw.githubusercontent.com/StarryJam/PicDock/main/imgimage-20220315163520476.png)

```c#
//敌人AI控制器
public partial class EnemyController : MonoBehaviour, IUnitController
{
    public UnitInstructions Instructions { get; private set; }

    private FiniteStateMachine<EnemyController> _stateMachine;
    
    //缓存玩家的GameObject
    private GameObject _player;

    public float tracingRange = 5;
    public float attackingRange = 3;

    private void Awake()
    {
        _stateMachine = new FiniteStateMachine<EnemyController>(this);
        _ConfigStateMachine();
        _stateMachine.Awake();
        //开始时获取玩家单位
        _player = GameObject.Find("Player");
    }

    //状态机配置
    void _ConfigStateMachine()
    {
        _stateMachine
            .AddTransition(EnemyController_State.Idle, EnemyController_State.Attacking,
                EnemyController_Condition.EnterAttackingRange)
            .AddTransition(EnemyController_State.Idle, EnemyController_State.Tracing,
                EnemyController_Condition.EnterTracingRange)
            .AddTransition(EnemyController_State.Tracing, EnemyController_State.Attacking,
                EnemyController_Condition.EnterAttackingRange)
            .AddTransition(EnemyController_State.Tracing, EnemyController_State.Idle,
                EnemyController_Condition.LeaveTracingRange)
            .AddTransition(EnemyController_State.Attacking, EnemyController_State.Idle,
                EnemyController_Condition.LeaveAttackingRange)
            .AddTransition(EnemyController_State.Attacking, EnemyController_State.Tracing,
                EnemyController_Condition.BetweenAttackingAndTracingRange)
            .DefaultStateID = EnemyController_State.Idle;
    }

    private void Start()
    {
        _stateMachine.Start();
    }

    private void Update()
    {
        _stateMachine.Update();
    }

    //获取和玩家的距离
    private float _GetDistanceWithPlayer()
    {
        return Vector3.Distance(transform.position, _player.transform.position);
    }
}
```

给另一个单位加上EnemyController组件之后的运行效果（蓝色由玩家控制黄色又电脑控制）：

![UnitTest3](https://raw.githubusercontent.com/StarryJam/PicDock/main/imgUnitTest3.gif)



### 总结

到此为止，我们从零开始，一步步完成了状态机的搭建，基于C#的反射功能做到了一定的自动化装配，将原本的类依赖改为接口依赖轻松实现了复合状态机，最后在Demo中尝试在两个独立的系统中本别应用了我们状态机的完整功能。在我重新实现这个系统的过程中，也对之前的代码进行了不少的优化，比如彻底隔离了对状态类实例直接进行操作，对配装过程进行了链式编程的优化等，可谓是温故而知新了。当然其实在做Demo的时候也不难发现，这个系统还是有相当大的优化空间的，最明显的比如条件类和状态类的新建过程过于繁琐，然后条件类的抽象程度相当低，无法复用代码，即使是数值上的不同也需要新建一个类来实现，这点在这一个Demo中展现得淋漓尽致，仅仅为了判断距离，新建了五个类文件，之后的优化应该是朝着填表或可视化界面配置状态机，以及代码自动化创建的方向努力了。但是不管怎么说能够从头实现这个系统还是有一定的成就感的！

