这篇文章描述如何创建一个简单的只显示"Hello World"页面模块。这个方法很好的展示如何控制页面的生命周期。

另一个简单的模块示例:[Quick Start - Get module blueprint](http://orchardjumpstart.codeplex.com/)
> **This guide has been marked for review.** If you are just getting started with Orchard module development you should read the [Getting Started with Modules course](Getting-Started-with-Modules) first. It will introduce you to building modules with Orchard using Visual Studio Community, a free edition of Visual Studio. 

# Introduction

Orchard is built on top of ASP.NET MVC, which means that if you already know that framework you should feel right at home. If not, do not worry as we'll explain everything we're doing.

MVC is a pattern where concerns are neatly separated: there is a model (M) for the data, a controller (C) that orchestrates the UI and determines how it operates on the model, and a view (V) whose only responsibility is to display what the controller hands it.

In the case of our Hello World module, we won't have any data so the model will be of no concern to us. We will just have a controller and a view. All the rest will be some necessary plumbing to declare what we're doing to Orchard. We will come back to these concepts and recapitulate once we've built our module.


Orchard模块是一系列扩展，可以在其他Orchard站点中重用。模块是通过[MVC Areas](https://docs.asp.net/en/latest/mvc/controllers/areas.html)来实现的.
一个Orchard模块是一个包含一个清单文件的简单area. 模块可以使用Orchard APIs(但这并不是必须的).


# Generating the Module Structure

在生成模块文件结构之前，需要下载安装并启用 **Code Generation** 模块,更多信息,参考[Command-line Code Generation](Command-line-scaffolding)
启用之后,打开Orchard命令行，通过下面的命令创建一个名为`HelloWorld`的模块：

    
    codegen module HelloWorld


# Modifying the Manifest

You should now have a new HelloWorld folder under the Modules folder of your Orchard web site. In this folder, you'll find a module.txt file. Open it and customize it as follows:
在Orchard Web站点的Modules文件夹下已经生成了一个HelloWorld文件夹, 在这个文件夹有一个module.txt的文件，打开并自定义成如下:

    
    name: HelloWorld
    antiforgery: enabled
    author: The Orchard Team
    website: http://orchardproject.net
    version: 0.5.0
    orchardversion: 1.8.1
    description: The Hello World module is greeting the world and not doing much more. 
    features:
        HelloWorld:
            Description: A very simple module.
            Category: Sample


这个文本文件是描述我们的模块. 其中的信息会在管理页面中使用.

> Note: 清单文件支持空格和tab缩进, 推荐使用空格。

# Adding the Route

HelloWorld模块需要处理/HelloWorld相对Url, 为此需要在HelloWorld文件夹中创建Routes文件定义路由。
    
    using System.Collections.Generic;
    using System.Web.Mvc;
    using System.Web.Routing;
    using Orchard.Mvc.Routes;
    
    namespace HelloWorld {
        public class Routes : IRouteProvider {
            public void GetRoutes(ICollection<RouteDescriptor> routes) {
                foreach (var routeDescriptor in GetRoutes())
                    routes.Add(routeDescriptor);
            }
    
            public IEnumerable<RouteDescriptor> GetRoutes() {
                return new[] {
                    new RouteDescriptor {
                        Priority = 5,
                        Route = new Route(
                            "HelloWorld", // this is the name of the page url
                            new RouteValueDictionary {
                                {"area", "HelloWorld"}, // this is the name of your module
                                {"controller", "Home"},
                                {"action", "Index"}
                            },
                            new RouteValueDictionary(),
                            new RouteValueDictionary {
                                {"area", "HelloWorld"} // this is the name of your module
                            },
                            new MvcRouteHandler())
                    }
                };
            }
        }
    }


这个路由描述了Url与controller actions的映射, 上述代码将地址HelloWorld映射到area HelloWorld的Home Controller的Index action。

# Creating the Controller

The new module also has a Controllers folder ready to be filled. Create the following HomeController.cs file in that folder:

    
    using System.Web.Mvc;
    using Orchard.Themes;
    
    namespace HelloWorld.Controllers {
        [Themed]
        public class HomeController : Controller {
            public ActionResult Index() {
                return View("HelloWorld");
            }
        }
    }


This is the controller that will handle the requests for the HelloWorld URL. The default action, index, is requesting that the HelloWorld view gets rendered.

注意controller类的Themed属性，它是请求当前主题视图

# Creating the View

In the Views folder, create a folder named Home. In the Views\Home folder, create the following HelloWorld.cshtml file:

    
    <h2>@T("Hello World!")</h2>


This file is specifying the core contents of our view. All the chrome around it will get added by the current theme's default layout.

Notice that we used the T helper function that makes this view ready to be localized. This is not mandatory but it's a nice touch.
这里使用了T帮助方法来实现本地化，这不是强制的。

# Adding the new files to the project

We're almost done. The only task remaining is to declare to the system the set of files in the module for dynamic compilation.

Open the HelloWorld.csproj file in a text editor and add the following lines after one of the &lt;/ItemGroup&gt; tags:

    
    <ItemGroup>
      <Compile Include="Routes.cs"/>
      <Compile Include="Controllers\HomeController.cs"/>
    </ItemGroup>


Also add the following to the ItemGroup section that already has other Content tags:

    
    <Content Include="Views\Home\HelloWorld.cshtml" />


# Activate the Module

Finally, you need to activate your new module. In the command line, type:
最后我们需要启用这个模块，可以通过下面的命令, 也可在模块管理页面点击启用

    
    feature enable HelloWorld


You could also have done this from the "Features" screen in the site's admin UI.

# Use the Module

You may now add /HelloWorld to the URL of your Orchard site in your favorite web browser and obtain a nice Hello World message:

![The UI for our completed module](../Attachments/Building-a-hello-world-module/HelloWorld.PNG)

# Conclusion

In this tutorial, we have built a very simple module that handles a route (/HelloWorld) through the home controller's index action and serves a simple view that gets skinned by the current theme. We have done so with only free tools and in a way that differs very little from what you would do in a regular ASP.NET MVC area. We did get a few things for free by making this an Orchard module, such as activation/deactivation of the module, or theming of the view with no effort on our part.

Hopefully this will get you started with Orchard and prepare you to build more elaborate modules.

The code for this topic can be downloaded from here: [HelloWorld.zip](../Attachments/Building-a-hello-world-module/HelloWorld.zip)
