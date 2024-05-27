---
title: WPF从入门到入坟 - 02XAML
comments: false
date: 2024-05-23 20:12:51
tags:
   - WPF从入门到入坟
categories:
    - C#
    - WPF
keywords: 
    - XAML
description:
top_img:
cover: https://cnlicm-blog-image.oss-cn-shenzhen.aliyuncs.com/img/20240527195528.png
abbrlink:
---



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