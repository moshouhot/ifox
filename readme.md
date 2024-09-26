IFoxCAD类库从入门到精通  
 0. 前言   
使用IFoxCAD类库之前，您需要通读一遍cad的官方教程至Creating and Editing Entities 章节，教程的地址  
，如果看不懂英文的（其实你可以用浏览器的翻译插件看），可以去看中文翻译版《Autocad .net 开发指南2012版》。   
通读的目的不是让你写多少程序，而是知道cad的二次开发到底是个什么样子，api的构成是什么，特别是了解符号表是什么，符号表的构成。   
此文档由IFox的群友自发编写，不足之处还请谅解。  
0.1 项目来源  
起初 雪山飞狐（又狐哥） 在明经论坛发布了开源库，后来狐哥自己的项目进行了极大的丰富后形成NFox类库。然后 落魄山人 在征得 雪山飞狐 的同意后，对NFox类库进行了整理，增加了注释等，重新发布了NFox类库。  
后来，经过一段时间的更新后，由于莫名其妙的原因NFox类库挂掉了。而这时山人同学已经基本吃透NFox类库，考虑到NFox的封装过于复杂，遂进行了重构。  
重构的类库命名为IFoxCAD， 寓意为：I(爱)Fox(狐哥)，本项目发布于Inspire Function（中文名：跃动方程） 组织下，感谢 小轩轩 给起的名字。  
0.2 版本计划  
目前ifox受支持的版本为 0.5.2.4 ，0.6.1，0.7.2这三个主要的版本。  
其中0.5.2.4版本支持net35，net40，net45  
0.6.1版本支持net40，net45  
0.7以后的版本仅支持net48 (net48依然可以正确的运行在cad2014上)  
0.5版本作为低版本用户的维护版本，在没有维护人员接手的前提下，不进行任何的功能性更新，仅有重大bug修复更新，nuget版本号为0.5.*。  
0.6.*版本，支持net40-net45。此版本不进行功能性更新，仅有重大bug修复更新。  
v0.7及以上的版本，全部升级至net48，发布nuget 0.7.*版本。此版本进行功能性更新。  
请根据自己的需求选择对应的版本。  
经测试net48项目(不使用net40以上CADApi)可以在CAD2013及以上项目正常使用  
1. 前置知识  
1.1. AutoCAD的二次开发API的结构  
 本节内容基本照抄桌子的Managed .NET Developer's Guide (.NET)  
AutoCAD .NET API是由不同的DLL文件组成的，这些文件包含了大量的类、结构、方法和事件，提供了对绘图文件或应用程序中对象的访问。每个DLL文件都定义了不同的命名空间，用来根据功能组织库中的组件。  你会经常使用的AutoCAD .NET API的主要DLL文件是：       
● AcCoreMgd.dll，在编辑器内工作时使用，发布和打印，以及定义可以从AutoLISP调用的命令和函数。       
● AcDbMgd.dll，在处理存储在绘图文件中的对象时使用。       
● AcMgd.dll，在处理应用程序和用户接口时使用。      
● AcCui.dll，在处理自定义文件时使用。  
IFox 已内置上述类库，详见模板配置文件："AutoCAD.NET" Version="23.0.0">。  
1.1.1. 了解AutoCAD的对象层次结构  
AutoCAD .NET API主要是由对象构建的。每个暴露的对象都精确代表了AutoCAD的一个部分。在AutoCAD .NET  API中的 ，有许多不同类型的对象。在AutoCAD .NET API中有代表性的一些对象是。  
● 图形对象，如线、弧、文本和尺寸  
● 样式设置，如图层、线型和尺寸样式     
● 组织结构，如图层、组和块       
● 绘图显示，如视图和视口       
● 甚至是图形和AutoCAD应用程序    
这些对象的结构是分层的，AutoCAD应用对象位于根部。这种层次结构通常被称为  "对象模型"。下面的插图显示了应用程序对象和BlockTableRecord中的实体之间的基本关系，例如模型空间。在AutoCAD .NET  API中还有许多对象在这里没有表示。  
  
1.1.2. Application对象 (.NET)  
Application对象是AutoCAD .NET  API的根对象。从应用程序对象中，你可以访问主窗口以及任何打开的图形。一旦你有了一个图形，你就可以访问图形中的对象。 例如，Application对象有一个DocumentManager属性，它返回DocumentCollection对象。这个对象提供了对当前在AutoCAD中打开的图形的访问，允许你创建、保存和打开图形文件。应用程序对象的其他属性提供了对应用程序特定数据的访问，如信息中心、主窗口和状态栏。MainWindow属性允许访问应用程序的名称、大小、位置和可见性。 虽然应用程序对象的大多数属性允许访问AutoCAD .NET API中的对象，但也有一些属性引用了AutoCAD  ActiveX®自动化中的对象。这些属性包括应用程序对象的COM版本（AcadApplication）、菜单栏（MenuBar）、加载的菜单组（MenuGroups）和偏好设置（Preferences）。   
1.1.3. 文档对象  Document (.NET)  
文档对象实际上是一个AutoCAD图形，它是DocumentCollection对象的一部分。你使用DocumentExtension和DocumentCollectionExtention对象来创建、打开和关闭图形文件。文档对象提供了对数据库对象的访问，数据库包含了所有的图形和大多数非图形的AutoCAD对象。 与数据库对象一起，文档对象提供对状态栏、文档打开的窗口、编辑器和事务管理器对象的访问。编辑器对象提供了对各种功能的访问，这些功能用于从用户那里获得点或输入的字符串或数值形式的输入。 事务管理器对象用于在一个被称为事务的单一操作下访问多个数据库对象。事务可以被嵌套，当你完成一个事务时，你可以提交或放弃所做的改变。   
  
1.1.4. 数据库对象 Database(.NET)   
数据库对象包含所有的图形对象和大多数非图形的AutoCAD对象。数据库中包含的一些对象是实体、符号表和命名字典。数据库中的实体代表了图形中的图形对象。线、圆、弧、文本、填充体和多段线都是实体的实例。用户可以在屏幕上看到一个实体，并可以对其进行操作。 你通过Document对象的Database成员属性访问当前文档的Database对象。 Application.DocumentManager.MdiActiveDocument.Database    
符号表和字典   
符号表和字典对象提供对非图形对象（块、层、线型、布局等）的访问。每个图形都包含一组固定的符号表，而一个图形中的字典数量可以根据AutoCAD中使用的功能和应用类型而变化。新的符号表不能被添加到数据库中。 符号表的例子是层表（LayerTable）和块表（BlockTable），前者包含层表记录，后者包含块表记录。所有的图形实体（线、圆、弧等等）都属于一个块表记录。默认情况下，每张图纸都包含预定义的模型和图纸空间的块表记录。每个图纸空间的布局都有自己的块表记录。 字典是一个容器对象，可以包含任何AutoCAD对象或XRecord。词典要么存储在数据库中的命名对象词典下，要么作为表记录或图形实体的扩展词典。命名对象字典是与数据库相关的所有字典的主表。与符号表不同，新的字典可以被创建并添加到命名对象字典中。 注意：字典对象不能包含绘图实体。  
VBA/ActiveX对照表    
AutoCAD .NET API 中的数据库对象与 ActiveX  自动化库中的文档对象类似。要访问ActiveX自动化库的Document对象中的大多数属性，你需要使用AutoCAD .NET  API的Document和Database对象。    
  
1.1.5 CAD各版本关系  
1.2. VS2022的使用  
ifoxcad类库必须使用vs2022的全新版本，因为ifoxcad类库采用了最新的c#语法。  
为什么要这么做 ，就是因为cad二次开发圈子你们落后了，该跟上潮流了。新语法真香！！！  
1.2.1. 基础.net类库  
默认Visual Studio 2022 不再支持安装 .NET Framework 4.5 组件，选择组件里面已经不能选择4.5 / 4.0 的框架了。如果你要是使用0.5-0.6版本的ifoxcad，需要安装net4.0/4.5框架。  
解决方案如下：  
最小化安装vs2019，然后安装net4.0/4.5框架。  
利用nuget的microsoft.netframework.referenceassemblies.net45包，下载之后解压复制：build.NETFramework\v4.5\ 到 C:\Program Files (x86)\Reference Assemblies\Microsoft\Framework.NETFramework\v4.5。  
net40同理处理。  
最新【一键处理】方法：（不保证一直有效，取决于工具作者）  
https://github.com/MrXhh/VSTools/releases  
1）下载 VS2022Net4NotCompileFix  
2）右键管理员执行  
3）重启VS  
1.2.2. 类库调试设置  
vs2022新建的类库项目是不能直接设置调试启动程序的。但是cad的调试需要启动cad，所以需要新建个调试文件后设置为可执行文件模式。  
  
注：上图界面路径为：菜单栏-> 调试->xxx调试属性。  
按上图的顺序，点击1，点击2来删除默认的配置，点击3来新建个配置，选择 可执行文件。  
  
按上图的顺序依次设置，命令行参数/nologo的设置主要是为了加快启动速度。  
1.3.C#语言的部分语法  
最新的语法支持，ifox类库应用了部分新的语法，所以需要在项目文件里启用新的语法支持，来更好的使用ifox。 <LangVersion>preview</LangVersion>。  
using  
//可以使用    
using var xxx = new xxx();   
xxxx;  
//来代替原来的   
using (var xxx = new xxx())  
{  
    xxxx;  
}  
//两者的区别就是新的语法更简洁，using作用域是在函数结束的时候自动释放。  
全局using  
利用global using 语法可以进行全局using引用，这样就不用每个文件都有using一堆的引用。需要将所有的global using都放在一个文件里，文件名为GlobalUsings.cs。  
global using System;  
global using System.Collections;  
global using System.Collections.Generic;  
global using System.IO;  
global using System.Linq;  
global using System.Text;  
global using System.Reflection;  
global using System.Text.RegularExpressions;  
namespace  
文件范围限定namespace，将文件的缩进减少一级。  
//可以使用  
namespace sourcetest;  
  
public class Command  
{  
    [CommandMethod("Test")]  
    public void Test()  
    {  
        using var tr = new DBTrans();  
  
  
    }  
}  
//代替原来的  
namespace sourcetest  
{  
    public class Command  
    {  
        [CommandMethod("Test")]  
        public void Test()  
        {  
            using var tr = new DBTrans();  
  
  
        }  
    }  
}  
可空类型  
C# 新版本对于null做了很多的语法糖以及可空类型来帮助开发者减少因null处理不当导致的错误。  
首先开启可空类型 <Nullable>enable</Nullable>  
声明可空的变量   
string? a = null;  // 声明可空的字符串类型  
int? a = null; // 声明可空的整形  
int b = null; // 报错  
string c = null; // 报错  
具体参见 https://learn.microsoft.com/zh-cn/dotnet/csharp/nullable-references  
2. IFoxCad的架构说明  
IFoxCAD是基于NFOX类库的重制版，主要是提供一个最小化的内核，即DBTrans、SymbolTable、ResultData、SelectFilter等基础类，其他的功能都通过扩展方法的方式来实现。  
其重制的原因在于原NFOX类库的封装过于厚重，初学者理解起来困难，重制版希望做到最小化的内核，方便理解，然后丰富的扩展函数来实现大量的功能，便于学着现有的教程中那套基于Database扩展函数封装思路的初学者快速的入门。  
2.1 为什么要构建DBTrans类？  
主要是为封装cad的Transaction类的，为何如此封装有如下原因：  
虽然可以继承Transaction类，但是由于其构造函数为受保护的，同时其参数不能很方便的传递，所以即便cad在使用的时候也是调用TransactionManager的StartTransaction方法，所以直接继承Transaction类进行扩展并不方便。   
由于cad实体图元和非实体图元几乎都存储在数据库里，也就是Database里，所以目前市面上的教程基本都是基于Database的扩展函数进行封装。但是cad本身其实推荐的都是利用事务（Transaction）来对数据库进行增删改的操作，但是默认的Transaction类仅仅提供了几个方法，每次操作数据库或者修改图元都需要手动进行大量的重复性操作，这部分操作几乎都被封装为函数活跃于每个重复的轮子里。那么狐哥转变思路，继续不考虑数据库的操作而是延续cad的思路，着重封装关于Transaction的操作。  
DBTrans类主要就是为了完成事务的自动化处理而存在的，其完成的任务主要为：  
优化原桌子API的事务类的功能，比如获取对象的函数采用泛型的方法，可以直接返回对应的类型  
自动提交事务，节省多余的代码  
ifox推荐对事务的使用是简洁的，高性能的，因此根据大量的工程实践提出采用一个事务贯穿始终的用法。  
DBTrans类里基本的封装就是Transaction，然后是Document、Database、Editor、符号表、命名字典等，而这些其实都是CAD二次开发关于图元操作经常打交道的概念。  
DBTrans的每个实例都具有这些属性，而这些属性就对应于CAD的相关类库，通过这些属性就可以对数据进行相应的操作。特别是符号表中最常用的就是块表，通过对块表的操作来实现添加图元等。  
2.2 为什么要构建SymbolTable类  
主要是为了统一处理9个符号表，具体原因如下：  
其实CAD的API对于符号表都是继承自SymbolTable类，符号表记录都是继承自SymbolTableRecord类，所以其实这个自定义的类叫SymbolTable是和CAD的内部API有命名上的冲突的，希望给我给个贴近自定义的理念的类名。  
CAD的默认API关于符号表和符号表记录是隔离关系的，就是说符号表和符号表记录在API上是没有关系的，只是数据库里每个符号都映射着相应的符号表记录，所以为了对应符号表和符号表记录，写了SymbolTable类。  
通过这个类，就可以统一的处理符号表和符号表记录了，比如层表的处理就从原来首先获取层表对象->新建层表记录对象->打开层表的写模式->添加层表记录，变成新建层表的关联类实例->添加层表记录。  
有了这个类，DBTrans类就可以直接通过属性获取符号表的关联关系，然后进行符号表的处理。  
3. 事务管理器和符号表  
3.1 事务管理器  
3.1.1 事务管理器介绍  
事务管理器是CAD .NET二次开发过程绕不过去的一个部分，只要是涉及到读写cad数据的地方几乎都推荐在事务里完成。利用事务管理器可以自动的在退出事务的时候执行释放对象等操作，防止程序员不能释放对象，造成cad崩溃。  
但是，在日常的使用中，会发现每次开启事务，然后完成的都是差不多的任务，然后每次都要调用commit()函数，每次都要获取到符号表，每次要写模式，读模式等提权降级操作，但是这些操作其实都可以自动完成的，因此 ifoxcad 类库提供事务管理器类来完成本来需要手工完成的工作，让用户可以更方便的处理事务内的程序。  
用事务管理器类可以完成：  
原生CAD提供的事务管理器的全部操作  
方便的符号表操作  
方便的基础属性操作  
方便的对象获取操作  
方便的字典操作  
事务管理器类的类名为：DBTrans。开启默认的事务管理器写法为：  
using var tr = new DBTrans()；//采用了新的语法  
3.1.2 原生的事务管理器操作  
关于cad提供的原生事务管理器的操作不是本文档的重点，因为那操作起来麻烦，不够集中的将需要在事务内的操作做统一管理。  
3.1.3 符号表操作  
ifoxcad 类库的符号表其实是一个符号表的泛型类，直接将符号表和符号表记录包装为一个整体。不用担心，在实际使用的过程中，你几乎不会关心符号表的构成原理。  
Ifoxcad 类库里采用如下的符号来表示9大符号表。  
然后怎么使用呢？使用符号表一共分几步呢？  
using DBTrans tr = new DBTrans()； // 第一步，开启事务  
var layerTable = tr.LayerTable;// 第二步，获取图层表    
 // 事务结束并自动提交  
上面是一个获取层表的例子，其他的符号表都是一样的写法，因为这些符号表都是事务管理器的属性。那么获取到符号表之后能做些什么？  
向符号表里添加元素  
using DBTrans tr = new DBTrans()；  
// 第一步，开启事务  
var layerTable = tr.LayerTable;  
// 第二步，获取图层表  
layerTable.Add("1");// 返回值为ObjectId  
// 第三步，向层表里添加一个元素，也就是新建一个图层。  
// 事务结束并自动提交  
每个符号表都有Add函数，而且提供了不止一个重载函数。  
添加和获取符号表里的元素  
想要添加和获取符号表内的某个元素非常的简单：  
using DBTrans tr = new DBTrans()； // 第一步，开启事务  
  
var layerTable = tr.LayerTable; // 第二步，获取图层表  
layerTable.Add("1"); // 第三步，添加名为“1”的图层，即新建图层  
ObjectId id = layerTable["1"]; // 第四步，获取图层“1”的id。     
// 事务结束并自动提交  
每个符号表都提供了索引形式的获取元素id的写法。  
线型表  
// 两种方式  
// 第一种，直接调用tr.LinetypeTable.Add("hah")函数，然后对返回值ObjectId做具体的操作。  
// 第二种，直接在Action委托里把相关的操作完成。  
tr.LinetypeTable.Add(  
                   "hah",  
                   ltt =>   
                   {  
                       ltt.AsciiDescription = "虚线";  
                       ltt.PatternLength = 0.95; //线型的总长度  
                       ltt.NumDashes = 4; //组成线型的笔画数目  
                       ltt.SetDashLengthAt(0, 0.5); //0.5个单位的划线  
                       ltt.SetDashLengthAt(1, -0.25); //0.25个单位的空格  
                       ltt.SetDashLengthAt(2, 0); // 一个点  
                       ltt.SetDashLengthAt(3, -0.25); //0.25个单位的空格  
                   });  
