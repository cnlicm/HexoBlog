---
title: WPF从入门到入坟 - 07Application类
comments: false
date: 2024-05-31 20:43:22
tags:
    - WPF从入门到入坟
categories:
    - C#
    - WPF
keywords:
    - Application
    - App
description:
top_img: 
cover: https://cnlicm-blog-image.oss-cn-shenzhen.aliyuncs.com/img/wpf.png
abbrlink:
---

## 应用程序的生命周期

&emsp;&emsp;在 WPF 中，应用程序会经历简单的生命周期。在应用程序启动后，将立即创建应用程序对象。在应用程序运行时触发各种应用程序事件，您可以选择监视其中的某些事件。最后，当释放应用程序对象时，应用程序将结束。

### 创建Application对象

&emsp;&emsp;手动创建Application对象启动Wpf应用

1. 设置项目属性的`EnableDefaultApplicationDefinition`属性为false使WPF应用不自动生成Main函数

    ```XML
    <PropertyGroup>
        ...
        <!-- +++ -->
        <EnableDefaultApplicationDefinition>false</EnableDefaultApplicationDefinition>
        <!-- +++ -->
    </PropertyGroup>
    ```
1. 新建自己的文件

    ```C#
    class Program
    {
        [STAThread]
        public static void Main(string[] args)
        {
            App app = new App();
            MainWindow mainWindow = new MainWindow();
            app.Run(mainWindow);
        }
    }
    ```

### 应用程序的关闭方式

&emsp;&emsp;通常，只要还有窗口尚未关闭，Application类就保持应用程序处于有效状态。如果这不是期望的行为，可调整`Application.ShutdownMode`属性和。如果手动实例化Application对象，就需要在调用Run()方法之前设置ShutdownMode属性。如果使用App.xaml文件，那么可在XAML文件中简单设置ShutdownMode属性

<center>ShutdownMode枚举值</center>

|名称|说明|
|---|---|
|OnLastWindowClose|这是默认行为——只要至少还有一个窗口存在，应用程序就保持运行状态|
|OnMainWindowClose|这是传统方式——只要主窗口还处于打开状态，应用程序就保持运行状态|
|OnExplicitShutdown|除非调用`Application.Shutdown()`，否则应用程序就不会结束|

### 应用程序事件

<center>应用程序事件</center>

|名称|说明|
|---|---|
|Startup|该事件在调用`Application.Run()`之后，并且在主窗口显示之前发生。可使用该事件检查所有命令行参数，命令行参数时通过`StartupEventArg.Args`属性作为数组提供的|
|Exit|该事件在应用关闭时，并在`Run()`即将返回之前发生。此时不能取消关闭|
|SessionEnding|该事件在Windows对话结束时发生——例如，当用户注销或关闭计算机时（通过检查`SesstionEndingCancelEventArgs.ReasonSessionEnding`属性可以确定原因）。也可以通过将`SessionEndingEventArgs.Cancel`属性设置为true来取消关闭应用程序。否则，当事件处理程序结束时，WPF将调用`Application.Shutdown()`方法
|Activated|当激活应用程序中的窗口时发生该事件。当从另一个Windows程序切换到该应用时会发生该事件。当第一次显示窗口时也会发生该事件
|Deactivated|当取消激活应用程序中的窗口时发生该事件。当切换到另一个Windows程序时耶夫一发生该事件
|DispatcherUnhandledException|在应用程序（主应用程序线程）中的任何位置，只要发生未处理的异常，就会发生该事件（应用程序会驳货这些异常）。通过响应该事件，可记录重要错误，甚至可选择不处理这些异常，并通过将`DispatcherUnhandledExceptionEventArgs.Handled`属性设置为true继续运行应用程序。只有当可以确保应用程序仍然处于合法状态时可以继续运行时，才这样处理|

&emsp;&emsp;下面时一个自定义的应用程序类，他重写了`OnSessionEnding()`方法，并且给如果设置了相应的标志，该方法会阻止关闭系统和应用程序自身

