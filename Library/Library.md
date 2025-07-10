# Library

## 静态库

- 静态库的基本概念：

> 在链接阶段，会将汇编生成的目标文件.o与引用到的库一起链接打包到可执行文件中。其与可执行文件是一体的。

- 静态库的生成：

> ![image-20250708170734633](pictures\image-20250708170734633.png)
>
> 将配置类型转为静态库，点击生成->生成解决方案。
>
> 可以在项目目录里发现以该工程命名的.lib文件。

- 静态库的使用

> - 在工程配置中的调用：
>
> > 首先将库文件（.lib）和.h文件移动并添加至工程。
> >
> > ![image-20250708171054209](pictures\image-20250708171054209.png)
> >
> > 在属性面板里找到连接器->输入->附加依赖项，将库文件输入进去。
>
> - 在代码中语句加载lib调用。
>
> > ```c
> > #ifdef _DEBUG
> > 
> > #pragma comment(lib,"./Debug/LibTst.lib")
> > 
> > #else
> > 
> > #pragma comment(lib,".Debug/LibTst.lib")
> > 
> > #endif
> > ```
> >
> > 在代码的开头输入以上的代码。

## 动态库

> 动态函数库就是存放在系统中的某个特定位置，提供了一些大部分程序都会使用到的功能集合。这样主程序在编译的过程当中就不需要把这部分功能编译到自己的程序中。只需要到系统中特定的位置直接调用动态函数库的功能就行了。
>     好处：程序自身的体积不会因为动态函数库变大
>     缺点：就是程序运行过程中使用到了这些函数库内的功能时，万一系统特定的位置没有对应的动态库。就会造成程序崩溃或者各种奇怪的问题。

- 动态库的生成：

> - 通过导出语句生成：
>
> 同静态库的创建类似，区别在于多了一些语句
>
> ```c
> #pragma once
> #ifndef _LIB_H_
> #define _LIB_H_
> 
> //宏定义导出
> #ifdef LIB__//如果没有定义LIB 就定义 LIB __declspec(dllexport)
> #define LIB __declspec(dllexport)//导出
> #else
> #define LIB __declspec(dllimport)//导入
> #endif // LIB__//如果没有定义LIB 就定义 LIB
> 
> #endif
> ```
>
> 对于变量和函数之前也要求添加对应库的名称。
>
> ```c
> #pragma once
> #ifndef _LIB_H_
> #define _LIB_H_
> 
> //宏定义导出
> #ifdef LIB__//如果没有定义LIB 就定义 LIB __declspec(dllexport)
> #define LIB __declspec(dllexport)//导出
> #else
> #define LIB __declspec(dllimport)//导入
> #endif // LIB__//如果没有定义LIB 就定义 LIB
> 
> //定义双链表结构体
> typedef struct LIB Node {
>     int data;
>     struct Node* pre;
>     struct Node* next;
> }Node;
> 
>     LIB Node* initList(void);//初始化双循环链表
>     LIB void headInsert(Node* L, int data);//头插法
>     LIB void tailInsert(Node* L, int data);//尾插法 
>     LIB void deleat(Node* L, int data);//删除
>     LIB void printList(Node* L);//打印
> 
> #endif 
> ```
>
> > 或者：
> >
> > 创建生成 .dll文件的工程，在其中创建所需要作为库的文件（书写方式与上述相同），之后进行生成。
> >
> > ==ps：==导出模式下记得在引用库文件之前进行声明。
>
> - 通过模块文件生成：
>
> > 1. 首先创建`*.def`文件，之后编写`*.def`文件。
> >
> > ```
> > LIBRARY UnitTest_dll   #LIBRARY之后的为要生成的文件名
> > 
> > EXPORTS
> > Add                   #将要导出的文件
> > ```
> >
> > 2. 将配置改为动态库（.dll)。
> > 3. 生成解决方案。

- 动态库的调用：

> 配置文件和代码加载lib与静态库相同。
>
> - 在代码中加载dll
>
> > 首先要引用`windows.h`库，之后创建一个代表一类函数的别名，之后创建一个实例句柄`HINSTANCE DLL` 其中`DLL`可自定义。之后创建一个函数指针，指向要检索的函数（已知类型的==上述创建的别名==），加载动态库文件，获取函数地址，使用函数。
> >
> > ```c
> > #include<stdio.h>
> > #include<windows.h>
> > 
> > typedef struct  Node {
> >     int data;
> >     struct Node* pre;
> >     struct Node* next;
> > }Node;
> > 
> > typedef void (*Load_Func1)(Node* L, int data);//创建函数别名
> > typedef void (*Load_Func2)(Node* L);
> > typedef Node* (*Load_Func3)(void);
> > 
> > int main() {
> >     HINSTANCE DLL;//创建指向动态库的句柄
> >     Load_Func1 headInsert, tailInsert, deleat;//定义函数指针
> >     Load_Func2 printList;
> >     Load_Func3 initList;
> >     DLL = LoadLibrary(L"Dll1.dll");//加载动态库“Dll1.dll”文件
> >     if (DLL != NULL) {//如果加载到了就加载所需要的函数
> >         headInsert = (Load_Func1)GetProcAddress(DLL, "headInsert");
> >         tailInsert = (Load_Func1)GetProcAddress(DLL, "tailInsert");
> >         deleat = (Load_Func1)GetProcAddress(DLL, "deleat");
> >         printList = (Load_Func2)GetProcAddress(DLL, "printList");
> >         initList = (Load_Func3)GetProcAddress(DLL, "initList");
> > 
> >         Node* L = initList();//调用函数
> >         headInsert(L, 1);
> >         headInsert(L, 1);
> >         headInsert(L, 2);
> >         printList(L);
> >         tailInsert(L, 2);
> >         printList(L);
> >         deleat(L, 1);
> >         printList(L);
> > 
> >         FreeLibrary(DLL);//释放动态库
> >     }
> > }
> > 
> > /*   
> > *   LIB Node* initList(void);//初始化双循环链表
> > *   LIB void headInsert(Node* L, int data);//头插法
> > *   LIB void tailInsert(Node* L, int data);//尾插法 
> > *   LIB void deleat(Node* L, int data);//删除
> > *   LIB void printList(Node* L);//打印
> > * 
> > *   Node* L = initList();
> > *   headInsert(L, 1);
> > *   headInsert(L, 1);
> > *   headInsert(L, 2);
> > *   printList(L);
> > *   tailInsert(L, 2);
> > *   printList(L);
> > *   deleat(L, 1);
> > *   printList(L);
> >  */
> > ```
> >
> > 