// 这段代码同时演示了 ifoxcad 类库关于符号表的public ObjectId Add(string name, Action<TRecord> action)这个函数的用法。  
// 或者直接调用：  
tr.LinetypeTable.Add("hah", "虚线",0.95,new double[]{0.5,-0.25,0,-0.25});  
// 获取线型表  
tr.LinetypeTable["hah"];  
其他符号表的操作类同。如果类库没有提供的Add函数的重载，那么Action委托可以完成你想完成的所有事情。  
3.1.4 基础属性操作  
事务管理器类提供了Document、 Editor 、Database三个属性来在事务内部处理相关事项。  
同时还提供了关于字典的相关属性。  
3.1.5 对象获取操作  
提供了1个泛型 GetObject<T>函数的重载来根据ObjectId来获取到对象。  
3.2. 符号表  
3.2.1 概述  
每个图形文件都包含有9个固定的符号表。不能往数据库里添加新的符号表。如图层表（LayerTable），其中包含图层表记录；还有块表（BlockTable），其中包含块表记录等。所有的图形实体（线、圆、弧等等）都属于一个块表记录。缺省情况下，任何图形文件都包含为模型空间和图纸空间预定义的块表记录。每个符号表都有对应的符号表记录，可以理解为符号表是一个集合，而符号表记录是这个集合的元素。CAD的符号表和符号表记录的对应关系如下：  
3.2.2 符号表操作  
那么如何来操作这些符号表呢？下面是一个使用默认API新建图层的例子：  
Document acDoc = Application.DocumentManager.MdiActiveDocument;  
Database acCurDb = acDoc.Database;  
using (Transaction acTrans = acCurDb.TransactionManager.StartTransaction())  
{  
  // 返回当前数据库的图层表  
  LayerTable acLyrTbl = acTrans.GetObject(acCurDb.LayerTableId,OpenMode.ForRead) as LayerTable;  
  // 检查图层表里是否有图层 MyLayer  
  if (acLyrTbl.Has("MyLayer") != true)  
  {  
    // 以写模式打开图层表  
    acLyrTbl.UpgradeOpen();  
    // 新创建一个图层表记录，并命名为”MyLayer”  
    LayerTableRecord acLyrTblRec = new LayerTableRecord();  
    acLyrTblRec.Name = "MyLayer";  
    // 添加新的图层表记录到图层表，添加事务  
    acLyrTbl.Add(acLyrTblRec);  
    acTrans.AddNewlyCreatedDBObject(acLyrTblRec,     true);  
    //提交修改  
    acTrans.Commit();  
  }  
  // 关闭事务，回收内存；  
}   
上面的例子用了20多行的代码来完成一个很简单的功能，这就是AutoCAD提供的API太过于基础，没有进行进一步的封装造成的。那么如果我们单独为图层表封装一个函数来处理图层表，其他的8个符号表也要同样的各自封装函数，这样看起来没什么问题，但是对于代码的复用却没有很好的考虑进去。仔细思考一下，其实对于符号来说无非就是增删改三个主要的操作等，对于符号表记录来说无非就是一些属性的操作，增加实体的操作等。那么有没有一种办法可以统一管理9个符号表呢？其实AutoCAD提供了9个符号表和符号表记录的抽象基类，SymbolTable和SymbolTableRecord，但是这两个类提供的功能又很简单，只有寥寥几个函数和属性，完全不能满足我们的需求。因此ifoxcad类库提供了符号表类来封装9个符号表的大部分功能。那么用类库来完成上述的操作是什么样子的呢？见下面的例子：  
// 以下代码采用最新的c#版本语法  
using var tr = new DBTrans(); // 打开事务  
var layertable = tr.LayerTable.Add("MyLayer"); //添加图层  
同样的功能我们只需要两行就可以搞定了。那么有同学会问了，我同样单独对每个符号表的封装一样可以达到这个效果？是的，确实可以达到一样的效果，但是我只封装了一次，只是针对符号表的差异部分做了一些增量的处理，其他的代码都是复用的，而你要写9次。  
言归正传，通过上述的例子，我们会发现几个现象：  
符号表的操作是在事务内。  
符号表成了事务的属性  
添加符号表记录到符号表调用Add函数就可以了(其实提供了好多的重载，来完成不同的细粒度的操作)。  
符号表的操作都在事务内，这样由事务统一管理符号表的变动，减少出错的可能。  
符号表作为事务的属性，那么获取符号表记录就变成了属性的索引值。  
var layertable = tr.LayerTable["MyLayer"];  
不管是什么符号表，都是一个Add函数搞定添加操作。  
而删除就是：  
tr.LayerTable.Remove("1");  
注意，这里的关于删除图层的操作需要调用Delete函数  
看，我教会了你操作图层表，那么其他的8个表你都会了，都是一样的操作。  
3.2.3 块表添加图元  
一般的情况下，添加图元的操作都是要在事务里完成。目前大部分的添加图元的自定义函数都是Database或Editor对象的扩展函数。但是实际上添加图元的本质是读写图形数据库，具体的手段是对块表里的块表记录的读写。而实际的操作其实都是在事务里完成，也就是说事务其实是为了操作的原子性存在的。所以符合cad操作规则的写法其实应该是事务作为一系列操作的主体来进行。因此ifoxcad类库的封装思路为：扩展块表记录的函数，在事务管理器类里通过属性调用AddEntity函数来添加图元。  
对于这个添加图元的操作，一共分为如下几步：  
创建图元对象，可以在事务外创建，也可以在事务内创建。  
打开要添加图元的块表记录，在事务内打开。  
添加图元到块表记录  
下面看示例：  
添加图元到当前空间  
// 以下代码采用最新的c#版本语法  
using var tr = new DBTrans(); //开启事务管理器  
var line1 = new Line(new Point3d(0, 0, 0), new Point3d(1, 1, 0)); //定义一个直线  
tr.CurrentSpace.AddEntity(line1); // 将直线添加到当前绘图空间的块表记录  
添加图元到当前/模型/图纸空间  
// 以下代码采用最新的c#版本语法  
using var tr = new DBTrans(); //开启事务管理器  
var line1 = new Line(new Point3d(0, 0, 0), new Point3d(1, 1, 0)); //定义一个直线  
tr.CurrentSpace.AddEntity(line1); // 将直线添加到当前绘图空间的块表记录  
tr.ModelSpace.AddEntity(line1); // 将直线添加到当前模型空间的块表记录  
tr.PaperSpace.AddEntity(line1); // 将直线添加到当前图纸空间的块表记录  
块表  
块表这里需要特殊的说明一下：  
比如说添加一个块，用如下代码：  
tr.BlockTable.Add(blockName, btr => btr.AddEntity(ents));  
这里的blockName就是块名，ents就是图元列表。这种方式可以更细粒度的控制定义的块。  
插入块参照，比如：  
tr.CurrentSpace.InsertBlock(point,objectid);   
// 用于插入块参照，提供了重载函数来满足不同的需求  
其他函数的介绍  
tr.BlockTable.GetRecord() 函数，可以获取到块表的块表记录，同理层表等符号表也有同样的函数。  
tr.BlockTable.GetRecordFrom() 函数，可以从文件拷贝块表记录，同理层表等符号表也有同样的函数。  
tr.BlockTable.GetBlockFrom() 函数，从文件拷贝块定义。示例见7.2节  
  4. 快速入门及其他特殊说明  
4.1 快速入门  
（适用于动手能力，解决问题能力强的朋友，否则请使用项目模版来创建适用ifox的项目，项目模版见4.4章节）  
打开vs，新建一个standard类型的类库项目，注意，需要选择类型的时候一定要选standard2.0  
  
双击项目，打开项目文件：  
修改项目文件里的<TargetFramework>netcore2.0</TargetFramework>为<TargetFrameworks>NET45</TargetFrameworks>。其中的net45，可以改为NET45以上的标准TFM（如：net45、net46、net47等等）。同时可以指定多版本。具体的详细的教程见 VS通过添加不同引用库，建立多条件编译。  
在 <PropertyGroup> xxx </PropertyGroup> 中增加 <LangVersion>preview</LangVersion>，主要是为了支持最新的语法，本项目采用了最新的语法编写。项目文件现在的内容类似如下：  
<Project Sdk="Microsoft.NET.Sdk">  
   <PropertyGroup>  
       <TargetFramework>net45</TargetFramework>  
       <LangVersion>preview</LangVersion>  
   </PropertyGroup>  
</Project>  
右键项目文件，选择管理nuget程序包。  
  
在nuget程序里搜索ifox，记得将包括预发行版打钩。截止本文最后更新时，nuget上最新的支持net35的版本为ifox.cad.source 0.5.2.x版本和ifox.Basal.source 0.5.2.x版本。点击安装就可以。  
  
添加引用，在新建的项目里的cs文件里添加相关的引用。由于源码包提供的是不包括using引用的源码，所以需要自行添加一个GlobalUsings.cs文件到项目里，然后里面需要把所有的引用都写上。  
using Autodesk.AutoCAD.ApplicationServices;  
using Autodesk.AutoCAD.EditorInput;  
using Autodesk.AutoCAD.Runtime;  
using Autodesk.AutoCAD.Geometry;  
using Autodesk.AutoCAD.DatabaseServices;  
using IFoxCAD.Cad;  
添加代码  
[CommandMethod(nameof(Hello))]  
public void Hello()  
{  
    using DBTrans tr = new();  
    var line1 = new Line(new Point3d(0, 0, 0), new Point3d(1, 1, 0));  
    tr.CurrentSpace.AddEntity(line1);  
    // 如果您没有添加<LangVersion>preview</LangVersion>到项目文件里的话：按如下旧语法：  
    // using(var tr = new DBTrans())  
    // {  
    //     var line1 = new Line(new Point3d(0, 0, 0), new Point3d(1, 1, 0));  
    //     tr.CurrentSpace.AddEntity(line1);  
    // }  
}  
这段代码就是在cad的当前空间内添加了一条直线。  
生成，然后打开cad，netload命令将刚刚生成的dll加载。如果需要调试需要设置启动程序为cad，见1.2.2章节。  
运行hello命令，然后缩放一下视图，现在一条直线和一个圆已经显示在屏幕上了  
4.2 屏蔽IFox的元组、索引、范围功能  
特别提醒： 考虑到早期的框架没有提供System.Range类型(net core 开始提供)、System.Index类型(net core 开始提供)、System.ValueTuple类型(net 47开始提供)，本项目IFox.Basal包里包含了他们。 如果引用了包含System.Range等类型的第三方包（如IndexRange等），请在项目文件中定义NOINDEX、NORANGE、NOVALUETUPLE常量，以避免重复定义。上述代码能起作用的前提是用源码包，普通包暂时无解。  
<PropertyGroup Condition="'$(TargetFramework)' == 'NET47'">      
    <DefineConstants>$(Configuration);NOINDEX;NORANGE;NOVALUETUPLE</DefineConstants>  
</PropertyGroup>  
NOINDEX、NORANGE、NOVALUETUPLE 分别针对三种类型，哪种类型冲突就定义哪种。  
  
4.3 编译 IFox 源码工程  
安装好VS2022，其他的vs不支持。  
编译本项目需要你准备好git，具体的安装教程可以去网上搜索一下。当然也可以利用vs的git来完成。  
首先在gitee上fork本项目到你的账号，然后clone到本地。  
原生git使用命令行，打开终端/powershell/cmd，cd到你要存放的目录，然后运行下面的命令，把里面的yourname替换为你的名字，这样就在本地创建了一个ifoxcad文件夹，里面就是本项目的所有源码。  
git clone https://gitee.com/yourname/ifoxcad.git  
当然也可以采用vs的图形化操作，打开vs，选择 克隆存储库-->填入仓库地址和存放路径-->点击克隆。新手小白推荐用此办法。  
打开ifoxcad文件夹，双击解决方案文件，打开vs，等待项目打开，加载nuget包，然后生成就可以了。  
切记，不要用低版本的vs打开本项目，因为本项目采用了某些新的语法，所以老版本的vs是不兼容的，也就是说用了低版本的vs，你连编译都过不去。  
4.4 IFoxCad 项目模版  
项目模板目前分为两种，其中一种是vs的扩展插件，一种是dotnet项目模板。其中的vs的扩展插件只支持0.5.2.4版本，dotnet项目模板支持0.6.1以上的版本。  
如果你要开发2012版本以下的cad，那么你只能使用0.5.2.4版本的ifox，所以你将不得不使用vs的扩展插件，并且这个版本的ifox将不再提供功能更新。  
如果你要开发2012以上的，你可以使用0.6.1版本以上的ifox，0.6版本的ifox取消了net35的支持，但是还是继续支持net40和net45版本，可以通用于2012版本以上的所有cad。使用dotnet模板来轻松创建项目。  
但是如果没有低版本需求，我们强烈建议你直接开发net48版本的，不使用CADnet40以上CADApi或许可支持CAD2012-2024，这样就可以使用最新的ifox的版本和最新的功能。  
4.4.1 vs扩展插件（老版本，适用于0.5.2.4版本的ifox，如果你要使用net35的cad的话，可以用这个版本，否则请看4.4.2章节的）  
可以在vs扩展菜单-管理扩展中搜索ifoxcad，即可安装项目模板。使用项目模版可以方便的创建支持多目标多版本的使用ifoxcad类库的项目和类。如果无法在vs的市场里下载，就去上面的QQ群里下载。  
项目模版里的自动加载选择了简单api，ifox还提供了一套功能更强大的api，具体的可以参考14章的内容。  
4.4.2 dotnet项目模板  
建议尽可能的抛弃老的版本，因为ifox的后续发展计划就是取1消老版本的支持，0.7版本开始只支持net48版本，因此后续的新功能等等都只在新版本中提供。  
使用dotnet项目模板是很简单的，同时还可以创建acad，gcad，zcad的项目，这是老的vs扩展插件所不能实现的功能。  
首先你要了解一些基本的命令行的知识，比如打开终端，比如切换目录等。  
IFox项目模板地址IFoxCad.Templates 0.7.2  
打开终端，win10/11的系统，右键开始菜单的win徽标，就可以打开powershell或者终端。  
安装项目模板  
打开终端后，在打开的命令行窗口输入：  
dotnet new uninstall IFoxCad.Templates::0.7.1  
如果你使用win10的powershell，可能会出现命令报错，这时候你可以将上一行命令里的install替换为-i  
dotnet new -i IFoxCad.Templates::0.7.1   // 仅适用于win10 powershell 报错的时候  
使用项目模板  
项目模板可以直接用命令行的方式使用，也可以直接打开vs创建新的项目。  
命令行的方式：  
dotnet new ifox.0.7 -C acad -n acadtest  // 创建autocad项目，并将项目命名为acadtest  
dotnet new ifox.0.7 -C zcad -n zcadtest  // 创建zwcad项目，并将项目命名为zcadtest  
dotnet new ifox.0.7 -C gcad -n gcadtest  // 创建gstarcad项目，并将项目命名为gcadtest  
或者在vs里直接创建新项目，项目类型选择下图里的第二个模板。然后根据提示进行就可以了。  
  
4.4.3 模板使用的中的问题  
在使用模板创建了新项目后，有些同学会出现无法编译报错的问题。  
使用vs扩展插件创建的项目，请打开项目文件或者打开nuget管理器，将ifox的包版本调整为0.5.2.4版本大概率会解决你的问题。  
使用dotnet项目模板创建的项目，大概率是你安装了0.6.1版本，请按如下步骤删  除老模板，清理缓存后重新安装模板。  
打开终端，按顺序输入如下命令：  
dotnet new uninstall IFoxCad.Templates  
rmdir  C:\Users\你的名字\.templateengine -force  
dotnet new install IFoxCad.Templates::0.6.1.1  
  