```C#
public partial class App : Application
{
    private bool _unsavedData = false;
    public bool UnsavedData
    {
        get { return _unsavedData; }
        set { _unsavedData = value; }
    }

    protected override void OnStartup(StartupEventArgs e)
    {
        base.OnStartup(e);

        UnsavedData = true;
    }

    protected override void OnSessionEnding(SessionEndingCancelEventArgs e)
    {
        base.OnSessionEnding(e);

        if (UnsavedData)
        {
            e.Cancel = true;
            MessageBox.Show($"The application attempted to be closed as a result of {e.ReasonSessionEnding.ToString()}. This is not allowed, as you have unsaved data.");
        }
    }

}
```
## Application类的任务

### 显示初始界面

&emsp;&emsp;使用WPF提供的简单初始界面特性：

1. 为项目添加图像文件（通常时.bmp、.png或.jpg文件）
1. 在Solution Explorer 中选择图像文件
1. 将Build Action修改为`SplashScreen`

&emsp;&emsp;下次运行应用程序时，图像会立即在屏幕中央显示出来。一旦准备好运行时环境，而且`Application_Startup`方法执行完毕，应用程序的第一个窗口就将显示出来，这是初始界面图形会很快消失。以上添加初始界面的朝族哟，WPF编译器为自动生成的App.g.cs文件添加与下面类似的代码：

```C#
SplashScreen splashScreen = new SplashScreen("splashScreenImage.png");
splashScreen.Show(true); // 设置为false后，初始界面不会自动关闭，需要手动调用splashScreen.Close()进行关闭

MyApplication.App app = new MyApplication.App();
app.InitializeComponent();
app.Run();
```
### 处理命令行参数

&emsp;&emsp;为处理命令行参数，需要相应`Aplication.Startup`事件。命令行参数时通过`StartupEventArgs.Args`属性作为字符串数组提供的

```C#
protected override void OnStartup(StartupEventArgs e)
{
    base.OnStartup(e);

    UnsavedData = true;

    var win = new FileViewer();

    if (e.Args.Length > 0)
    {
        string file = e.Args[0]; // 获取命令行参数
        if (System.IO.File.Exists(file))
        {
            win.LoadFile(file);
        }
    }

}
```

### 访问当前Application对象

&emsp;&emsp;通过静态的`Application.Current`属性，可在应用程序的任何位置获取当前应用程序实例，从而在窗口之间进行基本交付，因为任何窗口都有可以访问当前Application对象，并通过Application对象获取主窗口的应用：

```C#
Window mainWindow = Application.Current.MainWindow;
MessageBox.Show("The main window is " + mainWindow.Title);
```

&emsp;&emsp;子啊窗口中还可以检查`Application.Windows`集合的内容，该集合提供了所有当前打开窗口的引用：

```C#
foreach(Window window in Application.Current.Windows)
{
    MessageBox.Show(window.Title + " is open.");
}
```

### 在窗口之间进行交互

&emsp;&emsp;正如在前面已经看到的，自定义应用程序类是放置响应不同应用程序事件的代码的好地方。应用程序类还可以很好地达到另一个目的:保存重要窗口的引用，使一个窗口可访问另一个窗口。

```C#
public partial class App : Application
{
    // 可存储整个应用程序的文档，达到每个窗口共享的效果
    private List<Document> documents = new List<Document>();

    public List<Document> Documents
    {
        get {return documents;}
        set {documents = value;}
    }
}
```

### 单实例应用程序

&emsp;&emsp;通常，只要愿意就可以加载 WPF 应用程序的任意多个副本。某些情况下，这种设计是非常合理的。但在另外一些情况下，这可能会成为问题，当构建基于文档的应用程序时更是如此。

{% note info %}
对于单实例应用程序，WPF本身并未提供自带的解决方法，但可使用几种变通方法。基本技术时当出发`Application.Startup`事件时，检查另一个应用程序实例时候已在运行。最简单的方法时使用`全局的mutex对象`（mutex对象时操作系统提供的用于进程间通信的同步对象）。这种方法很简单，但功能有限。最重要的是，应用程序的新实例无法与已经存在的应用程序实例进行通信。
{% endnote %}

{% note danger %}
第一种方法较为简单直接，适合大多数场景。第二种方法更为复杂，但可以处理一些特定的需求，例如在已有实例中显示新的内容。
{% endnote %}

#### 使用Mutex

&emsp;&emsp;使用Mutex可以确保只有一个实例在运行。如果尝试启动第二个实例，新的实例将检测已有的实例在运行并退出

