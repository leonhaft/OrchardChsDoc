## Introduction
This is part two of a four part course. It will get you started with a gentle introduction to extending Orchard at the code level. You will build a very simple module which contains a widget that shows an imaginary featured product. 

It will teach you some of the basic components of module development and also encourage you to use best-practices when developing for Orchard.

If you haven't read the previous part of this course then you can go back to the overview to [learn about the Getting Started with Modules course](Getting-Started-with-Modules).

## Expanding the widget to make it more dynamic

This meant we missed out a few classes that would be in a normal data-driven module and some of the classes we did add were pretty much empty.

Now that you understand the basic workflow of building a module we're going to go back and take a second look at it, adding in some new classes and expanding out the Widget.

By the time we have finished with this part we will have made the widget more dynamic. To achieve this goal we are going to:

  * Add a `Boolean` property to signify if the item is on sale.
  
  * Dip into the Orchard API to hide the widget if the page we are viewing is the featured product.
  
## Getting setup for the lesson
You should have already completed part one of this course before you move on to this part. This means you should have a copy of Orchard with the completed module work. 

If you have the original Solution and module files available then:

  1. Open Visual Studio
  
  1. Open the Solution you created in the first part of the course
  
If for some reason you don't have these files you can play catch-up by following these steps:

  1. Follow the [setting up for a lesson](Setting-up-for-a-lesson) guide.

  1. Download the [completed module source code from part 1](../Attachments/getting-started-with-modules-part-1/Orchard.LearnOrchard.FeaturedProduct-Part1-v1.0.zip).
  
  1.  Extract the archive into the modules directory at `.\src\Orchard.Web\Modules\`.
  
  1.  Run Orchard by pressing `Ctrl-F5`, go to the admin dashboard, select Modules from the navigation menu and enable the module.

Now we can begin the lesson by starting to build in database functionality to the Featured Product module.

## Add a ContentPartRecord class
In the first part of this course we created a simple `ContentPart` class called `FeaturedProductPart`. Because we weren't storing anything at the time this class was just an empty placeholder. 

We are going to go back and give it a property to store some data in the next section but first we need to create a `ContentPartRecord` class.

The `ContentPartRecord` class is used by Orchard to store content part data in a database.

Let's add the class to the module and then start wiring in the `Boolean` `IsOnSale` property:

  1. Locate your FeaturedProduct module in the `Solution Explorer`, `Right click` on the `Models` folder and choose `Add`, `Class...`
  
  1. In the `Add New Item` dialog enter `FeaturedProductPartRecord` in to the `Name:` field and press `Add`.
  
  1. Derive the class from `ContentPartRecord`:
  
         public class FeaturedProductPartRecord : ContentPartRecord
         
  1. Add the namespace by pressing `Ctrl-.`:
  
      ![](../Attachments/getting-started-with-modules-part-2/record-addusing.png)
  
  1. Add a public property to the class. Place your cursor in the main body of the class. Type `prop` then press the `Tab` key twice:
  
      ![](../Attachments/getting-started-with-modules-part-2/record-addprop.png)
      
     This will insert a code snippet for an automatically implemented public property.
      
  1. In the first placeholder (you can just start typing and it will replace the `int` automatically) set it to `virtual bool` then press `Tab`, type `IsOnSale` and press `Enter` to complete the property.
  
      ![](../Attachments/getting-started-with-modules-part-2/record-addpropdetail.png)
  
The important thing to remember for your `ContentPartRecord` classes is that each of the properties you want to store in the database should be marked with the `virtual` keyword. This is so that `NHibernate`, the database system used by Orchard, can inject its underlying plumbing. 

You should now end up with a file called `.\Models\FeaturedProductPartRecord.cs` with the contents:

    using Orchard.ContentManagement.Records;
    
    namespace Orchard.LearnOrchard.FeaturedProduct.Models {
      public class FeaturedProductPartRecord : ContentPartRecord {
        public virtual bool IsOnSale { get; set; }
      }
    }

## Update the ContentPart
You now have a class that will provide the interface between your database and your content part (`FeaturedProductPartRecord`).

The first change to the content part will be to let it know about this record class. Then we will add a public property which mirrors the data class and specifies how it will store its data:

  1. Open the `ContentPart` file located in `.\Models\FeaturedProductPart.cs`
  
  1. Add a generic type parameter to the FeaturedProductPart class by changing this:
  
         public class FeaturedProductPart : ContentPart
     
     To this:
     
         public class FeaturedProductPart : ContentPart<FeaturedProductPartRecord>
         
  1. Add the public property to the class:
  
          [DisplayName("Is the featured product on sale?")]
          public bool IsOnSale {
            get { return Retrieve(r => r.IsOnSale); }
            set { Store(r => r.IsOnSale, value); }
          }
          
  1. Add the namespace for the `DisplayName` attribute

The `DisplayName` attribute is a feature provided by ASP.NET MVC. Later on when we build the editor view for the admin dashboard it will use this phrase instead of simply displaying "IsOnSale" 

You should now end up with a `FeaturedProductPart.cs` file that looks like this:

    using System.ComponentModel;
    using Orchard.ContentManagement;

    namespace Orchard.LearnOrchard.FeaturedProduct.Models {
      public class FeaturedProductPart : ContentPart<FeaturedProductPartRecord> {
        [DisplayName("Is the featured product on sale?")]
        public bool IsOnSale {
          get { return Retrieve(r => r.IsOnSale); }
          set { Store(r => r.IsOnSale, value); }
        }
      }
    }

The `Retrieve()` and `Store()` methods come when you inherit from `ContentPart<T>`. 

Under the hood your data is stored in two places. There is the underlying database and something called the infoset.

## Understanding data storage in Orchard

Orchard提供了一个不可思议的模块化结构。做到这一点的方法是把一切切成小的组件，称为内容组件。Orchard在运行时构建这些内容组件.


为了在数据库存储这些所有数据组件，这些数据分割在多个表. 从数据库中获取这些数据将会有很多 `JOIN`. 这意味着这不是最快的方式.

通过建立为一个二级数据缓存减少这些影响到最小。所有不同内容部件都编码到一个XML文件，然后存储到数据库表的一列，例如: 

    <Data>
      <IdentityPart Identifier="0e8f0d480f0d4d4bb72ad3d0c756a0d4" />
      <CommonPart CreatedUtc="2015-09-24T19:37:19.5819846Z" 
        ModifiedUtc="2015-09-24T19:37:19.6720488Z" 
        PublishedUtc="2015-09-24T19:37:19.6810555Z" />
      <WidgetPart RenderTitle="true" Title="Featured Product" Position="1"
        Zone="AsideFirst" Name="" CssClasses="" />
    </Data>


为了替换从3个表(IdentityPart, CommonPart and WidgetPart)里面获取数据, Orchard只是查询一个xml区块。
`Retrieve()`和`Store()`方法自动存储数据。


## Upgrading using a data migration

在第一部分课程我们创建了`Migrations.cs`，这个类的`Create()` 返回了`1`, 当Orchard执行完这个方法，它将存储 `1` 到数据库.


现在我们对模块的数据存储进行一些修改，我们添加一个表来存储数据。我们添加一个新的方法 `UpdateFrom1()`. Orchard会在下次站点加载时自动找到该方法，并运行这次数据迁移。这个方法的最后我们返回 `2`，这表示未来如果我们修改，可以添加`UpdateFrom2()`方法。我们可以返回一个更高的数字。

  1. 打开 `Migrations.cs` .
  
  1. 在  `Create()` 方法下面, 添加如下方法:
  
        public int UpdateFrom1() {
          SchemaBuilder.CreateTable(typeof(FeaturedProductPartRecord).Name,
            table => table
              .ContentPartRecord()
              .Column<bool>("IsOnSale"));
          return 2;
        }


我们使用`SchemaBuilder`类来创建一个新表。第一个参数是表的名称，这将匹配我们创建ContentPartRecord。


> **Did you notice the extra `s` I accidentally typed into the table name?**
>   `Migrations.cs` 看起来如下:

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
    
        public int UpdateFrom1() {
          SchemaBuilder.CreateTable(typeof(FeaturedProductPartRecord).Name, 
            table => table
              .ContentPartRecord()
              .Column<bool>("IsOnSale"));
    
          return 2;
        }
      }
    }

## Add a handler
探究连接模块代码与数据库快要完成了。最后为 `ContentPartRecord`注册 `StorageFilter`.

`StorageFilter`类仓储对象持久化到数据库，在这个例子中`FeaturedProductPartRecord` 是我们需要注册的类。

在`Handler`内注册，尽管这不是这个类的唯一作用。可以把这个类当成ASP.MVC的过滤器。这是用于处理特定事件发生，但没有指定content类型。

举个例子，建立一个分析模块去监听`Loaded` 事件来记录使用。
模块里的这个handler并不是很复杂，但是它要实现一些来支持持久化。

  1. 在模块项目创建 `Handlers`文件夹. 然后在这个文件夹下添加类`FeaturedProductHandler`
  
  1. 使之继承 `ContentHandler`，然后添加引用:
  
         public class FeaturedProductHandler : ContentHandler
  
  1. 我们现在需要添加一个标准样板构造函数:
  
        public FeaturedProductHandler(
          IRepository<FeaturedProductPartRecord> repository) {
            Filters.Add(StorageFilter.For(repository));
        }
        
## Update the driver to support an editor view
这次通过这个驱动来获取另外2个样板方法。
第一个方法是`Editor()`, 用于构建显示在管理页编辑shape，这是个`GET`请求

第二个方法是 `Editor()` ，它重载用于处理表单提交，将数据提交到数据库，这是`POST`请求。

  1. 打开文件夹 `.\Drivers\`下的`FeaturedProductDriver.cs`.
  
  1. 在 `Display()` 方法下, 粘贴下面代码:
  
        protected override DriverResult Editor(FeaturedProductPart part, 
          dynamic shapeHelper) {
            return ContentShape("Parts_FeaturedProduct_Edit",
              () => shapeHelper.EditorTemplate(
                TemplateName: "Parts/FeaturedProduct",
                Model: part,
                Prefix: Prefix));
        }

        protected override DriverResult Editor(FeaturedProductPart part,
          IUpdateModel updater, dynamic shapeHelper) {
            updater.TryUpdateModel(part, Prefix, null, null);
            return Editor(part, shapeHelper);
        }
        
按照上面介绍，第一个`Editor()`方法，用于在管理页展示编辑表单。它返回了一个名为`Parts_FeaturedProduct_Edit`的content shape. 我们稍后在更新`placement.info`会利用这些信息。


我们利用`shapeHelper`工厂来为当前content part(我们的FeaturedProductPart)创建一个shape，这和用于`Display()`方法的工厂一样。

使用`.EditorTemplate()` 方法来传递设置值。默认，Orchard将所有编辑视图放在`EditorTemplates` 文件夹。

The `Model` parameter passes in the data, in our case a data structure that includes the `IsOnSale` value. This will be accessible inside the Razor view and we will use it to determine what to display.

The `Prefix` is a text value that is prepended to the names of the shapes internally. This makes sure that the names are unique. The `Prefix` value that we supply is a method that comes with deriving from `ContentPartDriver` that just uses a `typeof()` to get the name of the class:

    protected virtual string Prefix { get { return typeof(TContent).Name; } }

> In more complicated projects this method can combine multiple shapes together and return them as a single `CombinedShape`.

The second overload of the `Editor()` method is used when the administrator submits a page that has the `EditorTemplate` form inside it. You can see the bare minimum code here which just attempts to pass the form data to the correct internal model. As long as it doesn't have any errors the model will then be persisted to the database automatically by Orchard. 

It then calls the original `POST` version of the `Editor()` method with the form data already included in the model. This means that after the form has been submitted and the page loads up again it will have the form fields pre-populated with the data that has been entered.

> In more complicated projects you will see further processing being completed after the `TryUpdateModel()` call. 

> Don't forget, browsing the Orchard source code is an invaluable tool to learn more about its inner workings. A good reference module for both this and the last tip is the `WidgetPartDriver` located in `.\Orchard.Web\Modules\Orchard.Widgets\Drivers\WidgetPartDriver.cs`. It shows both the combining of shapes and extra validation in the editor update method.

You should now end up with a `FeaturedProductDriver.cs` file that looks like this:

    using Orchard.ContentManagement;
    using Orchard.ContentManagement.Drivers;
    using Orchard.LearnOrchard.FeaturedProduct.Models;
    
    namespace Orchard.LearnOrchard.FeaturedProduct.Drivers {
      public class FeaturedProductDriver : ContentPartDriver<FeaturedProductPart> {
        protected override DriverResult Display(FeaturedProductPart part, string displayType, dynamic shapeHelper) {
          return ContentShape("Parts_FeaturedProduct", () => shapeHelper.Parts_FeaturedProduct());
        }
    
        protected override DriverResult Editor(FeaturedProductPart part, dynamic shapeHelper) {
          return ContentShape("Parts_FeaturedProduct_Edit",
            () => shapeHelper.EditorTemplate(
              TemplateName: "Parts/FeaturedProduct",
              Model: part,
              Prefix: Prefix));
        }
    
        protected override DriverResult Editor(FeaturedProductPart part, IUpdateModel updater, dynamic shapeHelper) {
          updater.TryUpdateModel(part, Prefix, null, null);
          return Editor(part, shapeHelper);
        }
      }
    }

## Add the EditorTemplate view
The editor template view uses the same underlying technology as the front-end view, a Razor view template. Orchard keeps all of its editor templates inside the `EditorTemplates` folder of the `Views` folder.

When you're building the administration form you will normally use many of the ASP.NET MVC helper methods such as `@Html.CheckBoxFor(model => model.IsOnSale)`. 

For this reason you will find that editor template views are strongly-typed views and start off with the `@model` command. This will mean that you get the IntelliSense while using `model` to build the forms. 

The editor form for this module will have a single section to represent the Boolean value that we have added to the model. When building the view you should wrap each form item inside a `<fieldset>` tag.

Add the EditorTemplate view by following these steps:

  1. Add a new folder inside the `Views` folder called `EditorTemplates` (`Right click` on the `View` folder in the solution explorer, click `Add`, `New Folder` and type `EditorTemplates`).
  
  1. Add another folder inside that called `Parts`.
  
  1. Add a new `.cshtml` Razor view within the `Parts` folder called `FeaturedProduct.cshtml`
  
  1. Within the `FeaturedProduct.cshtml` view file add the following HTML markup:
  
          @model Orchard.LearnOrchard.FeaturedProduct.Models.FeaturedProductPart
          
          <fieldset>
            <div class="editor-label">
              @Html.LabelFor(model => model.IsOnSale)
            </div>
            <div class="editor-field">
              @Html.CheckBoxFor(model => model.IsOnSale)
              @Html.ValidationMessageFor(model => model.IsOnSale)
            </div>
          </fieldset>

When you edit the widget in the admin dashboard you will see a simple edit form:

![](../Attachments/getting-started-with-modules-part-2/template-admin.png)

You can see that it is automatically pulling through the `[DisplayName]` attribute value that we used earlier.

## Update the front-end view
Now that we have gone through all the steps to surface the `IsOnSale` property we can finally use it to make a decision in the front-end view.

If you have experience with normal ASP.NET MVC Razor views you will know that you can blend your `C#` code in with the HTML including features such as using conditional statements to control the visibility of certain sections.

