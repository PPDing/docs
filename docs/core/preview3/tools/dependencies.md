---
title: Managing dependencies in .NET Core Preview 3 tooling
description: Preview 3 brings about changes to how dependencies are managed
keywords: CLI, extensibility, custom commands, .NET Core
author: blackdwarf
manager: wpickett
ms.date: 11/12/2016
ms.topic: article
ms.prod: .net-core
ms.technology: .net-core-technologies
ms.devlang: dotnet
ms.assetid: 74b87cdb-a244-4c13-908c-539118bfeef9
---

Managing dependencies in .NET Core Preview 3 tooling
----------------------------------------------------

# Overview
With the move of .NET Core projects from project.json to csproj and MSBuild, a significant invesment also happened that resulted in unification of the project file and assets that allow tracking of depenencies. For .NET Core projects this is similar to what project.json did. There is no separate JSON or XML file that tracks NuGet dependencies. With this change, we've also introduced another type of *reference* into the csproj syntax called the `<PackageReference>`. 

This document describes the new reference type. It also shows how to add a package dependency using this new reference type to your project. 

# The new <PackageReference> element
The `<PackageReference>` has the following basic structure:

```xml
<PackageReference Include="<Package_ID>">
    <Version>PACKAGE_VERSION</Version>
</PackageReference>
```

If you are familiar with MSBuild, it will look familiar to the other [reference types]() that already exist. The key is the `Include` statement which specifies the package id that you wish to add to the project. The `<Version>` child element specified the version to get. The versions are specified as per [NuGet version rules](https://docs.nuget.org/ndocs/create-packages/dependency-versions#version-ranges).  

> **Note:** if you are not faimilar with the overall `csproj` syntax, you can use the [MSBuild project reference documentation]() to get acquainted.  

Adding a dependency that is available only in a specific target is done using conditions:

```xml
<PackageReference Include="<Package_ID>" Condition="'$(TargetFramework)' == 'netcoreapp1.0'">
    <Version>PACKAGE_VERSION</Version>
</PackageReference>
```

The above means that the dependency will only be valid if the build is happening for that given target. The `$(TargetFramework)` in the condition is a MSBuild property that is being set in the project. For most common .NET Core applications, you will not need to do this. 

# Adding a dependency to your project
Adding a dependency to your project is straightforward. Here is an example of how to add `JSON.net` version `9.0.1` to your project. Of course, it is applicable to any other NuGet dependency. 

When you open your project file, you will see two or more `<ItemGroup>` nodes. You will notice that one of the nodes already has `<PackageReference>` elements in it. You can add your new dependency to this node, or create a new one; it is completely up to you as the result will be the same. 

In this example we will use the default template that is dropped by `dotnet new`. This is a simple console application. When we open up the project, we first find the `<ItemGroup>` with already existing `<PackageReference>` in it. We then add the following to it:

```xml
<PackageReference Include="Newtonsoft.Json">
    <Version>9.0.1</Version>
</PackageReference>
```
After this, we save the project and run the `dotnet restore` command to install the dependency. 

The full project looks like this:

```xml
<Project ToolsVersion="15.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Import Project="$(MSBuildExtensionsPath)\$(MSBuildToolsVersion)\Microsoft.Common.props" />
  
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp1.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <Compile Include="**\*.cs" />
    <EmbeddedResource Include="**\*.resx" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.NETCore.App">
      <Version>1.0.1</Version>
    </PackageReference>
    <PackageReference Include="Newtonsoft.Json">
        <Version>9.0.1</Version>
    </PackageReference>
    <PackageReference Include="Microsoft.NET.Sdk">
      <Version>1.0.0-alpha-20161104-2</Version>
      <PrivateAssets>All</PrivateAssets>
    </PackageReference>
  </ItemGroup>
  
  <Import Project="$(MSBuildToolsPath)\Microsoft.CSharp.targets" />
</Project>
```

# Removing a dependency from the project
Removing a dependency from the project file involves simply removing the `<PackageReference>` from the project file. 


