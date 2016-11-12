Building a Web CMS (Content Management System) is unlike building a regular web application: it is more like building an application container. When designing such a system, it is necessary to build extensibility as a first-class feature. This can be a challenge as the very open type of architecture that's necessary to allow for great extensibility may compromise the usability of the application: everything in the system needs to be composable with unknown future modules, including at the user interface level. Orchestrating all those little parts that don't know about each other into a coherent whole is what Orchard is all about.

This document explains the architectural choices we made in Orchard and how they are solving that particular problem of getting both flexibility and a good user experience.


# Architecture

<table cellspacing="2" cellpadding="2" border="1" style="width:100%">
<tr>
<td colspan="4" align="center">Modules
</tr><tr>
<td colspan="4" align="center">Core
</tr><tr>
<td colspan="4" align="center">Orchard Framework
</tr><tr>
<td align="center" style="width:25%">ASP.NET MVC
<td align="center" style="width:25%">NHibernate
<td align="center" style="width:25%">Autofac
<td align="center" style="width:25%">Castle
</tr><tr>
<td colspan="2" align="center">.NET
<td colspan="2" align="center">ASP.NET
</tr><tr>
<td colspan="4" align="center">IIS or Windows Azure
</tr>
</table>

# Orchard Foundations

The Orchard CMS is built on existing frameworks and libraries. Here are a few of the most fundamental ones:

- [ASP.NET MVC](http://www.asp.net/mvc): ASP.NET MVC is a modern Web development framework that encourages separation of concerns.
- [NHibernate](http://nhforge.org/): NHibernate is an object-relational mapping tool. It handles the persistence of the Orchard content items to the database and considerably simplifies the data model by removing altogether the concern of persistence from module development. You can see examples of that by looking at the source code of any core content type, for example Pages.
- [Autofac](http://code.google.com/p/autofac/): Autofac is an [IoC container](http://en.wikipedia.org/wiki/Inversion_of_control). Orchard makes heavy use of dependency injection. Creating an injectable Orchard dependency is as simple as writing a class that implements IDependency or a more specialized interface that itself derives from IDependency (a marker interface), and consuming the dependency is as simple as taking a constructor parameter of the right type. The scope and lifetime of the injected dependency will be managed by the Orchard framework. You can see examples of that by looking at the source code for IAuthorizationService, RolesBasedAuthorizationService and XmlRpcHandler.
- [Castle Dynamic Proxy](http://www.castleproject.org/projects/dynamicproxy/): 生成动态代理.

The Orchard application and framework are built on top of these foundational frameworks as additional layers of abstraction. They are in many ways implementation details and no knowledge of NHibernate, Castle, or Autofac should be required to work with Orchard.

# Orchard Framework


Orchard framework是Orchard最底层。它包含应用程序引擎和一些不能隔离到模块的部分. 这些通常基础模块都必须依赖. 可以把它当成Orchard的基础类库.

## Booting Up Orchard(启动Orchard)

Orchard应用程序启动时, 将创建一个Orchard Host, 它是一个在应用程序域级的单例. 

然后, 这个Host将使用ShellContextFactory为当前用户创建Shell. 多用户模式是应用程序的多个实例,实例运行在同一应用程序域 , 实例之间的用户是隔离的. Shell在用户级是个单例且实际上表现给用户. 它是将有效地提供同时保持模块的编程模型的多租户的隔离的对象.

一旦Shell被创建, 将从ExtensionManager中获取可用扩展列表. 扩展包括模块和主题. 默认实现将检查模块和主题文件夹. 

同时, Shell从ShellSettingsManager中获取用户设置列表. 默认从`App_Data`子文件夹中获取这些设置, 但可实现从其他文件夹获取. 

然后Shell获取CompositionStrategy对象,并从当前Host可用扩展列表和当前用户设置中用它准备IoC容器. 返回的结果并不是一个IoC容器, 是ShellBlueprint, 它是依赖、控制器和记录蓝图. 

然后将每个用户的ShellSettings列表和ShellBlueprint抛给ShellContainerFactory.CreateContainer, 并获取一个ILifetimeScope, 它使IoC容器作用于用户级, 这样模块就可以为当前用户注入依赖而不需要其他任何指定. 

## Dependency Injection(依赖注入)

在Orchard中标准的注入依赖方式是创建一个继承自IDependency接口或其他已继承的接口, 并实现这个接口. 使用时, 可在构造函数中传入接口参数. 应用程序框架将找到所有依赖并观察实例和注入实例. 

有三种不用的依赖作用域, 选择合适的接口继承:

- Request： 每个新的Http请求创建一个依赖实例, 请求处理后将销毁. 通过继承IDependency来使用. 这个对象的创建应该合理简单.
- Object: 接口每次对象依赖都创建一个对象, 对象从不分享. 通过继承ITransientDependency来使用, 这些对象的创建必须非常便宜. 
- Shell: 每个Shell/用户创建一个实例, 通过继承ISingletonDependency来使用. 只有在shell中保持一个状态才使用. 

### Replacing Existing Dependencies(替换已存在的依赖)

可通过在给类增加OrchardSuppressDependency属性(Attribute)来标识替换已存在的依赖, 需要将替换类型全称作为参数.

### Ordering Dependencies(依赖顺序)

一些依赖并不唯一而是列表的部分. 例如, handlers将同时激活. 在一些例子中需要修改依赖顺序, 可通过修改模块的清单文件，使用Priority属性, 例如:

    
    Features:
        Orchard.Widgets.PageLayerHinting:
            Name: Page Layer Hinting
            Description: ...
            Dependencies: Orchard.Widgets
            Category: Widget
            Priority: -1



## ASP.NET MVC

Orchard是构建于ASP.NET MVC, 但为了增加主题和用户隔离等, 需要引入一个额外的间接层,

例如, 一个特定的视图被请求后, LayoutAwareViewEngine将介入. 严格来说, 这并不是一个新的视图引擎，它并不关心实际的呈现, 但它包含根据当前主题来查找正确的视图并委托视图呈现工作给实际的视图引擎.

类似, 我们有路由提供程序、模型绑定和控制器工厂作为ASP.NET MVC的单一入口点, 在下面来发送请求适合的作用域对象

例如路由, 我们有n个路由提供程序(一般来自模块), 一个路由发布程序将与ASP.NET MVC交互, 同样的事情也发生在模型绑定和控制器工厂. 

## Content Type System(内容类型系统)

Orchard的内容管理实际的类型系统, 在某些方式比.NET类型系统更加丰富和动态. 为提供灵活性: 类型必须在运行时构建, 呈现内容管理相关. 

### Types, Parts, and Fields(类型, 元件, Fields)

Orchard能处理任意内容类型, 包括站点管理员创建的动态. 这些内容类型是每个特定内容元件的聚合. 原因是许多内容跨多个内容类型.


例如, 一个博客文章, 一个产品, 一个视频剪辑可能都有一个路由地址, 评论和标记. 出于这个原因, 路由地址, 评论和标记在Orchard被当作单独的内容部件. 这种方式, 评论管理模块可只开发一个, 应用到任意内容类型, 包括评论模块作者都不知道.

元件本身包含属性和内容fields. 内容fields同样可以像元件那样复用: 一个特定field类型可用于多种元件和内容类型. 元件和fields的不同在于它们的操作规模和语义.

Fields比元件更优雅一点. 例如, 一个field类型可描述电话号码, 一个坐标, 尽管一个元件通常描述整个评论或标记. 

但重要的不同是语义: 你想写一个元件它实现的是一个 "is a" 关系, field实现的是 "has a" 关系. 

例如, 一件衬衫 **is a** 产品, 它 **has a** SKU和价格. 不能说衬衫有一个产品或者衬衫是一个价格或SKU. 

这样你就知道这件衬衫内容类型是有产品元件组成, 这个产品元件由一个名为"price"的Money字段和一个名为"SKU"的字符串字段.

另一个不同是每个内容类型都只有一个给定的类型, 这样就讲得通它是"is a" 关系, 尽管一个元件可含有任意数量的给定类型字段. 另一种说法,元件的字段是一个由字段类型字符串值字典, 尽管内容类型是一个元件类型列表.

还有一种如何选择元件和字段的方式: 如果你想每个内容类型多过一个对象实例, 那就需要的是字段.


### Anatomy of a Content Type(一个内容类型剖析)

内容类型由内容元件构成. 内容元件从代码角度来说通常与之联系的包括:

- 记录, 元件数据POCO对象
- 模型类, 实际元件, 继承自`ContentPart<T>`, 其中 T 是上述记录
- 仓储. 仓储不需要模块作者实现, Orchard使用了一个泛型仓储.
- handlers. 实现自IContentHandler, 一组事件处理如OnCreated,OnSaved. 总体来说, handlers介入内容生命周期以完成一些任务. 也可以在内容构造器中参与实际内容. 在基类ContentHandler中包含一个过滤器集合, 允许handler为内容类型添加一般行为. 例如, Orchard有一个StorageFilter, 它使内容元件声明持久化变的非常简单: 只需要t添加代码`Filters.Add(StorageFilter.For(myPartRepository));` ,Orchard将会处理数据的持久化. 另一个过滤器例子, ActivatingFilter负责元件组合成一个内容类型: 请求`Filters.Add(new ActivatingFilter<BodyAspect>(BlogPostDriver.ContentType.Name));` 增加内容元件主体

- 驱动. 驱动比较友好, 更加专注的handlers(因此减少了灵活性), 驱动与特定的内容元件关联. 继承自`ContentPartDriver<T>`(T为内容元件类型). Handlers并不需要指定内容元件类型. 驱动可以看成特定元件的控制器, 他们通常构建shapes来通过主题引擎呈现. 

## Content Manager

所有的内容都可以通过ContentManager对象访问到. ContentManager有方法查询内容, 查看内容和管理发布状态. 

## Transactions(事务)

Orchard自动为每个HTTP请求创建一个事务. 这表示所有请求期间的操作都在事务中. 如果请求中代码取消事务, 所有的数据操作都将回滚. 如果事务没有被其他方式显式取消, 所有操作都将最请求最后提交.


## Request Lifecycle(请求生命周期)

这部分, 我们举个例子: 请求一个博客文章. 
In this section, we'll take the example of a request for a specific blog post.

当一个请求到达后, 应用第一次检查可用的各种模块的路由并找到博客模块路由, 而后路由可解析这次博客请求的控制器方法. 然后这个方法从内容管理中(通过请求BuildDisplay)获取页对象模型(POM)

一个博客文章有自己控制器, 但并不是所有内容类型都是. 例如, 动态内容类型将由更加通用的ItemController负责. ItemController的Display方法与博客控制器做的事情几乎一样: 从内容管理获取内容并根据结果构建POM.

布局视图引擎将根据当前主题找出正确的视图然后将模型与Orchard组合一起.

在视图里, 更多动态shape可创建, 如区域定义.

实际呈现视图由主题引擎来完成, 找到正确的模板或外形方法并按外观顺序递归渲染每个外形.

## Widgets(小部件)

小部件是含有Widget内容和Widget样子的内容类型. 像其他内容类型一样, 也是有部件和字段组成. 这表示小部件可和其他内容类型使用同样版本和渲染逻辑. 他们同样分享构建块, 任何已存在的部件都可以作为小部件的一部分.

小部件通过widget层添加到页中. 层是一些小部件集, 他们有名称, 规则来决定该在哪个页上显示, 小部件与区域的顺序和设置相关.

每个层的规则是由IronRuby表达式来表达. 这些表达式可使用任何已IRuleProvider实现. Orchard已经有两个此类实现: url和验证.

## Site Settings

Orchard站点是一个内容项, 它使模块与部件紧密结合成为可能. 这是模块如何共享站点设置. 站点设置是按用户隔离. 

## Event Bus(事件总线)

Orchard与之模块通过为依赖创建实现接口来公开了延伸点, 这些接口实现可被注入. 

接入延伸点可通过实现接口, 或实现一个具有相同名称和方法的接口. 换句话说, Orchard并不严格要求强类型接口对应, 它使接入扩展延伸点并不需要引入. 

这只是一个Orchard事件总线实现. 当一个扩展点请求注入实现, 一个消息将在事件总线上获取发布. 一个对象监听着这个事件总线,发出消息到类中的继承方法.

## Commands(命令)

Orchard里多个操作可以通过命令行来完成与管理页一样的. 通过实现ICommandHandler并标记命令名称属性(attribute)开放命令. 

Orchard命令行工具在运行时通过模拟Web站点环境应用反射检查程序集来发现可用命令. 

## Search and Indexing(搜索和索引)

搜索和索引默认使用Lucene实现, 不过可以使用其他索引引擎替换默认实现. 

## Caching(缓存)

Orchard缓存依赖ASP.NET缓存, 但Orchard通过ICache开放了一个API, 调用这个方法. 使用Orchard的这个API的主要优势在它是用户隔离的.  

## File Systems(文件系统)

Orchard里的文件系统是抽象的, 存储可被映射到物理文件系统或Azure二进制存储. 

## Users and Roles(用户和角色)

Users也是内容项(虽然不能路由到), 可在profile模块中扩展字段. 
角色是内容部件与用户紧密结合. 

## Permissions(权限)

每个模块可暴露一系列权限, 同时这些权限如何默认授权到Orchard默认角色中. 

## Tasks(任务)

模块可通过调用IScheduledTaskManager的实现类的CreateTask方法安排任务. 任务通过实现IScheduledTaskHandler来执行. 这个方法检查任务类型名称并决定是否处理. 
任务可运行在ASP.NET线程池里的独立线程. 

## Notifications(通知)

模块可通过INotifier实现类的方法来显示消息到管理视图. 多个通知做作为任何请求的一部分. 

## Localization(本地化)
本地化通过调用`@T("This string can be localized")`来请求. 查看[Using the localization helpers](Using-the-localization-helpers)了解更多. Orchard资源管理可从本地PO文件中跟据区域获取资源字符串。

内容项本地化通过不同的机制: 内容项本地化不同版本根据内容项单独存储在物理路径. 

文化管理是根据当前文化区域字符串. 默认实现返回站点设置项, 但可实现从用户资料或浏览器设置中获取. 

## Logging(日志)

日志通过ILogger的实现来完成. 不同的实现可发送日志实体到不同的存储类型. Orchard使用[Castle.Core.Logging](https://github.com/castleproject/Windsor/blob/master/docs/logging-facility.md)日志.

# Orchard Core

Orchard.Core程序集包含一系列Orchard必须运行的模块. 其他模块可安全的引用这些模块. 

例如,Core模块有feeds, navigation, routable.

# Modules(模块)

Orchard默认内置了一些模块(博客, 页面), 但第三方模块也可生成. 

模块是ASP.NET MVC Area与mainifes.txt文件的Orchard模块

模块通常包含事件处理, 内容类型和其他模块呈现模块.

模块动态每次修改后从源代码编译. 

模块必须放置在Modules文件夹(Orchard/web/Modules/newModule),且文件夹名称必须与编译后dll名相同. 

# Themes(主题)

Orchard基本设计原则是所有生成的HTML都可以用主题替换, 包括由模块生成的标签. 约定文件必须在主题文件夹有层次.

Orchard渲染机制基础是shapes. 主题引擎的工作是找到当前主题, 并给主题决定最好的渲染每个shape的方式. 每个shape有一个默认渲染定义于模块视图文件夹的模板中或shape代码方法中. 默认渲染可能会被当前主题重写. 主题有自己的模板版本或有shape自己的方法.

主题有上层, 允许子主题特别或适应父主题. Orchard自带一个基本主题叫主题机器, 可使使用父主题设计更加简单.

主题可包含代码与模块包含的方式一样: 它们可有自己的csproj文件,有利于动态编译. 这将允行主题定义shape方法, 但同时暴露管理视图的任何设置.

当前主题的选择是由IThemeSelector实现类来完成, 它为任何请求返回一个主题名称和优先级. 这允许多选择来贡献给主题选择. Orchard自带了4种ITemeSelector实现: 

- SiteThemeSelector 选择这个主题配置为当前用户或优先级低站点.
- AdminThemeSelector 接管并返回高优先级admin主题不过当前URL是否是Admin URL.
- PreviewThemeSelector 重写站点当前预览主题
- SafeModeThemeSelector 在 "safe mode" 下的唯一选择,这通常是在安装期间. 它的优先级非常低.