然后创建个新的项目测试是否还有编译报错。  
有同学会提示CallerArgumentExpression 特性报错  
此功能需要 .NET 7.0 SDK和C# 10 可用。  
网上找到的相关内容：  
对于面向较旧 .NET 或 .NET Standard 的较旧项目，CallerArgumentExpressionAttribute 不可用。幸运的是，只要安装了 .NET 7.0 SDK，C# 10 和此功能仍然可以与它们一起使用。只需手动将 CallerArgumentExpressionAttribute 类添加到您的项目中，并像内置属性一样使用它。  
所以需要在vs里安装一下net7的运行时试试，如果不行，就单独安装net7的sdk。  
4.5 使用IFoxCad的几种方式  
目前ifox提供了三种使用方式，建议一般的用户使用第二种源码包的形式。有志于本项目发展并想提交点代码的可以选择第三种。  
第一种是直接使用普通的nuget包。目前的新的版本已不在提供普通包。  
此种方式使用便捷，只要在项目中引用了IFox.CAD.ACAD的包，就可以直接使用了。缺点一是无法控制ifox提供的元组功能的屏蔽，导致和其他的三方包的冲突；二是生成目录里带有ifox的dll。  
第二种是使用源码包。  
此种方式使用便捷，只要在项目中引用了IFox.Basal.Source和IFox.CAD.Source两个nuget包就可以直接使用了。优点就是使用简单，生成的目录里没有ifox的dll，同时还可以通过定义预处理常量的方式屏蔽ifox提供的元组等功能。缺点就是无法修改源码，即便解包修改了，也不会同步到nuget上。  
第三种是使用git子模块。  
此种方法使用步骤复杂，需要熟悉git及其子模块的使用，需要引用ifox里的共享项目文件。优点就是可以使用最新的代码，可以修改代码。具体的可以参考如下说明进行：  
4.6 让 IFox 作为您的子模块[需要有较强的动手能力]  
IFox的develop、v0.6、v0.7分支是一个多cad版本分支,您可以利用此作为您的git项目子模块.  
子模块是以共享工程的方式加入到您的工程的,共享项目的名字为IFox.CAD.Shared和IFox.Basal.Shared  
千万不要用IFox.CAD.ACAD内的工程作为引用,否则您将遭遇cad加载失效。  
一些全局命名空间的缺少,我们也建议您使用全局命名空间来补充,您只需要按照IFox.CAD.ACAD的GlobalUsings.cs文件一样添加就好了。  
若您使用acad是09版本以下的，比如 07 08版本,建议你升级至09 版本以上。强烈建议升级至net48。  
使用子模块可以使您在nuget未更新前临时更改IFox项目的部分代码，以适应您当前项目的需要。  
使用子模块后每次IFox更新需要您手动更新子模块，和自动更新nuget不一样。  
4.6.1 项目添加子模块前准备  
一个好的项目组织可以使您的项目直观易读，因此建议您将项目改为如下图所示。assets文件夹为子模块区，统一存储您的子模块内容，以后您也可以拥有其他的子模块，统一放进这个文件夹就行。src文件夹为您的项目源代码区，可以自行组织源码。  
感谢zxk大佬提供的解决思路：先用模板0.7(vicwjb)建一个ifox项目,不要勾选项目和解决方案同目录，项目名为src，然后在根目录下新建assets目录，这样子模块就和自己的代码文件夹src平行关系了。  
使用子模块前请先学会git操作  
  
4.6.2开始将IFox放入您的子模块区  
使用git命令行进入到assets文件夹（此时文件夹内如果没有其他子模块应为空）。  
  
输入命令：  
git submodule add -b branch url folder   
branch是子模块分支名,url是子模块的git地址,folder是你要在本地存放的文件夹名  
git submodule add -b v0.6 https://gitee.com/inspirefunction/ifoxcad.git IFox  
gitbash里边执行上述代码就可以将IFox作为您的子模块。  
此处子模块的子模块的git地址也可以使用您foked之后自己仓库的IFox下载地址，如果使用自己的地址，您将可以更改此子模块的内容并提交PullRequestPR，使用官方的地址将不可以PR，除非您是管理员。  
4.6.3更新您的子模块  
做完上述操作，重新打开VisualStudio的Git窗口就可以看到这个IFox子模块。选中IFox存储库之后点击下箭头就可以在VisualStudio更新子模块。  
  
4.6.4使用子模块替换nuget package  
打开visual studio之后，会发现assets文件夹中并没有内容，此时右键->添加->现有项目,找到assets中的IFox.Basal.Shared.shproj这个文件就是basel的共享项目；再次重复上述操作找到IFox.CAD.Shared.shproj这个文件就是IFoxCad的共享项目。具体操作截图如下：  
如果你的项目里没有assets文件夹，那么你需要在vs里创建一个文件夹。（知识点，vs解决方案的文件夹和你电脑里的文件夹不是一个概念）  
  
  
  
此时，VisualStudio中的assets文件夹就有了这两个共享项目（如下图），然后在自己的项目  
依赖项->右键->添加共享项目引用，打开的窗口上，把刚才的两个项目前的对勾打上。这时您就用上了基于子模块的IFox。如果你使用了zxk大佬的快捷办法，那么你需要把模板自带的两个源码包ifox.*.source移除。  
  
  
  
4.6.5为子模块打上global usings 缺失引用补丁  
建议直接使用IFox的项目模板创建项目，不容易缺失命名空间。  
如果使用了项目模板还是缺少命名空间，建议在global usings 文件内加上以下几个命名空间：  
global using System.Linq.Expressions;  
global using System.Collections.ObjectModel;  
global using Microsoft.CSharp.RuntimeBinder;  
4.6.6 为IFox提交子模块代码更改  
如果您在4.6.2节使用的是自己的地址，在修改代码之后可以直接提交commit，并在网页端提交到主仓库。  
有更多问题可以在群里问@MUSIC-DIE  
5. 添加图元  
5.1 添加图元  
添加直线  
using DBTrans tr = new();  
Line line = new(new Point3d(0, 0, 0), new Point3d(1, 1, 0));  
tr.CurrentSpace.AddEntity(line); //当前空间添加图元  
  
// 添加直线  
[CommandMethod(nameof(Test_AddLine1))]  
public void Test_AddLine1()  
{  
    using DBTrans tr = new();  
    Line line1 = new(new Point3d(0, 0, 0), new Point3d(1, 1, 0));  
    Line line2 = new(new Point3d(0, 0, 0), new Point3d(1, 1, 0));  
    Line line3 = new(new Point3d(1, 1, 0), new Point3d(3, 3, 0));  
    tr.CurrentSpace.AddEntity(line1, line2, line3);  
}  
5.2 创建几何图元  
IFox提供了一些附加的创建几何图元的方法，在对应的类型的扩展类中，如圆弧为ArxEx，圆为CircleEx等，提供了一些诸如3点建圆等函数，如下演示，其他可根据类名自行发掘  
    [CommandMethod(nameof(Test_Drawarc))]  
    public void Test_Drawarc()  
    {  
        using DBTrans tr = new();  
        Arc arc1 = ArcEx.CreateArcSCE(new Point3d(2, 0, 0), new Point3d(0, 0, 0), new Point3d(0, 2, 0));// 起点，圆心，终点  
        Arc arc2 = ArcEx.CreateArc(new Point3d(4, 0, 0), new Point3d(0, 0, 0), Math.PI / 2);            // 起点，圆心，弧度  
        Arc arc3 = ArcEx.CreateArc(new Point3d(1, 0, 0), new Point3d(0, 0, 0), new Point3d(0, 1, 0));   // 起点，圆上一点，终点  
        tr.CurrentSpace.AddEntity(arc1, arc2, arc3);  
    }  
  
[CommandMethod(nameof(Test_DrawCircle))]  
public void Test_DrawCircle()  
{  
    using DBTrans tr = new();  
    var circle1 = CircleEx.CreateCircle(new Point3d(0, 0, 0), new Point3d(1, 0, 0));// 两点画圆（直径上的两个点）  
    var circle2 = CircleEx.CreateCircle(new Point3d(-2, 0, 0), new Point3d(2, 0, 0), new Point3d(0, 2, 0));// 三点画圆，成功  
    var circle3 = CircleEx.CreateCircle(new Point3d(-2, 0, 0), new Point3d(0, 0, 0), new Point3d(2, 0, 0));// 三点画圆，失败  
    tr.CurrentSpace.AddEntity(circle1, circle2!);  
    if (circle3 is not null)  
        tr.CurrentSpace.AddEntity(circle3);  
    else  
        Env.Printl("三点画圆失败");  
}  
  
    [CommandMethod(nameof(Test_AddPolyline3))]  
    public void Test_AddPolyline3()  
    {  
        using var tr = new DBTrans();  
  
        var pts = new List<Point3d>  
        {  
            new(0, 0, 0),  
            new(0, 1, 0),  
            new(1, 1, 0),  
            new(1, 0, 0)  
        };  
        var pline = pts.CreatePolyline();  
        tr.CurrentSpace.AddEntity(pline);  
  
        var pline1 = pts.CreatePolyline(p =>  
        {  
            p.Closed = true;  
            p.ConstantWidth = 0.2;  
            p.ColorIndex = 1;  
        });  
        tr.CurrentSpace.AddEntity(pline1);  
    }  
6. 图元操作  
6.1 遍历图元名字   
[CommandMethod(nameof(iterateEntity))]  
public void iterateEntity()  
{  
    using DBTrans tr = new DBTrans();  
    tr.CurrentSpace.ForEach(id =>  
    {  
        var ent = id.GetObject<Entity>();      
        //如果要修改图元时var entity1 = id.GetObject<Entity>(OpenMode.ForWrite);  
        ent?.GetRXClass().DxfName.Print();  
        Env.Print("\n ");  
        Env.Print("\nDXF 名称:    " + ent?.GetRXClass().Name);  
        //AcDbLine,AcDbPolyline,AcDbText  
        Env.Print("\n图元名称:    " + ent?.GetType().Name);  
        //Line,Polyline,DBText  
        Env.Print("\nObjectID:    " + ent?.ToString());  
        Env.Print("\nHandle:      " + ent?.Handle.ToString());  
        if (ent is Line acdbLine)//如果ent是直线，转换为直线变量cdbLine  
        {  
            Env.Print("\nacDbLinet长度为： ");  
            //var txt = acdbLine.GetProperty("Length");//取得直度长度属性  
            var txt = acdbLine.Length;  // 这么写就可以了  
            txt.Print();  
        }  
        Env.Print("\n********************\n");  
    });  
}  
  
[CommandMethod(nameof(Test_PrintDxfname))]  
public void Test_PrintDxfname()  
{  
    using var tr = new DBTrans();  
  
    tr.CurrentSpace.ForEach(id => {  
        id.GetObject<Entity>()?.GetRXClass().DxfName.Print();  
    });  
}  
 6.2 移动  
ent.Move(pt1, pt2);  
6.3 旋转  
ent.Rotation(new(100, 0, 0), Math.PI / 2);  
6.4 镜像  
// 沿两点组成的线镜像  
ent.Mirror(pt1, pt2);  
// 沿面镜像  
var plane = new Plane(Point3d.Origin, Vector3d.ZAxis);  
ent.Mirror(plane);  
// 沿对称点镜像  
ent.Mirror(pt);  
6.5 缩放  
ent.Scale(scaleCenter,scaleValue);  
 7. 块表的使用  
7.1 查  
查找名为“自定义块”的块表中的图块记录。  
using var tr = new DBTrans();  
if (tr.BlockTable.Has("自定义块"))  
{  
    //要执行的操作  
}  
遍历块表并打印所有的块表的图块名称（包含*ModelSpace*和*PaperSpace*） 。  
using var tr = new DBTrans();  
tr.BlockTable.GetRecordNames().ForEach(action: (blockname) => blockname.Print());  
统计所有名为“自定义块”的数量。  
public void Test_DBTrans_BlockCount()  
{  
    using var tr = new DBTrans();  
    var i = tr.CurrentSpace  
            .GetEntities<BlockReference>()  
            .Where(brf => brf.GetBlockName() == "自定义块")  
            .Count();  
    Env.Print(i);  
}  
7.2 增  
添加自定义块记录。  
// 块定义  
[CommandMethod(nameof(Test_BlockDef))]  
public void Test_BlockDef()  
{  
    using DBTrans tr = new();  
    // var line = new Line(new Point3d(0, 0, 0), new Point3d(1, 1, 0));  
    tr.BlockTable.Add("test",  
        btr =>  
        {  
            btr.Origin = new Point3d(0, 0, 0);  
        },  
        () => // 图元  
        {  
            return new List<Entity> { new Line(new Point3d(0, 0, 0), new Point3d(1, 1, 0)) };  
        },  
        () => // 属性定义  
        {  
            var id1 = new AttributeDefinition() { Position = new Point3d(0, 0, 0), Tag = "start", Height = 0.2 };  
            var id2 = new AttributeDefinition() { Position = new Point3d(1, 1, 0), Tag = "end", Height = 0.2 };  
            return new List<AttributeDefinition> { id1, id2 };  
        }  
    );  
    // ObjectId objectId = tr.BlockTable.Add("a");// 新建块  
    // objectId.GetObject<BlockTableRecord>().AddEntity();// 测试添加空实体  
    tr.BlockTable.Add("test1",  
    btr =>  
    {  
        btr.Origin = new Point3d(0, 0, 0);  
    },  
    () =>  
    {  
        var line = new Line(new Point3d(0, 0, 0), new Point3d(1, 1, 0));  
        var acText = new TextInfo("123", Point3d.Origin, AttachmentPoint.BaseLeft)  
                    .AddDBTextToEntity();  
  
        return new List<Entity> { line, acText };  
    });  
}  
插入块定义。  
[CommandMethod(nameof(Test_InsertBlockDef))]  
public void Test_InsertBlockDef()  
{  
    using DBTrans tr = new();  
    var line1 = new Line(new Point3d(0, 0, 0), new Point3d(1, 1, 0));  
    var line2 = new Line(new Point3d(0, 0, 0), new Point3d(-1, 1, 0));  
    var att1 = new AttributeDefinition() { Position = new Point3d(10, 10, 0), Tag = "tagTest1", Height = 1, TextString = "valueTest1" };  
    var att2 = new AttributeDefinition() { Position = new Point3d(10, 12, 0), Tag = "tagTest2", Height = 1, TextString = "valueTest2" };  
    tr.BlockTable.Add("test1", line1, line2, att1, att2);  
  
  
    var ents = new List<Entity>();  
    var line5 = new Line(new Point3d(0, 0, 0), new Point3d(1, 1, 0));  
    var line6 = new Line(new Point3d(0, 0, 0), new Point3d(-1, 1, 0));  
    ents.Add(line5);  
    ents.Add(line6);  
    tr.BlockTable.Add("test44", ents);  
  
  
    var line3 = new Line(new Point3d(5, 5, 0), new Point3d(6, 6, 0));  
    var line4 = new Line(new Point3d(5, 5, 0), new Point3d(-6, 6, 0));  
    var att3 = new AttributeDefinition() { Position = new Point3d(10, 14, 0), Tag = "tagTest3", Height = 1, TextString = "valueTest3" };  
    var att4 = new AttributeDefinition() { Position = new Point3d(10, 16, 0), Tag = "tagTest4", Height = 1, TextString = "valueTest4" };  
    tr.BlockTable.Add("test2", new List<Entity> { line3, line4 }, new List<AttributeDefinition> { att3, att4 });  
    // tr.CurrentSpace.InsertBlock(new Point3d(4, 4, 0), "test1"); // 测试默认  
    // tr.CurrentSpace.InsertBlock(new Point3d(4, 4, 0), "test2");  
    // tr.CurrentSpace.InsertBlock(new Point3d(4, 4, 0), "test3"); // 测试插入不存在的块定义  
    // tr.CurrentSpace.InsertBlock(new Point3d(0, 0, 0), "test1", new Scale3d(2)); // 测试放大2倍  
    // tr.CurrentSpace.InsertBlock(new Point3d(4, 4, 0), "test1", new Scale3d(2), Math.PI / 4); // 测试放大2倍,旋转45度  
  
    var def1 = new Dictionary<string, string>  
    {  
        { "tagTest1", "1" },  
        { "tagTest2", "2" }  
    };  
    tr.CurrentSpace.InsertBlock(new Point3d(0, 0, 0), "test1", atts: def1);  
    var def2 = new Dictionary<string, string>  
    {  
        { "tagTest3", "1" },  
        { "tagTest4", "" }  
    };  
    tr.CurrentSpace.InsertBlock(new Point3d(10, 10, 0), "test2", atts: def2);  
    tr.CurrentSpace.InsertBlock(new Point3d(-10, 0, 0), "test44");  
}  
获取文件里的块定义：  
public void Test_BlockFile()  
{  
    using DBTrans tr = new();  
    var id = tr.BlockTable.GetBlockFrom(@"C:\Users\vic\Desktop\test.dwg", false);  
    //getblockfrom函数的第一个参数是文件路径，第二个参数是是否强制覆盖本图的块定义  
    tr.CurrentSpace.InsertBlock(Point3d.Origin, id);  
}  
7.3 改  
// 修改块定义  
[CommandMethod(nameof(Test_BlockDefChange))]  
public static void Test_BlockDefChange()  
{  
    using DBTrans tr = new();  
  
    tr.BlockTable.Change("test", btr =>  
    {  
        //修改块名，改成当前块名（可以获取到动态块名）  
        btr.Name = btr.GetBlockName();  
        foreach (var id in btr)  
        {  
            var ent = tr.GetObject<Entity>(id);  
            using (ent!.ForWrite())  
            {  
                if (ent is Dimension dBText)  
                {  
                    dBText.DimensionText = "234";  
                    dBText.RecomputeDimensionBlock(true);  
                }  
                if (ent is Hatch hatch)  
                    hatch.ColorIndex = 0;  
            }  
        }  
    });  
    tr.Editor?.Regen();  
}  
7.4 删  
删除名为“自定义块”的块表记录。  
using var tr = new DBTrans();  
tr.BlockTable.Remove("自定义块");  
8. 层表的使用  
8.1 查  
查看层表中是否含有名为“MyLayer”的图层。  
using var tr = new DBTrans();  
if(tr.LayerTable.Has("MyLayer"))  
{  
    //要执行的操作  
}  
遍历图层表并打印每个图层的名字 。  
using var tr = new DBTrans();  
tr.LayerTable.GetRecordNames().ForEach(action: (layname) => layname.Print());  
8.2 增  
创建一个名为“MyLayer”的图层，要求图层颜色为红色，线宽为0.3mm，可打印。  
using var tr = new DBTrans();  
tr.LayerTable.Add("MyLayer",it =>  
    {  
        it.Color = Color.FromColorIndex(ColorMethod.ByColor, 1);  
        it.LineWeight = LineWeight.LineWeight030;  
        it.IsPlottable = true;  
    });  