```C#
public partial class App : Application
{
    private static Mutex? _mutex = null;

    protected override void OnStartup(StartupEventArgs e)
    {
        const string appName = "SingleInstanceApp";
        bool createdNew;

        _mutex = new Mutex(true, appName, out createdNew);

        if (!createdNew)
        {
            // 应用程序已在运行！退出新实例
            MessageBox.Show("应用程序已在运行!");
            Application.Current.Shutdown();
            return;
        }

        base.OnStartup(e);
    }
}
```

#### 使用Windows API

&emsp;&emsp;通过P/Invoke使用Windows API来实现单实例

1. 创建一个`NativeMethods`类

    ```C#
    internal static class NativeMethods
    {
        public const int HWND_BROADCAST = 0xffff;
        public static readonly int WM_SHOWFIRSTINSTANCE = RegisterWindowMessage("WM_SHOWFIRSTINSTANCE|" + App.AppGuid);

        [DllImport("user32")]
        public static extern int RegisterWindowMessage(string message);

        [DllImport("user32")]
        public static extern bool PostMessage(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam);

        [DllImport("user32")]
        public static extern IntPtr FindWindow(string classname, string windowname);
    }
    ```
1. 更新`App.xaml.cs`

    ```C#
    public partial class App : Application
    {
        private static Mutex _mutex = null;
        public const string AppGuid = "your-app-guid-here";

        protected override void OnStartup(StartupEventArgs e)
        {
            bool createdNew;
            _mutex = new Mutex(true, "Global\\" + AppGuid, out createdNew);

            if (!createdNew)
            {
                NativeMethods.PostMessage((IntPtr)NativeMethods.HWND_BROADCAST, NativeMethods.WM_SHOWFIRSTINSTANCE, IntPtr.Zero, IntPtr.Zero);
                Application.Current.Shutdown();
                return;
            }

            base.OnStartup(e);
            ShowMainWindow();
        }

        private void ShowMainWindow()
        {
            MainWindow mainWindow = new MainWindow();
            mainWindow.Show();

            IntPtr handle = (new WindowInteropHelper(mainWindow)).Handle;
            HwndSource source = HwndSource.FromHwnd(handle);
            source.AddHook(new HwndSourceHook(WndProc));
        }

        private IntPtr WndProc(IntPtr hwnd, int msg, IntPtr wParam, IntPtr lParam, ref bool handled)
        {
            if (msg == NativeMethods.WM_SHOWFIRSTINSTANCE)
            {
                ShowMainWindow();
            }

            return IntPtr.Zero;
        }
    }
    ```

## 程序集资源

{% note info %}
程序集资源又称为二进制资源，因为它们作为不透明的二进制数据被嵌入到已编译的程序集中
{% endnote %}

### 添加资源

&emsp;&emsp;可同构项目添加文件，并在Properties窗口中将其Build Action属性设置为Resource来添加自己的资源。为成功地使用程序集资源，无比注意以下两点：

- 不能将Build Action属性错误地设置为 Embedded Resource。尽管所有程序集资源都被定义为嵌入的资源，当Embedded Resource生成操作会在另一个更难访问的位置放置二进制数据。在WPF应用程序中，假定总是使用Resource生成类型
- 不要在Propect Properties窗口中使用Resource选项卡。WPF不支持这种类型的资源URI

{% note info %}
WPF将程序集资源和其他BAML资源合并到单独的流中。单独的资源流是哟个以下格式命名：`AssemblyName.g.resources`
{% endnote %}

### 检索资源

&emsp;&emsp;显然，添加资源非常容易，但到底如何使用它们了？可以采用多种方法来使用资源

- 通过静态方法`Application.GetResourceStream()`

    ```C#
    StreamResourceInfo sri = Application.GetResourceStream(new Uri("images/winter.jpg",UriKind.Relative));
    ```
- 自行访问`AssemblyName.g.resource`资源流，并查询所需的对象

    ```C#
    Assembly assembly = Assembly.GetAssembly(this.GetType());
    string resourceName =assembly.GetName().Name + ".g";
    ResourceManager rm = new ResourceManager(resourceName,assembly);

    using(ResourceSet set = rm.GetResourceSet(CaltureInfo.CurrentCulture,true,true))
    {
        UnmanageMemoryStream s = (UnmanageMemoryStream)set.GetObject("images/winter.jpg",true);
    }
    ```

