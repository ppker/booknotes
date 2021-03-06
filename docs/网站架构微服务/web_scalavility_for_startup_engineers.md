
> Everything Should Be Made as Simple as Possible, But Not Simpler. - Albert Einstein

我觉得编写软件的过程就是和和复杂度斗争的过程，无论是编写传统的pc软件还是web应用。

## 简单
软件天然错综复杂，当你衡量一个东西是否过于简单的时候，首先回答是对谁而言以及对什么时候而言。对于工程师而言，简单是让别的工程师以一种最容易的方式使用你的方案。

- 隐藏复杂与构建抽象:软件变庞大之后很难一窥全貌，无法保持简单，但是可以保持局部简单。达到局部简单的主要方式实在设计和实现两个方面上，任何单个的类、模块、应用的设计目标和工作原理能被快速理解，而不需要深入到里头的细节。添加抽象层，隐藏复杂性。
- 避免过度设计：好的设计是能在后期不断添加新的功能特性，而不是一开始就开发一个大系统。每次进行软件设计的时候问题自己，这个设计是否可以更简单而且在将来依然可以保持弹性？斟酌权衡
- 尝试测试驱动开发：在某些方法上搞TDD可以获得一个全新的视角。TDD能然你避免写冗余的无用代码，同事单元测试用例可以被当做文档，暂时代码的意图、用法、期望结果等。同时TDD能然你以使用者的角度看待接口，从而简化API。无论是否使用TDD，你都要以用户（使用API）的人的视角思考，提高接口易用性。
- 从软件设计的简化范例中学习：Grails，Hadoop，Google Maps API。

## 低耦合
低耦合对系统的健康和伸缩至关重要。
- 促进低耦合：小心管理依赖，类之间、模块之间、应用之间的以依赖。类应该只有一个目的，是最小的抽象粒度，尽量少暴露类的内部信息。
- 避免不必要的耦合：尽可能多隐藏，少暴露。画系统设计图可以暴露循环依赖，一个良好的模块架构图应该看起来像一棵树（有向无环图），而不是社交网络图。
- 低耦合范式：通过阅读别人的代码吸取经验。比如unix的命令行及管道的设计。

## DRY
软件开发工程中低效的地方有很多，代码重复、缺乏自动化的重复（部署、测试、配置等）、大量复制粘贴代码、不必要的重复发明轮子等，“代码不会用第二次就凑合一下”的态度
- 复制粘贴代码：使用抽象，组合继承等消除重复，使用设计模式、共享类库等消除重复。

## 基于约定编程
基于约定编程或者说基于接口编程，是将客户端代码和功能提供者代码解耦的重要方式。让客户端依赖于约定编程。 约定在不同层次有不同的意义，比如低层次就是方法和函数的签名，高层次可以是web服务的文档。HTTP协议就是个注明的例子。


## 画架构图
有些时候需要做前期设计，可以帮助自己和他人更好了解设计。
- 用例图：定义系统用户是谁，可以做哪些操作。
- 类图：接口、类、关键方法名和它们之间的关系，展现类、接口及其交互关系。
- 模块图：模块之间的依赖和交互，可以是一个包或者组件。

## 单一职责
类应该只有一个职责。
－改善单一职责：类的代码不过长，确保类的以来不超过５个接口／类，确保一个类有一个明确的目标，能用一句话总结类的职责作为注释。

## 开闭原则
需求变更或者增加新功能时，你不需要修改现有代码。想扩展开放，对修改关关闭。MVC就是个好例子。重构代码分解任务并能复用。

## 依赖注入
一种降低耦合并且改善开闭特性的技术。依赖注入对类需要创建的对象提供一个引用实例，而无需类自己创建依赖对象。一般是把类需要的对象当成一个参数传递进去。这样类就能聚焦自己的主要职责，无需知道谁来提供依赖的实例。比如我们要实现一个播放器类，播放器必须有CD才能播放，我们可以在构造函数中把CD的实例作为构造函数的参数传递进去。

## 控制反转(IOC)
IOC是一种从类中移除职责的方法，从而使类更简单，与系统其他部分更少耦合。就是你的类实例不必知道自己何时被创建、被谁使用、自己的依赖又是如何被创建的。你的类是一个插件，外部决定这些类什么时候被创建，如何被使用。

## 为伸缩性设计

伸缩性方案可以被浓缩成三个基本设计方法：

- 增加副本：同一组件重复部署到多态服务器，最适用于无状态的服务，有状态的服务难以用这种方式伸缩。该方案需要找个方法同步状态。
- 功能分割: 基于功能将系统分隔成独立的子系统，常用于底层。
- 数据分片: 在每台服务器上只部署一部分数据，无共享架构，每个节点完全自治，几乎有无限的伸缩性。但是也是代价最高、最昂贵的技术，难点在于要访问数据需要找到数据所在的分区服务器，如果一个查询在涉及分区，实现会变得低效和困难。


## 自愈设计
设计系统要考虑高可用和系统自愈能力，设计一个伸缩性架构必须把各种失效状态视为常态，而不是特殊对待，要不停地想哪里会出错，出错后怎么办。系统能在最短时间内恢复，并且能自动化完成，就是自愈。