We will now update the module to show a red "ON SALE!" box in the widget when `IsOnSale` is set to `true`:

  1. Open up the view file located at `.\Views\Parts\FeaturedProduct.cshtml`
  
  1. Copy this CSS snippet into the `<style>` block at the top of the view:

        .sale-red {
          background-color: #d44950;
          color: #fff;
          float: right;
          padding: .25em 1em;
          display: inline;
        }
  
  1. Add this snippet in to the main body, above the first `<p>` tag:
  
        @if (Model.ContentPart.IsOnSale) {
          <p class="sale-red">ON SALE!</p>
        }

You will notice that this view doesn't feature an `@model` directive at the top. This is because the model is a dynamic class which contains the content part information and various other properties supplied by Orchard. This means that you can't define it in the view beforehand as the class isn't a named type and IntelliSense doesn't know what properties are going to be available at run-time. 

Because of this you are on your own when typing up the views. It can be helpful to add breakpoints to inspect the structure of the model at run-time if you don't know what value you're looking for.

The `@if (Model.ContentPart.IsOnSale) { }` line simply checks if the value is true and if so it dynamically shows the HTML contained within the curly braces, displaying a red "ON SALE!" banner within the widget.

You should now have a `FeaturedProduct.cshtml` file that looks like this:

    <style>
      .btn-green {
        padding: 1em;
        text-align: center;
        color: #fff;
        background-color: #34A853;
        font-size: 2em;
        display: block;
      }
      .sale-red {
        background-color: #d44950;
        color: #fff;
        float: right;
        padding: .25em 1em;
        display: inline;
      }
    </style>
    @if (Model.ContentPart.IsOnSale) {
      <p class="sale-red">ON SALE!</p>
    }
    <p>Todays featured product is the Sprocket 9000.</p>
    <p><a href="/sprocket-9000" class="btn-green">Click here to view it</a></p>

