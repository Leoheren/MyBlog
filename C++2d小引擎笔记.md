---
title: C++2d小引擎笔记
date: 2026-05-09 19:47:00
tags: 闲杂笔记
description: 
image: 
---

## 前置小知识
#### 编译器（compiler）
c++是编译语言，也就是说在你写完后需要一个编译器编译成二进制代码给电脑才能变成一个可执行文件。

**在IDE或终端写c++    =>    调用某个编译器变成10101010二进制      =>     构建可执行文件**

而通过终端直接调用编译器需要一遍又一遍地输入一些规则（比如：使用c++17规则构建，即时报错，指定文件名，指定你要链接的库等等）   
所以需要一个配置文件一次性填完这些规则，后面就不用再填写了。
教程中用的是GNU make，所以makefile正是这样的配置文件
之后只需要点出make build，就能自动配置规则了

#### 堆(heap)与栈(stack)与垃圾回收(garbage collect)
游戏中需要创建游戏对象。来看看c++是如何实例化对象和在内存中创建对象的：

可以在栈中创建对象，栈中创建的是连续的内存，如果创建对象时不使用new关键字，`Enemy enemy`则会在栈中创建，栈是先进后出的结构，一个个推进去，再一个个弹出来。离开作用域将会自动释放内存。
- 栈中，将会分配连续的内存块
- 编译器知晓内存块的大小
- 栈有固定的大小（取决于操作系统）
- 不必操心栈的内存分配或释放，编译器可以自动分配（创建对象时自动调用构造函数，离开作用域自动调用析构函数释放内存。
栈有固定的大小限制，如果超出大小就会栈溢出（stack overflow），所以针对比较复杂庞大的对象，可以考虑放在堆中。

堆中没有内存分配限制，是自由的，创建的时候使用new关键字`Enemy* enemy = new Enemy()`，虽然大小不限制，但是需要记得手动释放内存`delete enemy`，不然就会造成内存泄漏（Memory Leak).
- 堆中，内存通常是动态分配的，而不是连续的
- 堆没有固定大小限制，想拿多少拿多少
- 但堆的处理速度要比栈慢不少
- 堆需要手动释放内存

在堆中创建对象步骤：分配内存（malloc）→调用构造函数初始化（constructor）**这两个操作被整合为new关键字**
删除对象：调用析构函数摧毁（destructor）→释放内存（free）**这两个操作被整合为delete关键字**

不过后面会用到智能指针（smart point）来辅助我们进行这些操作，以免我们忘记管理内存。

而像c#这种，就有垃圾回收机制（garbage collect），也免除了各种麻烦的内存管理操作。

#### main文件与.h文件与.cpp文件
一切的主入口都是main函数，一般它单独写在一个“main”文件里，每次运行都先执行main函数，这点和c#差不多。

