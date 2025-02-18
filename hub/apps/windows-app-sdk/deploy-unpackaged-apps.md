---
title: Deploy unpackaged apps that use the Windows App SDK
description: This article provides instructions for deploying unpackaged apps that use the Windows App SDK.
ms.topic: article
ms.date: 05/21/2021
keywords: windows win32, windows app development, Windows App SDK 
ms.author: zafaraj
author: zaryaf
ms.localizationpriority: medium
---

# Deploy unpackaged apps that use the Windows App SDK

This article provides guidance about deploying non-[MSIX](/windows/msix) packaged apps that use the Windows App SDK to other computers.

> [!IMPORTANT]
> Unpackaged app deployment is currently supported in the [preview release channel](preview-channel.md) and [experimental release channel](experimental-channel.md) of the Windows App SDK. This feature is not supported for use by apps in production environments.

## Overview

Before configuring your apps for deployment, review [the Windows App SDK deployment architecture](deployment-architecture.md) to learn more about the dependencies your app takes when it uses the Windows App SDK, including dynamic dependencies.

You can test deployment of unpackaged apps that use the Windows App SDK using the Windows App SDK installer (.exe). The installer contains a copy of the MSIX packages for the Windows App SDK run time components, which includes the [Framework](deployment-architecture.md#framework-packages-for-packaged-and-unpackaged-apps), [Main](deployment-architecture.md#main-package) and [Dynamic Dependency Lifetime Manager (DDLM)](deployment-architecture.md#dynamic-dependency-lifetime-manager-ddlm) packages. By default, the installer will automatically detect the system architecture and install the MSIX packages in either the `X64` or `X86` architectures.

Alternatively, you can directly install the MSIX packages for the Windows App SDK runtime components in your development environment or through your app's setup program.

## Prerequisites

1. Download the 1.0 Preview 1 Windows App SDK [Installer and MSIX packages](https://aka.ms/windowsappsdk/1.0-preview1/msix-installer) to your development computer. For older versions, see [Downloads](downloads.md). 

2. If your development computer or the deployment computer is running Windows 10 version 1909 or an earlier version, make sure sideloading is enabled. Sideloading is automatically enabled on Windows 10 version 2004 and later.

    > [!NOTE]
    > Experimental and Preview versions of the Windows App SDK require that sideloading is enabled to install the runtime.  

    To confirm whether sideloading is enabled on Windows 10 version 1909 and earlier versions:

    1. Open **Settings**.
    2. Click **Update & Security** > **For developers**.
    3. In the **Use developer features** section, make sure the **Sideload apps** or **Developer mode** setting is selected (the **Developer mode** setting includes sideloading as well as other features).

        > [!NOTE]
        > If the computer is managed in an enterprise environment, the computer may have a policy that disables the ability to modify these settings. If so, you may get an error when you or your app tries to install the the Windows App SDK runtime. In this case, you must contact your IT Professional to enable sideloading or **Developer mode**. 

## Run the Windows App SDK installer in your development environment

You can test deployment in your development environment by running the Windows App SDK Installer: 

- **WindowsAppRuntimeInstall.exe** if you are using version 1.0 Preview 1 
- **WindowsAppSDKInstall.exe** if you are using version 1.0 Experimental 
- **ProjectReunionInstall.exe** if you are using version 0.8 Preview 

You can also run the installer with no user interaction and suppress all text output with `WindowsAppRuntimeInstall.exe --quiet` or `WindowsAppSDKInstall.exe --quiet` or `ProjectReunionInstall.exe --quiet`, depending on the Windows App SDK version. 

After the installation is complete, you can run your unpackaged app. For an example of how to build and run an unpackaged app that uses the Windows App SDK, see [Tutorial: Build and deploy an unpackaged app that uses the Windows App SDK](tutorial-unpackaged-deployment.md).

## Launch the Windows App SDK installer from your setup program

To test deployment of your unpackaged app to end users, you can launch the .exe installer from your setup program or MSI by using [ShellExecute](/windows/win32/shell/launch). This is very similar to other runtime installers you may have used, like .NET, Visual C++, or DirectX.

For a code example that demonstrates how to run the Windows App SDK Installer from your setup program, see the **RunInstaller** function in the [installer functional test](https://aka.ms/testruninstaller) for the Windows App SDK.

## Directly deploy the MSIX packages from your setup program

As an alternative to using the Windows App SDK installer for deployment to end users, you can manually deploy the MSIX packages through your app's program or MSI. This option can be best for developers who want more control or need offline installations.

For an example that demonstrates how your setup program can install the MSIX packages, see [install.cpp](https://aka.ms/testinstallpackages) in the Windows App SDK installer code.

## Using the Windows App SDK features at run time

Unpackaged apps must use the *bootstrapper API* before they can use Windows App SDK features such as WinUI, App lifecycle, MRT Core, and DWriteCore. This feature enables unpackaged apps to dynamically take a dependency on the Windows App SDK framework package at run time.

For more information, see [Reference the Windows App SDK framework package at run time](reference-framework-package-run-time.md) and [Tutorial: Build and deploy an unpackaged app that uses the Windows App SDK](tutorial-unpackaged-deployment.md).

## Using features in other framework packages at run time

In addition to the bootstrapper API, the Windows App SDK also provides a broader set of C/C++ functions and WinRT classes that implement the *dynamic dependency API*. This API is designed to be used to reference any framework package dynamically at run time. You only need to use the dynamic dependency API if you want to dynamically reference framework packages other than the Windows App SDK framework package.

For more information, see the [Use MSIX framework packages dynamically from your desktop app](../desktop/modernize/framework-packages/index.md).

## Related topics

- [Runtime architecture and deployment scenarios](deployment-architecture.md)
- [Tutorial: Build and deploy an unpackaged app that uses the Windows App SDK](tutorial-unpackaged-deployment.md)
- [Check for installed versions of the Windows App SDK runtime](check-windows-app-sdk-versions.md)
- [Remove outdated Windows App SDK runtime versions from your development computer](remove-windows-app-sdk-versions.md)
- [Release channels](release-channels.md)
- [Deploy packaged apps that use the Windows App SDK](deploy-packaged-apps.md)
