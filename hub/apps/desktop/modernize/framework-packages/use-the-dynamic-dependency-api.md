---
description: Learn about how to use the dynamic dependency API.
title: Use the dynamic dependency API to reference framework packages at run time
ms.topic: article
ms.date: 09/18/2021
ms.localizationpriority: medium
---

# Use the dynamic dependency API to reference framework packages at run time

The *dynamic dependency API* enables unpackaged apps (that is, apps that do not use MSIX for their deployment technology) to reference and use framework packages such as [WinUI2](../../../winui/winui2/index.md) and the DirectX Runtime. For more information about framework package dependencies, see [MSIX framework packages and dynamic dependencies](framework-packages-overview.md).

Specifically, the dynamic dependency API provides ways to manage the *install-time references* and *run-time references* for framework packages. For more information about these types of references, see [Servicing model for framework packages](framework-packages-overview.md#servicing-model-for-framework-packages). 

> [!NOTE]
> Unlike other framework packages, you cannot use the dynamic dependency API to reference the [Windows App SDK framework package](../../../windows-app-sdk/deployment-architecture.md#framework-packages-for-packaged-and-unpackaged-apps) in your unpackaged app. Instead, you must use the [bootstrapper API](/windows/windows-app-sdk/api/win32/_bootstrap/) provided by the Windows App SDK. The bootstrapper API is a specialized form of the dynamic dependency API that is designed to take dependencies on the Windows App SDK framework package. For more information, see [Reference the Windows App SDK framework package at run time](../../../windows-app-sdk/reference-framework-package-run-time.md).

## Use the dynamic dependency API

There are two implementations of the dynamic dependency API that you can choose from, depending on your target platform and scenario:

- The Windows App SDK provides both C/C++ functions (in [msixdynamicdependency.h](/windows/windows-app-sdk/api/win32/msixdynamicdependency)) and WinRT types (in the [Microsoft.Windows.ApplicationModel.DynamicDependency namespace](/windows/windows-app-sdk/api/winrt/microsoft.windows.applicationmodel.dynamicdependency)) that implement the dynamic dependency API. This implementation of the API can be used by apps that target Windows 10 version 1809 and later as well as Windows 11, but it has [some limitations](#dynamic-dependency-api-limitations-in-the-windows-app-sdk).
- Windows 11 also provides C/C++ functions that implement the dynamic dependency API (in [appmodel.h](/windows/win32/api/appmodel)). This implementation of the API can be used only by apps that target Windows 11.

To use the dynamic dependency API in your unpackaged app to take a dependency on a framework package, follow this general pattern in your code.

### 1. Create an install-time reference

In your app's installer or during the first run of your app, call one of the following functions or methods to specify a set of criteria for the framework package you want to use. This informs the OS that your app has a dependency upon a framework package that meets the specified criteria. If one or more framework packages are installed that meet the criteria, Windows will ensure that at least one of these framework packages will remain installed until the install-time reference is deleted.

- Windows 11 (C/C++): [TryCreatePackageDependency](/windows/win32/api/appmodel/nf-appmodel-trycreatepackagedependency)
- Windows App SDK (C/C++): [MddTryCreatePackageDependency](/windows/windows-app-sdk/api/win32/msixdynamicdependency/nf-msixdynamicdependency-mddtrycreatepackagedependency)
- Windows App SDK (WinRT): [PackageDependency.Create](/windows/windows-app-sdk/api/winrt/microsoft.windows.applicationmodel.dynamicdependency.packagedependency.create)

The criteria you specify includes the package family name, minimum version, and architectures, but you cannot indicate a specific framework package. When you add a run-time reference to the framework package, the API chooses the highest version that satisfies the specified criteria.

You must also specify a *lifetime artifact*, which can be the current process, a file, or a registry key that indicates to the system that the app is still available. If the specified artifact no longer exists, the OS can assume the dependency is no longer needed and it can uninstall the framework package if no other apps have declared a dependency on it. This feature is useful for scenarios where an app neglects to remove the install-time pin when it is uninstalled.

This API returns a dependency ID that must to be used in other calls to create run-time references and to delete the install-time reference.

### 2. Add a run-time reference

When your app needs to use the framework package, call one of the following functions or methods to request access to the specified framework package and add a run-time reference for it. Calling this API informs the OS that the framework package is in active use and to handle any version updates in a side-by-side manner (effectively delay uninstalling or otherwise servicing the older version until after the app is done using it). If successful, the app may activate classes and use content from the framework package.

- Windows 11 (C/C++): [AddPackageDependency](/windows/win32/api/appmodel/nf-appmodel-addpackagedependency)
- Windows App SDK (C/C++): [MddAddPackageDependency](/windows/windows-app-sdk/api/win32/msixdynamicdependency/nf-msixdynamicdependency-mddaddpackagedependency)
- Windows App SDK (WinRT): [PackageDependency.Add](/windows/windows-app-sdk/api/winrt/microsoft.windows.applicationmodel.dynamicdependency.packagedependency.add)

When you call this API, you must pass in the dependency ID that was returned when you created the install-time reference and the desired rank to use for the framework package in the process's package graph. This API returns the full name of the framework package that was referenced, and a handle that is used to keep track of the active-use dependency. If there are multiple framework packages installed that meet the criteria that you specified when you created the install-time reference, the API chooses highest version that satisfies the criteria.

### 3. Remove the run-time reference

When your app is done using the framework package, call one of the following functions or methods to remove the run-time reference. Typically, your app will call this API during shut down. This API informs the OS that it is safe to remove any unnecessary versions of the framework package.

- Windows 11 (C/C++): [RemovePackageDependency](/windows/win32/api/appmodel/nf-appmodel-removepackagedependency)
- Windows App SDK (C/C++): [MddRemovePackageDependency](/windows/windows-app-sdk/api/win32/msixdynamicdependency/nf-msixdynamicdependency-mddremovepackagedependency)
- Windows App SDK (WinRT): [PackageDependencyContext.Remove](/windows/windows-app-sdk/api/winrt/microsoft.windows.applicationmodel.dynamicdependency.packagedependencycontext.remove)

When you call this API, you must pass in the handle that was returned when you added the run-time reference.

### 4. Delete the install-time reference

When your app is uninstalled, call one of the following functions or methods during unto delete the install-time reference. This API informs the OS that it is safe to remove the framework package if no other apps have a dependency on it.

- Windows 11 (C/C++): [DeletePackageDependency](/windows/win32/api/appmodel/nf-appmodel-deletepackagedependency)
- Windows App SDK (C/C++): [MddDeletePackageDependency](/windows/windows-app-sdk/api/win32/msixdynamicdependency/nf-msixdynamicdependency-mdddeletepackagedependency)
- Windows App SDK (WinRT): [PackageDependency.Delete](/windows/windows-app-sdk/api/winrt/microsoft.windows.applicationmodel.dynamicdependency.packagedependency.delete)

When you call this API, you must pass in the dependency ID that was returned when you created the install-time reference.

## Dynamic dependency API limitations in the Windows App SDK

When you use the dynamic dependency API in the Windows App SDK to take a dependency on a framework package, this API requires help via another installed package and running process to inform Windows that the framework package is in use and to block servicing the framework while it is being used. This is called a *lifetime manager* component.

The Windows App SDK provides a lifetime manager component for its framework package called the [Dynamic Dependency Lifetime Manager (DDLM)](../../../windows-app-sdk/deployment-architecture.md#dynamic-dependency-lifetime-manager-ddlm). However, no other framework packages currently provide a similar lifetime manager component from Microsoft.

The dynamic dependency API implementation in Windows 11 (in [appmodel.h](/windows/win32/api/appmodel)) does not have this limitation.

## Related topics

- [MSIX framework packages](framework-packages-overview.md)
- [Dynamic dependencies specification](https://github.com/microsoft/WindowsAppSDK/blob/main/specs/dynamicdependencies/DynamicDependencies.md)
- [Runtime architecture and deployment scenarios for the Windows App SDK](../../../windows-app-sdk/deployment-architecture.md)
- [Reference the Windows App SDK framework package at run time](../../../windows-app-sdk/reference-framework-package-run-time.md)