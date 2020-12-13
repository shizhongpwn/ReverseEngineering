# DLL注入

> 最近一直搞windows,参加了一些实战，颇有体会，记录一下。

## 原理
DLL注入将含有恶意代码的DLL通过被远程进程调用的方式来启动恶意代码，这对于免杀是一种常见切有效的方法。
### DLL如何注入进正常程序？
​	恶意代码的加载器通过让远程进程调用Windows API **LoadLibrary**的方式强制远程进程加载恶意DLL进而实现注入，注入之后，OS会自动的调用DLLMain函数。

​	首先恶意代码加载器需要的是要注入的进程的句柄，这一般也是先通过API函数(CreateToolhelp32Snapshot,Process32First,Process32Next)发现进程，可以想象成遍历某个目录中的文件来找到想要的文件，发现进程后就会提取PID，通过windows API OpenProcess来获取进程的句柄。

​	在远程进程中调用VirtualAllocEx（如果为该API提供远程进程的句柄那么其将在远程进程中分配内存空间）来开辟内存空间，然后，调用WriteProcessMemory来将恶意DLL的名称字符写入到VirtualAllocEx函数开辟的内存空间。

​	我们可以通过加载Kernel32.dll来确定LoadLibrary的函数地址，最后我们就可以通过CreateRemoteThread实现注入，该API最重要的三个参数：受害进程的句柄，LoadLibrary的地址，以及受害进程中DLL的名称字符串。

### 恶意DLL的特点
恶意DLL的DLLMain函数一般只包含恶意程序（他也不会写一个完整正常的DLL吧？）。

一旦注入程序，恶意DLL中的恶意程序会获得和注入进程相同的权限。

### 如何识别是否是DLL注入

启动器代码中的API，通过启动器中不会含有恶意程序，但是其会通过调用大量的危险API实现将附带的DLL注入到电脑的其他进程当中。

### 例子

![image-20200329161603909](F:\笔记\恶意代码实战\注入\DLL注入\DLL注入.assets\image-20200329161603909.png)