## Update the placement.info
Before we run the module to see our updated widget in action we need to make a change to the `placement.info` file. The module should run without errors at this stage but until we set up a `<place>` for the editor template you won't be able to configure the `IsOnSale` variable.

  1. Open the `placement.info` file located in the root folder of the module.
  
  1. Within the `<placements>` tag add the following `<place>` tag:
  
        <Place Parts_FeaturedProduct_Edit="Content:7.5"/>
         
     The order of the `<place>` tags doesn't matter.

The name of the place that we are targeting `Parts_FeaturedProduct_Edit` was defined in the driver class when we configured it in the `EditorTemplate()` shape factory method. The shape will be injected into the local zone named "content" with a weight of 7.5. In this case the weight of 7.5 will move it down to the bottom of the form.

> When developing your modules it is common to forget this last stage. While you might not always be able to remember to update the `placement.info` before you run, you should take a moment to remember the solution.

> Whenever the shape isn't displayed where it was expected, think `placement.info` first.

You should now have a `placement.info` file that looks like this:

    <Placement>
      <Place Parts_FeaturedProduct="Content:1"/>
      <Place Parts_FeaturedProduct_Edit="Content:7.5"/>
    </Placement>

## Trying the module out in Orchard
Great! You have completed another stage of the development. Now its time to load the website up in the browser and play with the new feature you just built.

  1. Within Visual Studio, press `Ctrl-F5` on your keyboard to start the website without debugging enabled (its quicker and you can attach the debugger later if you need it).
  
  1. Navigate to the admin dashboard.
  
  1. Click `Widgets` in the side menu.
  
  1. If you follow the guide correctly in part 1, you should see your `Featured Product Widget` in the list under `AsideFirst`. Click on the word `Featured Product` (this may vary depending the title you entered when you set the widget up):
  
    ![](../Attachments/getting-started-with-modules-part-2/testing1-editwidget.png)
    
  1. The `Edit Widget` page will be displayed. Scroll down to the bottom to find the setting we added:
  
    ![](../Attachments/getting-started-with-modules-part-2/testing1-configure.png)
  
    Tick the checkbox. 
    
  1. Click `Save`.
  
  1. Navigate back to the homepage of the website by clicking on the site title in the top corner of the admin dashboard:
  
    ![](../Attachments/getting-started-with-modules-part-2/testing1-gotohomepage.png)
   
