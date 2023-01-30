# Hello SDL

![preview](https://lazyfoo.net/tutorials/SDL/01_hello_SDL/preview.png)

看上去你学了一些C++基础，但是你肯定厌恶了重复编写简短的命令行程序了对吧。为了能够使用图像、声音、键盘、游戏手柄等功能，你需要一些能将硬件功能转为C++能与之交互的API（Application Programmer‘s Interface）。

这就是SDL所做的事。它抽象出了Window、Linux、Mac、Android、iOS等平台的硬件功能，将其包装成统一的操作接口，方便你通过编写SDL代码并将其轻松编译到它支持的任何平台。为了使用SDL，需要首先安装一下。

这些安装教程用于辅助你配置好SDL。某些特定的公司（此处应保持匿名）的IDE配置界面随着每个版本的发行都在不断变化，所以自教程制作以来可能那些配置流程已经发生了变化。为了防止教程最近一次更新以来IDE的操作界面发生改变，你可以自己查看一下[SDL在GitHub上的文档](https://github.com/libsdl-org/SDL/tree/main/docs)以检查现在以及教程最近一次更新后的区别。

SDL作为一个动态链接库，包含了3个部分：

* 头文件（Library.h）
* 库文件（对Windows而言是Library.lib，对\*nix而言是libLibrary.a）
* 二进制文件（对Windows而言是Library.dll，对\*nix而言是libLibrary.so）

你的编译器在编译是需要能够找到头文件，这样才能知道SDL_Init()以及其他SDL函数或结构体是什么。你可以将SDL头文件所在的位置给你的编译器配置上作为一个额外的搜索路径，或将头文件放到编译器默认的搜索路径下。如果编译器抱怨找不到SDL.h，就意味着头文件并没有放到编译器会去搜索的位置上。