8.3 改  
查找名为“MyLayer2”的图层，并将图层“MyLayer2”的名称改为“MyLayer3”，颜色改为2号色，设为不可打印。  
using var tr = new DBTrans();  
if (tr.LayerTable.Has("MyLayer2"))  
{  
    tr.LayerTable.Change("MyLayer2", lt => {  
        lt.Name = "MyLayer3";  
        lt.Color = Color.FromColorIndex(ColorMethod.ByAci, 2);  
        lt.IsPlottable = false;  
        });  
}  
8.4 删  
尝试删除图层。  
using var tr = new DBTrans();  
tr.LayerTable.Delete("0");// 删除图层 0  
tr.LayerTable.Delete("Defpoints");// 删除图层Defpoints  
tr.LayerTable.Delete("1");// 删除不存在的图层 1  
tr.LayerTable.Delete("2");// 删除有图元的图层 2  
tr.LayerTable.Delete("3");// 删除图层 3  
强制删除图层。  
using var tr = new DBTrans();  
tr.LayerTable.Remove("2"); // 强制删除存在图元的图层 2  
9. 字体样式表的使用  
9.1 查  
查找名为“宋体”的字体样式。  
using var tr = new DBTrans();  
if(tr.TextStyleTable.Has("宋体"))  
{  
    //要执行的操作  
}  
9.2 增  
[CommandMethod(nameof(Test_TextStyle))]  
public void Test_TextStyle()  
{  
    using var tr = new DBTrans();  
    tr.TextStyleTable.Add("宋体", "宋体.ttf", 0.8);  
  
    tr.TextStyleTable.Add("宋体1", FontTTF.宋体, 0.8);  
    tr.TextStyleTable.Add("仿宋体", FontTTF.仿宋, 0.8);  
    tr.TextStyleTable.Add("fsgb2312", FontTTF.仿宋GB2312, 0.8);  
    tr.TextStyleTable.Add("arial", FontTTF.Arial, 0.8);  
    tr.TextStyleTable.Add("romas", FontTTF.Romans, 0.8);  
    tr.TextStyleTable.Add("daziti", ttr =>  
    {  
        ttr.FileName = "ascii.shx";  
        ttr.BigFontFileName = "gbcbig.shx";  
    });  
}  
9.3 改  
public void Test_TextStyleChange()  
{  
    using var tr = new DBTrans();  
    tr.TextStyleTable.AddWithChange("宋体1", "simfang.ttf", height: 5);  
    tr.TextStyleTable.AddWithChange("仿宋体", "宋体.ttf");  
    tr.TextStyleTable.AddWithChange("fsgb2312", "Romans", "gbcbig");  
}  
9.4 删  
删除名为“宋体”的字体样式。  
using var tr = new DBTrans();  
var textId = tr.TextStyleTable["宋体"];  
tr.TextStyleTable.Remove(textId);  
10. 线型表的使用  
10.1 查  
每个AutoCAD图形会自动加载3种线形：BYBLOCK、BYLAYER和CONTINUOUS，可以通过下述方式获得这三种线型的ObjectId。  
using var tr = new DBTrans();  
SymbolUtilityServices.GetLinetypeByBlockId(tr.Database);  
SymbolUtilityServices.GetLinetypeByLayerId(tr.Database);  
SymbolUtilityServices.GetLinetypeContinuousId(tr.Database);  
查看线型表中是否含有名为“CENTER”的线型。  
using var tr = new DBTrans();  
if (tr.LinetypeTable.Has("CENTER"))  
{  
    //要执行的操作  
}  
遍历线型表并打印每个线型的名字 。  
using var tr = new DBTrans();  
tr.LinetypeTable.GetRecordNames().ForEach(action: (linetypeName) => linetypeName.Print());  
10.2 增  
加载已有线型  
从acadiso.lin线型文件中加载指定线型 CENTER ，并返回 CENTER 线型的 ObjectId。  
using var tr = new DBTrans();  
if(!tr.LinetypeTable.Has("CENTER"))  
{  
    tr.Database.LoadLineTypeFile("CENTER", "acadiso.lin");  
    return tr.LinetypeTable["CENTER"];  
}  
加载自定义*.lin文件里的所有线型（已有线形会触发异常）  
 try  
 {  
     using var tr = new DBTrans();  
     tr.Database.LoadLineTypeFile("*", "D:\\文件名.lin");  
 }  
 catch (Exception)  
 {  
  
 }  
