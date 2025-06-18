---
title: "Get started with Windows development using WinUI - Windows apps"
source: "https://learn.microsoft.com/en-us/windows/apps/get-started/start-here?tabs=vs-2022-17-10#1-enable-developer-mode"
author:
  - "[[jwmsft]]"
published:
created: 2025-05-16
description: "List of steps to get started developing Windows apps with WinUI and the Windows App SDK."
tags:
  - "clippings"
---
## Start developing Windows apps

Welcome to Windows app development. This guide will take you through the steps needed to begin creating apps using the latest Windows development frameworks: the Windows App SDK and WinUI. It will also point you to resources that will help you learn more about Windows development. If you are already comfortable developing apps for Windows, but want to know more about the latest tools, please see [Develop Windows desktop apps](https://learn.microsoft.com/en-us/windows/apps/develop/).

Windows has a special mode for developers that adjusts security settings in order to let you run the apps you're working on. You'll need to enable Developer Mode before you can build, deploy, and test your app using Visual Studio.

To enable Developer Mode:

- Open Windows Settings and navigate to the **[System > For developers](https://learn.microsoft.com/en-us/windows/apps/get-started/)** page.
- Toggle the Developer Mode switch to **On** and confirm your choice in the confirmation dialog.

For more information about Developer Mode, see [Enable your device for development](https://learn.microsoft.com/en-us/windows/apps/get-started/enable-your-device-for-development).

You'll use [Visual Studio](https://learn.microsoft.com/en-us/visualstudio/ide/), Microsoft's comprehensive integrated development environment (IDE), to create your WinUI app. It's the preferred development tool of many Windows developers and it will help you write, debug, and deploy your apps. The project templates in Visual Studio will quickly get you started with projects for Windows and many other platforms.

Use the link below to download and install the latest Visual Studio. The installer will walk you through the steps, but if you find you need detailed instructions, see [Install Visual Studio](https://learn.microsoft.com/en-us/visualstudio/install/install-visual-studio).

[Download Visual Studio](https://visualstudio.microsoft.com/downloads/)

The free Visual Studio Community Edition includes everything you need to create your apps. If you're working with a development team or enterprise, you might need Visual Studio Professional or Visual Studio Enterprise. See [What is Visual Studio?](https://learn.microsoft.com/en-us/visualstudio/get-started/visual-studio-ide) for more info.

While installing Visual Studio, you need to install the workloads and components required for developing with WinUI and the Windows App SDK. After installation, you can open the Visual Studio Installer app and select **Modify** to add workloads and components.

On the **Workloads** tab of the Visual Studio Installer app, select the following workloads and components:

- [Visual Studio 2022 version 17.10 and later](https://learn.microsoft.com/en-us/windows/apps/get-started/?tabs=vs-2022-17-10#tabpanel_1_vs-2022-17-10)
- [Visual Studio 2022 version 17.1 - 17.9](https://learn.microsoft.com/en-us/windows/apps/get-started/?tabs=vs-2022-17-10#tabpanel_1_vs-2022-17-1)

- For **C#** app development using the Windows App SDK, select **WinUI application development**.

![A screenshot of the Visual Studio installer UI with the WinUI application development workload selected.](https://learn.microsoft.com/en-us/windows/apps/get-started/images/hello-world/vs-workload-winui.png)

A screenshot of the Visual Studio installer UI with the WinUI application development workload selected.

- For **C++** app development using the Windows App SDK, select **WinUI application development**.
	- Then in the **Installation details** pane, under the **WinUI application development** node, select **C++ WinUI app development tools**. (This will also select any additional required components.)

![A screenshot of the Visual Studio installer UI with the C++ WinUI app development tools selected.](https://learn.microsoft.com/en-us/windows/apps/get-started/images/hello-world/vs-workload-winui-cpp.png)

A screenshot of the Visual Studio installer UI with the C++ WinUI app development tools selected.

Visual Studio project templates include all the files you need to quickly create your app. In fact, after you create your project from a WinUI app template, you'll already have an app that you can run, and then add your code to.

To create a new project using the WinUI C# Blank App project template:

1. Open Visual Studio and select **Create a new project** from the launch page. (If Visual Studio is already open to the editor, select **File** > **New** > **Project**):
	![Create a new project](https://learn.microsoft.com/en-us/windows/apps/get-started/images/hello-world/start-project.png)
	Create a new project
2. Search for `WinUI` and select the `Blank App, Packaged (WinUI 3 in Desktop)` C# project template, then click **Next**:
	![Blank, packaged WinUI 3 C# desktop app](https://learn.microsoft.com/en-us/windows/apps/get-started/images/hello-world/create-project.png)
	Blank, packaged WinUI 3 C# desktop app
3. Specify a project name, then click **Create**. You can optionally specify a solution name and directory, or leave the defaults. In this image, the `Hello World` project belongs to a `Hello World` solution, which will live in `C:\Projects\`:
	![Specify project details](https://learn.microsoft.com/en-us/windows/apps/get-started/images/hello-world/configure-project.png)
	Specify project details
4. Click the Debug "Start" button to build and run your project:  
	![Build and run your project](https://learn.microsoft.com/en-us/windows/apps/get-started/images/hello-world/start-click.png)
	Build and run your project
	  
	Your project will build, be deployed to your local machine, and run in debug mode:  
	![Hello World project built and running](https://learn.microsoft.com/en-us/windows/apps/get-started/images/hello-world/click-me.png)
	Hello World project built and running
5. To stop debugging, close the app window, or click the debug "Stop" button in Visual Studio.

The `MainWindow` class included with the project template includes some sample code that needs to be removed to make room for your content.

1. Double-click `MainWindow.xaml` in **Solution Explorer** to open it. You should see XAML markup for a `StackPanel` control.
2. Delete the XAML for the `StackPanel`. (You'll add your own content in it's place as you create your app.)
	```
	<!-- ↓ Delete this. ↓ -->
	<StackPanel Orientation="Horizontal" HorizontalAlignment="Center" VerticalAlignment="Center">
	    <Button x:Name="myButton" Click="myButton_Click">Click Me</Button>
	</StackPanel>
	```
	If you try to run your app now, Visual Studio will throw an error along the lines of `The name 'myButton' does not exist in the current context`. This is because you deleted the `Button` control named `myButton`, but it's still referenced in the `MainPage.xaml.cs` code-behind file. Delete the reference in the code file, too.
3. Double-click `MainWindow.xaml.cs` in **Solution Explorer** to open it.
4. Delete the `myButton_Click` event handler.
	```csharp
	public sealed partial class MainWindow : Window
	{
	    public MainWindow()
	    {
	        this.InitializeComponent();
	    }
	    // ↓ Delete this. ↓
	    private void myButton_Click(object sender, RoutedEventArgs e)
	    {
	        myButton.Content = "Clicked";
	    }
	    // End delete.
	}
	```
5. Save the file by pressing CTRL + SHIFT + S, clicking the Save All icon in the tool bar, or by selecting the menu **File** > **Save All**.

The Windows App SDK (and WinUI, which is part of it) is distributed as a [NuGet](https://learn.microsoft.com/en-us/nuget/what-is-nuget) package. This means updates can be released out-of-sync with Windows and Visual Studio. As a result, the Visual Studio template you used to create your project might not reference the latest Windows App SDK NuGet package. To ensure you have the latest features and fixes, you should update your NuGet packages every time you create a new project in Visual Studio.

To update the Windows App SDK NuGet package for your project:

- In Visual Studio, with your project loaded, select **Tools** > **NuGet Package Manager** > **Manage NuGet Packages for Solution...**.
- If an update is available, it will appear on the Updates page. Check the box next to the listed update. (To include prerelease updates, check the "Include prerelease" option. To learn more about what's included in an update, see the [release notes](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/stable-channel).)
- Click the **Update** button, then click **Apply** in the Preview changes dialog, then accept the license terms to finish installing the update.

![A screenshot of the NuGet package manager in Visual Studio showing a Windows App SDK package update.](https://learn.microsoft.com/en-us/windows/apps/get-started/images/hello-world/nuget-update.png)

A screenshot of the NuGet package manager in Visual Studio showing a Windows App SDK package update.

Now your project is using the latest WinUI features that are available, and it's ready for you to make it your own.

[Complete a tutorial to create a note app with WinUI](https://learn.microsoft.com/en-us/windows/apps/tutorials/winui-notes/)

- To get an idea of what WinUI has to offer, check out the WinUI Gallery app.
	> The **WinUI 3 Gallery** app includes interactive examples of most WinUI 3 controls, features, and functionality. Get the app from the [Microsoft Store](https://www.microsoft.com/store/productId/9P3JFPWWDZRC) or get the source code on [GitHub](https://github.com/microsoft/WinUI-Gallery)
- Learn more about [WinUI fundamentals](https://learn.microsoft.com/en-us/windows/apps/develop/).
- Explore [Fluent Design](https://learn.microsoft.com/en-us/windows/apps/design/) principles.
- Find [samples and tools](https://learn.microsoft.com/en-us/windows/apps/get-started/samples) to help you develop apps more efficiently.

---

## Additional resources

---

Training

Module

[Create a cross-platform app with.NET MAUI - Training](https://learn.microsoft.com/en-us/training/modules/build-mobile-and-desktop-apps/?source=recommendations)

Learn how to use Visual Studio with.NET MAUI (Multi-platform Application User Interface) to create a cross-platform app.

Certification

[Microsoft Certified: Power Platform Developer Associate - Certifications](https://learn.microsoft.com/en-us/credentials/certifications/power-platform-developer-associate/?source=recommendations)

Demonstrate how to simplify, automate, and transform business tasks and processes using Microsoft Power Platform Developer.

---

Events

[Join AI Skills Fest Challenge](https://aka.ms/AISkillsFest_LearnT2_TechChallenges)

Apr 9, 12 AM - May 28, 4 PM

Sharpen your AI skills and enter the sweepstakes to win a free Certification exam

[Register now!](https://aka.ms/AISkillsFest_LearnT2_TechChallenges)