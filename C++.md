# C++

---

## 为什么选择c++

​	C语言是一门面相过程的编程语言，虽然可以通过函数指针做到类类似于**OOP**的效果，但是比较麻烦，且可读性较低。

-   随着维护的程序量越来越庞大，传统的面相过程是从计算机角度考虑问题，对每一步骤进行逻辑上的分析注重于程序基本实现，目标功能完成就是成功。相对而言不注重维护与修改，可移植性差一些。
-   而使用面相对象的思想从人的角度考虑问题，对每个被操作的对象进行观测与控制，注重于可重复单元内部操作的封装、新增属性与扩充内容的继承同一范畴下相似数据类型的相似操作集成的多态，利于开发与维护。

## C with class

​	在嵌入式领域，由于资源往往相对紧张，部分人员会采用 “C with class” 的写法，以此来舍弃 C++ 中一些资源占用较多的特性，仅保留类与命名空间等较为简单且不会显著增加调用开销的特性进行开发。

> 像是STL等会占用大量的RAM或者ROM。

​	然而，对于我们所使用的 C 板 A 板，其资源较为充裕，一般不会出现因使用完整 C++ 而导致资源不足的问题。不过，即便如此，“C with class” 的开发方式仍有一定价值，因为它在保持代码简洁性和可读性方面具有一定优势，比如通过类的封装特性可以更好地组织代码结构，使代码更易于维护和扩展，同时使用命名空间可以有效避免命名冲突，提高代码的模块化程度。

>   STL 除外，其中一些复杂的组件可能会占用较多的 RAM，且调试起来相对麻烦

我接下来的讲解的代码都是C++特性比较多的~~~因为C++代码有设计感~~。

## 类

类可以看作一个**大的结构体**，结构体内部的成员变量都是public属性的，而类是可以自定义的成员变量属性。

>   如果不显示声明成员那么默认是private。

-   自定义访问属性

-   类的内部可以将函数直接当作成员

-   ......

### 构造函数

#### 1.初始化

构造函数可以当作初始化来用，声明类时可以传入参数用作初始化，比如下面C语言代码。

~~~C++
void TDQuadratic_Init(TDQuadratic* td, float r, float h)
{
    td->r = r;
    td->h = h;
}

//调用

//需要调用Init方法初始化
int main(void)
{
    TDQuadratic td	//全局变量
    TDQuadratic_Init(&td, 100, 0.005)	//调用初始化方法
    
    return 0;
}
~~~

如果可以用C++构造函数就可以写成这样，并且不用调用`Init()`方法。

~~~C++
class TDquadratic
{
	TDquadratic(float r = 300.0f, float h = 0.001f) : r(r), h(h)
	{
	}
}

//类的声明
TDquadratic td(100, 0.005);
~~~

***构造时间不对会有BUG，如果需要解决这个bug得用延时构造***。但是我还没遇到过。

#### 2.构造函数列表

如果在一个类里面声明一个带参的构造函数，是不能像上面代码块展示的直接在后面传参，而是需要使用构造函数列表。

~~~c++
// 构造函数定义，使用初始化列表
class Gimbal
{
public:
	Gimbal::Gimbal()
    	: adrc_yaw_vel(Alg::LADRC::TDquadratic(100, 0.005), 8, 40, 0.1, 0.005, 16384),
      	// 速度pid的积分
      	pid_yaw_angle{0, 0},
      	// pid的k值
      	kpid_yaw_angle{8, 0, 0}
	{
    	// 其他初始化逻辑（如果有）
	}
    
private:
    Kpid_t kpid_yaw_angle;         // yaw轴pid增益
    PID pid_yaw_angle;             // yaw轴pid计算
    Alg::LADRC::Adrc adrc_yaw_vel; // adrc的速度环
}



~~~

## 继承

继承的特性我用到的和网上说的差不多，按着视频看就完事。

**不到万不不得以，不要使用多继承**

>   多继承会让程序的可读性大大降低，并且会有菱形继承的问题

## 多态

​	如果使用到多态的技巧，那肯定要扯到虚函数了，***虚函数会多一次函数调用的开销***，这是因为虚函数调用需要通过虚函数表来查找实际调用的函数地址，从而实现动态绑定。