.h文件也叫头文件，.cpp文件叫实现文件。有点类似接口的感觉，头文件里只包含要实现的各种函数，没有具体的实现函数体，而实现文件中将对应的头文件用`#include<>引用出来，再具体实现。
头文件就类似目录，把对应的.cpp能干什么都罗列出来了。

还有需要记住的是，头文件一般得加上这样的保护壳，以免链接时重复调用

```c++
#ifndef GAME_H
#define GAME_H
#include<___>
#include<___>
#include<___>
//...
#endif
```


#### 链接器（Linker）

再回到刚刚说的步骤：

**在IDE或终端写c++    =>    调用某个编译器变成10101010二进制      =>     构建可执行文件**

了解更详细一点的步骤，于是：

**c++ 源码   =>    预处理器preprofessor读取源码中带#的语句，这是预处理指令   =>   编译器compiler检查函数调用，参数名，参数顺序是否合规，但不知具体实现   =>   链接器Linker找到这些函数的实现，真正进行构建，拉取、链接所有库，构建在一起    =>     构建可执行文件**


#### 检查依赖项与库
别忘了检查库与依赖项是否完备。在makefile中配置好要链接的库的实现，用一个简单的main.cpp去看看是否都能调用，这样后面就不用担心这些配置上的东西了，安心写src文件了


**以上均为教师用Linux终端使用GCC进行演示，阐述原理。接下来要用Win系统上的VisualStudio再重新过一遍流程**


我们所需要的第三方库有：
- glm（数学库）
- imgui（提供编辑器窗口以及实时监视运行状况）
- sol（支持lua嵌入c++的库）
- lua（脚本逻辑）
- sdl2（提供图形、声音、字体底层功能的跨平台库）
其中前三个，只需要在我们的项目中放入源码即可。因为他们只有.h.cpp.a文件，无需外部依赖项.lib.dll.a文件。
后两个也可以直接放源码，但每次编译都要跑一遍这俩太费时间。自己项目文件夹中只包含必要的头文件，我们直接下载用预先把实现文件编译好的二进制文件.lib.dll.a即可。这样就不用每一次编译都要本地重新跑一次实现文件，而是链接器直接链接对应的原始的二进制文件库，减少了编译的时间。

- 直接放第三方源码简单直接，但是每多一个第三方，编译时间将大幅增长。
 - 链接二进制库文件省去编译的时间，但是配置会非常麻烦。
 
**根据具体的项目需要，可以选择到底是放完整的第三方库源码，还是链接二进制文件**

这里跟着教程就把lua和sdl2配置成链接库的形式了

在win系统上配置第三方库比起另外两个平台稍显麻烦，不，应该是非常麻烦，耗了我一堆时间！需要在vs中的属性配置非常多的路径，稍微哪个地方不对就会报各种bug，反正后面两个，sdl2和lua的动态链接库dll和静态链接库lib都要保证放对地方路径输对，其次调用时自己的代码也要注意细节大小写这些。


## 正篇
### 一、游戏循环（GameLoop）
#### 1.基本实现

![[屏幕截图 2025-07-29 144455.png]]
如图，是一个大致的游戏循环的思路，处理输入→更新游戏逻辑→渲染。
而实际落实到代码上，完成一个基础的游戏循环是这样的：
![[屏幕截图 2025-08-05 110729.png]]
可以看到在前面多了一个Setup函数，用来在预设各种初始的参数（敌人玩家的初始位置，初始速度等等）
而下面是头文件所罗列的目前完成基本循环所需的所有函数及常变量

![[屏幕截图 2025-08-05 110706.png]]

可以看到又多了Initialize()和Destroy()，前者是程序开始时用来初始化各种第三方库的，后者则是负责打破循环退出程序的。Run()是用来封装所有循环流程的。
所以最终实现出来的全流程是：初始化第三方→设定好游戏内初始参数→执行游戏循环→打破游戏循环退出游戏。

#### 2.限制帧率与DeltaTime
游戏循环的三个步骤都被包进while循环语句中，而while语句是处理器指令，这带来两个严重的问题：
- 帧率无上限，一切都以最高帧率来跑，不同性能的电脑能跑的最高帧率不同，游戏速度就无法统一了，并且使用while会CPU占用率拉到满，性能爆炸（限制帧率，使用SDL_Delay）
- 进行帧率限制后，游戏速度还是会受具体的帧率影响，30帧与60帧的速度还是不一样（deltatime）
实现限制帧率后，我们让所有电脑跑该程序都保持在了1秒30帧或60帧的指定帧数，众生平等。
但是我们思维依旧停留在“每帧移动...像素”上，30帧和60帧的速度仍不公平，而deltatime将实时获取每帧之间的时间差并转换成秒单位，每次移动都乘上一个delta，这样就将思维转变成了“每秒移动...像素”上，这样不论是不同的电脑还是不一样的帧数设置，都将以统一的速度运行游戏。
（有了deltatime后，就算放开帧率限制，帧率无上限，物体依旧按照原来的速度移动）
![[屏幕截图 2025-08-05 111812.png]]
#### 3.关于SDL Renderer的补充
以下是SDL Renderer的API使用
同时可以看出操作渲染器进行渲染的步骤，并不是Load或Draw函数就能简单的绘出
首先，加载PNG纹理，再根据PNG大小指定一块渲染范围，再在指定的范围里进行渲染。
（有点类似种田，得在地里开垦出一块田才能中下去菜）

![[屏幕截图 2025-08-05 111822.png]]

### 二、日志(Logger)
记录游戏进程信息，记录报错的功能是非常重要的。课程中简单实现了一个日志系统，用来学习并了解日志的原理。基本上就是新写了个Logger类封装了一些cout语句，c++中获取时间日期的方法有很多种，我这里就简单使用了ctime库进行显示，后面的cout也很粗暴。可以结合strftime再封装一个函数更代码更简洁，但是莫名报错就没用了......不过这只是学习了解用，体验日志系统是如何来的，能跑就行:D
![[屏幕截图 2025-08-05 163527.png]]

而后创建数据结构记录日志，以便后期查看。
![[屏幕截图 2025-08-05 163542.png]]
教程也提过，如果是正式的项目生产中最好还是用现成的第三方日志库，速度质量都有保障。
用于c++的spdlog库和c的log-c库都是教程中所大力推荐的MIT协议库，往后正式项目中可以作使用参考。

tips:常整理文件夹结构，保持干净整洁。另外整理完了记得检查头文件引用路径。

### 三、引擎架构
#### 对象继承设计（Object Inherited Desgin） 
对象继承纵然很直观，但是随着游戏项目的庞大，会有越来越多的对象需要组织，这时就难以判断到底怎么规划对象的基类，再加上c++允许多重继承，会出现菱形继承的现象，很容易造成项目的混乱。![[屏幕截图 2025-08-06 110857.png]]
#### 基于组件设计（Component-Based Design）
基于组件的设计也就是Unity中使用的设计，主要关于Entity（实体）与component（组件）。实体即unity中的游戏对象，是一个抽象的概念，根据设计者想要设计的东西往里面塞组件，这些组件即是代表了实体的能力。比如一个直升机，可以移动加上transform组件，可以显示图像加上sprite组件，可以发生碰撞加上collision组件。。。就如同拼积木一样组成了直升机。假如后面需要添加装饰的花草，由于只是装饰，那就只添加sprite、transform组件即可。这样的好处就是非常的灵活，不需要考虑复杂的继承关系，想要什么对象直接拿组件拼装就可以。
![[屏幕截图 2025-08-06 140012.png]]
同时会有一个注册表类（也可以叫什么entities manager，world，之类的）管理这些实体，删除添加或改变每个实体上的组件。（比如游戏中玩家开启无敌模式，就需要这个类删除直升机的collision组件）用代码表现如下：
![[屏幕截图 2025-08-06 140038.png]]
可以看到，实体类中会有一个vector组件列表记录自己所包含的所有组件，并且有自己的更新绚烂逻辑。组件类下也会带update()与render()，组件也会有自己的逻辑，游戏运行时实体每一tick都执行自己的update()，同时把身上组件的update()一并执行，所有组件的update()共同形成了该实体的逻辑，render()同理。

#### ECS设计（Entity-Component-System Design）
上面的基于组件的设计还带有一些面向对象的思想，而进一步重组的ECS设计就更偏向于面向数据思想一些。面向对象思想对人类更加友好，但是对电脑性能不友好，而面向数据恰恰相反，写起来虽麻烦，但性能的提升是巨大的。

ECS设计在EC基础之上做了以下重构：
- Entity只是一个ID
- Component不再包含任何逻辑，只是纯数据
- Component不再受制于实体，他们作为数据，自己组织自己。比如在EC设计中，实体中有关于组件的列表，是实体给组件开了个vector数据结构用，其实就是按照实体本身的能力组织组件，还是从实体，从对象思考出发的，还是在面向对象的思想当中。但是ECS设计中，追求的是高性能，实体仅被当作标识符，组件被当作纯数据，我们不用在实体中给组件开数据结构用，我们让组件自己另找一种更高性能的数据结构（比如组件序列，组件池）组织在连续的内存中，这样组件自己组织自己，性能会更好。![[屏幕截图 2025-08-06 141223.png]]
可以看出，EC设计实体给组件指针开的这个vector指向的是随机位置的内存，电脑读取时会很费劲
![[屏幕截图 2025-08-06 143621.png]]
而在ecs中，组件另寻数据结构，使得数据在连续的内存上，电脑访问就会更加快速。
理解了这一点，就理解了面向数据与面向对象的区别，也理解了ECS的核心。
- 新增System用来在组件与实体上实际执行逻辑
一个系统只对某些实体感兴趣，比如移动系统只对有transform组件和velocity组件的实体感兴趣。


根据这样的思路，那么：AIcomponent会有一个专门的AIsystem，CollisonComponent会有一个CollisonSystem。。。这样写肯定麻烦不少，但是对项目性能与可持续发展大有裨益。
代码思路实现如下：
![[屏幕截图 2025-08-06 145134.png]]
当然，还是会有注册表类来管理一切。

### 四、ECS基本实现
只是学习了解ECS的基本实现思路和学习c++的部分知识，真项目还是建议用EnTT等成熟的第三方库

#### 1.ECS文件结构
以下是ECS相关文件在源代码文件夹下的结构。
ECS的核心实现单独放在ECS文件下，有.h和.cpp文件
Component和System分别有一个独立的文件夹。

![[屏幕截图 2025-08-08 111605.png]]

#### 2.实体类
ECS中的实体基本上就只包含一个ID,示范代码：
```cpp
class Entity {
    private:
        int id;

