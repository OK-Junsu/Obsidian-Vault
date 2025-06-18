---
title: "Install tools for the Windows App SDK - Windows apps"
source: "https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/set-up-your-development-environment?tabs=cs-vs-community%2Ccpp-vs-community%2Cvs-2022-17-10%2Cvs-2022-17-1-b"
author:
  - "[[stevewhims]]"
published:
created: 2025-05-16
description: "Configure your development computer with the tools required to build Windows apps with the Windows App SDK] and WinUI 3."
tags:
  - "clippings"
---
## Install tools for the Windows App SDK

Configure your development computer with the tools required to build Windows apps using the [Windows App SDK](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/) (stable release channel) and [WinUI](https://learn.microsoft.com/en-us/windows/apps/winui/winui3/).

Before installing any tools, see [System requirements for Windows app development](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/system-requirements).

**\[Visual Studio 2022 and later\]** Install the required tools and workloads using the console and one of the following commands. These commands will open Visual Studio Installer with any missing workloads selected, for which you can select **Modify** to install the required workloads.

- [Visual Studio Community](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/?tabs=cs-vs-community%2Ccpp-vs-community%2Cvs-2022-17-10%2Cvs-2022-17-1-b#tabpanel_1_cs-vs-community)
- [Visual Studio Enterprise](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/?tabs=cs-vs-community%2Ccpp-vs-community%2Cvs-2022-17-10%2Cvs-2022-17-1-b#tabpanel_1_cs-vs-enterprise)
- [Visual Studio Professional](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/?tabs=cs-vs-community%2Ccpp-vs-community%2Cvs-2022-17-10%2Cvs-2022-17-1-b#tabpanel_1_cs-vs-professional)

```
winget install "Visual Studio Community 2022"  --override "--add Microsoft.VisualStudio.Workload.ManagedDesktop Microsoft.VisualStudio.ComponentGroup.WindowsAppSDK.Cs" -s msstore
```

- [Visual Studio Community](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/?tabs=cs-vs-community%2Ccpp-vs-community%2Cvs-2022-17-10%2Cvs-2022-17-1-b#tabpanel_2_cpp-vs-community)
- [Visual Studio Enterprise](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/?tabs=cs-vs-community%2Ccpp-vs-community%2Cvs-2022-17-10%2Cvs-2022-17-1-b#tabpanel_2_cpp-vs-enterprise)
- [Visual Studio Professional](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/?tabs=cs-vs-community%2Ccpp-vs-community%2Cvs-2022-17-10%2Cvs-2022-17-1-b#tabpanel_2_cpp-vs-professional)

```
winget install "Visual Studio Community 2022"  --override "--add Microsoft.VisualStudio.Workload.NativeDesktop  Microsoft.VisualStudio.ComponentGroup.WindowsAppSDK.Cpp"  -s msstore
```

The following sections describe how to install the required tools and workloads manually.

Use the following links to install Visual Studio 2022 (recommended) or Visual Studio 2019. You can choose between the free Visual Studio Community Edition, Visual Studio Professional, or Visual Studio Enterprise. Before installing either, see [System requirements for Windows app development](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/system-requirements).

[Download Visual Studio 2022](https://learn.microsoft.com/en-us/visualstudio/releases/2022/release-notes)

[Download Visual Studio 2019](https://learn.microsoft.com/en-us/visualstudio/releases/2019/release-notes)

During Visual Studio installation, you have the option to install workloads and components (you can also open the Visual Studio Installer and select **Modify** to add workloads and components after installation). We recommend installing the following:

- [Visual Studio 2022 version 17.10 and later](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/?tabs=cs-vs-community%2Ccpp-vs-community%2Cvs-2022-17-10%2Cvs-2022-17-1-b#tabpanel_3_vs-2022-17-10)
- [Visual Studio 2022 version 17.1 and 17.9](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/?tabs=cs-vs-community%2Ccpp-vs-community%2Cvs-2022-17-10%2Cvs-2022-17-1-b#tabpanel_3_vs-2022-17-1-a)
- [Other Visual Studio versions](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/?tabs=cs-vs-community%2Ccpp-vs-community%2Cvs-2022-17-10%2Cvs-2022-17-1-b#tabpanel_3_vs-other)

From within the Visual Studio Installer app:

- On the **Workloads** tab:
	- For C# app development using the Windows App SDK, select **WinUI application development**.
	- For C++ app development using the Windows App SDK, select **WinUI application development**
		- Then in the **Installation details** pane of the installation dialog box, under the **WinUI application development** node, select **C++ WinUI app development tools**. (This will also select any additional required components.)

The [Windows App SDK](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/) includes Visual Studio project and item templates for creating and developing apps that use the WinUI 3 library to implement the user interface.

- [Visual Studio 2022 version 17.1 and later](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/?tabs=cs-vs-community%2Ccpp-vs-community%2Cvs-2022-17-10%2Cvs-2022-17-1-b#tabpanel_4_vs-2022-17-1-b)
- [Visual Studio 2022 version 17.0](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/?tabs=cs-vs-community%2Ccpp-vs-community%2Cvs-2022-17-10%2Cvs-2022-17-1-b#tabpanel_4_vs-2022-17-0)
- [Visual Studio 2019](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/?tabs=cs-vs-community%2Ccpp-vs-community%2Cvs-2022-17-10%2Cvs-2022-17-1-b#tabpanel_4_vs-2019)

If you followed the instructions in [Required workloads and components](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/?tabs=cs-vs-community%2Ccpp-vs-community%2Cvs-2022-17-10%2Cvs-2022-17-1-b#required-workloads-and-components) above, the templates should already be installed.

Select **C#** or **C++** as the language, **Windows** as the platform, and **WinUI** as the Project type to create a new Windows App SDK project.

Optionally, install [Template Studio for WinUI (C#)](https://marketplace.visualstudio.com/items?itemName=TemplateStudio.TemplateStudioForWinUICs) to accelerate the creation of new.NET WinUI apps using a wizard-based UI. Select from a variety of project types and features to generate a project template customized for you.

For more versions of the Windows App SDK, see [Downloads for the Windows App SDK](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/downloads).

In releases 1.0.3 and 1.1 Preview 2 and later, the Windows App SDK uses Hybrid C/C++ runtime library linkage (hybrid CRT linkage). This is a CRT linkage technique that simplifies deployment. Whether you're a C++ application developer or a C++ library developer, here are some resources for learning about hybrid CRT linkage:

- [Hybrid CRT linkage coding guidelines](https://github.com/microsoft/WindowsAppSDK/blob/main/docs/Coding-Guidelines/HybridCRT.md) on GitHub.
- The hybrid CRT linkage segment of the [WinUI community call](https://www.youtube.com/watch?v=bNHGU6xmUzE&t=977s) from April 20, 2022.

To create your first WinUI 3 app that uses the Windows App SDK, see [Create your first WinUI 3 project](https://learn.microsoft.com/en-us/windows/apps/winui/winui3/create-your-first-winui3-app).

Also see [Use the Windows App SDK in an existing project](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/use-windows-app-sdk-in-existing-project).

- [Windows App SDK](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/)
- [Windows App SDK release channels](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/release-channels)
- [Install tools for preview and experimental channels of the Windows App SDK](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/preview-experimental-install)
- [System requirements for Windows app development](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/system-requirements)
- [Windows App SDK and supported Windows releases](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/support)
- [Downloads for the Windows App SDK](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/downloads)
- [Create your first WinUI 3 project](https://learn.microsoft.com/en-us/windows/apps/winui/winui3/create-your-first-winui3-app)
- [Use the Windows App SDK in an existing project](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/use-windows-app-sdk-in-existing-project)

---

## Additional resources

---

Training

Module

[Use developer tools to extend Power Platform - Training](https://learn.microsoft.com/en-us/training/modules/developer-tools-extend/?source=recommendations)

This module will focus on the available developer tools that can help you perform development activities with Power Platform.

Certification

[Microsoft Certified: Power Platform Developer Associate - Certifications](https://learn.microsoft.com/en-us/credentials/certifications/power-platform-developer-associate/?source=recommendations)

Demonstrate how to simplify, automate, and transform business tasks and processes using Microsoft Power Platform Developer.

---

Events

[Join AI Skills Fest Challenge](https://aka.ms/AISkillsFest_LearnT2_TechChallenges)

Apr 9, 12 AM - May 28, 4 PM

Sharpen your AI skills and enter the sweepstakes to win a free Certification exam

[Register now!](https://aka.ms/AISkillsFest_LearnT2_TechChallenges)