You should now see your product is marked as being on sale:

![](../Attachments/getting-started-with-modules-part-2/testing1-final.png)

> **Bonus Exercise:** Go back into the admin dashboard, uncheck the setting, save and come back again to see how the it no longer shows as being on sale.

## Download the code for this lesson
You can download a copy of the module so far at this link:

  * [Download Orchard.LearnOrchard.FeaturedProduct-Part2-v1.0.zip](../Attachments/getting-started-with-modules-part-2/Orchard.LearnOrchard.FeaturedProduct-Part2-v1.0.zip)
  
To use it in Orchard simply extract the archive into the modules directory at `.\src\Orchard.Web\Modules\`. If you already have the module installed from a previous part then delete that folder first.

> For Orchard to recognize it the folder name should match the name of the module. Make sure that the folder name is `Orchard.LearnOrchard.FeaturedProduct` and then the modules files are located directly under that.

## Conclusion
This part of the course has expanded your knowledge to touch on some of the data storage and content management features in Orchard. 

You have added in the common module development classes that we didn't cover in the first part. You've experienced using the data migrations to incrementally update your data. You've also seen the basics of creating an admin interface and using it to update the configuration settings of a widget.

You can now see the core process of adding a variable to your module and then implementing it, working successively up the layers to surface it in the admin dashboard and the front-end view.  

In the next part of the getting started with modules course we will look at [working with content items at the code level](Getting-Started-with-Modules-Part-3).