    public:
        Entity(int id): id(id) {};
        Entity(const Entity& entity) = default;
        int GetId() const;
       
        Entity& operator =(const Entity& other) = default;
        bool operator ==(const Entity& other) const { return id == other.id; }
        bool operator !=(const Entity& other) const { return id != other.id; }
        bool operator >(const Entity& other) const { return id > other.id; }
        bool operator <(const Entity& other) const { return id < other.id; }

        template <typename TComponent, typename ...TArgs> void AddComponent(TArgs&& ...args);
        template <typename TComponent> void RemoveComponent();
        template <typename TComponent> bool HasComponent() const;
        template <typename TComponent> TComponent& GetComponent() const;

        // Hold a pointer to the entity's owner registry
        class Registry* registry;
};
```
私有的只有一个id，公共的只有两个构造函数和一个获取自身id的函数，基本实现非常的简单。
后面的运算符重载和对注册表类功能的引用都是优化编码体验的。


 tips：运算符重载
像上面适当使用可让实体引用时更加方便清晰。
过量使用会造成混乱，小心使用。
#### 3.组件类
组件中只包含数据，组件根据功能不同，所包含的数据也不同，同时需要一个特定ID来代表它们在签名中在组件池中的位置，所以，一个基类包含所有组件都必须有的id，再派生模板类用来被各种具体的组件继承。
```cpp
struct IComponent {
    protected:
        static int nextId;
};

