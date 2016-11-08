## Introduction
This four part course will get you started with a gentle introduction to extending Orchard at the code level. You will build a very simple module which contains a widget that shows an imaginary featured product. 

It will teach you some of the basic components of module development and also encourage you to use best-practices when developing for Orchard.

In this first part we are going to set up our dev environment, scaffold a module and then build a simple `Widget` inside it. 

## Prerequisites
This course assumes the following:

  * You have some experience using Orchard and understand its core concepts. Refreshers and links to related guides will be provided.
  
  * You can read and write C# code.
  
  * You have *some* experience with ASP.NET MVC. This doesn't need to be deep but you should be aware of Razor templates, views, strongly-typed models and similar basics.
  
The course was written and tested against Orchard v1.9.2. It should work in new 1.x branch releases as they come out.

## Getting help
If you get stuck or need some support at any point in the course there are several places you can turn:

  1. Post a question in the [official support forums on CodePlex](http://orchard.codeplex.com/discussions).
  
  1. Post a question on [Stack Overflow tagged with OrchardCMS](http://stackoverflow.com/questions/tagged/orchardcms).
  
  1. Open an issue on the [Orchard Doc GitHub repo](https://github.com/OrchardCMS/OrchardDoc/issues).

## Setting up
First things first, you need to follow the [setting up for a lesson](Setting-up-for-a-lesson) guide.
首先，需要按照[指导](Setting-up-for-a-lesson)完成环境的搭建
       
## Getting the most out of this course
Writing an Orchard module that actually does something is going to contain a *minimum* of 9 different files. You will need to do a lot of development before you can run your module code and see it working in Orchard.


At first you might be overwhelmed by this, but here is a little tip; don't be. Just forge ahead with the tutorial and don't worry if terms like drivers, content parts, or placements seem unfamiliar at the moment. As you continue with your module development you will come across these files many times over. Before long you will start recognizing these core files and you will see how it all fits together.

## Course structure
Throughout the course we will alternate between discussing topics and implementing them. The discussion may contain example code or other example scenarios. 

So that there is no confusion for you as to what you should be doing, when it comes to implementing these lessons into the module it will be explained step-by-step via numbered lists.

Later on in the course, as the topics become more advanced, we may go through several sections of discussion before wrapping up the lessons into changes to the codebase.

You will also occasionally come across **Bonus Exercise** sections. These are completely optional. You can skip them, complete them at the time, or come back after completing the course to complete them. They are suggested when there is an extra feature you could implement using the skills you have just learned.

## Getting started
Now that you've completed all of the setup tasks you will have a fresh copy of Orchard configured and ready to go.

The rest of this part of the course will walk you through the process required to scaffold an empty module and then build a simple `Widget` inside of it.

## Command line scaffolding with Orchard.exe
You should now be looking at Visual Studio. Down the side, in your Solution Explorer window you will see many files and folders. 

The first step to take is to collapse all of the projects down. Its a long list and we need to be able to see an overview of the solution so we can start working with it. You don't need to collapse these individually by hand however:

  1. If your Solution Explorer window is not visible click `View`, `Solution Explorer`. 
  
  1. Click the `Collapse All` icon in the toolbar along the top of the solution explorer. It looks like this:
     
    ![](../Attachments/getting-started-with-modules-part-1/collapse-all.png)

If you expand your `Modules` folder you will see a long list of the modules which come packaged with Orchard:

![](../Attachments/getting-started-with-modules-part-1/modules-list.png)

There is a utility that is packaged with each copy of Orchard which will let us add our own module into this list. It is called `orchard.exe`. This is a command line utility which will scaffold up a new empty module and add it to the main solution. There are also other commands you can use with this utility. 

To scaffold a new module:

  1. Press the `Save All` button (or press `Ctrl-Shift-S`). Its a good practice to always save before using the command line utility. Many of its commands will make changes to your solution and if you have unsaved changes you will get merge conflicts.
  
  1. In the Solution Explorer, scroll down to the `Orchard.Web` project. It should be the very last project in the solution.
  
  1. Right click on the `Orchard.Web` project and choose `Open Folder in File Explorer`:
     
     ![](../Attachments/getting-started-with-modules-part-1/open-orchard-web.png)
  
  1. Open the `bin` folder
  
  1. Locate `orchard.exe` in the list and double click it to open.
  
     ![](../Attachments/getting-started-with-modules-part-1/open-orchard-exe.png)
     
     > **Note:** If you don't see `orchard.exe` in the `bin` folder then you didn't follow the steps in the [setting up for a lesson](Setting-up-for-a-lesson) guide. You need to have built the solution at least once for this file to exist. Press `Ctrl-Shift-B` within Visual Studio to build the solution.
  
  1. After a short pause while it loads you will then be presented with the Orchard command line:
  
     ![](../Attachments/getting-started-with-modules-part-1/orchard-command-line.png)
      
     > **Note:** There is a separate article where you can [learn more about orchard.exe and its features](Using-the-command-line-interface). You don't need to read it to understand this course but it will be useful to review in the future as part of your overall training.
  
  1. Type the following command: `feature enable Orchard.CodeGeneration` and press `enter`.
  
     ![](../Attachments/getting-started-with-modules-part-1/enable-codegeneration.png)
     
     This will activate the code generation features of `orchard.exe`. 
     
     > **Note:** If you get an error saying `No command found matching arguments "feature enable Orchard.CodeGeneration"` then you didn't follow the steps in the [setting up for a lesson](Setting-up-for-a-lesson) guide. You need to run the solution and go through the Orchard Setup screens before this command is available.
     
     The code generation command that we will be using is `codegen module`.
     
  1. Type `help codegen module` and press enter to see the syntax for this command. To see details about all of the commands available type `help commands`. 
  
    Like the rest of Orchard CMS, the orchard.exe command shell is extendable. The total number of commands available can vary depending on what features / modules you have loaded. In a future tutorial we will look at extending orchard.exe with our own commands.
    
  1. Scaffold the module by entering the following command: `codegen module Orchard.LearnOrchard.FeaturedProduct`. 
  
     If you read the help in the last step you might be wondering why we didn't include the `/IncludeInSolution:true` argument. This defaults to true so you don't need to add it.
     
  1. Close the Orchard command-line window.
  
  1. This has now created a new, empty module in the file system. Switching back to Visual Studio should show you the `File Modification Detected` dialog:
    
    ![](../Attachments/getting-started-with-modules-part-1/reload-solution.png)
    
    Click `Reload`.
    
    > **Note:** If you had unsaved changes in your Solution file then click the `Dismiss` option and add the project manually. In the Solution Explorer, `Right click` on the `Modules` folder. Choose `Add`, `Existing Project`, then navigate to `.\src\Orchard.Web\Modules\Orchard.LearnOrchard.FeaturedProduct\`, select `Orchard.LearnOrchard.FeaturedProduct.csproj` and press `Open`.

The basic framework for a module now exists inside the modules section of your solution:

![](../Attachments/getting-started-with-modules-part-1/scaffold-complete.png)

## Core concepts refresher
If you are at the stage of wanting to build modules for Orchard then you should already be familiar with the concept of Content Types, Widgets, Content Items and Content Parts. These are all things that you can manage via the admin dashboard and you will have worked with them if you have built any kind of site in Orchard. To refresh your memory:

  - **Content Type**: 一个content模板，最普通的一个例子是`Page`

  
  - **Widgets**: You can also make a content type that works as a `Widget`. The `Widget` is a special variation of content type which can be placed into one of the many `Zones` a template defines. It's manageable via the admin dashboard at run-time. Content types can opt-in to this system by configuring their `Stereotype` setting to `Widget`.  
`Widget` 是个特殊content类型，它可以放入 `Zones`,它可以通过管理页面在运行时设置。

  
  - **Content Item**: This is an instance of a specific content type. When you create a new `Page` in Orchard and fill it with content that is a `Content Item` with a `Content Type` of `Page`.

  这是一个特别content实例，创建`Page`时，`Content Item` 用于填充
  
  - **Content Part**: A small module providing some specific functionality. The `Content Type` is made up by attaching various `Content Parts` to it. For example you could have a comments content part. It just manages a block of comments for whatever it is attached to. The same comments content part could be attached to a `Page` content type, a `Blog` content type, or within a `Widget`.
一个小的模块提供一些特定功能。`Content Type`是由不同的`Content Parts`组成.例如评论，它只是提供一个评论功能，它同样可以放入`Page`、`Blog`或 `Widget`。


## What we will be building
As you might have guessed from the module name, we are going to build a very simple featured product module. This first step into extending Orchard will be a small one. 


这个模块是一个`Widget` ，它将显示一个特色商品静态信息列表。它没有任何可定制设置。也不需要查看数据库。`Widget` 是一个很好的开始点，因为它不需要担心菜单设置，标题，URLS或整合到管理页面

It will be a simple banner which you can display on your site by adding a widget via the admin dashboard. This will be enough to show the core concepts of a module. We will come back and make improvements in the next three parts of this course.

Let's get started with some development by adding classes and other files to our module.

## Content part
The content part class is the core data structure. When you scaffolded the module it automatically made you a `Models` folder. Now we need to add the `FeaturedProductPart` class to this folder:

![](../Attachments/getting-started-with-modules-part-1/add-contentpart-class.png)

  1. `Right click` on the `Models` folder.
  
  1. Choose `Add`
  
  1. Choose `Class...`

  1. In the `Name:` field type `FeaturedProductPart`
  
  1. Click `Add`
  
Your new class will be created and opened up in the Visual Studio editor.

> **Important note:** 为了Orchard组织内容部分类，这些类的命名空间必须以`.Models`结尾

content part类需要继承 `ContentPart` class.

通常我们会添加公共属性用于存储所有相关的数据，但此示例没有。

![](../Attachments/getting-started-with-modules-part-1/contentpart-add-inheritance.png)


## Data migrations

当模块在管理页面启用后，Orchard会执行一个数据迁移。数据迁移的目的注册模块中使用数据。


我们暂时不使用这个功能，但是这个迁移同样用于升级。数据迁移类可修改，也可以转换已有的数据来满足新的要求。


The data migration class can be created by hand, following a similar process as the last section but we can also scaffold it with the `orchard.exe` command line. Let's dive back in to the command line and add a data migration class to the module.

数据迁移类可手动创建，可以通过之前的`orchard.exe` 命令行:

  1. Press the `Save All` button (or press `Ctrl-Shift-S`). Its a good practice to always save before using the command line utility. Many of its commands will make changes to your solution and if you have unsaved changes you will get merge conflicts.
  
  1. In the Solution Explorer, scroll down to the `Orchard.Web` project. It should be the very last project in the solution.
  
  1. Right click on the `Orchard.Web` project and choose `Open Folder in File Explorer`:
     
     ![](../Attachments/getting-started-with-modules-part-1/open-orchard-web.png)
  
  1. Open the `bin` folder.
  
  1. Locate `orchard.exe` in the list and double click it to open.
  
     ![](../Attachments/getting-started-with-modules-part-1/open-orchard-exe.png)
  
  1. After a short pause while it loads you will then be presented with the Orchard command line:
  
     ![](../Attachments/getting-started-with-modules-part-1/orchard-command-line.png)
  
  1. We enabled the code generation feature when scaffolding the module but if you have been playing with Orchard or are just using this guide as a reference it can't hurt to run the command a second time to make sure. 
  
     Type the following command: `feature enable Orchard.CodeGeneration` and press `enter`.
  
     ![](../Attachments/getting-started-with-modules-part-1/enable-codegeneration.png)
     
     This will activate the code generation features of `orchard.exe`. The command that we will be using is `codegen datamigration`. 
     
  1. Type `help codegen datamigration` and press enter to see the syntax for this command. To see details about all of the commands available type `help commands`. 
  
    Like the rest of Orchard CMS, the orchard.exe command shell is extendable. The total number of commands available can vary depending on what features / modules you have loaded. In a future tutorial we will look at extending orchard.exe with our own commands.
    
  1. Scaffold the data migration class by entering the following command: `codegen datamigration Orchard.LearnOrchard.FeaturedProduct`.
  
  1. Close the Orchard command-line window.
  
  1. This has now created a new data migration the file system called `Migrations.cs`. It will be in the root folder of your module. 
  
     Switching back to Visual Studio should show you the `File Modification Detected` dialog:
    
    ![](../Attachments/getting-started-with-modules-part-1/reload-solution.png)
    
    Click `Reload`.
    
    > **Note:** If you had unsaved changes in your Solution file then click the `Dismiss` option and add the class manually. In the Solution Explorer, `right click` on the `Orchard.LearnOrchard.FeaturedProduct` folder. Choose `Add`, `Existing Item`, then navigate to `.\src\Orchard.Web\Modules\Orchard.LearnOrchard.FeaturedProduct\`, select `Migrations.cs` and press `Add`.


模块根目录文件夹下已经有了`Migrations.cs`文件，默认包含一个空的只返回整型的`Create()`方法. 

之前讨论的 `Widget` 是一个`ContentType` with a `Stereotype` of `Widget`.A `ContentType` 只是 `ContentPart`集合。每个`ContentType` 应该包含`CommonPart`，它将提供一些基本例如所有者和创建日期字段。我们需要添加`WidgetPart`。最后我们添加 `FeaturedProductPart`.


现在我们更新`Create()`方法来实现下列计划:

  1.打开`Migrations.cs`
  
  1. 用以下内容替换 `Create()` :
  
        public int Create() {
          ContentDefinitionManager.AlterTypeDefinition(
            "FeaturedProductWidget", cfg => cfg
              .WithSetting("Stereotype", "Widget")
              .WithPart(typeof(FeaturedProductPart).Name)
              .WithPart(typeof(CommonPart).Name)
              .WithPart(typeof(WidgetPart).Name));
          return 1;
        }
        
     Orchard doesn't have a `CreateTypeDefinition` method so even within the create we still used `AlterTypeDefinition`. If it doesn't find an existing definition then it will create a new content type.
   
  1.添加相关引用
   
  1. 添加`Orchard.Widgets`项目引用
  
That's all for the data migration, your `Migrations.cs` should now look like this:

    using Orchard.ContentManagement.MetaData;
    using Orchard.Core.Common.Models;
    using Orchard.Data.Migration;
    using Orchard.LearnOrchard.FeaturedProduct.Models;
    using Orchard.Widgets.Models;
    
    namespace Orchard.LearnOrchard.FeaturedProduct {
        public class Migrations : DataMigrationImpl {
    
            public int Create() {
                ContentDefinitionManager.AlterTypeDefinition(
                  "FeaturedProductWidget", cfg => cfg
                    .WithSetting("Stereotype", "Widget")
                    .WithPart(typeof(FeaturedProductPart).Name)
                    .WithPart(typeof(CommonPart).Name)
                    .WithPart(typeof(WidgetPart).Name));
                return 1;
            }
        }
    }

## Update dependencies as you go along


在 `Create()`里我们在 `WidgetPart`引入了一个依赖。

这表示我们的模块不能离开`Orchard.Widgets`而运行。


为了让Orchard知道我们引入了这个依赖，我们需要在清单文件(`Module.txt`)中记录它。
这是个YAML格式，它存储了模块的元数据，包括名称、作者、描述、依赖。


>注意:在模块添加引用后必须马上在清单文件中记录，如果不记录这个信息，将会在运行时引起异常

让我们更新这个清单文件以记录`Orchard.Widgets` 依赖：

  1. 打开 `Module.txt`
  
  1. 最后的3行 (因为这只有一个模块): 
  
         Features:
             Orchard.LearnOrchard.FeaturedProduct:
                 Description: Description for feature Orchard.LearnOrchard.FeaturedProduct.
    

     在`Description:`后面添加一行，然后添加如下:
     
         Features:
             Orchard.LearnOrchard.FeaturedProduct:
                 Description: Description for feature Orchard.LearnOrchard.FeaturedProduct.
                 Dependencies: Orchard.Widgets
                 
      
## How is all this magic working?
So far the `ContentPart` class has been magically detected as long as it uses the `.Model` namespace, now the data migration is automatically detected just for deriving from `DataMigrationImpl`. How is all of this happening?


Orchard使用[Autofac](http://autofac.org/)，一个控制反转容器。如果有兴趣了解可查看Orchard整合[how Orchard works](How-Orchard-works).


## Content part driver
Everything you see in Orchard 由 `Shapes`构成. 可查看 [accessing and rendering shapes](Accessing-and-rendering-shapes) guide. 


一个content部件驱动是一个类，它构成了shapes，它可以用于查看编辑content部件。一个基本驱动类包含3个方法，一个用于前端视图展现显示驱动，一个用于在管理页编辑表单编辑驱动和一个用于处理表单提交方法。



在驱动里创建shapes，可以向视图传递数据。视图将在下个章节讨论。


我们正在创建的部件没有可配置项，所以这个驱动只需要`Display` 方法。其他方法将在第二部分再查看。

There aren't any command line scaffolding commands for setting up new drivers so you will need to create it manually:

我们没有命令行来创建一个新的驱动，所以我们需要手动:

  1. 创建 `Drivers` 文件夹
  
  1. 添加新类 `FeaturedProductDriver` 
     
  1. 扩展这个类使其继承`ContentPartDriver<FeaturedProductPart>`.
  

未来我们会在这个驱动做其他很多工作. 当前只需要简单来衔接shape到视图。修改这个类:

   1. Inside your `FeaturedProductDriver` class add this single method:
   
        protected override DriverResult Display(FeaturedProductPart part, 
          string displayType, dynamic shapeHelper) {
          return ContentShape("Parts_FeaturedProduct", () => 
            shapeHelper.Parts_FeaturedProduct());
        }

方法表示当展示`FeaturedProductPart`时返回一个`Parts_FeaturedProduct`shape。
默认Orchard将在`Views\Parts\FeaturedProduct.cshtml` 检查这个shape。

## View
Orchard使用Razor模板来显示它的shapes. 我们可以提供强类型数据模型。

创建视图

  1. 在`Views` 文件夹添加 `Parts`文件夹.
  
  1. 在`Parts`文件夹添加 `FeaturedProduct.cshtml`.
  
  1. 修改这个文件:
  
        <style>
          .btn-green {
            padding: 1em;
            text-align: center;
            color: #fff;
            background-color: #34A853;
            font-size: 2em;
            display: block;
          }
        </style>
        <p>Today's featured product is the Sprocket 9000.</p>
        <p><a href="~/sprocket-9000" class="btn-green">Click here to view it.</a></p>

## Placement

最后一步，驱动类增加设置以告诉Orchard如何呈现content part. Placemeng用于告诉Orchard去哪呈现这些组件。

`placement.info` 文件在模块根目录下。它是个xml文件，可参考[understanding placement.info](Understanding-placement-info)

添加`placement.info`文件，然后添加下面的内容: 

        <Placement>
          <Place Parts_FeaturedProduct="Content:1"/>
        </Placement>
  

`Content:1`是shape的区域和优先级. 一个shape一般有多个区域。通常包括头部，内容，meta和根部，这些区域可以组合。在这个例子中 `Content` 是主内容区域. 


优先级表示部件会展示在顶部区域. 在一些复杂的模块会有多个shapes。设置不同的优先级可以按顺序组织同一区域的shapes。


主题开发者可以在placement.info定义布局。这样主题作者就可以自定义主题而不用修改代码。这表示模块升级到新版本主题开发者不会更新。

## Trying the module out in Orchard
最后的几个步骤是启用这个模块然后将部件放置到激活的模板。

  1. 编译解决方案

  1. 登录管理页，进入`Modules` 
  
  1. 现在可以将部件添加到网站，进入`Widgets`页面。
   
  1. 在 `AsideFirst` 点击 `Add` button:
   
     ![](../Attachments/getting-started-with-modules-part-1/activate-addtolayer.png)
     
  1.  `Featured Product Widget`会在列表中 :
  
     ![](../Attachments/getting-started-with-modules-part-1/activate-featuredproductwidget.png)

  1. 可以保留默认设置，可以设置 `Title` to `Featured Product`:
  
     ![](../Attachments/getting-started-with-modules-part-1/activate-widgettitle.png)

  1. 点击 `Save` .
  
回到主页，可以看到这个模块在站点里

![](../Attachments/getting-started-with-modules-part-1/enable-viewmodule.png)


我们创建页面，所以点击后会提示404. 


## Download the code for this lesson
可以在这下载代码:

  * [Download Orchard.LearnOrchard.FeaturedProduct-Part1-v1.0.zip](../Attachments/getting-started-with-modules-part-1/Orchard.LearnOrchard.FeaturedProduct-Part1-v1.0.zip)
  

## Conclusion
This first guide in the module introduction course has shown the main components of a module.

In the next part we will extend the module to add some interactivity to the module. This means adding database backing, an editor view, configuration settings and we will dip our toes in with some of the Orchard API features.

In the final part of the course we will review the module and clean it up to ensure we follow development best practices that have been missed so far.

This was a long guide. Take a break now and when you're refreshed come back and [read part two of the course](Getting-Started-with-Modules-Part-2).