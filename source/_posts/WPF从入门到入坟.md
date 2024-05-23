---
title: WPF从入门到入坟
comments: false
date: 2024-05-22 21:41:34
tags:
    - C#
    - WPF
categories:
    - C#
    - WPF
keywords:
description:
top_img:
cover: https://cnlicm-blog-image.oss-cn-shenzhen.aliyuncs.com/img/wpf.png
abbrlink:
---

## WPF概述

&emsp;&emsp;WPF(`Windows Presentation Foundation`)是用于 Windows 的现代图形显示系统。与之前出现的其他技术相比，WPF发生了根本性变化，引入了“内置硬件加速”和“分辨率无关”等创新功能;本章将介绍这两项功能。    

### WPF应用特点

- **硬件加速** 。通过DirectX执行所有WPF绘图操作，以便充分利用现代显卡的最新功能
- **分辨率无关**。WPF能够根据系统DPI设置，很长灵活地方大和缩小显示的内容，以使其适合所用的显示器和显示选择
- **控件无固定外观**。可自由定制外观
- **声明式用户界面**。通过XAML不必编写代码即可创建窗口
- **基于对象的绘图**。即使准备在更低级的可视化层(而非高级元素层)上工作，也不需要使用绘图和像素进行工作，而是创建图形对象并让 WPF 尽可能最优化地显示出来。

### WPF体系结构

&emsp;&emsp;WPF 使用多层体系结构。在顶层，应用程序与完全由托管C#代码编写的一组高层服务进行交瓦。至于将.NET 对象转换为Direct3D纹理和三角形的实际工作，是在后台由一个名为milcore.dll 的低级非托管组件完成的。milcore.dll 是使用非托管代码实现的，因为它需要和Direct3D紧密集成，并且它对性能极其敏感。

#### 体系结构

<center>WPF 应用程序中各层的工作情况</center>

