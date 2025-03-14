---
description: Build a C# .NET 5 and C++ desktop (Win32) application with WinUI 3 and basic Win32 interop capabilities using the Platform Invocation Services, or PInvoke.
title: Build a C# .NET 5 app with WinUI 3 and Win32 interop
ms.date: 08/02/2021
ms.topic: article
keywords: windows 10, windows 11, uwp, COM, win32, winui, interop
ms.author: kbridge
author: Karl-Bridge-Microsoft
ms.localizationpriority: high
ms.custom: 19H1
---

# Build a C# .NET 5 app with WinUI 3 and Win32 interop

In this article, we step through how to build a basic **C# .NET 5** application with WinUI 3 and Win32 interop capabilities using Platform Invocation Services ([PInvoke](https://github.com/dotnet/pinvoke)).

## Prerequisites

1. Set up your development environment and install the latest Windows App SDK VSIX from [Install developer tools](../../windows-app-sdk/set-up-your-development-environment.md#4-install-the-windows-app-sdk-extension-for-visual-studio).
2. Test your configuration by [creating a new project that uses the Windows App SDK](create-your-first-winui3-app.md). 

## Basic managed C#/.NET 5 app

For this example, we'll specify the location and size of the app window, convert and scale it for the appropriate DPI, disable the window minimize and maximize buttons, and finally query the current process to show a list of modules loaded in the current process.

We're going to build our example app from the initial template application (see [Prerequisites](#prerequisites)), but before we start, lets have a look at the structure and content of the template app.

### The application solution

By default, the Visual Studio Solution for an app contains two projects: The application itself, and another for creating an [MSIX](/windows/msix) app package (see [WinUI 3 project templates in Visual Studio](winui-project-templates-in-visual-studio.md) for more details).

:::image type="content" source="images/build-basic/solution-explorer-pinvoke.png" alt-text="Screenshot of the Visual Studio Solution Explorer showing the file structure of the initial template app.":::<br/>*Solution Explorer showing the file structure of the initial template app.*

> [!NOTE]
> At this time, only packaged apps can be deployed using the [Stable release channel for the Windows App SDK](../../windows-app-sdk/stable-channel.md).
>
> [Deploying unpackaged apps that use the Windows App SDK](../../windows-app-sdk/deploy-unpackaged-apps.md) is supported in the [Experimental release channel for the Windows App SDK](../../windows-app-sdk/experimental-channel.md).

### The MainWindow.xaml file

With WinUI 3, you can now create instances of the [Window](/windows/winui/api/microsoft.ui.xaml.window) class in XAML markup.

The XAML [Window](/windows/winui/api/microsoft.ui.xaml.window) class has been extended to support desktop windows, turning it into an abstraction of each of the low-level window implementations used by the UWP and desktop app models. Specifically, CoreWindow for UWP and window handles (or HWNDs) for Win32.

The following code shows the MainWindow.xaml file from the initial template app, which uses the [Window](/windows/winui/api/microsoft.ui.xaml.window) class as the root element for the app.

```xaml
<Window
    x:Class="WinUI_3_basic_win32_interop.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:WinUI_3_basic_win32_interop"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <StackPanel Orientation="Horizontal" HorizontalAlignment="Center" VerticalAlignment="Center">
        <Button x:Name="myButton" Click="myButton_Click">Click Me</Button>
    </StackPanel>
</Window>
```

### Configuration

1. To call Win32 APIs exposed in User32.dll, add the open source [**PInvoke.User32** NuGet package](https://github.com/dotnet/pinvoke) to the VS project (from the Visual Studio menus, select **Tools -> NuGet Package Manager -> Manage NuGet Packages for Solution...** and search for "Pinvoke.User32"). For more details, see [Calling Native Functions from Managed Code](/cpp/dotnet/calling-native-functions-from-managed-code).

   :::image type="content" source="images/build-basic/nuget-pkg-manager-pinvoke.png" alt-text="Screenshot of the Visual Studio NuGet Package Manager with PInvoke.User32 selected.":::<br/>*NuGet Package Manager with PInvoke.User32 selected.*

   Confirm installation was successful by checking the **Packages** folder in the VS project.

   :::image type="content" source="images/build-basic/solution-explorer-packages-pinvoke.png" alt-text="Screenshot of the Visual Studio Solution Explorer Packages with PInvoke.User32.":::<br/>*Solution Explorer Packages with PInvoke.User32.*

   Next, double-click the application project file (or right click and select "Edit project file") to open the file in a text editor and confirm the project file now includes the NuGet `PackageReference` for "PInvoke.User32".

   :::code language="xml" source="samples/WinUI-3-basic-win32-interop/WinUI-3-basic-win32-interop/WinUI-3-basic-win32-interop.csproj" highlight="17":::

### Code

1. In the App.xaml.cs code-behind file, we get a handle to the [Window](/windows/winui/api/microsoft.ui.xaml.window) using the [WindowNative.GetWindowHandle](/windows/windows-app-sdk/api/win32/microsoft.ui.xaml.window/nf-microsoft-ui-xaml-window-iwindownative-get_windowhandle) WinRT COM interop method (see [Call WinRT COM interop interfaces from .NET 5+ apps](/windows/apps/desktop/modernize/winrt-com-interop-csharp)).

   This method is called from the app's [OnLaunched](/windows/winui/api/microsoft.ui.xaml.application.onlaunched) handler, as shown here:

   :::code language="csharp" source="samples/WinUI-3-basic-win32-interop/WinUI-3-basic-win32-interop/app.xaml.cs" id="OnLaunched":::

1. We then call a `SetWindowDetails` method, passing the Window handle and preferred dimensions.

   In this method:

   - We call [GetDpiForWindow](/windows/win32/api/winuser/nf-winuser-getdpiforwindow) to get the the dots per inch (dpi) value for the window (Win32 uses actual pixels while WinUI 3 uses effective pixels). This dpi value is used to calculate the scale factor and apply it to the width and height specified for the window.
   - We then call [SetWindowPos](/windows/win32/api/winuser/nf-winuser-setwindowpos) to specify the desired location of the window.
   - Finally, we call [SetWindowLong](/windows/win32/api/winuser/nf-winuser-setwindowlongw) to disable the *Minimize* and *Maximize* buttons.

   :::code language="csharp" source="samples/WinUI-3-basic-win32-interop/WinUI-3-basic-win32-interop/app.xaml.cs" id="SetWindowDetails" highlight="3,8,11":::

1. In the MainWindow.xaml file, we use a [ContentDialog](/windows/winui/api/microsoft.ui.xaml.controls.contentdialog) with a [ScrollViewer](/windows/winui/api/microsoft.ui.xaml.controls.scrollviewer) to display a list of all the modules loaded for the current process.

   :::code language="xaml" source="samples/WinUI-3-basic-win32-interop/WinUI-3-basic-win32-interop/MainWindow.xaml" range="10-19":::

1. In the MainWindow.xaml.cs code-behind file, we replace the `MyButton_Click` event handler with the following code.

   Here, we get a reference to the current process by calling [GetCurrentProcess](/dotnet/api/system.diagnostics.process.getcurrentprocess). We then iterate through the collection of [Modules](/dotnet/api/system.diagnostics.process.modules) and append the filename of each [ProcessModule](/dotnet/api/system.diagnostics.processmodule) to our display string.

   :::code language="csharp" source="samples/WinUI-3-basic-win32-interop/WinUI-3-basic-win32-interop/MainWindow.xaml.cs" id="myButton_Click":::

1. Compile and run the app.
1. After the window appears, select the "Display loaded modules" button.

   :::image type="content" source="images/build-basic/runtime-app-process-modules.png" alt-text="Screenshot of the basic Win32 interop application described in this topic.":::<br/>*The basic Win32 interop application described in this topic.*

## Summary

In this topic we covered accessing the underlying window implementation (in this case Win32 and HWNDs) and using Win32 APIs along with the WinRT APIs. This demonstrates how you can use existing desktop application code when creating new WinUI 3 desktop apps.

For a more extensive sample, see the [AppWindow gallery sample](https://github.com/microsoft/WindowsAppSDK-Samples/tree/main/Samples/Windowing) in the [Windows App SDK Samples](https://github.com/microsoft/WindowsAppSDK-Samples) GitHub repo.

## See also

- [Windows App SDK](../../windows-app-sdk/index.md)
- [Stable release channel for the Windows App SDK](../../windows-app-sdk/stable-channel.md)
- [Manage app windows](../../windows-app-sdk/windowing/windowing-overview.md)
- [Windows App SDK Samples](https://github.com/microsoft/WindowsAppSDK-Samples)
