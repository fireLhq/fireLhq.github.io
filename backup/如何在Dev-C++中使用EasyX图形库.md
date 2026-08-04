# Dev-C++配置EasyX

## 1.下载EasyX库

打开官网：[EasyX Graphics Library for C++](https://easyx.cn/)

<img width="1910" height="896" alt="Image" src="https://github.com/user-attachments/assets/f9697f40-adc9-4b50-ab4d-f8ec3816d8a8" />

<img width="1910" height="896" alt="Image" src="https://github.com/user-attachments/assets/df22558e-1046-4353-8d6f-803e6b303567" />

<img width="1910" height="896" alt="Image" src="https://github.com/user-attachments/assets/27876219-70c2-443b-a15f-848bd819dfbe" />

<img width="1910" height="896" alt="Image" src="https://github.com/user-attachments/assets/8bfcd848-d891-4d92-9659-42f6a35f121d" />

下载后会得到**easyx4mingw_25.9.10.zip**文件，内部结构如下

<img width="414" height="155" alt="Image" src="https://github.com/user-attachments/assets/cc23c21f-e67e-44cc-aa41-1e2af42b5389" />

将文件解压并重命名后放到一个自己能找到的地方，如放到**D:\EasyX**，该文件下就直接放置上面几个文件夹：

```text
D:\EasyX\
├──include\ #头文件，程序引用的入口
├──lib32\ #32位库文件，用于32位MinGW GCC
├──lib64\ #64位库文件，用于64位MinGW GCC
├──lib-for-devcpp_5.4.0\ #32位库文件，用于5.4.0版Dev-C++
└──readme.txt #说明文档
```

## 2.在Dev-C++中引入库、C++包含文件

这里以**小熊猫Dev-C++ 6.7.5**为例，其它版本如**原版Dev-C++ 5.11**操作类似

依次点击：**工具→编译选项→编译器配置→目录**

### 2.1引入库

在**库**中添加目录：**lib64**，并移至顶部，注意根据自己实际**MinGW**和**Dev-C++版本**选择
引入后如：**D:\EasyX\lib64**

<img width="1920" height="1017" alt="Image" src="https://github.com/user-attachments/assets/2dd5206b-4a8c-4044-8280-a402ddafffe7" />

### 2.2引入C++包含文件

仍然在上个步骤的**目录**中

在**C++包含文件**中添加目录：**include**，并移至顶部
引入后如：**D:\EasyX\include**

<img width="1920" height="1017" alt="Image" src="https://github.com/user-attachments/assets/024b9cb0-56ad-4e79-a84f-53f8bad02881" />

## 3.配置链接参数

在上个步骤的**编译器选项**中，点击**自动链接**，并依次添加**EaxyX**两个头文件的链接参数

<img width="1920" height="1017" alt="Image" src="https://github.com/user-attachments/assets/703f097f-8db4-4f5b-83b5-db81ac273204" />

对于**项目**来说，需要在**项目→项目属性→参数→链接器**中输入`-leasyx -lgdi32 -lole32`，如果是编译运行**单个.cpp文件**则不用这步

<img width="1920" height="1017" alt="Image" src="https://github.com/user-attachments/assets/3be56de7-8a8e-4265-92ec-35374c7c983d" />

## 4.使用EasyX库

上面的步骤完成后就可以使用了，用下面的示例代码测试一下

```c++
#include <conio.h>
#include <easyx.h>

int main()
{
	initgraph(500, 500);
	circle(250, 250, 220);
	_getch();
	closegraph();
	return 0;
}
```

演示效果

<img width="1920" height="1017" alt="Image" src="https://github.com/user-attachments/assets/13017992-aa3f-47dd-b9d3-c85f2dafd0a3" />

可以看到成功绘制出图像！但编译器有个警告：**[Warning] ignoring '#pragma comment ' [-Wunknown-pragmas]**，直接忽略即可。`#pragma comment(lib, "...")`是**Microsoft Visual C++**专用语法，作用是告诉**Visual Studio**编译这个程序时，自动链接某个库。我们用的**Dev-C++**编译器不认识这个**pragma**，所以将它忽略了并给出了警告。前面的步骤我们已经手动链接了参数，所以完全不影响使用，因此直接忽略警告。
