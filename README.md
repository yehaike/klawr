Klawr (Experimental WIP)
================
A set of Unreal Engine 4 plugins that will allow users to write game-play code in C# (and eventually other CLI languages such as F#), in game projects targeting the Windows platform. The primary aim of this project is to make it possible to do anything that can currently be done in Blueprints in C# scripts, in order to eliminate the need for complex Blueprint spaghetti.

The current development focus is on script components (see UKlawrScriptComponent), these are actor components whose functionality is implemented in a C# class. The C# class attached to a script component can access any Blueprint accessible property or function exposed by the actor it is attached to or any of its sibling components (this is still limited at present). Eventually the properties and methods defined in a C# script component will also be exposed to Blueprints.

Some time was also spent developing script objects, which allow for "subclassing" in C# of almost any UObject-derived class, actors included, but that's currently on the back-burner.

NOTE: This project is still highly experimental, and not very useful (though it's getting there).

Check the [Wiki](https://github.com/enlight/klawr/wiki) for additional details.

Prerequisites
========
- Unreal Engine 4.4.0 or later (you'll need the source from GitHub)
- Visual Studio 2013
- .NET Framework 4.5 or later

Overview
======
The core of the CLR hosting code is located in the Klawr.ClrHost libraries, there is a C++ library that bootstraps the CLR, and a couple of C# libraries that handle hosting related tasks and provide some core classes for use by the auto-generated UE4 C# API as well as user scripts. The C++ library is linked against KlawrRuntimePlugin.

**KlawrCodeGeneratorPlugin** generates C# wrappers for UObject subclasses from the reflection information gathered by  **Unreal Header Tool** from UFUNCTION and UPROPERTY decorators in the engine/game source. This is the same reflection information that underpins much of the functionality provided by Blueprints.

**KlawrRuntimePlugin** executes user scripts written in C#, scripts have access to the wrapped UE4 API generated by KlawrCodeGeneratorPlugin.

Building
======
This is a two phase process, in the first phase the CLR hosting libraries must be built, in the second phase the plugins themselves.

Source Location
-----------------
The directory hierarchy of this repository matches that of the UE4 source repository, once you check out this project you can just copy the Engine directory into your UE4 source checkout and everything should end up in the right place. However, if you plan on making any changes to the source you'll probably want create a couple of directory junctions instead of copy/pasting files back and forth. To that end I've provided **CreateLinks.bat**, which you can right-click on and **Run as administrator** directly from the checkout root. The batch file will prompt you to enter the path to the UE4 source checkout that you'd like to use to build Klawr, and will then create the necessary directory junctions.

Now everything should be in the right place, from here on any paths are relative to the UE4 source checkout.

Disable Built-in Script Plugins
-------------------------------
The UE4 source contains some built-in Script plugins, these are mostly meant as a reference for plugin developers, but they get in the way because the Unreal Build Tool (UBT) wastes time building and running them. So, before building the Klawr plugins you'll need to make a couple of one-line changes to the UE4 source to get the built-in Script plugins out of the way, you can either make the changes by hand or apply the two patch files that I've provided (ScriptPluginPatch.diff and UnrealHeaderToolPatch.diff). If you're making the changes by hand comment out the following line in `Engine/Plugins/Script/ScriptPlugin/Source/ScriptPlugin/Private/ScriptPlugin.cpp`

```#include "GeneratedScriptLibraries.inl"
```

and replace the following line in `Engine/Source/Programs/UnrealHeaderTool/UnrealHeaderTool.Target.cs`

```AdditionalPlugins.Add("ScriptGeneratorPlugin");
```

with

```AdditionalPlugins.Add("KlawrCodeGeneratorPlugin");
```

Now UBT won't waste time building the ScriptGeneratorPlugin, the ScriptGeneratorPlugin won't waste time generating wrapper functions that aren't used by anything, and ScriptPlugin won't fail to build because ScriptGeneratorPlugin didn't produce **GeneratedScriptLibraries.inl**.

Libraries
---------
1. Open `Engine\Source\ThirdParty\Klawr\Klawr.ClrHost.sln` in VS2013.
2. Set the **Solution Configuration** to either **Release** or Debug.
3. Set the **Solution Platform** to **x64** (this is very important, Win32 builds are not supported yet).
4. Select **Build Solution** to build all the libraries.

Assuming that the build finished with no errors you can move on to phase two.

Plugins
--------
1. Configure Unreal Header Tool to use the KlawrCodeGeneratorPlugin, to do so add/edit the **Plugins** section in `Engine\Programs\UnrealHeaderTool\Saved\Config\Windows\Engine.ini`:

    ```
    [Plugins]
    ProgramEnabledPlugins=KlawrCodeGeneratorPlugin
    ScriptExcludedModules=ScriptPlugin
    ScriptExcludedModules=ScriptEditorPlugin
    ScriptExcludedModules=ScriptGeneratorPlugin
    ```

2. Run GenerateProjectFiles.bat in your UE4 source checkout.
3.  Open **UE4.sln** in VS2013.
4.  Set the **Solution Configuration** to **Debug Editor** or Development Editor.
5.  Set the **Solution Platform** to **Win64** (this is very important, Win32 builds are not supported yet).
6. Select **Build Solution** to build everything.

During the build you may see a bunch of console windows popup briefly, don't panic, this is just KlawrCodeGeneratorPlugin building the UE4 C# wrappers assembly.

Using
====
**Don't!** Yet. But if you insist:

1. Enable KlawrRuntimePlugin and KlawrEditorPlugin in UnrealEd, restart as requested.
2. On the Content Browser menu select New->Blueprints->Klawr Blueprint.
3. Enter a name for the new Blueprint (it's going to contain a component), e.g. MyComponent
4. Enter the filename for your new C# script, this will default to the name of the Blueprint.
5. Enter the location for your new C# script, this is relative to the game project directory, and should be outside the Content directory. The default is the Scripts directory (will be created if it doesn't already exist), but you can also set the location to the Source directory (if your project has one), or Source\Scripts, or any directory within the aforementioned directories (excluding Content).
6. Press OK to create the Blueprint.
7. The C# scripts assembly will be rebuilt.
8. The new Blueprint component can now be attached to an actor, just open/create an actor Blueprint and add the Blueprint component to it.
9. Nothing happens because I'm still working on it! :)

License
=====
Klawr is licensed under the MIT license.

Klawr uses the pugixml library to parse XML, it was written by Arseny Kapoulkine and is licensed under the MIT license.
