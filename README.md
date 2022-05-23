**WARNING: THIS TEMPLATE IS NOT FOR NEWBIES, THIS IS FOR EXPERIENCES PROGRAMMERS ONLY. NEWBIES SHOULD NOT PROCEED TO TRY IT**

**This won't cover how to mod games in general, hooking functions, etc that every other online tutorial already covers. This template simply tells you how to use them**


# Introduction
Imgui mod menu for il2cpp and other native android games, and some codes used from [Octowolve](https://github.com/Octowolve/Hooking-Template-With-Mod-Menu). KittyMemory, MSHook, And64InlineHook and AY Obfuscator included. 
Support Android 4.4.x up to Android S Preview. ARMv7, x86 and ARM64 architecture supported. However x86 is deprecated for Unity games so x86 is not our priority

Preview:


# Prerequisites
Before we can jump head first into working a template, we need to go over a few things.

* **AN EXPERIENCED MODDER, NOT A BEGINNER:** You should be able to mod any games in general, like modifying .so files, dll files, smali files, etc.
* Basic knowledge of smali dalvik opcodes to modify smali
* Basic knowledge ARM and ARM64 assembly, to be able to patch hex (No need x86)
* Basic knowledge of C++  (JNI is optional)
* Be able to write hook function in C++ (Not really needed, but recommended if you want to do advanced modding in the future)
* Basic awareness of how Android layout works in XML and Java. This project only use Java for layout but you will learn it easly
* Time and patience: Don't start working on this if you have deadlines or important work. Take your time to read, learn and get used to work with this project.
* DIY (Do it yourself): Yes, you must be able to do things yourself, not depending being spoonfeed. We are not the teachers.
* An inquisitive mind

# What you need
* Android Studio 4 and up: https://developer.android.com/studio
* NDK Installed via Android Studio (Cmake is not needed) https://developer.android.com/studio/projects/install-ndk#default-version
* Apktool: [Apktool.jar](https://ibotpeaches.github.io/Apktool/) or any 3rd party tools
* [APK Easy Tool](https://forum.xda-developers.com/android/software-hacking/tool-apk-easy-tool-v1-02-windows-gui-t3333960). To get main activity: 
* Any text editor. We use [Notepad++](https://notepad-plus-plus.org/downloads/)

# Installation
Download this repo as ZIP, or clone using any git tools

Extract the source to your desired location. The location must **NOT** contain any spaces or symbols

Open the project

Please wait for a while, it will index and sync the project for the first time, takes around a minute depending your computer performance

After it's done, you can start working!

#### KittyMemory patching usage:
```cpp
MemoryPatch::createWithHex([Lib Name], [offset], "[hex. With or without spaces]");
[Struct].get_CurrBytes().Modify();
[Struct].get_CurrBytes().Restore();

[Struct].get_TargetAddress();
[Struct].get_PatchSize();
[Struct].get_CurrBytes().c_str();

PATCHOFFSET("0x20D3A8", "00 00 A0 E3 1E FF 2F E1");
PATCHOFFSET_LIB("libFileB.so", "0x20D3A8", "00 00 A0 E3 1E FF 2F E1");
```

Example: https://github.com/MJx0/KittyMemory/blob/master/Android/test/src/main.cpp

Use an online ARM assembly converter like ARMConverter to convert ARM to HEX: https://armconverter.com/

#### Hook usage:
This macro works for both ARMv7 and ARM64. Make sure to use predefined macro `defined(__aarch64__)` and `defined(__arm__)` if you are targeting both archs

Strings for macros are automatically obfuscated. No need to obfuscate!
```cpp
HOOK("0x123456", FunctionExample, old_FunctionExample);
HOOK_LIB("libFileB.so", "0x123456", FunctionExample, old_FunctionExample);
HOOK_NO_ORIG("0x123456", FunctionExample);
HOOK_LIB_NO_ORIG("libFileC.so", "0x123456", FunctionExample);
HOOKSYM("__SymbolNameExample", FunctionExample, old_FunctionExample);
HOOKSYM_LIB("libFileB.so", "__SymbolNameExample", FunctionExample, old_FunctionExample);
HOOKSYM_NO_ORIG("__SymbolNameExample", FunctionExample);
HOOKSYM_LIB_NO_ORIG("libFileB.so", "__SymbolNameExample", FunctionExample);
```

Or

ARM64:
```cpp
A64HookFunction((void *) getAbsoluteAddress([Lib Name], [offset]), (void *)[function], (void **)&[old function]);
```

ARMv7/x86:
```cpp
MSHookFunction((void *) getAbsoluteAddress([Lib Name], [offset]), (void *)[function], (void **)&[old function]);
```

#### **Android.mk**

The make file for the c++ compiler. In that file, you can change the lib name on the `LOCAL_MODULE` line
When you change the lib name, change also on `System.loadLibrary("")` under OnCreate method on `MainActivity.java`
Both must have same name

# Implementing the menu to the target game

### 1. Know your game's main activity

Now we are looking for main activity, there are 2 ways to do

1. Decompile the game's APK file. Open `AndroidManifest.xml` and search after `<action android:name="android.intent.action.MAIN"/>`.

Example the game's main activity was `com.unity3d.player.UnityPlayerActivity`

![](https://i.imgur.com/FfOtc1K.png)

Be sure to enable Word wrap so it is easier to read

![](https://i.imgur.com/7DzU8d0.png)

2. APK Easy Tool since it can read out location of main activity without decompiling APK

![](https://i.imgur.com/JQdPjyZ.png)

Note it somewhere to remember it

### 2. Making corresponding changes in the files

Decompile the game APK

It is very easy to load the imgui just load your library from game's ,ain activity

### 3. Compiling game apk
 
Now compile and sign the apk, and install it on your device

Congrats. You have successfully implemented a mod menu.

Compile failed? read the log and look up on Google

If you face any problem, please read the [FAQ](#faq)

# Leeching concerns

Leeching as known as stealing code and offsets via reverse enginnering, and editing credits via file editing and recompiling. We all know that, right?

There are some simple protections in the template:
- Simple C++ string obfuscation called AY Obfuscator. Usage `OBFUSCATE("string here")` and with a key `OBFUSCATE_KEY("string here", 64-bit key here)`. Example `OBFUSCATE_KEY("Hello", 2353474243)` or in hex `OBFUSCATE_KEY("Hello", 0x3FE63DF21A3B)`. The key must not be too long or too short
- `string2Offset("")` to protect offsets
- Simple anti-leech measures that crashes if JNI functions are not called
- Quite harder to edit credits via smali
- Toast hidden inside `getFeatureList` in Main.cpp

These protection are NOT full protection, it does not stop them, it will only slow them down, this intent is to help you improve on your own by yourself. You should:
- Improve anti-leech measures on your own way
- Protect and encrypt your dex and lib. Find the tools or the projects by yourself, chinese based tools is not recommended as anti virus may flag your mod for malware (false positive). Don't tell anyone what protection you are using, don't let game developers get a hand of it
- Improve string obfuscators a lot more or use others which are not known. Make sure that obfuscator is not too simple
- Enable proguard, and add filters to make sure it does not break your project. See https://developer.android.com/studio/build/shrink-code
- Never share your project to someone
- Do not include any important stuff such as 'offline' username and password, instead add an additional layer, e.g. a web service handling the protected request
- And etc.

Never contact how to protect more, never complain that your mod has been leeched, that's all your responsibility! If you are really worry about leeching, or getting constantly leeched, and can't protect, just upload your project on Github. They will download from your Github instead leeching.

**Never tell us how to leech stuff, we are not interested getting involved in it, You will get blocked immediately!**

### I have a problem decompiling or compiling APK file
Search for the related issues on Google or on Apktool Github page: https://github.com/iBotPeaches/Apktool/issues

### I'm getting an error `Unsigned short value out of range: 65536` if I compile
The method index can't fit into an unsigned 16-bit value, means you have too many methods in the smali due to the limit 65535. Place your code on other classes, such as smali_classes2 instead. This work for Android 5 (Lollipop) and above only.

### I'm getting an error `ERROR: executing external native build for ndkBuild Android.mk. Affected Modules: app`
See: https://github.com/LGLTeam/Android-Studio-Solutions/wiki/Executing-external-native-build-for-ndkBuild-Android.mk

### I'm getting strange issues on Android Studio or Gradle
See: https://github.com/LGLTeam/Android-Studio-Solutions/wiki

### Why can't you just add the feature I want? For example closing animation?

Also known as: "You're a developer after all — it shouldn't be that hard!"

Since we can't spend my days on it, we have to prioritize the features and fixes that are likely to benefit the larger number of people. Features that is specific to your usage is not going to benefit that many users after all. This is important that you do not expect that anyone can do everything for you. Developing *is* hard, and even outside of supporting a feature, adding the code can take longer than you think!

With all this being said, remember that this is 100% Open Source. So if you really want a specific feature, try to do it yourself, or ask someone who are willing to help you.

### Why can't you just help or teach me modding the game? For example, hooking?

Anything else, such as how to hook, how to patch, how to bypass, what functions to mod, how il2cpp works, etc. is out of scope. We will not cover anything and trivial stuff every other tutorial online already covers. Instead, try to find a couple of tutorials to learn and mod the game yourself. It's a lot easier than you think. If you can't, find a couple of forums where you can ask your questions or ask the right modder for specific special features

### When there is a new update? I have waited for so long time

There is no ETA, we only push a commit when we want to. Please don't ask the same question over and over again. You could make something better yourself without having to wait

# Reporting issues/Cоntact
<details>
<summary>Cоntact:</summary>
Please stop and read this carefully.

Make sure you have readed FAQ and at least searching for answers.

If you have usage problems, try asking your questions on any forum sites. For example, if you have an issue with hooking or patching, bypassing security, or wanna mod PUBG and Free Fire, you should go to the **forums**. Here there are no teachers, or who deal with such issues.

Beginner/newbie/noobs and toxic peoples are **NOT** allowed to cоntact. They are annoying, you would be left **unanswered** and possibly get **BLOCKED**. Known leechers will be instant **BLOCKED**

Issue tracker is permanently disabled

Tеlеgram: @layout_musk


</details>

# Credits/Acknowledgements
Thanks to the following individuals whose code helped me develop this mod menu

* My Friend zTz - Imgui Setup and Touch for unity: https://github.com/ZTzTopia
* ozMod - Touch for unity (refference) :https://github.com/ozMod/BPM-ModMenu-ImGui
* MJx0 A.K.A Ruit - https://github.com/MJx0/KittyMemory
* Rprop - https://github.com/Rprop/And64InlineHook
* And everyone else who provided input and contributions to this project!

*LOVE FOR ALL
