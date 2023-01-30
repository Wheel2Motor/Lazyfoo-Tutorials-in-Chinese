# 在屏幕上显示一张图

<p align="center">
    <img src="https://lazyfoo.net/tutorials/SDL/02_getting_an_image_on_the_screen/preview.png">
</p>

既然你已经成功打开了一个窗口，接下来让我们往上面放一张图。

注意：从现在开始教程将只涵盖代码的关键部分。如果想要获得完成的程序，你将不得不下载完整的源代码。

```cpp
//启动SDL并创建窗口
bool init();

//载入媒体数据
bool loadMedia();

//释放媒体数据并关闭SDL
void close();
```

在第一个教程中，我们将所有代码都放在了主函数里。因为它是一段非常小的程序，所以我们能这样草草了事，但是在真实的项目中（比如视频游戏）你会希望自己的代码尽可能模块化。就是说你会更希望自己的代码各自放在干净的代码块中以便排查错误和复用。

这就意味着我们需要定义一组函数用于处理初始化、载入媒体、关闭SDL。我们将这些定义在接近源代码顶端的位置。

我收到许多电子邮件询问关于为什么调用这个“close”函数在C语言中会引发冲突，因为C语言不支持函数重载。这就是为什么在这个教程中使用C++的原因。所以这个函数叫“close”并不是一个bug。

```cpp
//即将用于渲染的窗口
SDL_Window* gWindow = NULL;

//用于渲染的窗口包含的surface
SDL_Surface* gScreenSurface = NULL;

//我们即将载入并显示在屏幕上的图片
SDL_Surface* gHelloWorld = NULL;
```

接下来我们来定义一些全局变量。一般来说在大型程序中你都会希望避免使用全局变量。我们在此处这么做的目的主要是因为我们希望源代码尽可能精简，但是在大型程序中，全局变量会使一切变得更加复杂。由于这不过是一段单个文件的代码而已，因此大可不必担心。

首先我们介绍一个叫做SDL Surface的新数据类型。SDL Surface只是一个图像数据类型，它保存了图像的像素信息以及所有渲染所需的其他数据。SDL Surface使用软件渲染，这意味着它使用的是CPU来进行渲染。当然也能使用硬件渲染图像，但稍微麻烦一点点，所以我们先从简单的入手来进行学习。后面教程中，我们会涵盖关于如何使用GPU加速来渲染图像。

我们即将用到的图像是来自一张屏幕当前的画面的图像（你在屏幕中看到的内容就是），我们将会从本地文件中载入这样一张图像。

注意这些SDL Surface都是指针。原因如下<1>我们将会采用动态内存分配的方式来载入图像<2>引用图像最好是通过内存地址的方式来进行引用。想象一下在你的游戏中有个一面砖墙，砖墙由同样的砖块重复渲染很多次构成（比如超级玛丽兄弟）。同时在内存中驻留多份同样的数据并重复渲染实在是过于浪费资源。

如你所见，我们已经将SDL初始化和窗口创建的代码放到了它应该存在的位置。唯一不同的就是多了个SDL_GetWindowSurface的调用。

我们希望能够在窗口中显示图像，为了达成这一目标，我们需要先获取到窗口对象中所包含的图像。所以我们调用SDL_GetWindowSurface来抓取窗口中包含的surface。

```cpp
bool loadMedia()
{
    //载入成功的标志变量
    bool success = true;

    //载入图片
    gHelloWorld = SDL_LoadBMP("02_getting_an_image_on_the_screen/hello_world.bmp");
    if (gHelloWorld == NULL)
    {
        printf("Unable to load image %s! SDL Error: %s\n", "02_getting_an_image_on_the_screen/hello_world.bmp", SDL_GetError());
        success = false;
    }

    return success;
}
```

在载入媒体的函数中我们使用SDL_LoadBMP。SDL_LoadBMP函数通过输入一个bmp文件路径作为入参，返回载入的surface。如果该函数返回了NULL，就意味着图像载入失败，那么我们就需要使用SDL_GetError获取错误信息，然后在命令行中打印出来。