// Used to assign a unique id to a component type
template <typename T>
class Component: public IComponent {
    public:
        // Returns the unique id of Component<T>
        static int GetId() {
            static auto id = nextId++;
            return id;
        }
};

```
tips：c++模板
模板在ECS中会大量使用，并且在模板函数的实现得直接写在头文件里，不能去cpp文件里实现

###### 组件签名
系统只对持有特定组件的实体感兴趣。如何得知有哪些实体是系统真正关注的？我们利用组件签名来解决此问题。
所谓组件签名，就是一组bitset，位集，由一组01开关组成的队列，比如总共有32个组件，那么组件签名这个bitset上就有32个01开关，那么如果5号组件和2号组件挂到实体上，那么bitset上对应的5号2号开关就打开0→1，其余的保持关闭状态，这样就能表示一个实体上挂有哪些组件了。
```cpp
const unsigned int MAX_COMPONENTS = 32;

////////////////////////////////////////////////////////////////////////////////
// Signature
////////////////////////////////////////////////////////////////////////////////
// We use a bitset (1s and 0s) to keep track of which components an entity has,
// and also helps keep track of which entities a system is interested in.
////////////////////////////////////////////////////////////////////////////////
typedef std::bitset<MAX_COMPONENTS> Signature;
```
###### 组件池
组件中包含的数据需要有数据结构来储存。这正是ECS的核心解决问题——性能问题
```cpp
////////////////////////////////////////////////////////////////////////////////
// Pool
////////////////////////////////////////////////////////////////////////////////
// A pool is just a vector (contiguous data) of objects of type T
////////////////////////////////////////////////////////////////////////////////
class IPool {
    public:
        virtual ~IPool() {}
};

template <typename T>
class Pool: public IPool {
    private:
        std::vector<T> data;

    public:
        Pool(int size = 100) {
            data.resize(size);
        }

        virtual ~Pool() = default;

        bool isEmpty() const {
            return data.empty();
        }

        int GetSize() const {
            return data.size();
        }

        void Resize(int n) {
            data.resize(n);
        }

        void Clear() {
            data.clear();
        }

        void Add(T object) {
            data.push_back(object);
        }

        void Set(int index, T object) {
            data[index] = object;
        }

        T& Get(int index) {
            return static_cast<T&>(data[index]);
        }

