---
title: "How to Setup and Use NDepend on macOS for .NET Development"
seoTitle: "How to Set Up and Use NDepend on macOS for .NET | Step-by-Step Guide"
seoDescription: "Learn how to set up and use NDepend on macOS for .NET projects. A step-by-step guide to running static code analysis, improving code quality, and exploring "
datePublished: Tue Dec 10 2024 21:35:14 GMT+0000 (Coordinated Universal Time)
cuid: cm4izcsw7000d08l9g8b26ycp
slug: how-to-setup-and-use-ndepend-on-macos-for-net-development
tags: dotnet, ndepend

---

Are you a .NET developer working on macOS, striving for high-quality code? If so, NDepend is one tool you should not overlook. In this post, we'll cover everything from setting it up to running your first analysis. Let's get started!

My recent exploration of [NDepend](https://www.ndepend.com/), thanks to an invitation from Patrick at NDepend, sparked my interest in its potential to enhance my codebase. As a Software Architect, I often use static code analysis tools to enhance code quality, and NDepend didn't disappoint.

## Getting Started on Mac

### Step 1: Download NDepend

Visit the official [NDepend](https://www.ndepend.com/) website and download the macOS version. Your download will be a ZIP file containing everything you need.

### Step 2: Extract and Set Up

Unzip the downloaded file to a location of your choice. Let's say we've extracted it to `~/NDepend` for this guide. All further CLI commands will work as if you extracted NDepend to this path.

### Step 3: Install .NET SDK (If You Haven't Already)

NDepend requires the .NET SDK to run. If you don't have it installed, you can download it from the [.NET website](https://dotnet.microsoft.com/download). Follow the installation instructions specific to macOS.

### Step 4: Register Evaluation or a license

This can be done using the `dotnet` command with the path to your NDepend file.

```bash
dotnet ~/Ndepend/net8.0/NDepend.Console.MultiOS.dll --RegEval
```

After you should see a message like this:

```bash
Evaluation registered on this machine

Evaluation 14 days left
```

## Creating a NDepend Project

Recently, I decided to try NDepend on my [VerticalSliceArchitecture](https://github.com/nadirbad/VerticalSliceArchitecture) project. I was pleasantly surprised by the detailed HTML report I received within seconds. The steps I followed were pretty straightforward.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXc3QIjq0UPmgdCX_XEzHstgU2iXUvCTpuiyYyksUxSYzp9Z7SJU-bL-C9Mv1Woizn6XrhLZBB9iWNygzLWWzV6q0-nGeDtRo7zwjECmAimtJVng-Lzf8p5ptxDfTgr5eEphZJpFqg?key=AAOimGJbPO2bMmy0WvPuizBc align="left")

With NDepend and the .NET SDK installed, you're ready to create your first NDepend project. Here's how to do it:

### Step 1: Open Terminal

Fire up your Terminal app. We'll be using the command line to run NDepend.

### Step 2: Create a NDepend project file

NDepend uses project files (.ndproj) to know what assemblies to analyze. `NDepend.Console.MultiOS.dll` can be used to create an NDepend project file (.ndproj extension). I created a project by pointing at my solution file e.g.:

```bash
dotnet ~/NDepend/net8.0/NDepend.Console.MultiOS.dll --CreateProject ./VerticalSliceArchitecture.ndproj ~/code/VerticalSliceArchitecture/VerticalSliceArchitecture.sln
```

If successful, you’ll get a confirmation message and the default NDepend project file will be created for analysis of your solution or project.

## Running the Analysis

Follow these steps to run your analysis and review the results.

### Step 1: Run NDepend Analysis

From the project directory, I run the following command:

```bash
dotnet ~/NDepend/net8.0/NDepend.Console.MultiOS.dll VerticalSliceArchitecture.ndproj
```

Replace `VerticalSliceArchitecture.ndproj` with the path to your NDepend project file.

### Step 2: Review the Results

Once the analysis completes, NDepend will generate a nice HTML report in the NDependOut folder. Open the `/NDependOut/NDependReport.html` report in your browser to explore various metrics, code rules, and potential issues found in your codebase. **Since my VerticalSliceArchitecture project code broke some rules, the command line exited non-zero.** This is great for build integration where I want to fail if rules get violated.

## Potential Enhancement

I believe there's always room for improvement. Here are a few suggestions that could refine the user experience further.

### Standardize Installation

Incorporating an installation method that aligns with usage of package managers, like Homebrew, or maybe if it is available as [dotnet tool](https://learn.microsoft.com/en-us/dotnet/core/tools/global-tools) would be great. For example executing `dotnet tool install ndepend-tool -g` sounds quite handy.

### Simplifying Command Line Usage

Running the cross-platform executable currently requires a somewhat lengthy command:

```bash
dotnet ~/path/to/net8.0/NDepend.Console.MultiOS.dll
```

The command line executable is used for various actions like creating projects, registering licenses, and running analyses. It could be more intuitive if commands followed a hierarchical command structure e.g. `<tool> <command> [subcommand] [options]`.

These suggestions aim to enhance the efficiency and user experience of NDepend, and that usage is intuitive and self-explanatory.

### Missing Exploring Code Architecture Diagrams on macOS

If you are working on Windows and using Visual Studio you will benefit of having graphs and code architecture diagrams like this [dependency graph](https://www.ndepend.com/docs/visual-studio-dependency-graph). These graphs are really helpful, and they helped me to improve code architecture of [VerticalSliceArchitecture]([https://github.com/nadirbad/VerticalSliceArchitecture](https://github.com/nadirbad/VerticalSliceArchitecture)) and resolve coupling issues, as noted here on \[NDepend blog on comparing Clean Architecture and Vertical Slice approach\]([https://blog.ndepend.com/vertical-slice-architecture-in-asp-net-core/#Note\_on\_this\_implementation](https://blog.ndepend.com/vertical-slice-architecture-in-asp-net-core/#Note_on_this_implementation)). Here is the example of dependency graph of VerticalSliceArchitecture project:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733510542365/50fdb9db-0e28-459f-ac0e-078a569897ea.webp align="center")

## Conclusion

And there you have it! You've successfully set up NDepend on your Mac, analyzed your .NET codebase.

NDepend is a powerful tool in maintaining high-quality code. By regularly analyzing your code, you can catch issues early, adhere to best practices. The tool's analysis capabilities and detailed reports ensure your code remains clean, efficient, well-structured.

While there are opportunities for enhancing the tool's user experience, such as streamlining installation and command structures, availability of graphs in HTML reports, NDepend remains a robust tool in the pursuit of code excellence.

## Bonus: Explore NDepend in Action with VerticalSliceArchitecture

If you're looking for a practical example of setting up and running NDepend, check out my [VerticalSliceArchitecture GitHub repository](https://github.com/nadirbad/VerticalSliceArchitecture/blob/ndepend-analysis/ndepend.md).

I've included:

- A complete step-by-step instruction guide for integrating NDepend.
- A helper shell [run-ndepend.sh](https://github.com/nadirbad/VerticalSliceArchitecture/blob/ndepend-analysis/run-ndepend.sh) script to streamline the process.

This example demonstrates how NDepend can be used effectively to analyze and enhance a real-world project.