新建自定义线型  
自定义一个DASHLINES线型。  
using var tr = new DBTrans();  
tr.LinetypeTable.Add("DASHLINES", (ltr) =>  
{  
    ltr.AsciiDescription = "虚线";//线型说明  
    ltr.PatternLength = 0.95;//组成线型的图案长度（划线、空格、点）  
    ltr.NumDashes = 4;//组成线型的图案数目  
    ltr.SetDashLengthAt(0, 0.5);//0.5个单位的划线  
    ltr.SetDashLengthAt(1, -0.25);//0.25个单位的空格  
    ltr.SetDashLengthAt(2, 0);//一个点  
    ltr.SetDashLengthAt(3, -0.25);//0.25个单位的空格  
});  
4.自定义一个带文字的线型。  
using var tr = new DBTrans();  
tr.LinetypeTable.Add("文字线型",  ltrText  =>  
{  
    //添加带文字的线型  
    ltrText.AsciiDescription = "文字";//线型说明  
    ltrText.PatternLength = 0.9;//组成线型的图案长度（划线、空格、点）  
    ltrText.NumDashes = 3;//组成线型的图案数目  
    ltrText.SetDashLengthAt(0, 0.5);//0.5个单位的划线  
    ltrText.SetDashLengthAt(1, -0.2);//0.2个单位的空格  
    ltrText.SetShapeStyleAt(1, tr.TextStyleTable["Standard"]);//设置文字的文字样式  
    //文字在线型的 X 轴方向上向左移动0.1个单位，在Y轴方向向下移动0.05个单位。  
    ltrText.SetShapeOffsetAt(1, new Vector2d(-0.1, -0.05));  
    ltrText.SetShapeScaleAt(1, 0.1);//文字的缩放比例  
    ltrText.SetShapeRotationAt(1, 0);//文字的旋转角度为0（不旋转）  
    ltrText.SetTextAt(1, "GAS");//文字内容  
    ltrText.SetDashLengthAt(2, -0.2);//0.2个单位的空格  
});  
5.将CENTER设为当前线型  
using var tr = new DBTrans();  
//查找是否包含CENTER线型  
if(!tr.LinetypeTable.Has("CENTER"))  
{  
    tr.Database.LoadLineTypeFile("CENTER", "acadiso.lin"); //导入CENTER线型  
}  
tr.Database.Celtype = tr.LinetypeTable["CENTER"];  
10.3 删  
卸载CENTER线型。  
using var tr = new DBTrans();  
tr.LinetypeTable["CENTER"].Erase();  
注意:不能卸载如下线型：  
BYBLOCK  
BYLAYER  
CONTINUOUS  
当前线型  
已使用的线型  
外部参照的线形  
块定义中的线型  
11. 其他符号表的使用  
11.1 标注  
11.1.1 增  
增加名称为DimStyleName，缩放比例为1，箭头类型为建筑标记的标注样式  
using var tr = new DBTrans()  
tr.DimStyleTable.Add("DimStyleName", e =>  
{  
    e.Dimlfac = 1 //缩放比例  
    //箭头 建筑标记，  
    //内置的类型均在Env.DimblkType枚举里  
    //可以通过Env.GetDimblkName()方法获取字符串  
    e.Dimblk = tr.BlockTable["_ARCHTICK"];  
      
}  
//此处action内的属性可在系统变量里查找，或者定义界面悬停也会有显示，内容较多。  
///https://help.autodesk.com/view/ACD/2020/CHS/?page=sysvars&q=D  
11.1.2 删  
方法类似于10.3，删除DimStyleName标注样式  
using var tr = new DBTrans()  
tr.DimStyleTable["DimStyleName"].Erase()  
11.1.3 查  
方法类似于10.2，查询是否存在DimStyleName标注样式  
using var tr = new DBTrans();  
if (tr.DimStyleTable.Has("DimStyleName"))  
{  
    //要执行的操作  
}  
11.2视图表及视图操作  
操作当前视图   
我们通过调用Editor对象的GetCurrentView()方法来访问模型空间或图纸空间中视口的当前视图。 GetCurrentView()方法返回一个ViewTableRecord 对象。我们就用ViewTableRecord对象来操作活动视口中视图的缩放、位置及方向。一旦修改了ViewTableRecord对象，我们就可以调用SetCurrentView()方法来更新活动视口中的当前视图。   
用来操作当前视图的常用属性：   
CenterPoint - DCS(显示坐标系)坐标系中视图的中心点；   
Height - DCS 坐标系中视图的高度，高度增加视图拉远，高度减小视图拉近；   
Target - WCS 坐标系中视图的目标；   
ViewDirection - WCS 坐标系中视图目标到视图观察点的矢量；   
ViewTwist - 视图扭转角(弧度)；   
Width - DCS 坐标系中视图的宽度，宽度增加视图拉远，宽度减小视图拉近；用来操作当前视图的函数   
本例代码是一个常用子程序，后面的示例中将用到。Zoom()函数接受 4 个参数，实现了缩放视图到边  
界、平移视图、视图居中以及按给定系数放大视图等功能。Zoom()函数要求所有坐标值为 WCS 坐标。   
视图表操作  
using var tr = new DBTrans();  
tr.ViewTable.Add("View2");//添加到View2视口  
tr.ViewTable.GetRecordNames().ForEach(action: (Viewname) => Viewname.Print());//遍历视口表并打印每个视口的名字。  
if (tr.ViewTable.Has("View2")) { Env.Print("\nView2视口存在"); }  
tr.ViewTable.Remove("View2");  
if (!tr.ViewTable.Has("View2")) { Env.Print("\nView2视口不存在"); }  
视图操作  
  
    [CommandMethod(nameof(Test_Zoom))]  
    public void Test_Zoom()  
    {  
        using DBTrans tr = new();  
        var res = Env.Editor.GetEntity("\npick ent:");  
        if (res.Status == PromptStatus.OK)  
            Env.Editor.ZoomObject(res.ObjectId.GetObject<Entity>()!);  
    }  
  
    [CommandMethod(nameof(Test_ZoomExtents))]  
    public void Test_ZoomExtents()  
    {  
        Env.Editor.ZoomExtents(  
    }  
11.3 视口  
  
11.4 UCS  
  
12. 拖拽类JigEx的使用  
JigEx类为ifox封装的拖拽类，使用它可以方便快捷简单高效的编写Jig。  
JigEx创建的时候有一个委托和一个容差参数，委托里面有两个参数，鼠标点mpw和队列queue,  
容差用来防止频繁响应导致的图元闪烁，一般按默认即可。  
鼠标点mpw为鼠标当前的wcs坐标  
队列queue是用于对Jig中的临时性图元进行显示  
同时提供worlddraw，用于对持久性图元进行显示。  
简单来说，JigEx内部创建的图元(临时性图元)放在queue里，在每个循环开始时会销毁上次的，外部的图元（持久型图元）使用worlddraw来显示  
下面为一段没有业务内容的JigEx创建。  
其中(mpw,queue)为委托，1e-6为容差，mpw为委托中的鼠标点(WCS)，queue为委托中的队列  
using var jig=new JigEx((mpw, queue) =>  
    {  
      
    },1e-6);  
    jig.DatabaseEntityDraw(worlddraw =>  
    {  
      
    });  
JigEx提供了SetOptions函数用来对拖拽获取鼠标点时的文字提示、鼠标样式、关键字、基点等参数进行设置，同时提供了返回值和委托的方式，方便深度定制的用户  
// 1.设置提示语和关键字  
jig.SetOptions("\n选择点", new Dictionary<string, string>() { { "A", "操作(A)" }, { "Q", "操作(Q)" } });  
  
// 2.设置基点，鼠标形状，提示语  
var jppo= jig.SetOptions(Point3d.Origin, CursorType.RubberBand, "\n选择点");  
  
// 3.使用返回值设置其他选项  
jppo.Keywords.Add("A", "A", "操作(A)");  
  
// 4.使用委托的方式设置  
jig.SetOptions(jppo =>  
{  
    jppo.Message = "\n选择点";  
    jppo.UseBasePoint = true;  
    jppo.BasePoint = Point3d.Origin;  
    jppo.Keywords.Add("A", "A", "操作(A)");  
});  
当然，queue和worlddraw未必都要使用，也可以选择性使用，只用其中一种。  
下面是一些简单的示例  
示例一：使用Jig在鼠标位置画一个半径为100的圆，我将分别用queue和worlddraw两种方式进行演示  
使用queue(不推荐此用法)  
using var jig=new JigEx((mpw, queue) =>  
{  
    var circle = new Circle(mpw,Vector3d.ZAxis,100);  
    queue.Enqueue(circle);  
});  
jig.SetOptions("\n选择圆心位置");  
var r1 = jig.Drag();  
if (r1.Status != PromptStatus.OK)  
    return;  
using var tr = new DBTrans();  
tr.CurrentSpace.AddEntity(jig.Entitys);  
可以看到JigEx封装了Entitys属性用来取出最后一次鼠标移动后加入queue中的图元  
使用worlddraw（推荐）  
var circle=new Circle(Point3d.Origin,Vector3d.ZAxis,100);  
using var jig=new JigEx((mpw, queue) =>  
{  
    circle.Center = mpw;  
});  
jig.DatabaseEntityDraw(worlddraw => worlddraw.Geometry.Draw(circle));  
jig.SetOptions("\n选择圆心位置");  
var r1 = jig.Drag();    
if (r1.Status != PromptStatus.OK)  
    return;  
using var tr = new DBTrans();  
tr.CurrentSpace.AddEntity(circle);  
大家应该已经看出上面所说的临时性和持久性的区别，queue只能添加在委托里创建的图元，而worlddraw可以绘制任意的图元，包括已经存在于数据库中的图元。  
那么这时候有人会说，那还要queue干嘛，都用worlddraw不就可以了？  
其实不然，多一种方式在处理较为复杂的问题时会让代码逻辑变得简单。  
示例二：从坐标原点画一条线，当鼠标在原点右侧时，在鼠标位置多画一个直径为100的圆  
var line = new Line(Point3d.Origin, Point3d.Origin);  
using var jig=new JigEx((mpw, queue) =>  
{  
    line.EndPoint = mpw;  
    if (mpw.X > 0)  
    {  
        var circle = new Circle(mpw, Vector3d.ZAxis, 100);  
        queue.Enqueue(circle);  
    }  
});  
jig.DatabaseEntityDraw(worlddraw => worlddraw.Geometry.Draw(line));  
jig.SetOptions("\n选择下一点");  
  
  
效果：  
  
可以看出通过queue可以较为容易的控制圆的显示，方便用来控制未必会添加的图元，而必然会显示的直线通过worlddraw来控制，再通过下一节的JigExTransient来显示其他的图元，编写复杂的功能将更加得心应手。  
13. 瞬态JigExTransient的使用  
此类是一个瞬态容器，放进容器的图元，会显示在图纸中，从容器中取出，则从图纸中消失，并且不用借助事务。用于临时显示一些图元，可配合Jig一起使用。  
  
使用演示如下：  
// 创建瞬态容器  
using var jet = new re();  
  
// new一个圆，并加入到瞬态容器中  
var circle= new Circle(Point3d.Origin,Vector3d.ZAxis,100);  
jet.Add(circle);  
  
// GetPoint仅用于暂停查看效果，在选择点时，坐标圆点已经显示了刚加到瞬态容器中的圆  
Env.Editor.GetPoint("\n选择点");  
  
// 对图元进行修改后Update，可更新图元的显示  
circle.Center = new Point3d(1000,0,0);  
circle.ColorIndex = 1;  
jet.Update(circle);  
  
var line = new Line(Point3d.Origin, new Point3d(200, 200, 0));  
jet.Add(line);  
  
// 在本次选择点时，圆已经修改了位置并更改了颜色，并且新增了一条线  
Env.Editor.GetPoint("\n选择点");  
  
// 顺态容器提供了属性用以拿到容器中所有的图元,方便进行操作  
// 但是注意，此Array为复制体，对其进行增删操作不会实际影响到瞬态容器  
Entity[] ents = jet.Entities;  
using var tr = new DBTrans();  
//tr.CurrentSpace.AddEntity(jet.Entities);  
tr.CurrentSpace.AddEntity(circle);  
// 瞬态容器会在Dispose的时候清空，未加入数据库的图元会清除显示  
注意：虽然没有经过事务，但仍然不能够多线程使用。  
另外，瞬态容器加入图元时，可设置TransientDrawingMode参数，使其达到亮显，置顶等效果。  
jet.Add(TransientDrawingMode.Highlight, circle);  
jet.Add(TransientDrawingMode.DirectTopmost, line);  
14. 自动加载和初始化的使用  
14.1 功能描述  
自动加载和初始化功能主要是为了完成自动写注册表和初始化两件事情二合一。如果你只想写注册表来自动加载或者只想写初始化代码，那么这个功能并不适合你。  
为什么要在把两个功能合并？  
因为cad的插件唯一能自动运行的方式就是初始化函数里的代码可以在CAD加载插件的时候自动执行。剩余的代码都需要通过命令来调用，那么为了自动写注册表完成自动加载，减少手动的麻烦，就只能在初始化函数里写注册表。所以这个功能是合并在一起的。  
注意：ifox提供了两套关于自动加载和初始化的API，其中简单版仅用于完成自动写注册表和初始化功能，功能版API可以完成其他的设置，并且采用特性的写法来支持多处初始化功能，因此可以按你的需求来选择不同的API来使用。  
14.2 简单版API  
提供了AutoLoad抽象类来完成自动加载和初始化功能，只要在需要初始化的类继承AutoLoad类，然后实现Initialize() 和 Terminate() 两个函数就可以了。特别强调的是，一个程序集里只能有一个类继承，不管是不是同一个命名空间。  
如果要将dll的目录加入支持文件目录，请在 Initialize 函数中调用Env.AppendSupportPath(CurrentDirectory.FullName);  
其他需要初始化执行的函数及设置都需要在 Initialize 函数中执行。  
14.3 功能版API  
声明: :简单版API与功能版API二选一，不能共存。  
模板文件内不含功能版API，使用功能版API需新建类AutoRegAssemEx。代码如下：  
/// <summary>  
/// 注册中心(自动执行接口):  
/// <para>  
/// 继承<see cref="AutoRegAssem"/>虚函数后才能使用<br/>  
/// 0x01 netload加载之后自动执行,写入启动注册表,下次就不需要netload了<br/>  
/// 0x02 反射调用<see cref="IFoxInitialize"/>特性和<see cref="IFoxAutoGo"/>接口<br/>  
/// 启动cad后的执行顺序为:<br/>  
/// 1:<see cref="AutoRegAssem"/>构造函数<br/>  
/// 2:<see cref="IFoxInitialize"/>特性..多个<br/>  
/// 3:<see cref="IFoxAutoGo"/>接口..多个<br/>  
/// 4:本类的构造函数<br/>  
/// <code>  
/// **** 警告 ****  
/// 如果不写一个 <see cref="CmdInit.AutoRegAssemEx"/> 储存这个对象,  
/// 而是直接写卸载命令在此,  
/// 第一次加载的时候会初始化完成,然后这个类生命就结束了,  
/// 第二次通过命令进入,会引发构造函数再次执行,留意构造函数的打印信息即可发现  
/// </code>  
/// </para>  
/// </summary>  
public class AutoRegAssemEx : AutoRegAssem  
{  
    //枚举值AutoRegConfig.All,默认包含去教育版功能；精简等特殊CAD版本，如果出现打印功能异常，请自行修改枚举值。  
    public AutoRegAssemEx() : base(AutoRegConfig.All)  
    {  
        CmdInit.AutoRegAssemEx = this;  
    }  
}  
public class CmdInit  
{  
    public static AutoRegAssemEx? AutoRegAssemEx;  
  
    /// 如果netload之后用 <see cref="IFoxRemoveReg"/> 删除注册表,  
    /// 由于不是也不能卸载dll,再netload是无法执行自动接口的,  
    /// 所以此时会产生无法再注册的问题...因此需要暴露此注册函数(硬来)  
    [CommandMethod(nameof(IFoxAddReg))]  
    public void IFoxAddReg()  
    {  
        Env.Printl($"加入注册表");  
  
        AutoRegAssemEx ??= new();  
        AutoRegAssemEx.RegApp();  
    }  
  
    /// <summary>  
    /// 卸载注册表信息  
    /// </summary>  
    [CommandMethod(nameof(IFoxRemoveReg))]  
    public void IFoxRemoveReg()  
    {  
        Env.Printl($"卸载注册表");  
  
        // 防止卸载两次,不然会报错的  
        AutoRegAssemEx?.UnRegApp();  
        AutoRegAssemEx = null;  
    }  
  
    [CommandMethod(nameof(Debugx))]  
    public void Debugx()  
    {  
        var flag = Environment.GetEnvironmentVariable("debugx", EnvironmentVariableTarget.User);  
        if (flag == null || flag == "0")  
        {  
            Environment.SetEnvironmentVariable("debugx", "1", EnvironmentVariableTarget.User);  
            Env.Printl($"vs输出 -- 已启用");  
        }  
        else  
        {  
            Environment.SetEnvironmentVariable("debugx", "0", EnvironmentVariableTarget.User);  
            Env.Printl($"vs输出 -- 已禁用");  
        }  
    }  
}  
自动执行特性，方法如下：  
/*  
 * 自动执行:特性  
 */  
public class Cmd_IFoxInitialize  
{  
    int TestInt = 0;  
  
    [IFoxInitialize]  
    public void Initialize()  
    {  
        Env.Printl($"开始自动执行,可以分开多个类和多个函数:{nameof(Cmd_IFoxInitialize)}.{nameof(Initialize)}+{TestInt}");  
    }  
  
    [IFoxInitialize]  
    public void Initialize2()  
    {  
        Env.Printl($"开始自动执行,可以分开多个类和多个函数,又一次测试:{nameof(Cmd_IFoxInitialize)}.{nameof(Initialize2)}");  
    }  
  
    //[IFoxInitialize(isInitialize: false)]  
    //public void Terminate()  
    //{  
    //    try  
    //    {  
    //        // 注意此时编辑器已经回收,所以此句引发错误  
    //        // 您可以写一些其他的释放动作,例如资源回收之类的  
    //        Env.Printl($"\n 结束自动执行 Terminate \r\n");  
    //        // 改用  
    //        Debugx.Printl($"\n 结束自动执行 Terminate \r\n");  
    //    }  
    //    catch (System.Exception e)  
    //    {  
    //        System.Windows.Forms.MessageBox.Show(e.Message);  
    //    }  
    //}  
  
    [IFoxInitialize]  
    public static void StaticInitialize()  
    {  
        Env.Printl($"开始自动执行,静态调用:{nameof(Cmd_IFoxInitialize)}.{nameof(StaticInitialize)}");  
    }  
}  
自动执行接口，方法如下：  
/*  
 * 自动执行:接口  
 */  
public class Cmd_IFoxInitializeInterface : IFoxAutoGo  
{  
    int TestInt = 0;  
    public Cmd_IFoxInitializeInterface()  
    {  
        Env.Printl($"开始自动执行,{nameof(IFoxAutoGo)}接口调用:{nameof(Cmd_IFoxInitializeInterface)}::{TestInt}");  
    }  
  
    public Sequence SequenceId()  
    {  
        return Sequence.Last;  
    }  
  
    public void Initialize()  
    {  
        Env.Printl($"开始自动执行,{nameof(IFoxAutoGo)}接口调用:{nameof(Initialize)}::{TestInt}");  
    }  
  
    public void Terminate()  
    {  
        Debugx.Printl($"开始自动执行,{nameof(IFoxAutoGo)}接口调用:{nameof(Terminate)}::{TestInt}");  
        //    try  
        //    {  
        //        // 注意此时编辑器已经回收,所以此句没用,并引发错误  
        //        Env.Printl($"结束自动执行 {nameof(Cmd_IFoxInitializeInterface)}.Terminate \r\n");  
        //    }  
        //    catch (System.Exception e)  
        //    {  
        //        System.Windows.Forms.MessageBox.Show(e.Message);  
        //    }  
    }  
}  
15. 选择集过滤器的使用  
15.1 选择集过滤器简介  
桌子提供了选择集过滤器是为了更精确的选择对象。可以通过使用选择过滤器来限制哪些对象被选中并添加到选择集，选择过滤器列表通过属性或类型过滤所选对象。  
在桌子的 .net api 中：选择过滤器由一对 TypedValue 参数构成。TypedValue 的第一个参数表明过滤器的类型（例如对象），第二个参数为要过滤的值（例如圆）。过滤器类型是一个 DXF 组码，用来指定使用哪种过滤器。  
默认的使用桌子api来创建选择集（带过滤器）分三步：  
创建一个TypedValue数组来定义过滤器条件  
  
   TypedValue[] acTypValAr = new TypedValue[1]; // 创建数组  
   acTypValAr.SetValue(new TypedValue((int)DxfCode.Start, "CIRCLE"), 0);   
   // 添加一个过滤条件，例如选择圆  
     
   // 如果要创建多个过滤条件怎么办？  
   TypedValue[] acTypValAr = new TypedValue[3];  
   acTypValAr.SetValue(new TypedValue((int)DxfCode.Color, 5), 0);  
   acTypValAr.SetValue(new TypedValue((int)DxfCode.Start, "CIRCLE"), 1);  
   acTypValAr.SetValue(new TypedValue((int)DxfCode.LayerName, "0"), 2);  
   // 实际上只要不停的往数组里添加条件就可以了  
创建SelectionFilter对象  
   // 将过滤器条件赋值给 SelectionFilter 对象  
   SelectionFilter acSelFtr = new SelectionFilter(acTypValAr);  
创建选择集  
   // 请求用户在图形区域选择对象  
   PromptSelectionResult acSSPrompt = acDocEd.GetSelection(acSelFtr);  
看起来很是简单对不对，单个条件和多个条件的过滤非常简单。当指定多个选择条件时，AutoCAD 假设所选对象必须满足每个条件。我们还可以用另外一种方式定义过滤条件。对于数值项，可以使用关系运算（比如，圆的半径必须大于等于 5.0）。对于所有项，可以使用逻辑运算（比如单行文字或多行文字）。使用 DXF 组码-4 或常量 DxfCode.Operator 表示选择过滤器中的关系运算符类型。运算符本身用字符串表示。  
比如：  
1. 过滤半径大于等于5.0的圆  
   TypedValue[] acTypValAr = {  
         new TypedValue((int)DxfCode.Start, "CIRCLE"),  
         new TypedValue((int)DxfCode.Operator, ">="),   
         new TypedValue(40, 5)  
   };  
2. 过滤单行文本或者多行文本  
   TypedValue[] acTypValAr = {  
         new TypedValue((int)DxfCode.Operator, "<or"),  
         new TypedValue((int)DxfCode.Start, "TEXT"),  
         new TypedValue((int)DxfCode.Start, "MTEXT"),  
         new TypedValue((int)DxfCode.Operator, "or>")  
   };  
3. 更复杂的过滤条件呢？比如选择的对象为不是位于0图层的直线，或者位于2图层的组码10的x坐标>10,y坐标>10的非圆图元。  
 对应的lisp代码如下：  
'((-4 . "<or")  
     (-4 . "<not")  
         (-4 . "<and")  
             (0 . "line")  
             (8 . "0")  
         (-4 . "and>")  
     (-4 . "not>")  
     (-4 . "<and")  
         (-4 . "<not")(0 . "circle")(-4 . "not>")  
         (8 . "2")  
         (-4 . ">,>,*")(10 10 10 0)  
   (-4 . "and>")  
(-4 . "or>"))    
   对应的c#代码：  
   TypedValue[] acTypValAr = {  
         new TypedValue((int)DxfCode.Operator, "<or"),  
                 new TypedValue((int)DxfCode.Operator, "<not"),  
                     new TypedValue((int)DxfCode.Operator, "<and"),  
                         new TypedValue((int)DxfCode.Start, "LINE"),  
                         new TypedValue((int)DxfCode.LayerName, "0"),  
                     new TypedValue((int)DxfCode.Operator, "and>"),  
                 new TypedValue((int)DxfCode.Operator, "not>"),  
                 new TypedValue((int)DxfCode.Operator, "<and"),  
                     new TypedValue((int)DxfCode.Operator, "<not"),  
                         new TypedValue((int)DxfCode.Start, "CIRCLE"),  
                     new TypedValue((int)DxfCode.Operator, "not>"),  
                     new TypedValue((int)DxfCode.LayerName, "2"),  
                     new TypedValue((int)DxfCode.Operator, ">,>,*"),  
                     new TypedValue(10, new Point3d(10,10,0)),  
                 new TypedValue((int)DxfCode.Operator, "and>"),  
         new TypedValue((int)DxfCode.Operator, "or>")  
   };  
这个过滤器是不是看起来很乱，一眼看去根本不知道是要过滤什么，写起来也很麻烦。所以说，虽然桌子提供了api，但是简单的过滤条件很好用，但是复杂的过滤条件就很复杂了。  
因此IFox类库提供了关于选择集过滤器的辅助类来帮助用户用更简单的方式来创建选择集的过滤器。  
4.原生api极简写法  
      var f = new SelectionFilter([  
           new (-4, "<or"),  
           new (-4, "<not"),  
           new (-4, "<and"),  
           new (0, "LINE"),  
           new (8, "0"),  
           new (-4, "and>"),  
           new (-4, "not>"),  
           new (-4, "<and"),  
           new (-4, "<not"),  
           new (0, "CIRCLE"),  
           new (-4, "not>"),  
           new (8, "2"),  
           new (-4, ">,>,*"),  
           new (10, new Point3d(10,10,0)),  
           new (-4, "and>"),  
           new (-4, "or>")  
   ]);  
     var ss = Env.Editor.SelectAll(f);  
    Env.Editor.SetImpliedSelection(ss.Value);  
15.2 类库过滤器对象与cad过滤器对应关系  
IFoxCad类库对于DxfCode.Operator枚举构建了一些辅助函数来表达关系运算和逻辑运算；提供了dxf函数来表达组码。其对应的关系如下表：  
15.3 具体用法  
IFoxCad类库提供了三种方式来构建过滤器，其实大同小异，就是写法不一样，用户可以根据自己的喜好来选择。  
第一种  
  var fd =  
      new OpOr    //定义一个 (-4 . "<or")(...)(-4 . "or>")  
      {  
          !new OpAnd //定义(-4 . "<not")(-4 . "<and")(...)(-4 . "and>")(-4 . "not>")  
          {  
              { 0, "line" }, //{组码，组码值}  
              { 8, "0" }, //{组码，组码值}  
          },  
          new OpAnd //定义(-4 . "<and")(...)(-4 . "and>")  
          {  
              !new OpEqual(0, "circle"), //定义(-4 . "<not")(...)(-4 . "not>")  
              { 8, "2" }, //{组码，组码值}  
              { 10, new Point3d(10,10,0), ">,>,*" }  //(-4 . ">,>,*")(10 10 10 0)  
          },  
      };  
  editor.SelectAll(fd); //这里直接传入fd就可以了  
    以上代码的含义为：选择的对象为不是位于0图层的直线，或者位于2图层的组码10的x坐标>10,y坐标>10的非圆图元。其同含义的lisp代码如下：  
  '((-4 . "<or")  
        (-4 . "<not")  
            (-4 . "<and")  
                (0 . "line")  
                (8 . "0")  
            (-4 . "and>")  
        (-4 . "not>")  
        (-4 . "<and")  
            (-4 . "<not")(0 . "circle")(-4 . "not>")  
            (8 . "2")  
            (-4 . ">,>,*")(10 10 10 0)  
        (-4 . "and>")  
    (-4 . "or>"))  
第二种  
  
  var f = OpFilter.Build(e =>   
              !(e.Dxf(0) == "line" & e.Dxf(8) == "0")   
              | (e.Dxf(0) != "circle" & e.Dxf(8) == "2" & e.Dxf(10) >= new Point3d(10, 10, 0)));  
  editor.SelectAll(f); //这里直接传入f就可以了  
  代码含义如第一种。  
第三种  
  var f2 = OpFilter.Build(  
    e =>e.Or(  
      !e.And(e.Dxf(0) == "line", e.Dxf(8) == "0"),  
      e.And(e.Dxf(0) != "circle", e.Dxf(8) == "2", e.Dxf(10) >= new Point3d(10, 10, 0)))  
  );  
  editor.SelectAll(f2); //这里直接传入f2就可以了  
  代码含义如第一种，第三种和第二种的写法非常像，区别就是关于 and 、or 、not 等运算符，一个是采用C#的语法，一个是采用定义的函数。and 与&等价，or与|等价，not 与！等价。  
15.4 选择集对象类SelectedObject  
选择集对象类是存储选择集的信息,存储有选择集的选择方式,选择点位置,等...  
15.5 带关键字的选择  
  [CommandMethod(nameof(Test_Ssget))]  
  public void Test_Ssget()  
  {  
      var action_a = () => { Env.Print("this is a"); };  
      var action_b = () => { Env.Print("this is b"); };  
  
      var keyword = new Dictionary<string, (string, Action)>  
          {  
              { "A", ("A",action_a) },  
              { "B", ("B",action_b) }  
          };  
  
      var ss = Env.Editor.SSGet(mode:":S", messages: ("get", "del"),  
                                       keywords: keyword);  
      Env.Print(ss!);  
  }  
  
  
mode  
避开系统关键字  
窗口(W)/上一个(L)/窗交(C)/框(BOX)/全部(ALL)/栏选(F)/圈围(WP)/圈交(CP)/编组(G)/添加(A)/删除(R)/多个(M)/前一个(P)/放弃(U)/自动(AU)/单个(SI)  
  
  
  