        T& operator [](unsigned int index) {
            return data[index];
        }
};
```
这里存的是某一个特定组件在不同实体上的不同数据，而后面结合注册表类的实现，池上再套一层vector，形成一个类似二维数组的东西，就实现了存储不同组件再不同实体上的数据的目的了
 
#### 4.系统类
说到系统，再结合先前说的组件签名，每一个系统自己会有一个组件签名，根据自己需要，执行RequireComponent这个函数，打开自己签名上的对应开关。每个实体上也有一个组件签名，加了什么组件，就打开对应的开关。注册表类里存有所有实体的组件签名，注册表负责检查实体的签名和系统的签名是否吻合，并进行加入操作。

这里的公开方法曝露给注册表类使用，因为注册表负责检查比对双方的签名，只有签名对应时，系统才会关注实体。
```cpp
////////////////////////////////////////////////////////////////////////////////
// System
////////////////////////////////////////////////////////////////////////////////
// The system processes entities that contain a specific signature
////////////////////////////////////////////////////////////////////////////////
class System {
    private:
        Signature componentSignature;
        std::vector<Entity> entities;

    public:
        System() = default;
        ~System() = default;
        
        void AddEntityToSystem(Entity entity);
        void RemoveEntityFromSystem(Entity entity);
        std::vector<Entity> GetSystemEntities() const;
        const Signature& GetComponentSignature() const;

        // Defines the component type that entities must have to be considered by the system
        template <typename TComponent> void RequireComponent();
};
```
#### 5.注册表类
注册表类，也叫实体管理类，世界管理类，协调者类，因为这个类负责进行所有的全局操作，增删查取实体，组件，系统，都有这个类来执行。最复杂，也是ECS最核心的实现。
先看私有，有实体数量，组件池动态数组，所有实体的组件签名，无序映射表来装系统，并且有待加入实体和待删除实体名单。注册表类要对于整个ECS的信息都知晓，才能在公共的函数中管理所有的元素。
可见公共函数中管理组件与系统的全是模板函数，只有创建实体和把实体加入系统两个函数不是，毕竟实体只是一个标识符罢了，是需要即时创建的，不是已经有的东西。
```cpp
////////////////////////////////////////////////////////////////////////////////
// Registry
////////////////////////////////////////////////////////////////////////////////
// The registry manages the creation and destruction of entities, add systems,
// and components.
////////////////////////////////////////////////////////////////////////////////
class Registry {
    private:
        int numEntities = 0;

        // Vector of component pools, each pool contains all the data for a certain compoenent type
        // [Vector index = component type id]
        // [Pool index = entity id]
        std::vector<std::shared_ptr<IPool>> componentPools;

        // Vector of component signatures per entity, saying which component is turned "on" for a given entity
        // [Vector index = entity id]
        std::vector<Signature> entityComponentSignatures;

        // Map of active systems
        // [Map key = system type id]
        std::unordered_map<std::type_index, std::shared_ptr<System>> systems;

        // Set of entities that are flagged to be added or removed in the next registry Update()
        std::set<Entity> entitiesToBeAdded;
        std::set<Entity> entitiesToBeKilled;

    public:
        Registry() {
            Logger::Log("Registry constructor called");
        }
        
        ~Registry() {
            Logger::Log("Registry destructor called");
        }

        // The registry Update() finally processes the entities that are waiting to be added/killed to the systems
        void Update();
        
        // Entity management
        Entity CreateEntity();

        // Component management
        template <typename TComponent, typename ...TArgs> void AddComponent(Entity entity, TArgs&& ...args);
        template <typename TComponent> void RemoveComponent(Entity entity);
		template <typename TComponent> bool HasComponent(Entity entity) const;
        template <typename TComponent> TComponent& GetComponent(Entity entity) const;

        // System management
        template <typename TSystem, typename ...TArgs> void AddSystem(TArgs&& ...args);
        template <typename TSystem> void RemoveSystem();
        template <typename TSystem> bool HasSystem() const;
        template <typename TSystem> TSystem& GetSystem() const;

        // Checks the component signature of an entity and add the entity to the systems
        // that are interested in it
        void AddEntityToSystems(Entity entity);
};


