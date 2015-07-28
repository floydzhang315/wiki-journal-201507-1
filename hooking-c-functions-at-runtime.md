# 运行时的挂钩 C 函数  --邵凯阳

来源:http://thomasfinch.me/blog/2015/07/24/Hooking-C-Functions-At-Runtime.html  

时间：2015年7月27  

作者：Thomas Finch

## 关于文本

钩子（Hook)是 Windows 消息处理机制的一个平台，该技术可以实现对消息的监视，具有很强大的功能。本文就是基于钩子的主要功能实现钩子在 C 中的应用，主要介绍了运行是挂钩 C 函数的基本步骤、相关代码和一些局限性。

## 文章内容

这是一份我最近尝试的关于在 C 中运行时函数挂钩的快速记录。对于钩入一个函数最基本的思想是用您自己的代码替换函数的代码，所以在调用该函数时，您的代码将被执行。运行时挂钩允许您更改程序的运行方式当被执行的程序没有自己的代码或者没有以任何方式对其文件进行实际修改的时候。运行时函数挂钩并不少见，并且用于 iOS 越狱调整(通过 [Cydia Substrate](http://www.cydiasubstrate.com/) 或[Substitute](https://github.com/comex/substitute) 平台提供技术支持)以及  [Xposed 框架](http://repo.xposed.info/module/de.robv.android.xposed.installer) 在 Android 程序中的使用。

如果您想在您自己的计算机上跟随了解这篇文章，您需要使用 Xcode 和 Xcode 命令行工具安装的 Mac。这些代码能够在[Github](http://repo.xposed.info/module/de.robv.android.xposed.installer)上找到。

### 示例程序

```   
//testProgram.c
#include <stdio.h>

int hookTargetFunction() {
    printf("Calling original function!\n");
    return 5;
}

int main() {
    printf("The number is: %d\n", hookTargetFunction());
    return 0;
} 
```  

编译和运行该程序可以得到下面的输出：

```
Calling original function!
The number is: 5
```

我们的目标是钩入 `hookTargetFunction` 这个函数并且更改该函数的返回值，使返回值不再是 5。

### 挂钩目标函数

我们挂钩目标函数的方法是通过创建一个动态库，当程序运行时加载它。这个动态库的构造函数会在 `main` 函数的目标可执行文件前执行，所以我们就可以在目标可执行文件运行之前在内存中修改它。若要运行我们替换的代码，我们需要在挂钩函数的开头插入跳转指令的机器代码。换句话说，当计算机尝试运行目标函数时，他将跳转到我们替换函数所在的位置并运行我们的代码。

这个过程的第一步就是创建包含一个构造函数和一个替换函数的动态库。

```
//inject.c
#include <stdio.h>

int hookReplacementFunction() {
    printf("Calling replacement function!\n");
    return 3;
}

__attribute__((constructor))
static void ctor(void) {
    printf("Dylib constructor called!\n");
}
```

当他和使用了 `DYLD_INSERT_LIBRARIES` 环境变量的目标程序被编译和加载后，我们能够看到他的构造函数在主程序之前被执行。

```
$ ls
inject.c    testProgram testProgram.c
$ clang -dynamiclib inject.c -o inject.dylib
$ DYLD_INSERT_LIBRARIES=inject.dylib ./testProgram
Dylib constructor called!
Calling original function!
The number is: 5
```

为了钩入目标函数，现在我们可以开始向构造函数中添加代码。由于 x86 跳转指令使用相对寻址，所以我们不能简单的在内存中给计算机一个地址让其跳转。首先，我们需要从目标函数中找到替换函数的抵消函数，这些可以通过获得进入每个函数的指针，然后从另一个指针中减去一个函数的指针。

```
void *mainProgramHandle = dlopen(NULL, RTLD_NOW);
int64_t *origFunc = dlsym(mainProgramHandle , "hookTargetFunction");
int64_t *newFunc = (int64_t*)&hookReplacementFunction;
int32_t offset = (int64_t)newFunc - ((int64_t)origFunc + 5 * sizeof(char));
```

在这个示例代码中有一些值得关注的事情。首先是使用 `dlopen` 来获得进入目标可执行文件的指针。`dlopen` 通常被用来加载共享库，但是 [根据其文档](http://linux.die.net/man/3/dlopen)，如果传递 NULL 作为文件名，它也可以用于访问主可执行文件。其次应该注意的是，跳转的偏移量实际采取的是下一条指令的地址，在这种情况下目标函数将增加 5 bytes，因为插入跳转指令的大小为 5 bytes。

在这篇文章中我省略了一小步，那就是使目标函数所在的内存是可写的，因为处于安全的考虑，在默认情况下内存仅仅是可读的和可执行的。一旦这些被完成，最后一步就是创建和插入跳转指令。x86 操作码是 E9，他与立即数偏移寻址一起是无条件跳转，因此我们将这作为指令的第一个字节，紧跟的是偏移。 

```
int64_t instruction = 0xE9 | offset << 8;
*origFunc = instruction;
```

这里是完成的 `inject.c` 文件:

```
#include <stdio.h>
#include <dlfcn.h>
#include <stdint.h>
#include <sys/mman.h>
#include <unistd.h>

int hookReplacementFunction() {
    printf("Calling replacement function!\n");
    return 3;
}

__attribute__((constructor))
static void ctor(void) {
    //Get pointers to the original and new functions and calculate the jump offset
    void *mainProgramHandle = dlopen(NULL, RTLD_NOW);
    int64_t *origFunc = dlsym(mainProgramHandle , "hookTargetFunction");
    int64_t *newFunc = (int64_t*)&hookReplacementFunction;
    int32_t offset = (int64_t)newFunc - ((int64_t)origFunc + 5 * sizeof(char));

    //Make the memory containing the original funcion writable
    //Code from http://stackoverflow.com/questions/20381812/mprotect-always-returns-invalid-arguments
    size_t pageSize = sysconf(_SC_PAGESIZE);
    uintptr_t start = (uintptr_t)origFunc;
    uintptr_t end = start + 1;
    uintptr_t pageStart = start & -pageSize;
    mprotect((void *)pageStart, end - pageStart, PROT_READ | PROT_WRITE | PROT_EXEC);

    //Insert the jump instruction at the beginning of the original function
    int64_t instruction = 0xe9 | offset << 8;
    *origFunc = instruction;
}
```

当他编译和执行完后，他确实改变了主程序的输出！

```
$ ls
inject.c    testProgram testProgram.c
$ ./testProgram 
Calling original function!
The number is: 5
$ clang -dynamiclib inject.c -o inject.dylib
$ DYLD_INSERT_LIBRARIES=inject.dylib ./testProgram
Calling replacement function!
The number is: 3
```

这里是另外一个执行过程和一些调试输出，显示了跳转指令插入目标函数的开始：

```
$ DYLD_INSERT_LIBRARIES=inject.dylib ./testProgram
Original function address: 0x1078abee0
Replacement function address: 0x1078b4c40
Offset: 0x8d5b
Before replacement: 
*(origFunc+0):  554889e5
*(origFunc+4):  488d3d73
*(origFunc+8):  00e84c00
*(origFunc+12): 00000089
After replacement: 
*(origFunc+0):  e95b8d00
*(origFunc+4):  488d3d73
*(origFunc+8):  00e84c00
*(origFunc+12): 00000089
Calling replacement function!
The number is: 3
```

### 局限性

挂钩这种方法的一个局限性是他要求目标函数至少是 5 bytes，用于插入跳转指令。这看起来似乎是一个愚蠢的限制，但创建这样小的函数也肯定是可能的（例如，只有单字节大小的 `ret` 指令）。我想不出解决这一问题的方式，毕竟对单字节进行操作时很艰难的。最直截了当的解决方法就是不挂钩小于 5 bytes的指令。

我遇到的另外一个问题是让这些代码运行在 Linux 上。出于某些原因，Linux 始终在一个高地址加载动态库，地址如此之高以至于偏移量溢出了可用的 32 bits。我不认为这是可以修复的，尽管也使用了跳转指令，因为偏移量的最大尺寸是 32 bits。但是，这个函数能够被挂钩通过另外一种方法——例如，将替换函数的地址压入堆栈，然后通过 `ret` 指令跳转到改地址。这种方法将会比简单的跳转花费更多的空间，但是这是我现在仅能想到的方法。
 
我希望您能够喜欢这篇文章！再一次，自由下载并且在您自己的计算机上测试这些代码。当您自己动手尝试时，您会体会到更多的乐趣！