15.6 选择集夹取状态和清除选择集夹取状态  
夹取选择集  
Env.Editor.SetImpliedSelection(obidCollection);  
清除选择集夹取  
ObjectId[] idarrayEmpty = new ObjectId[0];  
Env.Editor.SetImpliedSelection(idarrayEmpty);  
实体高亮  
Entity e;  
e.Highlight();  
16. ResultBuffer的使用  
这是什么，来个大佬完善一下  
使用ResultBuffer构建Lisp的list表与点表  
//'("IFoxCad" '("一代目" . "狐哥") '("二代目" . "山人") '("三代目" . "惊惊") '("四代目" . "DYH小白"))  
var rbArgs = new ResultBuffer  
{  
    new TypedValue((int)LispDataType.ListBegin),  
    new TypedValue((int)LispDataType.Text, "IFoxCad"),  
  
    new TypedValue((int)LispDataType.ListBegin),  
    new TypedValue((int)LispDataType.Text, "一代目"),  
    new TypedValue((int)LispDataType.Text, "狐哥"),  
    new TypedValue((int)LispDataType.DottedPair),  
  
    new TypedValue((int)LispDataType.ListBegin),  
    new TypedValue((int)LispDataType.Text, "二代目"),  
    new TypedValue((int)LispDataType.Text, "山人"),  
    new TypedValue((int)LispDataType.DottedPair),  
  
    new TypedValue((int)LispDataType.ListBegin),  
    new TypedValue((int)LispDataType.Text, "三代目"),  
    new TypedValue((int)LispDataType.Text, "惊惊"),  
    new TypedValue((int)LispDataType.DottedPair),  
  
    new TypedValue((int)LispDataType.ListBegin),  
    new TypedValue((int)LispDataType.Text, "四代目"),  
    new TypedValue((int)LispDataType.Text, "DYH小白"),  
    new TypedValue((int)LispDataType.DottedPair),  
  
    new TypedValue((int)LispDataType.ListEnd)  
};  
ResultBuffer相关的常量值，如常用的5005代表string，5006代表objectId  
namespace Autodesk.AutoCAD.Runtime  
{  
    public enum LispDataType  
    {  
        None = 5000,  
        Double = 5001,  
        Point2d = 5002,  
        Int16 = 5003,  
        Angle = 5004,  
        Text = 5005,  
        ObjectId = 5006,  
        SelectionSet = 5007,  
        Orientation = 5008,  
        Point3d = 5009,  
        Int32 = 5010,  
        Void = 5014,  
        ListBegin = 5016,  
        ListEnd = 5017,  
        DottedPair = 5018,  
        Nil = 5019,  
        T_atom = 5021  
    }  
}  
结果缓存——ResultBuffer  
　　结果缓存即 Autodesk.AutoCAD.DatabaseServices.ResultBuffer 类型，使用 ResultBuffer 对象时需要提供一个数据对，每个数据对包含一个数据类型描述和一个值，这些数据对是 Autodesk.AutoCAD.DatabaseServices.TypedValue 类的实例。  
　　TypedValue.TypeCode 属性是一个16位整型数据，它指明 TypedValue.Value 属性的数据类型，可接受的 TypeCode 值取决于 ResultBuffer 实例的使用范围，例如，适用于扩展记录定义的 TypeCode 值就不适合于 XData。而Autodesk.AutoCAD.DatabaseServices.DxfCode 枚举类型定义的码值则描述了 ResultBuffer 可能的数据类型。  
　　TypedValue.Value 属性是一个 System.Object 的实例，它可以包含任何类型的数据；但是，Value 的数据必须符合由 TypeCode 指明的类型。  
　　创建 ResultBuffer 方法有两种：  
　　一种是使用构造函数创建，即在声明 ResultBuffer 时将一个 TypedValue 作用参数传给 ResultBuffer：  
ResultBuffer  resBuf = new ResultBuffer(new TypedValue((int)DxfCode.Text, "我的扩展数据"));  
　　另一种是使用 ResultBuffer.Add() 方法来添加 TypedValue，可以添加多个TypedValue，但总数据大小不能超过128K：  
ResultBuffer resBuf = new ResultBuffer ();   
resBuf.Add(new TypedValue ((int)DxfCode.Text, "我的扩展数据"));   
resBuf.Add(new TypedValue ((int)DxfCode.Real, 20.0));   
=====================================  
ResulrBuffer扩展类分为三大类分别对应扩展字典/数据、Lisp数据和选择集过滤器  
其中  
XdataList类对应扩展字典/数据  
Lisp*类对应Lisp数据  
Op*类对应选择集过滤器  
测试代码：  
        [CommandMethod("tt1")]  
        public void Test1()  
        {  
            //扩展数据  
            XDataList lst =      new XDataList  
                {  
                    { 1001, "myapp" },  
                    { 1000, "hello" }  
                };  
  
            //过滤器的三种写法  
            var fd =  
                new OpOr  
                {  
                        !new OpAnd  
                        {  
                            { 0, "line" },  
                            { 8, "0" },  
                        },  
                        new OpAnd  
                        {  
                            !new OpEqual(0, "circle"),  
                            { 8, "2" },  
                            { 10, new Point3d(10,10,0), ">,>,*" }  
                        },  
                };  
  
            var p = new Point3d(10, 10, 0);  
  
            var f =  
                OpFilter.Build(  
                    e =>  
                        !(e.Dxf(0) == "line" & e.Dxf(8) == "0") |  
                        e.Dxf(0) != "circle" & e.Dxf(8) == "2" & e.Dxf(10) >= p  
                    );  
  
            var f2 =  
                OpFilter.Build(  
                    e =>  
                        e.Or(  
                        !e.And(e.Dxf(0) == "line", e.Dxf(8) == "0"),  
                        e.And(e.Dxf(0) != "circle", e.Dxf(8) == "2", e.Dxf(10) <= new Point3d(10, 10, 0)))  
                    );  
  
            Env.Print($"\nXdataList:{lst}\n{fd}\n{f}\n{f2}");  
  
        }  
  
        [LispFunction("mytt")]  
        public object LispTest(ResultBuffer rb)  
        {  
            LispList? lisplist1 = new LispList();  
            //隐式类型转换  ResultBuffer to LispList  
            lisplist1 = rb;  
  
            ResultBuffer rb2 = new ResultBuffer();  
            //隐式类型转换 LispList to ResultBuffer  
            rb2 = lisplist1;  
  
            return rb2;  
        }  
  
        [LispFunction("mytt2")]  
        public object LispTest2(ResultBuffer rb)  
        {  
            var lisplist1 =  
                new LispList  
                {  
                    0,  
                    new LispDottedPair{ 12, 13 }  
                };  
  
            ResultBuffer rb2 = new ResultBuffer();  
            //隐式类型转换 LispList to ResultBuffer  
            rb2 = lisplist1;  
  
            return rb2;  
        }  
命令: TT1  
XdataList:((1001,myapp)(1000,hello))  
(-4,<Or)(-4,<Not)(-4,<And)(0,line)(8,0)(-4,And>)(-4,Not>)(-4,<And)(-4,<Not)(0,circle)(-4,Not>)(8,2)(-4,>,>,*)(10,(10,10,0))(-4,And>)(-4,Or>)  
(-4,<Or)(-4,<Not)(-4,<And)(0,line)(8,0)(-4,And>)(-4,Not>)(-4,<And)(-4,<Not)(0,circle)(-4,Not>)(8,2)(-4,>,>,*)(10,(10,10,0))(-4,And>)(-4,Or>)  
(-4,<Or)(-4,<Not)(-4,<And)(0,line)(8,0)(-4,And>)(-4,Not>)(-4,<And)(-4,<Not)(0,circle)(-4,Not>)(8,2)(-4,<,<,*)(10,(10,10,0))(-4,And>)(-4,Or>)  
  
命令: (mytt2)  
(0 (12 . 13))  
  