```
具体的实现在源码文件看，这里只写大致思路。
另外写一下这个AddEntityToSystem的实现，以便了解比较关键的签名是怎么联系系统实体与组件的
```cpp
void Registry::AddEntityToSystems(Entity entity) {
    const auto entityId = entity.GetId();

    const auto& entityComponentSignature = entityComponentSignatures[entityId];
    
    for (auto& system: systems) {
        const auto& systemComponentSignature = system.second
        ->GetComponentSignature();

        bool isInterested = (entityComponentSignature & systemComponentSignature) == systemComponentSignature;

        if (isInterested) {
            system.second->AddEntityToSystem(entity);
        }
    }
}

```

tips：智能指针
- 唯一指针（unique_ptr)
- 共享指针（shared_ptr）
- 弱指针（weak_ptr）
主要是前两个比较重点
唯一指针用在不能共享所有权的对象上，唯一的一个东西，用make_unique创建一个智能指针，一旦超出所有者的作用域，智能指针将自动销毁释放内存。
共享指针跟唯一指针语法基本一致是给可以共享的物体用的，可以在最后一个所有者超出范围时才销毁对象。
有了智能指针，基本上用不上new关键字了，尽量去用智能指针


完成注册表类的工作后，基本上ECS的核心代码就算完成了，接下来就需要根据自己具体的需求写具体的组件和系统即可。
### 五、ECS搭建
#### 1.基本组件、系统搭建
关于具体组件与系统的实现，这里就给一个运动系统的例子简单说明：
运动系统很简单，它只关注持有变换组件和刚体组件的实体。
所以两个组件的代码如下，只包含必要的数据：
变换组件
```cpp
#ifndef TRANSFORMCOMPONENT_H
#define TRANSFORMCOMPONENT_H

#include <glm/glm.hpp>

struct TransformComponent{
	glm::vec2 position; 
	glm::vec2 scale;
	double rotation;

	TransformComponent(glm::vec2 position = glm::vec2(0,0), glm::vec2 scale = glm::vec2(1,1), double rotation = 0.0) {
		this->position = position;
		this->scale = scale;
		this->rotation = rotation;
	}
};


#endif

```

刚体组件
```cpp
#ifndef H_RIGIDBODYCOMPONENT
#define H_RIGIDBODYCOMPONENT

#include <glm/glm.hpp>

struct RigidBodyComponent {
	glm::vec2 velocity;

	RigidBodyComponent(glm::vec2 velocity = glm::vec2(0.0, 0.0)) {
		this->velocity = velocity;
	}
};

#endif 

```

运动系统
首先，在构造函数里调用RequireComponent把自己身上的签名的组件开关打开，这样注册表就能顺利比对实体组件签名对号入座。
其次系统获取对应的component数据，自己的update每帧会改变对应component里的数据，这样就完成了实体移动逻辑的目标
```cpp
#ifndef MOVEMENTSYSTEM_H
#define MOVEMENTSYSTEM_H

#include "../ECS/ECS.h"
#include "../Components/RigidBodyComponent.h"
#include "../Components/TransformComponent.h"

class MovementSystem :public System {

     public:
         MovementSystem() {
             
             RequireComponent<TransformComponent>();
             RequireComponent<RigidBodyComponent>();
         }

         void Update(double deltaTime) {
             //TODO:
             //循环系统关注的实体
            // for (auto entity: GetEntities()) {
               //根据实体的速度每帧更新实体的位置
            // }

             for (auto entity : GetSystemEntities()) {
                 auto& transform = entity.GetComponent<TransformComponent>();
                 const auto& rigidbody = entity.GetComponent<RigidBodyComponent>();
             
                 transform.position.x += rigidbody.velocity.x*deltaTime;
                 transform.position.y += rigidbody.velocity.y*deltaTime;

                 Logger::Log("Entity id = " + 
                     std::to_string(entity.GetId()) +
                     "position is now (" +
                     std::to_string(transform.position.x) + 
                     "," + 
                     std::to_string(transform.position.y) + ")");
             }
             
         }
};
#endif
```
#### 2.动画、碰撞等注意事项
AABB算法
改变源矩形


#### 3.ECS优化、打磨
模板会增加编译时间，在代码更难懂的情况下，有可能还会造成性能损失
删除实体，以及ID重用

### 六、资产管理
SDL渲染纹理的步骤很繁琐，同时为了以后加载动画地块等大量资产的方便，需要一个资产管理类写一些方便的函数加载资产。
这里面核心就是addtexture函数，写好它，game文件里只需加载一次这个路径，后面想要加载只需输入assetId即可。Gettexture后续也会在rendersystem中使用到
```cpp
#ifndef ASSETSTORE_H
#define ASSETSTORE_H