![WPF体系结构](https://cnlicm-blog-image.oss-cn-shenzhen.aliyuncs.com/img/20240522220928.png)

- `PresentationFramework.dll` 包含 WPF 顶层的类型，包括那些表示窗口、面板以及其他类型控件的类型。它还实现了高层编程抽象，如样式。开发人员直接使用的大部分类都来自这个程序集。
- `PresentationCore.dll` 包含了基础类型，如 UIElement 类和 Visual类，所有形状类和控件类都继承自这两个类。如果不需要窗口和控件抽象层的全部特征，可使用这一层，而且仍能利用 WPF 的渲染引擎。
- `WindowsBase.dll` 包含了更多基本要素，这些要素具有在WPF之外重用的潜能，如DispatcherObiect 类和 DependencyObiect类,这两个类引入了依赖项属性。
- `milcore.dll` 是 WPF 渲染系统的核心，也是媒体集成层(MediaIntegration Layer，MIL)的基础。其合成引擎将可视化元素转换为Direct3D所期望的三角形和纹理。尽管将milcore.dll 视为 WPF 的一部分，但它也是 Windows Vista和 Windows7的核心系统组件之一。实际上，桌面窗口管理器(Desktop Window Manager，DWM)使用 milcore.dll渲染桌面。
- `WindowsCodecs.dll` 是一套提供图像支持的低级 API(例如处理、显示以及缩放位图和JPEG 图像)。
- `Direct3D` 是一套低级 API，WPF 应用程序中的所有图形都由它进行渲染。
- `User32` 用于决定哪些程序实际占有桌面的哪一部分。所以它仍被包含在 WPF 中，但不再负责渲染通用控件。

#### 类层次关系

<center>类层次关系</center>

![类层次关系](https://cnlicm-blog-image.oss-cn-shenzhen.aliyuncs.com/img/20240522225239.png)

##### DispatcherObject

&emsp;&emsp;WPF 应用程序使用为人熟知的单线程亲和(Single-Thread Afnity，STA)模型，这意味着整个用户界面由单个线程拥有。从另一个线程与用户界面元素进行交互是不安全的。为方便使用此模型，每个 WPF 应用程序由协调消息(键盘输入、鼠标移动乃至框架处理，如布局)的调度程序管理。通过继承自 DispatcherObiect类,用户界面中的每个元素都可以检査代码是否在正确的线程上运行，并能通过访问调度程序为用户界面线程封送代码。

##### DependencyObject

&emsp;&emsp;在 WPF 中，主要通过属性与屏幕上的元素进行交互。在早期设计阶段，WPF的设计者决定创建一个更加强大的属性模型，该模型支持许多特性，例如更改通知、默认值继承以及减少属性存储空间。最终结果就是依赖项属性(dependencyproperty)特性。通过继承自 DependencyObject类，WPF类可获得对依赖项属性的支持。

##### Visual

&emsp;&emsp;在 WPF 窗口中显示的每个元素本质上都是 Visual对象。可将 Visual 类视为绘图对象，其中封装了绘图指令、如何执行绘图的附加细节(如剪裁、透明度以及变换设置)以及基本功能(如命中测试)。Visual类还在托管的WPF 库和渲染桌面的 milcore.dll 程序集之间提供了链接。任何继承自 Vsual 的类都能在窗口上显示出来。如果更愿意使用轻量级的 API创建用户界面，而不想使用 WPF 的高级框架特征，可直接使用 Visual 对象进行编程。

##### UIElement

&emsp;&emsp;UIElement 类增加了对 WPF 本质特征的支持，如布局、输入、焦点和事件(WPF 团队使用首字母缩写词 LIFE 来表示)。WPF实现了增强的称为路由事件(RoutedEvent)的事件路由系统，UIElement类中还添加了对命令的支持。

##### FrameworkElement

&emsp;&emsp;FrameworkElement类是 WPF 核心继承树中的最后一站。该类实现了一些全部由 UIElement类定义的成员。例如，UIElement类为WPF布局系统设置了基础，但FrameworkElement类提供了支持它的重要属性(如 HorizontalAlignment 和 Margin 属性)。UIElement 类还添加了对数据绑定、动画以及样式等核心特性的支持。

##### Shape

&emsp;&emsp;基本的形状类(如 Rectangle 类、Polygon 类、Ellipse 类、Line 类以及 Path 类)都继承自该类可将这些形状类与更传统的 Windows小组件(如按钮和文本框)结合使用。

##### Control

&emsp;&emsp;控件(control)是可与用户进行交互的元素。控件显然包括TextBox 类、Button 类和 ListBox类等。Control类为设置字体以及前景色与背景色提供了附加属性。但最令人感兴趣的细节是模板支持，通过模板支持，可使用自定义风格的绘图替换控件的标准外观。

##### ContentControl

&emsp;&emsp;ContentControl类是所有具有单一内容的控件的基类，包括简单的标签乃至窗口的所有内容。该模型给人印象最深刻的部分是:控件中的单一内容可以是普通字符串乃至具有其他形状和控件组合的布局面板

##### ItemControl

&emsp;&emsp;ItemsControl 类是所有显示选项集合的控件的基类，如ListBox和 TreeView 控件。列表控件十分灵活--例如，使用 ItemsControl类的内置特征，可将简单的 ListBox 控件变换成单选按钮列表、复选框控件列表、平铺的图像或是您所选择的完全不同的元素的组合。实际上，WPF中的菜单、工具栏以及状态栏都是特定的列表，并且实现它们的类都继承自ItemsContorl 类。

##### Panel

&emsp;&emsp;Panel 类是所有布局容器的基类,布局容器是可包含一个或多个子元素、并按特定规则对子元素进行排列的元素。这些容器是WPF布局系统的基础，要以最富有吸引力、最灵活的方式安排内容，使用这些容器是关键所在。

## XAML

{% note warning %}
    XAML对于WPF不是必需的，理解这一点很重要。Visual Studio当然可以使用Windows窗体方法，通过语句代码来构造WPF窗口。但如果这样的话，窗口将被限制在Visual Studio开发环境之内，只能由编程人员使用。
{% endnote %}

### XAML名称空间

``` XML
xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
xmlns:"x=http://schemas.microsoft.com/winfx/2006/xaml"
```

&emsp;&emsp;xmlns特性是XML中的一个特殊特性，他专门用来声明名称空间。这段标记声明了两个名称空间，在创建的所有WPF XAML
文档中都会使用这两个名称控件
- `http://schemas.microsoft.com/winfx/2006/xaml/presentation`是WPF核心名称空间。它包含了所有WPF类，包括用来构建用户界面的控件。在该例中，该名称空间的声明没有使用名称空间前缀，所以它成为整个文档的默认名称空间。换句话说，除非另行指明，每个元素自动位于这个名称空间
- `http://schemas.microsoft.com/winfx/2006/xam1` 是XAML 名称空间。它包含各种 XAML实用特性，这些特性可影响文档的解释方式。该名称空间被映射为前缀x。这意味着可通过在元素名称之前放置名称空间前缀x来使用该名称空间(例如<x:ElementName>)。

### 简单属性

&emsp;&emsp;可直接进行简单的赋值操作

```XML
<TextBox VerticalAlignment="Stretch" FontFamily="Verdana" FontSize="24" Foreground="Green" />
```
### 转换器

#### 简单转换器

1. 定义转换器

    ```C#
    // Visibility类型到bool类型的转换器
    [ValueConversion(typeof(Visibility), typeof(bool))] 
    public class VisibilityToBoolConverter : IValueConverter
    {
        // 定义静态属性(可选)
        public static VisibilityToBoolConverter Instance = new VisibilityToBoolConverter();

        public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
        {
            if (value is Visibility visibility)
            {
                return visibility == Visibility.Visible ? true : false;
            }
            return false;
        }

        public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        {
            if (value is bool visibility && visibility)
            {
                return Visibility.Visible;
            }
            return Visibility.Collapsed;
        }
    }
    ```
1. 使用转换器
   - 静态资源引用
    ``` XML
    <Window
        xmlns:converters="clr-namespace:MyApplication.Converters">
        <Window.Resource>
            <!--声明资源-->
            <converters:VisibilityToBoolConverter x:Key="VisibilityToBoolConverter"/>
        </Window.Resource>
        
        <Grid>
            <CheckBox IsChecked="{Binding MyFlag,Converter={StaticResource VisibilityToBoolConverter}}"/>
        </Grid>

    </Window>
    ```
   - 静态属性绑定
    ``` XML
    <Window
        xmlns:converters="clr-namespace:MyApplication.Converters">
        <!-- 无需声明声明资源-->
        <Grid>
           <CheckBox IsChecked="{Binding MyFlag,Converter={x:Static converters:VisibilityToBoolConverter.Instance}}"/>
        </Grid>

    </Window>
    ```

#### 多值转换器

1. 定义转换器

    ```C#
    public class FullNameConverter : IMultiValueConverter
    {
        public static FullNameConverter Instance = new FullNameConverter();

        public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
        {
            return string.Join("", values);
        }

        public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
        {
            throw new NotImplementedException();
        }
    }
    ```
1. 使用转换器

    ```XML
    <Window x:Class="WpfApp.MainWindow"
        xmlns:converters="clr-namespace:WpfApp.Converters">
        <StackPanel>
            <TextBox x:Name="firstName"/>
            <TextBox x:Name="secondName"/>
            <TextBlock>
                <TextBlock.Text>
                <!-- MultiBinding只能绑定常量，因此对于变量可以先赋值给一些控件上，然后通过Element+Path绑定 -->
                    <MultiBinding Converter="{x:Static converters:FullNameConverter.Instance}">
                        <Binding ElementName="firstName" Path="Text"/>
                        <Binding ElementName="secondName" Path="Text"/>
                    </MultiBinding>
                </TextBlock.Text>
            </TextBlock>
        </StackPanel>
    </Window>

    ```
### 复杂属性

&emsp;&emsp;虽然类型转换器便于使用，但他们不能解决所有的实际问题。例如，有些属性是完备的对象，这些对象具有自己的一组属性。幸运的是，XAML提供了另一种选择：属性元素语法（property-element syntax）。使用属性元素语法，可添加名称形式为Parent.PropertyName的子元素。

```XML
<Grid>
    <Grid.Background>
        <LinearGradientBrush StartPoint="0 0" EndPoint="1 0">
            <GradientStop Color="Red" Offset="0"/>
            <GradientStop Color="Yellow" Offset="0.5"/>
            <GradientStop Color="Blue" Offset="1"/>
        </LinearGradientBrush>
    </Grid.Background>
</Grid>
```
### 标记扩展

&emsp;&emsp;标记扩展可用于嵌套标签或XML特性中（用于XML特性的情况更常见）。标记扩展使用{标记扩展类 参数}语法。

    定义枚举扩展，以Extension结尾在使用时可省略后缀

```C#
public class EnumExtension : MarkupExtension
{
    private readonly Type _type;

    public EnumExtension(Type type)
    {
        if (type == null || !type.IsEnum)
        {
            throw new ArgumentNullException("Enum type is required.");
        }
        _type = type;
    }

    public override object ProvideValue(IServiceProvider serviceProvider)
    {
        var result = new List<KeyValuePair<string, Enum>>();
        var values = _type.GetEnumValues().Cast<Enum>();
        foreach (var value in values)
        {
            var field = _type.GetField(value.ToString());
            var attributes = field?.GetCustomAttributes(typeof(DescriptionAttribute), inherit: true);
            var desc = attributes?.Length > 0 ? ((DescriptionAttribute)attributes[0]).Description : value.ToString();
            result.Add(new KeyValuePair<string, Enum>(desc, value));
        }

        return result;
    }
}
```
    使用自定义枚举扩展，返回枚举类型的所有Description

```XML
<Window 
        xmlns:extensions="clr-namespace:WpfApp.Extensions"
        xmlns:constants="clr-namespace:WpfApp.Constants">
    <Grid>
        <ComboBox Width="200" Height="35" ItemsSource="{extensions:Enum constants:WeekType}" DisplayMemberPath="Key" SelectedValuePath="Value"/>
    </Grid>
</Window>
```

### 附加属性

&emsp;&emsp;附加属性是可用于多个控件但在另一个类中定义的属性。在WPF中，附加属性常用于控件布局。附加属性始终使用包含两个部分的命名形式：定义类型、属性名。这种包含两个部分的命名语法使XAML解析器能够区分开普通属性和附加属性

    通过附加属性绑定Password（由于安全考虑Password属性不是依赖属性，因此不能直接绑定）

```C#
public class PasswordBoxHelper
{
    #region Attach
    public static bool GetAttach(DependencyObject obj)
    {
        return (bool)obj.GetValue(AttachProperty);
    }

    public static void SetAttach(DependencyObject obj, bool value)
    {
        obj.SetValue(AttachProperty, value);
    }

    /// <summary>
    /// 指示是否启用密码绑定功能
    /// </summary>
    public static readonly DependencyProperty AttachProperty =
        DependencyProperty.RegisterAttached("Attach", typeof(bool), typeof(PasswordBoxHelper), new PropertyMetadata(false, OnAttachChanged));
    #endregion

    #region Password
    public static string GetPassword(DependencyObject obj)
    {
        return (string)obj.GetValue(PasswordProperty);
    }

    public static void SetPassword(DependencyObject obj, string value)
    {
        obj.SetValue(PasswordProperty, value);
    }

    /// <summary>
    /// 密码
    /// </summary>
    public static readonly DependencyProperty PasswordProperty =
        DependencyProperty.RegisterAttached("Password", typeof(string), typeof(PasswordBoxHelper), new PropertyMetadata(string.Empty, OnPasswordChanged));
    #endregion

    #region IsUpdating
    private static bool GetIsUpdating(DependencyObject obj)
    {
        return (bool)obj.GetValue(IsUpdatingProperty);
    }

    private static void SetIsUpdating(DependencyObject obj, bool value)
    {
        obj.SetValue(IsUpdatingProperty, value);
    }

    /// <summary>
    /// 是由静态依赖属性“IsUpdating”，用于跟踪PasswordBox密码更新状态
    /// </summary>
    private static readonly DependencyProperty IsUpdatingProperty =
        DependencyProperty.RegisterAttached("IsUpdating", typeof(bool), typeof(PasswordBoxHelper), new PropertyMetadata(false));
    #endregion

    private static void OnAttachChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        if (d is PasswordBox passwordBox)
        {
            if ((bool)e.OldValue)
            {
                passwordBox.PasswordChanged -= PasswordBox_PasswordChanged;
            }
            if ((bool)e.NewValue)
            {
                passwordBox.PasswordChanged += PasswordBox_PasswordChanged;
            }
        }
    }

    private static void OnPasswordChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        if (d is PasswordBox passwordBox)
        {
            passwordBox.PasswordChanged -= PasswordBox_PasswordChanged;
            // 防止在更新过程中引发无限循环
            if (!GetIsUpdating(passwordBox))
            {
                passwordBox.Password = (string)e.NewValue;
            }
            passwordBox.PasswordChanged += PasswordBox_PasswordChanged;
        }
    }

    private static void PasswordBox_PasswordChanged(object sender, RoutedEventArgs e)
    {
        if (sender is PasswordBox passwordBox)
        {
            SetIsUpdating(passwordBox, true);
            SetPassword(passwordBox, passwordBox.Password);
            SetIsUpdating(passwordBox, false);
        }
    }
}
```
    登录表单绑定Password

```XML
<Window 
        xmlns:extensions="clr-namespace:WpfApp.Extensions">
    <Window.DataContext>
        <local:MainWindowViewModel/>
    </Window.DataContext>
    <Grid>
        <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">
            <WrapPanel>
                <TextBlock Text="用户名：" VerticalAlignment="Center" Width="50"/>
                <TextBox Text="{Binding UserName}" Width="120" Height="30"/>
            </WrapPanel>
            <WrapPanel Margin="0 10 0 0">
                <TextBlock Text="密码：" VerticalAlignment="Center" Width="50"/>
                <PasswordBox 
                    extensions:PasswordBoxHelper.Attach="True"
                    extensions:PasswordBoxHelper.Password="{Binding Password,Mode=TwoWay,UpdateSourceTrigger=PropertyChanged}" 
                    Width="120" Height="30"/>
            </WrapPanel>
            <WrapPanel HorizontalAlignment="Center" Margin="0 10 0 0">
                <Button Command="{Binding LoginCommand}" Content="登录" Width="50"/>
            </WrapPanel>
        </StackPanel>
    </Grid>
</Window>
```
### 特殊字符与空白

&emsp;&emsp;XAML受到XML规则限制部分字符无法直接显示

|特殊字符|字符实体|
|-------|------|
|<center>小于号(<)</center>|<center>`&lt;`</center>|
|<center>大于号(>)</center>|<center>`&gt;`</center>|
|<center>&符号(&)</center>|<center>`&amp;`</center>|
|<center>引号(")</center>|<center>`&quot;`</center>|

### 字符串格式化

```XML
<!-- 时间格式化 -->
<TextBlock Text="{Binding CreateTime,StringFormat={}{0:yyyy/MM/dd HH:mm:ss}}"/>
<!-- 小数格式化 -->
<TextBlock Text="{Binding Money,StringFormat={}{0:F2}}"/>
<!-- 货币格式(前面加上$) -->
<TextBlock Text="{Binding Money,StringFormat={}{0:C2}}"/>
<!-- 数字格式，使用千位分隔符，保留2位小数。 -->
<TextBlock Text="{Binding Money,StringFormat={}{0:N2}}"/>
```