另外需要注意的是，这段代码假设你的当前工作目录下有一个名为“02_getting_an_image_on_the_screen”的目录，其包含了一张图片名为“hello_world.bmp”。当前工作目录就是你的应用程序认为它正在运行时所处的目录。一般来说，你的工作目录就是可执行文件所在的目录，但是有些程序比如Visual Studio会修改工作目录为vcxproj文件所在的位置。所以如果你的应用程序无法找到图像文件，确认一下图像是否在正确的位置。

还有，如果程序运行的时候还是无法找到图像，你的工作目录多半是有些问题。工作目录在不同的操作系统、不同的IDE之间都有所不同。如果google搜索如何修复工作目录也无法给出有效的解决方案，我建议将“02_getting_an_image_on_the_screen”的文件夹去掉，直接将“hello_world.bmp”放到外面，直到程序找到并成功加载了为止。

```cpp
void close()
{
    //释放surface
    SDL_FreeSurface(gHelloWorld);
    gHelloWorld = NULL;

    //销毁窗口
    SDL_DestroyWindow(gWindow);
    gWindow = NULL;

    //退出SDL子系统
    SDL_Quit();
}
```

在我们简洁的代码中，还像上一节那样销毁窗口并退出SDL，但还应该关注到已经载入的surface。我们使用SDL_FreeSurface来进行释放。无需担心窗口的的surface，SDL_DestroyWindow会来处理的。

养成将空指针赋值为NULL的好习惯。

为什么我们要去关注一个无论如何都即将被关闭的程序中的资源释放问题呢？难道程序返回后为其分配的内存资源不会被释放吗？

答案就是：我不知道。这是由操作系统决定的，操作系统可能会帮助你自动释放资源，但也有可能并不会。欢迎来到C++未定义行为的天坑世界。一条通行的规则就是，能手动避免的未定义行为就手动避免。这是针对未定义行为一个相当不错的例子，在此之前我已经见过很多工程师耗费数天时间查找由未定义行为引发的bug（顺带一提，在构造函数中千万不要调用虚函数）。从现在养成认真对待未定义行为的习惯吧，你不会希望哪天在团队项目的交付节点前因为未定义行为而忙得焦头烂额的。

```cpp
            //更新窗口的surface
            SDL_UpdateWindowSurface(gWindow);
```

当完成了希望在这一帧画面中显示的所有内容的绘制，我们需要使用SDL_UpdateWindowSurface更新一下屏幕。留意一下当你在屏幕上绘图的时候，一般来说我们不会直接在屏幕上当前显示的图像上进行绘图。默认情况下大多数渲染系统都使用的是双缓冲机制。双缓冲机制就是指一个前台缓冲区和一个后台缓冲区。（绘制是绘制在后台缓冲区上，通过SDL_UpdateWindowSurface将前、后台缓冲区进行互换）

当你调用绘制函数比如SDL_BlitSurface，会绘制到后台缓冲区。你所看到的是前台缓冲区。我们为什么要这么做是因为大多数游戏帧都需要绘制许多物体到屏幕上。如果我们只有一个前台缓冲，我们会看到物体绘制的过程，也就是说我们会看到没有绘制完成的帧。所以我们所做的事情就是，将所有物体都先绘制到后台缓冲区，一旦绘制完成我们就交换前后台缓冲区，这样用户就能看到完成的帧了。

这也意味着，你不需要每次绘制任何像素后都调用SDL_UpdateWindowSurface，只要在当前帧全部绘制完成后调用一次就够了。

```cpp
            //让窗口停留小技巧
            SDL_Event e;
            bool quit = false;
            while (quit == false) {
                while (SDL_PollEvent(&e)) {
                    if (e.type == SDL_QUIT)
                        quit = true;
                }
            }
        }
    }

    //释放资源，关闭SDL
    close();

    return 0;
}
```

既然我们已经将所有东西都绘制到窗口内了，我们用一个小技巧，让窗口不会立即消失。当等待结束后，我们关闭程序即可。

> 下载媒体文件和源代码：[下载链接](https://lazyfoo.net/tutorials/SDL/02_getting_an_image_on_the_screen/02_getting_an_image_on_the_screen.zip)
> 
> [返回SDL教程]()