#include <map>
#include <string>
#include <SDL.h>

class AssetStore {
    private:
        std::map<std::string, SDL_Texture*> textures;
        // TODO: create a map for fonts
        // TODO: create a map for audio

    public:
        AssetStore();
        ~AssetStore();

        void ClearAssets();
        void AddTexture(SDL_Renderer* renderer, const std::string& assetId, const std::string& filePath);
        SDL_Texture* GetTexture(const std::string& assetId);
};
#endif
```

```cpp
#include "./AssetStore.h"
#include "../Logger/Logger.h"
#include <SDL_image.h>

AssetStore::AssetStore() {
    Logger::Log("AssetStore constructor called!");
}

AssetStore::~AssetStore() {
    ClearAssets();
    Logger::Log("AssetStore destructor called!");
}

void AssetStore::ClearAssets() {
    for (auto texture: textures) {
        SDL_DestroyTexture(texture.second);
    }
    textures.clear();
}

void AssetStore::AddTexture(SDL_Renderer* renderer, const std::string& assetId, const std::string& filePath) {
    SDL_Surface* surface = IMG_Load(filePath.c_str());
    SDL_Texture* texture = SDL_CreateTextureFromSurface(renderer, surface);
    SDL_FreeSurface(surface);

    // Add the texture to the map
    textures.emplace(assetId, texture);

    Logger::Log("New texture added to the Asset Store with id = " + assetId);
}

SDL_Texture* AssetStore::GetTexture(const std::string& assetId) {
    return textures[assetId];
}
```
### 七、事件
事件系统的实现方式多种多样，这里列举两个，最终我们会选择第二种进行实现。
而实现的过程当中有相当多对我们cpp新手来说的新语法，比较难理解，要多思考
###### 被动检查（passive check）
A系统发出一事件，可能塞到一个容器里，B系统自己循环检查容器里是否有A发出的事件
###### 阻塞（blocking）
B系统提前订阅A系统要发出的事件，A系统一旦发出事件，将停止接下来的行动（blocking）去回调订阅了自己的函数，立即回调B系统的函数
###### 文件结构
src文件夹下，Events文件夹用来写具体发出的事件，EventBus文件夹用来写事件管理器
###### 实现：碰撞事件
###### 实现：事件总线
学习实现订阅某个事件和发出某个事件的函数，这里又要用到模板了。
###### 实现：事件处理程序



### 八、打磨
简单摄像机实现
剔除在摄像机之外的渲染
标签与群组（关于性能与数据结构的思考）
错误检查与验证（关于目前已写的整个ECS）
面向数据设计与组件池打包提高缓存命中率
数组结构与结构数组
valgrind网站（内存与缓存分析）
### 九、Imgui窗口
一些API使用

如何显示一个imgui窗口？

如何实现窗口停靠？

如何将游戏场景实时显示在imgui窗口上？
已知问题：声明SDL_Texture* 后正常释放纹理依然导致栈损坏，一切由SDL_Texture引发的问题，关闭窗口会导致栈损坏。现在又知道不论什么，二次声明SDLwindow和SDLrenderer都会引发同样的问题。并且该情况只在Game类里出现。是的在Game类里声明不管是否释放都会出现栈损坏。而另开一个文件声明SDL_Texture*， 就能暂时解决问题。

初始化顺序与关闭顺序是反着的

如何让窗口上游戏视图看起来不会抖动？看起来像素完美？

传统保留模式下的窗口实现与ImGui使用的即时模式的窗口实现
一个是强事件驱动，一个是靠每帧渲染


### 十、Lua脚本对接
借用第三方库sol来进行lua与c++的对接，sol相当于lua的一个小包装器。
 
如何利用sol来调用lua脚本文件中的函数？又如何在lua中调用c++ 函数？

如何利用sol提前检查lua脚本语法错误？

如何动态加载关卡？
1.从lua表里读取素材2.从lua表里读取实体

如何用Lua写脚本逻辑？
1.暴露c++函数（getposition，setposition。。。）供lua脚本函数使用2.运用sol库为实体添加脚本组件与组件上的函数3.创建脚本系统处理lua脚本中的逻辑函数4.如何利用sol真正把lua函数和c++函数绑定在一起