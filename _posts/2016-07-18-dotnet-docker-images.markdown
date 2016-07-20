---
layout: post
title:  "Dotnet Docker image usages"
date:   2016-07-18 17:06:58 -0700
categories: docker .NET
---

When using .NET and Docker some folks have been confused by the number of available images. To understand what each images is for, you need to understand the two different ways that you can choose to deploy a .NET Core application as well as some of the different ways people use containers. 

Let's start with the different ways you can deploy .NET apps:

1. Portable
    
    This is the default way to deploy and run an ASP.NET Core application. When you build a portable application then the output of `dotnet publish` is a `.dll` that you can run using `dotnet <appName>.dll`. However, you don't need the entire CLI just to run your already built `.dll`. You don't need restore, build, etc. You only need the dotnet command that can run your app.

2. Standalone
    
    If you choose to make your application standalone, then the output of your build is an executable binary (`.exe`) rather than a `.dll`. The executable is able to run your application and the output of `dotnet publish` includes all the files required to run your app without any dotnet runtime or SDK installed on the machine.

In addition to these two ways of deploying .NET Core applications, you have two common ways that you might want to use a container.

1. Containerizing your build
    
    In this case you are building your application in a container, and may or may not actually run your app in a container later. Typically you need many libraries and utilities inside your build container that you don't need at runtime.
2. Running your application
    
    In this case you are running your application inside a container, and typically want your image focused on just running your app without anything extraneous.

Armed with this information we can talk about the 3 images and what they are for:

- **1.0.0-core-deps**: This image contains only the pre-reqs required by the BCL. These are libraries like libicu, libssl, libunwind, etc. Useful for running standalone apps.
- **1.0.0-core**: This image inherits from `1.0.0-core-deps` and adds the .NET Core runtime. Useful for running portable apps.
- **1.0.0 (latest)**: This image contains the pre-reqs as well as the dotnet SDK and other dependencies that might be needed to build inside a container. Useful to containerize your build.

If you put each of these 4 pivots into a matrix, with the squares of the matrix being the best available Docker image, then you get this:


|            | Running App | Containerizing Build |
|------------|-------------|----------------------|
| **Standalone** | 1.0.0-core-deps | 1.0.0 (latest) |
| **Portable**   | 1.0.0-core | 1.0.0 (latest) |


If you just do `docker pull microsoft/dotnet` then you will get `1.0.0`. The reason for this that the `1.0.0` image is the most versatile, in that it can serve any of the jobs we have talked about so far. If you are running a standalone or portable application then they will hapilly work in the `1.0.0` image, it is just larger then it needs to be. However, if you try to build your app in `1.0.0-core` then you are going to have a hard time.