### pack URI

&emsp;&emsp;WPF使用pack URI语法寻址编译过的资源，其完整语法如下：

```XML
pack://application:,,,/AssemblyName;component/ResourceName
```
例如：

- 当前程序集下

```XML
<!-- 以下写法等价 -->
<Image Source="images/winter.jpg"/>
<Image Source="pack://application:,,,/images/winter.jpg"/>
```
- 其他程序集下

```XML
<!-- 资源图片位于WpfResources程序集的images目录下 -->
<Image Source="pack://application:,,,/WpfResources;component/images/winter.jpg"/>
```

### 内容文件

&emsp;&emsp;当嵌入文件作为资源时，会将文件放到编译过的程序集中，并且可以确保文件总是可用的。对于部署而言这是理想选择，并且可避免可能存在的问题。然而在有些情况下，使用这种方法并不方便：

- 希望改变资源文件，又不想重新编译应用程序
- 资源文件非常大
- 资源文件是可选的，并且可以不随程序集一起部署
- 资源是声音文件

{% note warning %}
WPF声音不支持程序集资源。因此，无法从资源流中析取音频文件并播放它们——至少，如果没有首先保存音频文件，就不能播放它们。这一局限是由于这些类使用的技术基础（Win32API和媒体播放器）造成的
{% endnote %}

&emsp;&emsp;显然，可使用应用程序部署文件，并为应用程序添加代码，进而从硬盘驱动器中读取这些文件来解决该问题。然而，WPF还有更方便的选择，使这一过程更加容易管理。可将这些未编译的文件专门标记为内容文件。
&emsp;&emsp;不能将内容文件嵌入到程序集中。然而，WPF为程序集添加了AssemblyAssociated-ContentFile 特性，公告每个内容文件的存在。该特性还记录了每个内容文件相对于可执行文件的位置(指示内容文件是否和可执行文件位于同一个文件夹中，或者位于某个子文件夹中)。最方便的是，当为能够理解资源的元素(如 Image 类)使用内容文件时，可使用相同的 URI 系统。
&emsp;&emsp;为测试该技术，为项目添加声音文件，在SolutionExplorer 中选择该文件，并在Properties窗口中`将 Build Action 属性改为 Content`。确保`将 Copy to Output Directory 属性设置为 CopyAlways`，以保证当生成项目时将声音文件复制到输出目录中。

```XML
<MediaELement Name="Sound" Source="Sounds/start.wav" LoadedBehavior="Manual"/>
```
## 本地化

&emsp;&emsp;当需要本地化窗口时，程序集资源也可以提供方便。使用资源，可根据 Windows操作系统的当前文化设置改变控件。对于需要翻译为不同语言的文本标签和图像，这尤其有用。

### 构建能够本地化的用户界面

&emsp;&emsp;在开始翻译任何内容前，首先需要考虑应用程序会如何响应内容变化。用户界面应当能够调整自身以适应动态的内容。下面列出建议采用的一些原则：

- 不使用硬编码的宽度或高度(或至少对那些包含不能滚动的文本内容的元素不使用硬编码的宽度和高度)。
- 将 Window.SizeToContent 属性设置为 Width、Height 或 WidthAndHeight，使窗口尺寸能够根据需要扩大(根据窗口结构的不同，并不总是需要这样，但有时是很有用的)。
- 使用 ScrollViewer 控件封装大量文本。

### 使应用程序为本地化做好准备

&emsp;&emsp;在第一个`<PropertyGroup>`元素中的任意地方添加以下元素：

```XML
<PropertyGroup>
    ...
	<UICulture>en-US</UICulture>
</PropertyGroup>
```

&emsp;&emsp;上面的标记告诉编译器，应用程序的默认文化是美式英语(显然，也可选择其他合适的文化)。一旦进行了这一修改，生成过程就会发生变化。下次编译应用程序时，最后会生成名为en-US 的子文件夹。在该文件夹中包含的是附属程序集，附属程序集与应用程序同名，而且扩展名为.resources.dll(如 LocalizableApplication.resources.dl)。附属程序集包含了应用程序的所有编译过的 BAML资源，以前这些资源保存在主应用程序的程序集中。