​	多态的前提是有**抽象的基类**，如果使用的好，会让代码有极高的复用率。

-   比如我的电机库：

[SG_GIMBAL_C_2025_2_7/User/BSP/Motor at main · YALDZJC/SG_GIMBAL_C_2025_2_7](https://github.com/YALDZJC/SG_GIMBAL_C_2025_2_7/tree/main/User/BSP/Motor)

使用`MotorBase.hpp`基类，先把调用方法定义好，再将`Parse()`方法抽象，让其他子类重写。这样就省去了每个电机库都要重新写一大堆的`get()`方法。

>   这个库还利用策略模式，将每个不同电机的减速比、力矩常数等考虑进去，再通过`Configure()`方法统一度量衡。



-   还一个例子就是遥控器管理库：

[SG_GIMBAL_C_2025_2_7/User/APP/Mod at main · YALDZJC/SG_GIMBAL_C_2025_2_7](https://github.com/YALDZJC/SG_GIMBAL_C_2025_2_7/tree/main/User/APP/Mod)

因为两款遥控器的控制逻辑完全不同，一个多是按钮(mini)，一个是拨杆(dr16)。为了统一遥控器接口，我将他们的各个按键状态直接抽象为各种模式

~~~c++
//基类写法
class IRemoteController
{
  public:
    virtual ~IRemoteController() = default;

    // 连接状态
    virtual bool isConnected() const = 0;

    // 云台模式
    virtual bool isVisionMode() const = 0;
    virtual bool isLaunchMode() const = 0;
    virtual bool isKeyboardMode() const = 0;
    virtual bool isStopMode() const = 0;
}
~~~

然后在子类复写，将每个模式的逻辑写清楚

~~~c++
/**
 * @brief DR16遥控器实现
 */
class DR16RemoteController : public IRemoteController
{
  public:
    bool isConnected() const override
    {
        return Dir_Event.getDir_Remote(); // Dir_Remote为true表示断连
    }

    bool isVisionMode() const override
    {
        return ((dr16.switchLeft() != Dr16::Switch::MIDDLE) && (dr16.switchRight() == Dr16::Switch::MIDDLE)) ||
               (dr16.mouse().right == true);
    }

    bool isLaunchMode() const override
    {
        return (dr16.switchRight() == Dr16::Switch::UP);
    }
}

/**
 * @brief Mini遥控器实现
 */
class MiniRemoteController : public IRemoteController
{
  public:
    MiniRemoteController()
    {
        // 初始化BSP::Key::SimpleKey类
        key_paused = BSP::Key::SimpleKey();
        key_fn_left = BSP::Key::SimpleKey();
        key_fn_right = BSP::Key::SimpleKey();
    }

    bool isConnected() const override
    {

    }

    /**
     * @brief 视觉模式为扳机键
     * 并且视觉准备标志位要为true
     * 并且视觉发送的绝对值大于0
     *
     * @return true
     * @return false
     */
    bool isVisionMode() const override
    {
        auto &remote = Mini::Instance();
        return ((remote.trigger() == Mini::Switch::UP) || (remote.mouse().right == true));
    }

    bool isLaunchMode() const override
    {
        return key_fn_left.getToggleState();
    }
}
~~~

这样就做到了多态，最后通过一个管理类，判断获取`MiniRemoteController`的指针还是`DR16RemoteController`的指针，就可以做到统一接口管理了。

~~~c++
auto *remote = Mode::RemoteModeManager::Instance().getActiveController();

if (remote->isStopMode())
{
    TASK::GIMBAL::gimbal.setNowStatus(TASK::GIMBAL::DISABLE);
}
else if (remote->isVisionMode() && Communicat::vision.getVisionFlag())
{
    TASK::GIMBAL::gimbal.setNowStatus(TASK::GIMBAL::VISION);
}
else if (remote->isKeyboardMode())
{
    TASK::GIMBAL::gimbal.setNowStatus(TASK::GIMBAL::KEYBOARD);
}
else
{
    TASK::GIMBAL::gimbal.setNowStatus(TASK::GIMBAL::NORMOL);
}
~~~

​	这样做的好处就在于：如果你需要加入一个新的遥控器，那么你只要完成BSP层的数据解析，然后在继承`IRemoteController`复写你的方法，就可以直接使用了，此外无需再修改任何一行代码，这就是多态带来的灵活性与极高的拓展性。

>   遥控器管理类还需要修改一下切换的时机。

**但是**使用多态可能会使代码结构变得复杂，以及在某些情况下可能会带来一定的性能开销，在实际开发中，需要根据具体需求权衡是否使用多态（~~我吃过这个亏~~。详情请看我的底盘代码，使用*状态模式*导致代码冗余，当时也不怎么会c++）。



## 封装

### 介绍

封装是一个经久不衰的话题了，封装有几个核心理念：

1. 信息隐藏
    即隐藏对象的内部实现细节，仅对外暴露必要的接口。这使得对象的使用者无需关心其内部复杂的实现逻辑，只需了解如何通过接口与对象交互。这种信息隐藏可以降低系统各部分之间的相互依赖，提高代码的可维护性和可扩展性。
2. 模块化
    封装是实现模块化编程的重要手段。通过将数据和相关操作封装在一个类或模块中，可以使代码结构更加清晰、模块化程度更高。每个模块相对独立，便于单独开发、测试和维护，同时也更容易在不同项目中进行复用。
3. 降低耦合度
    良好的封装可以降低系统各组件之间的耦合度。如果一个组件的内部实现被外部代码广泛依赖，那么对这个组件的任何修改都可能影响到其他部分。通过封装，将内部实现细节隐藏起来，仅提供明确的接口，可以减少组件之间的依赖关系，使系统更加稳定和灵活。
4. 提高代码的可读性和易用性
    封装可以使代码更加易读和易用。当一个类或模块的接口设计得简洁明了时，使用者可以快速理解其功能和用法，而无需深入研究其内部复杂实现。这有助于提高开发效率，降低学习成本。
5. 提高代码的复用性
    封装好的类或模块可以更容易地在不同的项目或系统中进行复用。只要接口设计得当，即使底层实现发生变化，只要接口保持不变，外部代码就可以继续使用而无需修改。

### 示例

*但是什么才叫一个好的封装？*

我展示我的写法：

**1. 库里只提供调用的方法，而不提供成员变量，如果需要修改成员变量，尽量使用`set()`方法**。换句话说，就是使成员变量不能直接被修改。
我最近写的代码，就拿火控举例：

~~~c++
void Class_ShootFSM::HeatLimit()
{
    auto CurL = BSP::Motor::Dji::Motor3508.getTorque(1);
    auto CurR = BSP::Motor::Dji::Motor3508.getTorque(2);

    auto velL = BSP::Motor::Dji::Motor3508.getVelocityRpm(1);
    auto velR = BSP::Motor::Dji::Motor3508.getVelocityRpm(2);

    Heat_Limit.setBoosterHeat(100, 10);
    Heat_Limit.setFrictionCurrent(CurL, CurR);
    Heat_Limit.setFrictionVel(velL, velR);
    Heat_Limit.setTargetFire(target_dail_omega);

    Heat_Limit.UpData();
}
~~~

可以看到我在实际调用的时候，只提供`set()`方法，只要将必要的`set()`完就可以直接使用这个库了。

这种做法使成员变量得到保护，只能通过指定的方法进行操作，增强了代码的安全性和可维护性。

> 单例模式的获取实例方法与依赖库另说。

**2. 如果需要读取成员变量，请尽量用`get()`方法获取**。我在备赛期间被问到的最多的问题就是：

*为什么这个值在这里能够正常显示，在另一个地方却不一样了？*

我给出的答案是：*肯定有个地方你给赋值了*。

意思就是：成员变量被外部的某个值**强制修改了**。

这个时候应该将成员变量的属性全部改为私有的，只对外提供公共的`get()`方法,外部的代码想要访问成员变量只能通过调取`get()`方法间接访问。

这种做法的好处就是你无法通过外部访问修改成员变量，**减少错误赋值的BUG**。
~~~c++
int main()
{
    float filter_vel = 0;
    
    filter_vel = td.x1;         //不要使用这种直接访问
    filter_vel = td.getX1();    //尽量使用这种安全访问

    return 0;    
}
~~~

**3. 整个库只会有一个更新方法，其他的方法在这个方法里自己更新**。就拿上述代码来说，只有一个`UpData`方法，不用再额外的调用

> 如果没有必要用多个更新方法的话，就~~尽量~~直接省去

再比如ADRC代码：
[SG_GIMBAL_C_2025_2_7/User/APP/Mod at main · YALDZJC/SG_GIMBAL_C_2025_2_7](https://github.com/YALDZJC/SG_GIMBAL_C_2025_2_7/tree/main/User/Algorithm/LADRC)
在`UpData()`方法中，将其他更新的方法集中更新,*减少外部能访问的成员函数*。

~~~c++
float Adrc::UpData(float feedback)
{
    // 跟踪微分器处理目标值
    td_.Calc(target_);

    // ESO估计系统状态和扰动
    ESOCalc(target_, feedback);

    // 计算控制律
    SefCalc();

    return u;
}
~~~

但是这样写也有弊端，比如会多一次函数调用的开销
> 不过，使用`get()`或者`set()`方法会多一次函数调用的开销，但在大多数情况下，这种开销是非常小的，可以忽略不计,

封装不仅仅是为了保护数据和减少 bug，还能提高代码的可读性、模块化程度以及便于后期的维护和扩展。

当然，在某些特殊情况下，可能需要直接访问成员变量，但这种情况相对较少，且需要经过谨慎的评估和权衡。

封装是面向对象编程中一个非常重要且实用的概念，熟练掌握封装技巧有助于编写出高质量、可靠的代码。

### 单一职责原则(SRP)

#### 介绍

单一职责原则是 SOLID 设计原则中的第一个原则，它强调 **一个类或模块应该有且只有一个引起变化的原因**。换句话说，一个类应该只负责一项明确的职责。
这一原则的核心价值在于：

1. **高内聚**：将相关功能集中在同一个模块中，避免功能分散。
2. **低耦合**：减少类之间的依赖关系，降低修改一个功能对其他模块的影响。
3. **可维护性**：当需求变化时，只需修改负责特定功能的类，而不影响其他部分。

#### 示例

但是有同学又要问了`HeatLimit`类明明同时负责三个职能

- 计算热量积累
- 控制拨盘的频率
- 存储热量数据

那么它违反了*SRP原则*了——吗？

我在这里 强调一下，代码不要过度设计，~~不要过度设计，不要过度设计~~

> 重要的事说三遍
>
> “设计时应根据实际需求判断职责边界。若多个操作服务于同一目标且变化方向一致，合并它们是合理的；反之则需拆分。”

`HeatLimit`类只负责 关于火控相关的职责：监控发弹量和限制拨盘频率

所有`set()`均为**实现一个目标**：控制热量

虽然涉及多个操作（如设置热量冷却与上限、摩擦轮电流、摩擦轮速度等），但这些操作本质上都属于「热量限制策略」的一部分（例如：热量限制的计算需要依赖摩擦轮的状态判断发弹量），它们都**可以被视为同一个职责的组成部分**。

### 开闭原则(OCP)

#### 介绍

开闭原则强调 **软件实体（类、模块、函数）应对扩展开放，对修改关闭**。即当需求变化时，应通过扩展（如继承、组合）来添加新功能，而不是修改已有代码。
这一原则的意义在于：

1. **稳定性**：已有代码经过充分测试，减少因修改引入新 Bug 的风险。
2. **灵活性**：通过抽象和接口定义扩展点，支持未来的功能迭代。

#### 示例

上述遥控器代码已经阐述的很清楚了，有什么不明白的可以问。

具体体现在上文：*这样做的好处就在于：如果你需要加入一个新的遥控器，那么你只要完成BSP层的数据解析，然后在继承`IRemoteController`复写你的方法，就可以直接使用了，此外无需再修改任何一行代码，这就是多态带来的灵活性与极高的拓展性。*

- “扩展”与“修改”

> “`IRemoteController` 抽象基类定义了稳定的接口，新增遥控器类型时只需添加子类（扩展），而无需修改已有遥控器逻辑或调用方代码（关闭修改）。”

- `OCP` 的核心是通过抽象（如接口、基类）定义扩展点。可以补充：

> “开闭原则的关键是**对抽象编程，而非具体实现**。通过抽象基类 `IRemoteController` 隔离了调用方与具体遥控器实现的依赖，使得系统可以无限扩展。”

`OCP `并不意味着完全禁止修改。当抽象接口无法满足新需求时，仍需调整接口，但应尽量兼容已有代码。

### 其他

上述两个只是**SOLID五大原则**中的其中两种，剩余的可以自行了解。

但是切记代码**不要过度设计**。

## 命名空间

### 为什么使用命名空间

1.   防止命名冲突：如果有两个命名相同的变量，使用命名空间进行分隔就不会冲突。
2.   模块化
3.   便于维护
4.   增加可读性

### 嵌套命名空间的好处

在c++17中更新的新特性，命名空间可以直接嵌套使用

使得命名空间使用起来更加简介~~这也是我为什么使用c++17的原因~~。

~~~C++
//c++17以后的嵌套类名
namespace APP ::Heat_Detector
{
}

//c++17以前
namespace APP
{
namespace Heat_Detector
{
        
}
}
~~~
​	在`vscode`中，如果我想访问我写的库，比如电机库，我只需要打出`BSP::`，然后就会提示出`DM`公司或者`DJI`公司,继续`::`就会弹出具体的电机型号，直接选择即可。不必再回想复杂的类名~~如果你的类名特别长的话~~。

<img src="./assets/%E7%A4%BA%E4%BE%8B%E5%B5%8C%E5%A5%97%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4.gif" alt="示例嵌套命名空间" style="zoom: 50%;" />



## 一些关键字
### 内联inline

#### inline介绍

​	inline 关键字用于建议编译器尝试将函数体内联到每个函数调用的位置。这意味着编译器会在函数调用的位置直接插入函数的代码，而不是执行常规的函数调用，从而减少函数调用的运行时开销。

​	使用 inline 定义一个函数时，实际上是在提示编译器：“如果可能的话，可以考虑避免这个函数的调用开销，直接展开它的代码。”然而，这只是一个优化建议，最终是否内联，取决于编译器的实现和对特定代码的评估。因此，inline 并不保证函数一定会被内联。

​	inline 在 C++ 中不仅用于优化函数调用的开销，还扮演着控制函数链接属性的关键角色。inline 关键字特别有助于解决程序中的多重定义问题，这通常发生在同一个头文件被多个源文件包含时。

​	C++ 标准要求函数和对象的定义在整个程序中必须唯一，违反这一规则通常会导致链接器错误。然而，inline 函数是一个例外。

​	由于 inline 函数在不同编译单元中的多个定义都被视为同一函数的实例，因此它们不会触发链接错误，即使这些函数的定义在代码中出现多次。

​	inline 函数的另一个关键特性是在编译时的处理。当编译器遇到 inline 函数时，它可能将函数体在每个调用点展开。即使没有展开，**编译器也保证这些函数定义在链接时是可用的，并视为同一符号，从而避免了链接冲突**。

​	所以，inline就有如下妙用：

#### inline的妙用

在c++17中，inline允许修饰变量。

写过*only head*库的同学都知道在`.h`或者`.hpp`文件里声明一个全局类后，根本就不需要在.c文件里创建全局实例。

但是你如果用了`.cpp`文件，就必须要在`.cpp`文件里先创建-+实例，非常不方便~~不优雅~~。

上述介绍，inline可以改变链接属性，如果利用c++17语法，用inline修饰的话，就可以做到只在`.hpp文`件中声明类`.cpp`可以不用管，就能做到和extern类似的效果。

~~~c++
/**
 * @brief 电机实例
 * 模板内的参数为电机的总数量，这里为假设有两个电机
 * 构造函数的第一个参数为初始ID，第二个参数为电机ID列表,第三个参数是发送的ID
 *
 */
inline GM2006<1> Motor2006(0x200, {1}, 0x200);
inline GM3508<2> Motor3508(0x200, {2, 3}, 0x200);
inline GM6020<1> Motor6020(0x204, {1}, 0x1FE);

//这样就直接可以全局调用了
~~~

### constexpr

#### 介绍

​	`constexpr`（constant expression）是 C++11 引入的关键字，用于声明常量表达式。

-   **常量表达式**：`constexpr` 用于声明编译期就能确定的常量值，如常量变量、常量函数等。这意味着它们的值***在编译时***就被计算出来，而不是在运行时。
-   **编译期计算**：编译器会强制要求 `constexpr` 声明的值必须在编译阶段确定，以确保其常量性。

-   **提升性能**：由于值在编译期计算，运行时无需再进行计算，减少了运行时开销。

> `constexpr`能修饰函数,但是我没用过,特性与修饰变量是不一样

#### 示例

一些单位的换算可以直接用`constexpr`得出系数，这样不用每次运行都重新计算。

~~~c++
//这里的staitc为强调意图是代码风格，可省略
static constexpr double deg_to_rad = 0.017453292519611;
static constexpr double rad_to_deg = 1 / 0.017453292519611;
~~~

### auto

#### 介绍：

- 对于c++这种严格类型的语言来说，auto会导致别人很难第一时间得知你的数据类型。

    > 如果IDE够牛逼的话，可以把鼠标放在auto上，可以显示数据类型，这样就不会有上述问题

- 如果代码足够清晰的话，auto能使代码变得简洁、工整。

- *Lambda 表达式*是匿名函数对象，其具体类型由编译器自动生成且比较复杂，无法直接用常规类型来表示，所以必须使用 auto 来进行类型推断。
- auto本身**不会有性能开销**，因为在编译期就能推导出数据类型。

#### 示例：

这里使用auto来获取遥控器实例的类型，`Mode::IRemoteController *remote`这是完整类型。

~~~c++
// 获取遥控器实例
auto *remote = Mode::RemoteModeManager::Instance().getActiveController();
// 更新遥控器数据
remote->update();
// 更新键盘状态
APP::Key::KeyBroad::Instance().Update(remote->getKeybroad());
~~~

但是auto千万不能滥用，否则会导致代码的可读性直接变得稀烂。

int、bool、float这些常用直接定义的时候就最好别auto了，除非是如下情况。

~~~c++
auto *remote = Mode::RemoteModeManager::Instance().getActiveController();
// 这里的remote_rx与remote_ry都是auto,但是不确定getRightX()的类型是什么
auto remote_rx = remote->getRightX();
auto remote_ry = remote->getRightY();
~~~

## 一些技巧

### 通信

在与其他设备通信时，大部分数据的内存都是按照协议规定排序的。对于这种已经严格划分内存区域的数据，可以使`std::memcpy`函数进行数据的读取与发送。

在hpp中利用alignas(uint64_t)将数据严格对齐，不要让编译器自动填充。

> **__attribute__**((packed))关键字也可以

然后使用**位域结构体根据手册**的数据内存大小，划分每个数据的字节数(如果数据正好是完整的字节，可以省略字节数的指定)。

~~~c++
/*.hpp*/
struct alignas(uint64_t) DjiMotorfeedback
{
    int16_t angle;
    int16_t velocity;
    int16_t current;
    uint8_t temperature;
    uint8_t unused;
};


/*.cpp*/
// 解析函数
/**
 * @brief 解析CAN数据
 *
 * @param RxHeader  接收数据的句柄
 * @param pData     接收数据的缓冲区
 */
void Parse(const CAN_RxHeaderTypeDef RxHeader, const uint8_t *pData)
{
    const uint16_t received_id = CAN::BSP::CAN_ID(RxHeader);

    for (uint8_t i = 0; i < N; ++i)
    {
        if (received_id == init_address + recv_idxs_[i])
        {
            std::memcpy(&feedback_[i], pData, sizeof(DjiMotorfeedback));
			//由于can发送的是大端数据，所以需要用__builtin_bswap16反转大小端序
            feedback_[i].angle = __builtin_bswap16(feedback_[i].angle);
            feedback_[i].velocity = __builtin_bswap16(feedback_[i].velocity);
            feedback_[i].current = __builtin_bswap16(feedback_[i].current);

            Configure(i);

            this->runTime_[i].dirTime.UpLastTime();
        }
    }
}
~~~

### 有限状态机

在使用**DT7遥控器**进行各种模式的切换时，我时常感觉用13，12，21，22这种基于*拨杆状态切换模式*的状态机过于反人类。

于是，刚开始学c++的利用刚开始学习的c++重构了一套基于状态模式的**事件驱动**的状态机，详情请看我的底盘代码：

[SG_CHASSIS_C_2024_11_5/MDK-ARM/User/Task/ChassisTask.cpp at main · YALDZJC/SG_CHASSIS_C_2024_11_5](https://github.com/YALDZJC/SG_CHASSIS_C_2024_11_5/blob/main/MDK-ARM/User/Task/ChassisTask.cpp)

我最初的设想很好，符合开闭原则，模式之间直接解耦，一键更新。但是对于RM工况来说，并不需要这么复杂的操作，模式切换较少。因为各个模式之间，貌似只有期望值不同，最后导致代码臃肿，很多重复逻辑(**不要过度设计**)。

- 对于底盘来说，其实只有两种模式。

    1. 底盘跟随

    2. 底盘不跟随

    3. 停止模式

        > 当我们把vw单独用旋钮拎出来后，小陀螺模式只是vw的期望值不同了而已。

- 而对于云台来说。

    1. 视觉期望值
    2. 遥控期望值
    3. 键鼠期望值
    4. 停止模式

        > 底盘可以不用专门设置底盘的键鼠模式，不过底盘需要对键鼠操作做一定拓展，比如加减功率这些

- 对于发射机构来说

    1. 摩擦轮期望值为0，拨盘期望值等于反馈值
    2. 摩擦轮设置期望值，拨盘设定连续期望值
    3. 摩擦轮设置期望值，拨盘设定单发期望值

使用简单的**基于事件驱动的有限状态机**便可以满足需求，这是我看完中科大的视频后的想法。

只需要继承有限状态机类就可以直接使用，简单，易懂，可读性高，拓展性也不差~~(RM工况)~~。

~~~c++
/*hpp*/
enum Gimbal_Status
{
    DISABLE,
    NORMOL,
    VISION,
    KEYBOARD
};

//继承父类
class Gimbal : public Class_FSM
{
}

/*cpp*/
switch (Now_Status_Serial)
{
case (GIMBAL::DISABLE): {

    // 如果失能则让期望值等于实际值pp
    filter_tar_yaw_pos = BSP::IMU::imu.getAddYaw();
    filter_tar_yaw_vel = BSP::IMU::imu.getGyroZ();

    filter_tar_pitch += BSP::Motor::DM::Motor4310.getAddAngleDeg(1);

    // 检测状态变化的上升沿（进入DISABLE状态）
    if (DM_state.getRisingEdge())
    {
        BSP::Motor::DM::Motor4310.On(&hcan2, 1);
        osDelay(1);
    }

    break;
}
case (GIMBAL::VISION): {
    // 视觉模式
    filter_tar_yaw_pos = Communicat::vision.getTarYaw();
    filter_tar_pitch = Communicat::vision.getTarPitch();
    break;
}
case (GIMBAL::KEYBOARD): {
    // 键鼠模式
    filter_tar_yaw_vel = remote->getMouseVelX() * 100000;
    filter_tar_pitch += remote->getMouseVelY() * 1000;
    // 一键掉头
    TurnAround();
    break;
}
case (GIMBAL::NORMOL): {
    // 正常状态
    filter_tar_yaw_vel = remote_rx * 150;
    filter_tar_pitch += remote_ry * 0.5f;
    break;
}
}
~~~

### 设计模式
#### 单例模式

在嵌入式软件代码中，我们常常使用全局变量来实现全局访问，比如写一个遥控器的库，就必须在里面声明全局的结构体或者类。然而，这种做法可能会带来一些问题，比如全局变量的生命周期管理、内存泄漏、线程安全等。单例模式可以很好地解决这些问题。

单例模式是一种常用的软件设计模式，用于确保一个类在一个程序中只有一个实例，并提供一个全局访问点`Instance()`。基于C++构造函数的特性。在构造函数时，将返回值类型修饰为静态，这样这个实例的生命周期就会被延长，直到程序结束。这样，我们可以确保只有一个实例存在，并且在程序结束时自动销毁。
##### 生命周期

在单例模式中，实例的生命周期由静态成员变量`instance_`控制。当第一次调用`Instance()`时，会创建一个实例并赋值给`instance_`。之后每次调用`Instance()`都会返回这个已存在的实例，而不会创建新的实例。

> 你也可以将初始化放在`Instance()`中, 这样就不用专门有一个初始化函数了

##### 饿汉模式

**实现原理：**

饿汉模式在类加载时就完成了实例的创建，通过将实例的初始化放在的类静态初始化块中来实现。由于类的静态初始化块只会在类第一次被加载时执行一次，因此可以保证实例的唯一性。

**优点：**

- 实现简单，线程安全，因为类的静态初始化过程是由虚拟机保证线程安全的。
- 实例的创建在类加载时就完成，避免了懒汉模式中可能出现的多次实例化问题。

**缺点：**

- 实例的创建在类加载时就完成，可能会导致资源浪费，因为实例可能在程序的整个生命周期中都没有被使用。

~~~c++
class Mini
{
	// 获取单例实例(饿汉模式)
	static Mini& Instance()
	{
	    return instance_;
	}
}
~~~

##### 懒汉模式

**实现原理：**

懒汉模式在第一次调用获取实例的方法时才创建实例，这样可以延迟实例的创建，直到实际需要时才进行实例化。

> 我之前说过构造函数有可能在初始化没完成就构造了，懒汉模式就可以一定程度上解决这一点——延时构造

**优点：**

- 实例的创建延迟到第一次使用时才进行，可以避免资源浪费。

**缺点：**

- 线程不安全的懒汉模式在多线程环境下可能会导致多个实例被创建，因此需要加入线程同步机制，这会增加一定的性能开销。

    > 单片机的多线程问题几乎可以忽略，因为是大部分单片机都是单核运行

~~~c++
class Mini
{
	// 获取单例实例(懒汉模式)
	static Mini &Instance()
	{
	    if (instance_ == nullptr)
	    {
	        instance_ = new Mini();
	    }
	    return *instance_;
	}
}
~~~



##### 全局变量与单例模式

| **特性**         | **全局变量**                             | **单例模式**                                             |
| ---------------- | ---------------------------------------- | -------------------------------------------------------- |
| **控制实例数量** | 无法保证唯一性，容易产生多个实例         | 严格保证一个类只有一个实例                               |
| **生命周期管理** | 生命周期与程序相同，无法手动控制销毁时间 | 生命周期可控，可以通过析构函数或销毁函数控制实例销毁时间 |
| **存储位置**     | 通常存储在全局数据区                     | 可以存储在栈上（饿汉模式）或堆上（懒汉模式）             |
| **使用方式**     | 直接访问，无抽象层                       | 通过接口访问，具有更高的抽象层次                         |
| **资源管理**     | 无法延迟初始化，资源可能被提前占用       | 支持延迟初始化，按需分配资源                             |
| **灵活性**       | 灵活性低，难以扩展和修改                 | 灵活性高，易于扩展和修改                                 |
| **可测试性**     | 依赖全局状态，测试困难                   | 易于模拟和测试，提高可测试性                             |
| **代码封装性**   | 破坏封装性，容易导致代码耦合度高         | 封装性好，降低代码耦合度                                 |
| **内存管理**     | 内存泄漏风险高，无法自动释放             | 内存管理可控，可以通过析构函数释放资源                   |


~~~c++
namespace BSP::Remote
{

class Mini
{
  public: // 公有成员函数
    // 释放单例实例
    static void DestroyInstance()
    {
        if (instance_ != nullptr)
        {
            delete instance_;
            instance_ = nullptr;
        }
    }
}
// inline Mini remote; 单例模式可以不用再声明全局变量   
} // namespace BSP::Remote

/**
 * @brief 视觉模式为扳机键
 * 并且视觉准备标志位要为true
 * 并且视觉发送的绝对值大于0
 *
 * @return true
 * @return false
 */
bool isVisionMode() const override
{
    auto &remote = Mini::Instance();
    return ((remote.trigger() == Mini::Switch::UP) || (remote.mouse().right == true));
}
~~~

#### 状态模式

状态模式



## STD

​	c++的std库其实有很多可以直接用的常用功能，比如`memcpy`、`clamp`、`fmax`、`fmin`等等，不需要再自己手动写类似函数。但是std库中一些比较占用单片机资源的尽量不要使用。



## STL

##  Template