命令: (mytt 1 '(2 . 3))  
(1 (2 . 3))  
  
  
  
  
17. 扩展数据、扩展记录、XDataList的使用  
17.1 扩展数据  
增  
// 测试扩展数据  
//要修改的appname  
static readonly string _appname = "myapp2";  
// 增  
[CommandMethod(nameof(Test_AddXdata))]  
public void Test_AddXdata()  
{  
    using DBTrans tr = new();  
    var appname = "myapp2";  
  
    tr.RegAppTable.Add("myapp1");  
    tr.RegAppTable.Add(appname); // add函数会默认的在存在这个名字的时候返回这个名字的regapp的id，不存在就新建  
    tr.RegAppTable.Add("myapp3");  
    tr.RegAppTable.Add("myapp4");  
  
    var line = new Line(new Point3d(0, 0, 0), new Point3d(1, 1, 0))  
    {  
        XData = new XDataList()  
            {  
                { DxfCode.ExtendedDataRegAppName, "myapp1" },  // 可以用dxfcode和int表示组码  
                { DxfCode.ExtendedDataAsciiString, "xxxxxxx" },  
                {1070, 12 },  
                { DxfCode.ExtendedDataRegAppName, appname },  // 可以用dxfcode和int表示组码,移除中间的测试  
                { DxfCode.ExtendedDataAsciiString, "要移除的我" },  
                {1070, 12 },  
                { DxfCode.ExtendedDataRegAppName, "myapp3" },  // 可以用dxfcode和int表示组码  
                { DxfCode.ExtendedDataAsciiString, "aaaaaaaaa" },  
                {1070, 12 },  
                { DxfCode.ExtendedDataRegAppName, "myapp4" },  // 可以用dxfcode和int表示组码  
                { DxfCode.ExtendedDataAsciiString, "bbbbbbbbb" },  
                {1070, 12 }  
            }  
    };  
    tr.CurrentSpace.AddEntity(line);  
}  
删  
[CommandMethod(nameof(Test_RemoveXdata))]  
public void Test_RemoveXdata()  
{  
    var res = Env.Editor.GetEntity("\n select the entity:");  
    if (res.Status == PromptStatus.OK)  
    {  
        using DBTrans tr = new();  
        var ent = tr.GetObject<Entity>(res.ObjectId);  
        if (ent == null || ent.XData == null)  
            return;  
  
        Env.Printl("\n移除前:" + ent.XData.ToString());  
  
        ent.RemoveXData(_appname, DxfCode.ExtendedDataAsciiString);  
        Env.Printl("\n移除成员后:" + ent.XData.ToString());  
  
        ent.RemoveXData(_appname);  
        Env.Printl("\n移除appName后:" + ent.XData.ToString());  
    }  
}  
改  
// 改  
[CommandMethod(nameof(Test_ChangeXdata))]  
public void Test_ChangeXdata()  
{  
    var res = Env.Editor.GetEntity("\n select the entity:");  
    if (res.Status != PromptStatus.OK)  
        return;  
    using DBTrans tr = new();  
    var data = tr.GetObject<Entity>(res.ObjectId)!;  
    data.ChangeXData(_appname, DxfCode.ExtendedDataAsciiString, "change");  
  
      
    Env.Printl(data.XData.ToString());  
}  
查  
// 查  
[CommandMethod(nameof(Test_GetXdata))]  
public void Test_GetXdata()  
{  
    using DBTrans tr = new();  
    tr.RegAppTable.ForEach(id =>  
        id.GetObject<RegAppTableRecord>()?.Name.Print());  
    tr.RegAppTable.GetRecords().ForEach(rec => rec.Name.Print());  
    tr.RegAppTable.GetRecordNames().ForEach(name => name.Print());  
    tr.RegAppTable.ForEach(reg => reg.Name.Print(), checkIdOk: false);  
  
    // var res = ed.GetEntity("\n select the entity:");  
    // if (res.Status == PromptStatus.OK)  
    // {  
    //    using DBTrans tr = new();  
    //    tr.RegAppTable.ForEach(id => id.GetObject<RegAppTableRecord>().Print());  
    //    var data = tr.GetObject<Entity>(res.ObjectId).XData;  
    //    ed.WriteMessage(data.ToString());  
    // }  
  
    // 查询appName里面是否含有某个  
    var res = Env.Editor.GetEntity("\n select the entity:");  
    if (res.Status == PromptStatus.OK)  
    {  
        var ent = tr.GetObject<Entity>(res.ObjectId);  
  
  
        XDataList data = ent.XData;  
        if (data.Contains(_appname))  
            Env.Printl("含有appName:" + _appname);  
        else  
            Env.Printl("不含有appName:" + _appname);  
  
        var str = "要移除的我";  
        if (data.Contains(_appname, str))  
            Env.Printl("含有内容:" + str);  
        else  
            Env.Printl("不含有内容:" + str);  
    }  
}  
17.2 扩展记录  
#define XTextString  
#if NewtonsoftJson  
using System.Diagnostics;  
using Newtonsoft.Json;  
using static IFoxCAD.Cad.WindowsAPI;  
  
namespace Test_XRecord;  
  
public class TestCmd_XRecord  
{  
    [CommandMethod(nameof(TestSerializeSetXRecord))]  
    public void TestSerializeSetXRecord()  
    {  
        var prs = Env.Editor.SSGet("\n 序列化,选择多段线:");  
        if (prs.Status != PromptStatus.OK)  
            return;  
  
        using var tr = new DBTrans();  
        var pls = prs.Value.GetEntities<Polyline>();  
        Tools.TestTimes(1, nameof(TestSerializeSetXRecord), () => {  
            foreach (var pl in pls)  
            {  
                if (pl == null)  
                    continue;  
  
                TestABCList datas = new();  
                for (int i = 0; i < 1000; i++)  
                {  
                    datas.Add(new()  
                    {  
                        AAA = i,  
                        BBB = i.ToString(),  
                        CCCC = i * 0.5,  
                        DDDD = i % 2 != 0,  
                        EEEE = new(0, i, 0)  
                    });  
                }  
  
                using (pl.ForWrite())  
                    pl.SerializeToXRecord(datas);  
            }  
        });  
    }  
  
    [CommandMethod(nameof(TestDeserializeGetXRecord))]  
    public void TestDeserializeGetXRecord()  
    {  
        var prs = Env.Editor.GetEntity("\n 反序列化,选择多段线:");  
        if (prs.Status != PromptStatus.OK)  
            return;  
  
        using var tr = new DBTrans();  
  
        TestABCList? datas = null;  
        Tools.TestTimes(1, nameof(TestDeserializeGetXRecord), () => {  
            var pl = prs.ObjectId.GetObject<Entity>();  
            if (pl == null)  
                return;  
#if XTextString  
            datas = pl.DeserializeToXRecord<TestABCList>();  
#endif  
  
#if ExtendedDataBinaryChunk  
            // 这里有数据容量限制,而且很小  
            var xd = pl.GetXDictionary();  
            if (xd == null)  
                return;  
            if (xd.XData == null)  
                return;  
  
            XDataList data = xd.XData;  
            var sb = new StringBuilder();  
            data.ForEach(a => {  
                if (a.TypeCode == (short)DxfCode.ExtendedDataBinaryChunk)  
                    if (a.Value is byte[] bytes)  
                        sb.Append(Encoding.UTF8.GetString(bytes));  
            });  
            datas = JsonConvert.DeserializeObject<TestABCList>(sb.ToString(), XRecordHelper._sset);  
#endif  
        });  
        if (datas == null)  
        {  
            Env.Printl("没有反序列的东西");  
            return;  
        }  
  
        var sb = new StringBuilder();  
        for (int i = 0; i < datas.Count; i++)  
            sb.Append(datas[i]);  
        Env.Printl(sb);  
    }  
}  
  
public static class XRecordHelper  
{  
    #region 序列化方式  
    internal static JsonSerializerSettings _sset = new()  
    {  
        Formatting = Formatting.Indented,  
        TypeNameHandling = TypeNameHandling.Auto  
    };  
  
    /// <summary>  
    /// 设定信息  
    /// </summary>  
    /// <param name="dbo">储存对象</param>  
    /// <param name="data">储存数据</param>  
    public static void SerializeToXRecord<T>(this DBObject dbo, T data)  
    {  
        var xd = dbo.GetXDictionary();  
        if (xd == null)  
            return;  
  
        // XRecordDataList 不能超过2G大小  
        const int GigaByte2 = 2147483647;  
        // 单条只能 16KiBit => 2048 * 16 == 32768  
        const int KiBit16 = (2048 * 16) - 1;  
  
        // 就算这个写法支持,计算机也不一定有那么多内存,所以遇到此情况最好换成内存拷贝  
        var json = JsonConvert.SerializeObject(data, _sset);// 此时内存占用了2G  
        var buffer = Encoding.UTF8.GetBytes(json);          // 此时内存又占用了2G..  
  
        BytesTask(buffer, GigaByte2, bts => {  
#if XTextString  
            XRecordDataList datas = new();  
            BytesTask(buffer, KiBit16, bts => {  
                datas.Add(DxfCode.XTextString, Encoding.UTF8.GetString(bts)); // 这对的  
                // datas.Add(DxfCode.XTextString, bts);//这样 bts 变成 "System.Byte[]"  
            });  
            xd.SetXRecord(typeof(T).FullName, datas);  
#endif  
  
#if ExtendedDataBinaryChunk  
            // 这里有数据容量限制,而且很小  
            var appname = typeof(T).FullName;  
            DBTrans.Top.RegAppTable.Add(appname);  
  
            XDataList datas = new();  
            datas.Add(DxfCode.ExtendedDataRegAppName, appname);  
            BytesTask(buffer, KiBit16, bts => {  
                datas.Add(DxfCode.ExtendedDataBinaryChunk, bts);  
            });  
            using (xd.ForWrite())  
                xd.XData = datas; // Autodesk.AutoCAD.Runtime.Exception:“eXdataSizeExceeded”  
#endif  
        });  
    }  
  
    [DebuggerHidden]  
    static void BytesTask(byte[] buffer, int max, Action<byte[]> action)  
    {  
        int index = 0;  
        while (index < buffer.Length)  
        {  
            // 每次 max,然后末尾剩余就单独  
            byte[] bts;  
            if (buffer.Length - index > max)  
                bts = new byte[max];  
            else  
                bts = new byte[buffer.Length - index];  
  
            for (int i = 0; i < bts.Length; i++)  
                bts[i] = buffer[index++];  
  
            action.Invoke(bts);  
        }  
    }  
  
    /// <summary>  
    /// 提取信息  
    /// </summary>  
    /// <typeparam name="T">类型</typeparam>  
    /// <param name="dbo">储存对象</param>  
    /// <returns>提取数据生成的对象</returns>  
    public static T? DeserializeToXRecord<T>(this DBObject dbo)  
    {  
        var xd = dbo.GetXDictionary();  
        if (xd == null)  
            return default;  
  
        var datas = xd.GetXRecord(typeof(T).FullName);  
        if (datas == null)  
            return default;  
  
        var sb = new StringBuilder();  
        for (int i = 0; i < datas.Count; i++)  
            sb.Append(datas[i].Value);  
  
        return JsonConvert.DeserializeObject<T>(sb.ToString(), _sset);  
    }  
    #endregion  
  
#if NET35  
    /// <summary>  
    /// 设置描述(容量无限)  
    /// </summary>  
    /// <param name="db"></param>  
    /// <param name="key"></param>  
    /// <param name="value"></param>  
    public static void SetSummaryInfoAtt(this Database db, string key, string value)  
    {  
        var info = new DatabaseSummaryInfoBuilder(db.SummaryInfo);  
        if (!info.CustomProperties.ContainsKey(key))  
            info.CustomProperties.Add(key, value);  
        else  
            info.CustomProperties[key] = value;  
        db.SummaryInfo = info.ToDatabaseSummaryInfo();  
    }  
    /// <summary>  
    /// 获取描述  
    /// </summary>  
    /// <param name="db"></param>  
    /// <param name="key"></param>  
    /// <param name="value"></param>  
    public static object? GetSummaryInfoAtt(this Database db, string key)  
    {  
        var info = new DatabaseSummaryInfoBuilder(db.SummaryInfo);  
        if (info.CustomProperties.ContainsKey(key))  
            return info.CustomProperties[key];  
        return null;  
    }  
#else  
    /// <summary>  
    /// 设置描述(容量无限)  
    /// </summary>  
    /// <param name="db"></param>  
    /// <param name="key"></param>  
    /// <param name="value"></param>  
    public static void SetSummaryInfoAtt(this Database db, string key, object value)  
    {  
        var info = new DatabaseSummaryInfoBuilder(db.SummaryInfo);  
        if (!info.CustomPropertyTable.Contains(key))  
            info.CustomPropertyTable.Add(key, value);  
        else  
            info.CustomPropertyTable[key] = value;  
        db.SummaryInfo = info.ToDatabaseSummaryInfo();  
    }  
    /// <summary>  
    /// 获取描述  
    /// </summary>  
    /// <param name="db"></param>  
    /// <param name="key"></param>  
    /// <param name="value"></param>  
    public static object? GetSummaryInfoAtt(this Database db, string key)  
    {  
        var info = new DatabaseSummaryInfoBuilder(db.SummaryInfo);  
        if (info.CustomPropertyTable.Contains(key))  
            return info.CustomPropertyTable[key];  
        return null;  
    }  
#endif  
}  
  
public class TestABCList : List<TestABC>  
{  
  
}  
  
[ComVisible(true)]  
[Serializable]  
[DebuggerDisplay("{DebuggerDisplay,nq}")]  
[DebuggerTypeProxy(typeof(TestABC))]  
[StructLayout(LayoutKind.Sequential, Pack = 4)]  
public class TestABC  
{  
    [DebuggerBrowsable(DebuggerBrowsableState.Never)]  
    private string DebuggerDisplay => ToString();  
  
    public int AAA;  
    public string? BBB;  
    public double CCCC;  
    public bool DDDD;  
    public Point3D EEEE;  
  
    public override string ToString()  
    {  
        return JsonConvert.SerializeObject(this, XRecordHelper._sset);  
    }  
}  
#endif  
  
18. 四叉树的使用  
namespace Test;  
  
#pragma warning disable CS8632 // 只能在 "#nullable" 注释上下文内的代码中使用可为 null 的引用类型的注释。  
/*  
 * 这里属于用户调用例子,  
 * 调用时候必须要继承它,再提供给四叉树  
 * 主要是用户可以扩展属性  
 */  
public class CadEntity : QuadEntity  
{  
    public ObjectId ObjectId;  
    // 这里加入其他字段  
    public List<QuadEntity>? Link;// 碰撞链  
    public System.Drawing.Color Color;  
    public double Angle;  
    public CadEntity(ObjectId objectId, Rect box) : base(box)  
    {  
        ObjectId = objectId;  
    }  
    public int CompareTo(CadEntity? other)  
    {  
        if (other == null)  
            return -1;  
        return GetHashCode() ^ other.GetHashCode();  
    }  
    public override int GetHashCode()  
    {  
        return (base.GetHashCode(), ObjectId.GetHashCode()).GetHashCode();  
    }  
}  
#pragma warning restore CS8632 // 只能在 "#nullable" 注释上下文内的代码中使用可为 null 的引用类型的注释  
  
public partial class TestQuadTree  
{  
    QuadTree<CadEntity>? _quadTreeRoot;  
    #region 四叉树创建并加入  
    [CommandMethod(nameof(Test_QuadTree))]  
    public void Test_QuadTree()  
    {  
        using DBTrans tr = new();  
  
        Rect dbExt;  
        // 使用数据库边界来进行  
        var dbExtent = tr.Database.GetValidExtents3d();  
        if (dbExtent == null)  
        {  
            // throw new ArgumentException("画一个矩形");  
  
            // 这个初始值的矩形是很有意义,  
            // 主要是四叉树分裂过程中产生多个Rect,Rect内有很多重复的double值,是否可以内存复用,以此减少内存大小?  
            // 接着想了一下,Rect可以是int,long,这样可以利用位运算它扩展和缩小,  
            // 最小就是1,并且可以控制四叉树深度,不至于无限递归.  
            // 而且指针长度跟值是一样的,所以就不需要复用了,毕竟跳转一个函数地址挺麻烦的.  
            // 但是因为啊惊懒的原因,并没有单独制作这样的矩形,  
            // 而且非常糟糕的是,c#不支持模板约束运算符,使得值类型之间需要通过一层接口来委婉处理,拉低了效率..引用类型倒是无所谓..  
            // 要么忍着,要么换c++去搞四叉树吧  
            dbExt = new Rect(0, 0, 1 << 10, 1 << 10);  
        }  
        else  
        {  
            var a = new Point2d(dbExtent.Value.MinPoint.X, dbExtent.Value.MinPoint.Y);  
            var b = new Point2d(dbExtent.Value.MaxPoint.X, dbExtent.Value.MaxPoint.Y);  
            dbExt = new Rect(a, b);  
        }  
  
        // 创建四叉树  
        _quadTreeRoot = new QuadTree<CadEntity>(dbExt);  
  
        // 数据库边界  
        var pl = dbExt.ToPoints();  
        var databaseBoundary = new List<(Point3d, double, double, double)>  
            {  
                (new Point3d(pl[0].X,pl[0].Y,0),0,0,0),  
                (new Point3d(pl[1].X,pl[1].Y,0),0,0,0),  
                (new Point3d(pl[2].X,pl[2].Y,0),0,0,0),  
                (new Point3d(pl[3].X,pl[3].Y,0),0,0,0),  
            };  
        tr.CurrentSpace.AddPline(databaseBoundary);  
  
        // 生成多少个图元,导致cad会令undo出错(八叉树深度过大 treemax)  
        // int maximumItems = 30_0000;  
        int maximumItems = 1000;  
  
        // 随机图元生成  
        List<CadEntity> ces = new();  // 用于随机获取图元  
        Tools.TestTimes(1, "画圆消耗时间:", () => {  
            // 生成外边界和随机圆形  
            var grc = GenerateRandomCircle(maximumItems, dbExt);  
            foreach (var ent in grc)  
            {  
                // 初始化图元颜色  
                ent!.ColorIndex = 1; // Color.FromRgb(0, 0, 0);// 黑色  
                var edge = ent.GeometricExtents;  
                // 四叉树数据  
                var entRect = new Rect(edge.MinPoint.X, edge.MinPoint.Y, edge.MaxPoint.X, edge.MaxPoint.Y);  
                var entId = tr.CurrentSpace.AddEntity(ent);  
  
                var ce = new CadEntity(entId, entRect)  
                {  
                    Color = RandomEx.NextColor()  
                };  
                ces.Add(ce);  
                /*加入随机点*/  
                var p = edge.MinPoint + new Vector3d(10, 10, 0);  
                entRect = new Rect(p.Point2d(), p.Point2d());  
                entId = tr.CurrentSpace.AddEntity(new DBPoint(p));  
                var dbPointCe = new CadEntity(entId, entRect);  
                ces.Add(dbPointCe);  
            }  
        });// 30万图元±3秒.cad2021  
  
        // 测试只加入四叉树的时间  
        Tools.TestTimes(1, "插入四叉树时间:", () => {  
            for (int i = 0; i < ces.Count; i++)  
                _quadTreeRoot.Insert(ces[i]);  
        });// 30万图元±0.7秒.cad2021  
  
        tr.Editor?.WriteMessage($"\n加入图元数量:{maximumItems}");  
    }  
  
    /// <summary>  
    /// 创建随机圆形  
    /// </summary>  
    /// <param name="createNumber">创建数量</param>  
    /// <param name="dbExt">数据库边界</param>  
    static IEnumerable<Entity?> GenerateRandomCircle(int createNumber, Rect dbExt)  
    {  
        var x1 = (int)dbExt.X;  
        var x2 = (int)(dbExt.X + dbExt.Width);  
        var y1 = (int)dbExt.Y;  
        var y2 = (int)(dbExt.Y + dbExt.Height);  
  
        var rand = RandomEx.GetRandom();  
        for (int i = 0; i < createNumber; i++)  
        {  
            var x = rand.Next(x1, x2) + rand.NextDouble();  
            var y = rand.Next(y1, y2) + rand.NextDouble();  
            yield return CircleEx.CreateCircle(new Point3d(x, y, 0), rand.Next(1, 100)); // 起点，终点  
        }  
    }  
  
    /* 啊惊: 有点懒不想改了*/  
#if true2  
  
    // 选择加入到四叉树  
    [CommandMethod(nameof(CmdTest_QuadTree21))]  
    public void CmdTest_QuadTree21()  
    {  
        var dm = Acap.DocumentManager;  
        var doc = dm.MdiActiveDocument;  
        var db = doc.Database;  
        var ed = doc.Editor;  
        ed.WriteMessage("\n选择单个图元加入已有的四叉树");  
  
        var ss = ed.Ssget();  
        if (ss.Count == 0)  
            return;  
  
        AddQuadTreeRoot(db, ed, ss);  
    }  
  
    // 自动加入全图到四叉树  
    [CommandMethod(nameof(CmdTest_QuadTree20))]  
    public void CmdTest_QuadTree20()  
    {  
        var dm = Acap.DocumentManager;  
        var doc = dm.MdiActiveDocument;  
        var db = doc.Database;  
        var ed = doc.Editor;  
        ed.WriteMessage("\n自动加入全图到四叉树");  
  
        var ss = new List<ObjectId>();  
        int entnum = 0;  
        var time1 = Timer.RunTime(() => {  
            db.Action(tr => {  
                db.TraverseBlockTable(tr, btRec => {  
                    if (!btRec.IsLayout)// 布局跳过  
                        return false;  
  
                    foreach (var item in btRec)  
                    {  
                        // var ent = item.ToEntity(tr);  
                        ss.Add(item);  
                        ++entnum;// 图元数量:100000, 遍历全图时间:0.216秒 CmdTest_QuadTree2  
                    }  
                    return false;  
                });  
            });  
        });  
        ed.WriteMessage($"\n图元数量:{entnum}, 遍历全图时间:{time1 / 1000.0}秒");  
  
        // 清空原有的  
        _quadTreeRoot = null;  
        AddQuadTreeRoot(db, ed, ss);  
    }  
  
    void AddQuadTreeRoot(Database db, Editor ed, List<ObjectId> ss)  
    {  
        if (_quadTreeRoot is null)  
        {  
            ed.WriteMessage("\n四叉树是空的,重新初始化");  
  
            Rect dbExt;  
            // 使用数据库边界来进行  
            var dbExtent = db.GetValidExtents3d();  
            if (dbExtent == null)  
            {  
                // throw new ArgumentException("画一个矩形");  
  
                // 测试时候画个矩形,在矩形内画随机坐标的圆形  
                dbExt = new Rect(0, 0, 32525, 32525);  
            }  
            else  
            {  
                dbExt = new Rect(dbExtent.Value.MinPoint.Point2d(), dbExtent.Value.MaxPoint.Point2d());  
            }  
            _quadTreeRoot = new(dbExt);  
        }  
  
        /* 测试:  
         * 为了测试删除内容释放了分支,再重复加入是否报错  
         * 先创建 CmdTest_QuadTree1  
         * 再减去 CmdTest_QuadTree0  
         * 然后原有黑色边界,再生成边界 CmdTest_Create00,对比删除效果.  
         * 然后加入 CmdTest_QuadTree2  
         * 然后原有黑色边界,再生成边界 CmdTest_Create00,对比删除效果.  
         */  
  
        List<CadEntity> ces = new();  
        db.Action(tr => {  
            ss.ForEach(entId => {  
                var ent = entId.ToEntity(tr);  
                if (ent is null)  
                    return;  
                var edge = new EdgeEntity(ent);  
                // 四叉树数据  
                var ce = new CadEntity(entId, edge.Edge)  
                {  
                    Color = Utility.RandomColor  
                };  
                ces.Add(ce);  
  
                edge.Dispose();  
            });  
        });  
  
        var time2 = Timer.RunTime(() => {  
            _quadTreeRoot.Insert(ces);  
        });  
        ed.WriteMessage($"\n图元数量:{ces.Count}, 加入四叉树时间:{time2 / 1000.0}秒");  
    }  
#endif  
  
    #endregion  
  
    /* 啊惊: 有点懒不想改了*/  
#if true2  
  
    #region 节点边界显示  
    // 四叉树减去节点  
    [CommandMethod(nameof(CmdTest_QuadTree0))]  
    public void CmdTest_QuadTree0()  
    {  
        var dm = Acap.DocumentManager;  
        var doc = dm.MdiActiveDocument;  
        // var db = doc.Database;  
        var ed = doc.Editor;  
        ed.WriteMessage("\n四叉树减区");  
  
        if (_quadTreeRoot is null)  
        {  
            ed.WriteMessage("\n四叉树是空的");  
            return;  
        }  
        var rect = GetCorner(ed);  
        if (rect is null)  
            return;  
        _quadTreeRoot.Remove(rect);  
    }  
  
    // 创建节点边界  
    [CommandMethod(nameof(CmdTest_CreateNodesRect))]  
    public void CmdTest_CreateNodesRect()  
    {  
        var dm = Acap.DocumentManager;  
        var doc = dm.MdiActiveDocument;  
        var db = doc.Database;  
        var ed = doc.Editor;  
        ed.WriteMessage("\n创建边界");  
  
        if (_quadTreeRoot is null)  
        {  
            ed.WriteMessage("\n四叉树是空的");  
            return;  
        }  
  
        // 此处发现了一个事务处理的bug,提交数量过多的时候,会导致 ctrl+z 无法回滚,  
        // 需要把事务放在循环体内部  
        // 报错: 0x6B00500A (msvcr80.dll)处(位于 acad.exe 中)引发的异常: 0xC0000005: 写入位置 0xFFE00000 时发生访问冲突。  
        // 画出所有的四叉树节点边界,因为事务放在外面引起  
        var nodeRects = new List<Rect>();  
        _quadTreeRoot.ForEach(node => {  
            nodeRects.Add(node);  
            return false;  
        });  
        var rectIds = new List<ObjectId>();  
        foreach (var item in nodeRects)// Count = 97341 当数量接近这个量级  
        {  
            db.Action(tr => {  
                var pts = item.ToPoints();  
                var rec = EntityAdd.AddPolyLineToEntity(pts.ToPoint2d());  
                rec.ColorIndex = 250;  
                rectIds.Add(tr.AddEntityToMsPs(db, rec));  
            });  
        }  
        db.Action(tr => {  
            db.CoverGroup(tr, rectIds);  
        });  
  
        // 获取四叉树深度  
        int dep = 0;  
        _quadTreeRoot.ForEach(node => {  
            dep = dep > node.Depth ? dep : node.Depth;  
            return false;  
        });  
        ed.WriteMessage($"\n四叉树深度是: {dep}");  
    }  
    #endregion  
  
#endif  
  
    #region 四叉树查询节点  
    // 选择范围改图元颜色  
    [CommandMethod(nameof(CmdTest_QuadTree3))]  
    public void CmdTest_QuadTree3()  
    {  
        Ssget(QuadTreeSelectMode.IntersectsWith);  
    }  
  
    [CommandMethod(nameof(CmdTest_QuadTree4))]  
    public void CmdTest_QuadTree4()  
    {  
        Ssget(QuadTreeSelectMode.Contains);  
    }  
  
    /// <summary>  
    /// 改颜色  
    /// </summary>  
    /// <param name="mode"></param>  
    void Ssget(QuadTreeSelectMode mode)  
    {  
        if (_quadTreeRoot is null)  
            return;  
  
        using DBTrans tr = new();  
        if (tr.Editor is null)  
            return;  
        var rect = GetCorner(tr.Editor);  
        if (rect is null)  
            return;  
  
        tr.Print("选择模式:" + mode);  
  
        // 仿选择集  
        var ces = _quadTreeRoot.Query(rect, mode);  
        ces.ForEach(item => {  
            var ent = tr.GetObject<Entity>(item.ObjectId, OpenMode.ForWrite);  
            ent!.Color = Color.FromColor(item.Color);  
            ent.DowngradeOpen();  
            ent.Dispose();  
        });  
    }  
  
    /// <summary>  
    /// 交互获取  
    /// </summary>  
    /// <param name="ed"></param>  
    /// <returns></returns>  
    public static Rect? GetCorner(Editor ed)  
    {  
        var optionsA = new PromptPointOptions($"{Environment.NewLine}起点位置:");  
        var pprA = ed.GetPoint(optionsA);  
        if (pprA.Status != PromptStatus.OK)  
            return null;  
        var optionsB = new PromptCornerOptions(Environment.NewLine + "输入矩形角点2:", pprA.Value)  
        {  
            UseDashedLine = true,// 使用虚线  
            AllowNone = true,// 回车  
        };  
        var pprB = ed.GetCorner(optionsB);  
        if (pprB.Status != PromptStatus.OK)  
            return null!;  
  
        return new Rect(new Point2d(pprA.Value.X, pprA.Value.Y),  
                        new Point2d(pprB.Value.X, pprB.Value.Y),  
                        true);  
    }  
    #endregion  
}  
  
// public partial class TestQuadTree  
// {  
//    public void Cmd_tt6()  
//    {  
//        using DBTrans tr = new();  
//        var ed = tr.Editor;  
//        // 创建四叉树,默认参数无所谓  
//        var TreeRoot = new QuadTree<CadEntity>(new Rect(0, 0, 32525, 32525));  
  
//        var fil = OpFilter.Build(e => e.Dxf(0) == "LINE");  
//        var psr = ed.SSGet("\n 选择需要连接的直线", fil);  
//        if (psr.Status != PromptStatus.OK) return;  
//        var LineEnts = new List<Line>(psr.Value.GetEntities<Line>(OpenMode.ForWrite)!);  
//        // 将实体插入到四岔树  
//        foreach (var line in LineEnts)  
//        {  
//            var edge = line.GeometricExtents;  
//            var entRect = new Rect(edge.MinPoint.X, edge.MinPoint.Y, edge.MaxPoint.X, edge.MaxPoint.Y);  
//            var ce = new CadEntity(line.Id, entRect)  
//            {  
//                // 四叉树数据  
//                Angle = line.Angle  
//            };  
//            TreeRoot.Insert(ce);  
//        }  
  
//        var ppo = new PromptPointOptions(Environment.NewLine + "\n指定标注点:<空格退出>")  
//        {  
//            AllowArbitraryInput = true,// 任意输入  
//            AllowNone = true // 允许回车  
//        };  
//        var ppr = ed.GetPoint(ppo);// 用户点选  
//        if (ppr.Status != PromptStatus.OK)  
//            return;  
//        var rect = new Rect(ppr.Value.Point2d(), 100, 100);  
//        tr.CurrentSpace.AddEntity(rect.ToPolyLine());// 显示选择靶标范围  
  
//        var nent = TreeRoot.FindNearEntity(rect);// 查询最近实体，按逆时针  
//        var ent = tr.GetObject<Entity>(nent.ObjectId, OpenMode.ForWrite);// 打开实体  
//        ent.ColorIndex = Utility.GetRandom().Next(1, 256);// 1~256随机色  
//        ent.DowngradeOpen();// 实体降级  
//        ent.Dispose();  
  
//        var res = TreeRoot.Query(rect, QuadTreeSelectMode.IntersectsWith);// 查询选择靶标范围相碰的ID  
//        res.ForEach(item => {  
//            if (item.Angle == 0 || item.Angle == Math.PI) // 过滤直线角度为0或180的直线  
//            {  
//                var ent = tr.GetObject<Entity>(item.ObjectId, OpenMode.ForWrite);  
//                ent.ColorIndex = Utility.GetRandom().Next(1, 7);  
//                ent.DowngradeOpen();  
//                ent.Dispose();  
//            }  
//        });  
//    }  
// }  
19. 曲线的操作  
19.1 打断曲线  
public class TestCurve  
{  
    [CommandMethod(nameof(Test_BreakCurve))]  
    public void Test_BreakCurve()  
    {  
        using DBTrans tr = new();  
        var ents = Env.Editor.SSGet()?.Value.GetEntities<Curve>();  
        if (ents is null)  
            return;  
        var tt = CurveEx.BreakCurve(ents.ToList()!);  
        tt.ForEach(t => t.ForWrite(e => e.ColorIndex = 1));  
        tr.CurrentSpace.AddEntity(tt);  
    }  
}  
  
20. 填充参数HatchInfo  
创建填充。  
[CommandMethod(nameof(TestHatchInfo))]  
public void TestHatchInfo()  
{  
    using var tr = new DBTrans();  
    var sf = new SelectionFilter(new TypedValue[] { new TypedValue(0, "*line,circle,arc") });  
    var ids = Env.Editor.SSGet(null, sf).Value?.GetObjectIds();  
    if (ids.Count() > 0)  
    {  
        HatchInfo hf = new HatchInfo(ids, false, null, 1, 0).Mode1PreDefined("Solid");  
        hf.Build(tr.CurrentSpace);  
    }  
}  
修改填充图案比例。  
[CommandMethod(nameof(TestHatchInfo2))]  
public void TestHatchInfo2()  
{  
    using var tr = new DBTrans();  
    var ht = tr.GetObject<Hatch>(Env.Editor.GetEntity("\n选择填充").ObjectId);  
    using (ht.ForWrite())  
    {  
        ht.PatternScale = 2;  
        ht.SetHatchPattern(ht.PatternType, ht.PatternName);  
    }  
    Env.Editor.Redraw(ht);  
  }  
21. 其他功能  
21.1 Lisp相关  
// 定义lisp函数  
[LispFunction(nameof(LispTest_RunLisp))]  
public static object LispTest_RunLisp(ResultBuffer rb)  
{  
    CmdTest_RunLisp();  
    return null!;  
}  
  
// 模态命令,只有当CAD发出命令提示或当前没有其他的命令或程序活动的时候才可以被触发  
[CommandMethod("CmdTest_RunLisp1")]  
// 透明命令,可以在一个命令提示输入的时候触发例如正交切换,zoom等  
[CommandMethod("CmdTest_RunLisp2", CommandFlags.Transparent)]  
// 选择图元之后执行命令将可以从 <see cref="Editor.GetSelection()"/> 获取图元  
[CommandMethod("CmdTest_RunLisp3", CommandFlags.UsePickSet)]  
// 命令执行前已选中部分实体.在命令执行过程中这些标记不会被清除  
[CommandMethod("CmdTest_RunLisp4", CommandFlags.Redraw)]  
// 命令不能在透视图中使用  
[CommandMethod("CmdTest_RunLisp5", CommandFlags.NoPerspective)]  
// 命令不能通过 MULTIPLE命令 重复触发  
[CommandMethod("CmdTest_RunLisp6", CommandFlags.NoMultiple)]  
// 不允许在模型空间使用命令  
[CommandMethod("CmdTest_RunLisp7", CommandFlags.NoTileMode)]  
// 不允许在布局空间使用命令  
[CommandMethod("CmdTest_RunLisp8", CommandFlags.NoPaperSpace)]  
// 命令不能在OEM产品中使用  
[CommandMethod("CmdTest_RunLisp9", CommandFlags.NoOem)]  
// 不能直接使用命令名调用,必须使用   组名.全局名  调用  
[CommandMethod("CmdTest_RunLisp10", CommandFlags.Undefined)]  
// 定义lisp方法.已废弃   请使用lispfunction  
[CommandMethod("CmdTest_RunLisp11", CommandFlags.Defun)]  
// 命令不会被存储在新的命令堆上  
[CommandMethod("CmdTest_RunLisp12", CommandFlags.NoNewStack)]  
// 命令不能被内部锁定(命令锁)  
[CommandMethod("CmdTest_RunLisp13", CommandFlags.NoInternalLock)]  
// 调用命令的文档将会被锁定为只读  
[CommandMethod("CmdTest_RunLisp14", CommandFlags.DocReadLock)]  
// 调用命令的文档将会被锁定,类似document.lockdocument  
[CommandMethod("CmdTest_RunLisp15", CommandFlags.DocExclusiveLock)]  
// 命令在CAD运行期间都能使用,而不只是在当前文档  
[CommandMethod("CmdTest_RunLisp16", CommandFlags.Session)]  
// 获取用户输入时,可以与属性面板之类的交互  
[CommandMethod("CmdTest_RunLisp17", CommandFlags.Interruptible)]  
// 命令不会被记录在命令历史记录  
[CommandMethod("CmdTest_RunLisp18", CommandFlags.NoHistory)]  
// 命令不会被 UNDO取消  
[CommandMethod("CmdTest_RunLisp19", CommandFlags.NoUndoMarker)]  
// 不能在参照块中使用命令  
[CommandMethod("CmdTest_RunLisp20", CommandFlags.NoBlockEditor)]  
#if !ac2008  
// acad09增,不会被动作录制器 捕捉到  
[CommandMethod("CmdTest_RunLisp21", CommandFlags.NoActionRecording)]  
// acad09增,会被动作录制器捕捉  
[CommandMethod("CmdTest_RunLisp22", CommandFlags.ActionMacro)]  
#endif  
#if !NET35  
// 推断约束时不能使用命令  
[CommandMethod("CmdTest_RunLisp23", CommandFlags.NoInferConstraint)]  
// 命令允许在选择图元时临时显示动态尺寸  
[CommandMethod("CmdTest_RunLisp24", CommandFlags.TempShowDynDimension)]  
#endif  
public static void CmdTest_RunLisp()  
{  
    // 测试方法1: (command "CmdTest_RunLisp1")  
    // 测试方式2: (LispTest_RunLisp)  
    var dm = Acap.DocumentManager;  
    var doc = dm.MdiActiveDocument;  
    var ed = doc.Editor;  
  
    var sb = new StringBuilder();  
    foreach (var item in Enum.GetValues(typeof(EditorEx.RunLispFlag)))  
    {  
        sb.Append((byte)item);  
        sb.Append(',');  
    }  
    sb.Remove(sb.Length - 1, 1);  
    var option = new PromptIntegerOptions($"\n输入RunLispFlag枚举值:[{sb}]");  
    var ppr = ed.GetInteger(option);  
  
    if (ppr.Status != PromptStatus.OK)  
        return;  
    var flag = (EditorEx.RunLispFlag)ppr.Value;  
  
    if (flag == EditorEx.RunLispFlag.AdsQueueexpr)  
    {  
        // 同步  
        Env.Editor.RunLisp("(setq a 10)(princ)",  
            EditorEx.RunLispFlag.AdsQueueexpr);  
        Env.Editor.RunLisp("(princ a)",  
            EditorEx.RunLispFlag.AdsQueueexpr);// 成功输出  
    }  
    else if (flag == EditorEx.RunLispFlag.AcedEvaluateLisp)  
    {  
        // 使用(command "CmdTest_RunLisp1")发送,然后 !b 查看变量,acad08是有值的,高版本是null  
        var strlisp0 = "(setq b 20)";  
        var res0 = Env.Editor.RunLisp(strlisp0,  
            EditorEx.RunLispFlag.AcedEvaluateLisp); // 有lisp的返回值  
  
        var strlisp1 = "(defun f1( / )(princ \"aa\"))";  
        var res1 = Env.Editor.RunLisp(strlisp1,  
            EditorEx.RunLispFlag.AcedEvaluateLisp); // 有lisp的返回值  
  
        var strlisp2 = "(defun f2( / )(command \"line\"))";  
        var res2 = Env.Editor.RunLisp(strlisp2,  
            EditorEx.RunLispFlag.AcedEvaluateLisp); // 有lisp的返回值  
    }  
    else if (flag == EditorEx.RunLispFlag.SendStringToExecute)  
    {  
        // 测试异步  
        // (command "CmdTest_RunLisp1")和(LispTest_RunLisp)4都是异步  
        var str = "(setq c 40)(princ)";  
        Env.Editor.RunLisp(str,  
            EditorEx.RunLispFlag.SendStringToExecute); // 异步,后发送  
        Env.Editor.RunLisp("(princ c)",  
            EditorEx.RunLispFlag.AdsQueueexpr); // 同步,先发送了,输出是null  
    }  
}  
21.2 使用CAD内部命令  
SendStringToExecute()函数会延时执行（非同步），它在.NET命令结束时才会被调用。  
var doc=Acap.DocumentManager.MdiActiveDocument;  
string fileName = "D:\\test.dwg";  
doc.SendStringToExecute("Saveas\n"+"LT2004\n"+fileName+"\n",true,false,false);  
需要同步执行时，可以用Editor.RunLisp()方法，设置RunLispFlag特性为RunLispFlag.AcedEvaluateLisp即可同步执行。  
Env.Print("\n123");  
Env.Editor.RunLisp("(princ \"\n456\")", EditorEx.RunLispFlag.AcedEvaluateLisp);  
Env.Print("\n789");  
执行结果为：  
命令: TEST  
123  
456  
789  
在执行命令前取消在执行的多个嵌套命令，可以参考以下方法。  
//创建Esc命令 By edata 代码来自adn blog https://forums.autodesk.com/t5/net/ribbon-image-resolution-issue/m-p/10325019  
string esc = "";  
string cmds = (string)Acap.GetSystemVariable("CMDNAMES");  
if (cmds.Length > 0)  
{  
    int cmdNum = cmds.Split(new char[] { '\'' }).Length;  
    for (int i = 0; i < cmdNum; i++)  
        esc += '\x03';  
}  
doc.SendStringToExecute(esc + cmdbtn.CmdStr + "\n", true, false, true);  
21.3 XrefFactory绑定参照  
public class TestCmd_BindXrefs  
{  
    //后台绑定  
  
    [CommandMethod(nameof(Test_Bind1))]  
    public static void Test_Bind1()  
    {  
        string fileName = @"D:\Test.dwg";  
        using var tr = new DBTrans(fileName,  
            fileOpenMode: FileOpenMode.OpenForReadAndAllShare/*后台绑定特别注意*/);  
        tr.XrefFactory(XrefModes.Bind);  
        tr.SaveDwgFile();  
    }  
  
    //前台绑定  
    [CommandMethod(nameof(Test_Bind2))]  
    public static void Test_Bind2()  
    {  
        using var tr = new DBTrans();  
        tr.XrefFactory(XrefModes.Bind);  
        tr.SaveDwgFile();  
    }  
}  
21.4 导出为WMF  
[CommandMethod(nameof(Test_ExportWMF), CommandFlags.Modal | CommandFlags.UsePickSet)]  
public void Test_ExportWMF()  
{  
    var psr = Env.Editor.SelectImplied();// 预选  
    if (psr.Status != PromptStatus.OK)  
        psr = Env.Editor.GetSelection();// 手选  
    if (psr.Status != PromptStatus.OK)  
        return;  
  
    var ids = psr.Value.GetObjectIds();  
    // acad21(acad08没有)先选择再执行..会让你再选择一次  
    // 而且只发生在启动cad之后第一次执行.  
    Env.Editor.ComExportWMF("D:\\桌面\\aaa.wmf", ids);  
}  
21.5 标注箭头类  
// 设置标注样式箭头为建筑标记  
dimStyleTableRecord.Dimblk = Env.GetDimblkId(Env.DimblkType.ArchTick);  
// 设置为点  
dimStyleTableRecord.Dimblk = Env.GetDimblkId(Env.DimblkType.Dot);  
cad中的样式都可以在枚举里找到  
  
  
22 CAD小众API  
22.1 Utils CAD工具类  
此类提供众多好用的方法,有同lisp功能的方法,有补充Editor类功能的方法,等  
Utils.ZoomAuto(1, 1, 1, 1, 1);//缩放视口，查看全部图元  
22.2 ZipExtractor 压缩包类  
CAD提供的读写压缩包的类  
22.3 LayerUtilities 图层工具类  
22.4 LayoutThumbnailEnumerator 布局缩略图类  
22.5 MlinesMgdServices 多线管理类  
22.6 SystemVariableEnumerator 系统变量枚举类  
22.7 CommandLineMonitorServices命令行监视器服务  
监控命令的选择信息,命令行信息,命令信息,等  
22.8 Acap.UIBindings CAD UI绑定的数据类  
此类中有CAD各种UI的信息,或许能操控一下特殊的UI,如鼠标移到填充夹点上弹出的设定原点,比例,角度;  
Acap.UIBindings.Collections此集合类中有各种符号表的信息,图层表,块表,等  
23.打开CAD自动加载DBX，ARX文件  
24.获取低版本没有的api入口  
25.PE的使用  
  
Todo   
  
