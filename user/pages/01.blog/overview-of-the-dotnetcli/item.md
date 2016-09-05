---
title: Overview of the dotnet-cli
date: 17:34 04/11/2016
taxonomy:
    category: blog
    tag: [dotnet]
---

If you have been using .NET Core or ASP.NET Core beta, you've probably made friends with `dnvm`, `dnu`, and `dnx`. These are the command line tools used to build and run .NET applications. It's a bit confusing that there are three different commands though, isn't it?

`dnu` is the Microsoft .NET Development Utility and is used to install dependencies, build, and package your app.

`dnx` is the Microsoft Execution environment and is used to run your app on the version of execution environment you're using, which is manged with `dnvm`.

I'm very happy that the team has decided to consolidate these three tools into one tool - the `dotnet` cli. This will happen in the rc2 release of ASP.NET Core and the former tools will no longer be needed. You can read about the reasoning on in the dotnet/cli repo [here](https://github.com/dotnet/cli/blob/rel/1.0.0/Documentation/intro-to-cli.md) and Scott Hanselman's excellent [blog post](http://www.hanselman.com/blog/ExploringTheNewNETDotnetCommandLineInterfaceCLI.aspx).

At the time of this writing, these are the available commands:

* `dotnet new`
* `dotnet restore`
* `dotnet build`
* `dotnet publish`
* `dotnet run`
* `dotnet repl`
* `dotnet pack`

There is also the `dotnet compile` and `dotnet test` commands, however from what I can tell these are only available on Windows at this time.

`dotnet compile` will bring with it the ability to compile either to IL or a native binary, which will be incredibly powerful.

You can get a Hello World dotnet program up and going in minutes on Windows, Linux, or OS X by using `dotnet new`, `dotnet restore`, and `dotnet run`. I imagine soon we will have the ability to do things like `dotnet new web` to scaffold out a simple web application, similar to the existing yeoman generator.

You can track progress of commands as they're being worked on at https://github.com/dotnet/cli/tree/rel/1.0.0/src/dotnet.

### Local commands

The feature of the cli that I am most excited about is the ability to add new commands locally. This is done by simply placing an executable on the $PATH with the naming convention of `dotnet-{command}`. Any options are also passed to the executable. It's sort of like a global version of npm scripts.

This is exciting to me because it opens up the possibility for a yeoman-like registry of dotnet utilities created by users. These utilities could be stored in nuget, and be used for all kinds of applications!

It's still early days, but the potential is certainly exciting.
