# Employee-Management-System-Based-on-Polymorphism
The system leverages the power of object-oriented programming, specifically the concept of polymorphism, to provide flexibility and extensibility.

### 01 #pragma once 
在C++编程中，#pragma once是一个预处理器指令，用于防止头文件的内容被多次包含。当编译器遇到#pragma once指令时，它会记住这个头文件，并且在后续的编译过程中，如果再次遇到这个头文件，它会忽略这个头文件的内容，从而避免了重复包含。

### 02 纯虚函数不需要.cpp
如果一个类只包含纯虚函数，并且没有其他成员函数或数据成员，那么这个类的定义完全可以只在头文件（.h文件）中完成，而不需要对应的源文件（.cpp文件）。在这里 纯虚函数 用 virtual void showInfo() = 0; 这样表示，有一个=0；

```cpp
// AbstractClass.h
class AbstractClass {
public:
    virtual void pureVirtualFunction1() = 0;  // 纯虚函数
    virtual void pureVirtualFunction2() = 0;  // 另一个纯虚函数
    virtual ~AbstractClass() {}  // 虚析构函数
};
```

在这个例子中，AbstractClass 是一个抽象类，包含两个纯虚函数和一个虚析构函数。所有这些函数都在头文件中被声明，且虚析构函数在头文件中被定义（虽然是空的）。因此，不需要.cpp文件。

### 03 测试多态
测试多态可以这样写，注意需要给worker一个基类worker的空指针。然后每次都用一个派生类去给到这个基类的指针，通过指针调用虚函数就能实现动态的多态(使用派生类的函数)。
```cpp
void test()
{
	Worker* worker = NULL;
	worker = new Employee(1, "张三", 1);
	worker->showInfo();
	delete worker;

	worker = new Manager(2, "李四", 2);
	worker->showInfo();
	delete worker;

	worker = new Boss(3, "王五", 3);
	worker->showInfo();
	delete worker;
}
```

这里的指针本身的内存是在栈上的 会随着test() 的结束而销毁，但是指针指向的内存，我们是new在堆上的，需要手动删除管理。

### 04 workerManager类的析构函数
因为wokerManager目前有两个成员:
```cpp
//记录文件中的人数个数
int m_EmpNum;

//员工数组的指针 
Worker** m_EmpArray;
```

其中 Worker** m_EmpArray; 是一个指针的指针，因为数组里存放的是指向worker的指针，所以这里是指针的指针来表示数组。
它的析构函数是这样的:
```cpp
WorkerManager::~WorkerManager()
{
	if (this->m_EmpArray != NULL)
	{
		delete[] this->m_EmpArray;
	}
}
```

不过GPT给出了更合理的方案:
```cpp
WorkerManager::~WorkerManager()
{
    if (this->m_EmpArray != NULL)
    {
        for (int i = 0; i < m_EmpNum; i++) {
            delete this->m_EmpArray[i];  // 删除每一个Worker对象
        }
        delete[] this->m_EmpArray;  // 删除Worker*数组
        this->m_EmpArray = NULL;  // 将指针设置为NULL，防止成为悬挂指针
    }
}
```
在这个例子中，WorkerManager的析构函数首先删除m_EmpArray数组中的每一个Worker对象，然后再删除m_EmpArray数组本身，最后将m_EmpArray设置为NULL来防止它成为一个悬挂指针。这样可以确保所有使用new分配的内存都被正确地释放，防止内存泄漏。

### 05 预处理和宏
```cpp
#define FILENAME "empFile.txt"
```
是的，#define FILENAME "empFile.txt"这行代码在C和C++语言中被称为预处理指令，特别是这是一个宏定义。预处理器是编译过程的第一步，它负责处理源代码中的预处理指令。

在这种特定的情况下，#define指令创建了一个宏，即在编译期间，预处理器会将源代码中所有的FILENAME替换为"empFile.txt"。

在C和C++编程语言中，#define预处理器指令用于创建宏，它会在编译时把所有宏名替换为宏定义中的内容。宏名和宏定义之间用空格分隔。这是一个基本的宏定义格式：
```cpp
#define macro_name macro_definition
```
在这个格式中：

macro_name 是你想要定义的宏的名称。每次预处理器在代码中遇到这个名称时，都会将其替换为定义。
macro_definition 是宏的定义，它可以是任何文本，包括代码。预处理器会在编译时将代码中的每个 macro_name 替换为这个定义。
```cpp
#define PI 3.14159
```
在这个例子中，每当预处理器在代码中遇到 PI，它都会用 3.14159 来替换它。

同样，这个宏定义：
```cpp
#define FILENAME "empFile.txt"
```
在这个例子中，FILENAME 将在编译时被替换为 "empFile.txt"。所以，如果你在代码中的任何地方写 FILENAME，编译器都会把它看作是 "empFile.txt"。

### 06 在.h中已经包含了的宏和include<>,在.cpp中需要再定义一次吗？
不需要。如果你已经在.h文件中定义了这个宏，那么任何包含（#include）这个.h文件的.cpp文件都可以访问这个宏。这是因为预处理器的#include指令实际上是把指定头文件的内容复制到.cpp文件中。

举个例子，你有一个头文件header.h：
```cpp
// header.h
#ifndef HEADER_H  // 防止头文件被重复包含
#define HEADER_H

#define FILENAME "empFile.txt"

// ... 其他代码 ...

#endif // HEADER_H
```

然后在.cpp文件中包含这个头文件：
```cpp
// main.cpp
#include "header.h"

int main() {
    std::ifstream file(FILENAME);
    // ... 其他代码 ...
    return 0;
}
```
这样，main.cpp就可以访问在header.h中定义的FILENAME宏了。

需要注意的是，为了防止头文件在一个.cpp文件中被重复包含，通常会在头文件中使用预处理器的#ifndef、#define和#endif指令来创建一个"包含保护"。这可以防止因重复包含头文件而导致的重定义错误。

### 07 ofstream 保存文件
ofstream标准库的操作是，要是本来没有这个文件，会新建一个。要是本来的文件里有内容，会覆盖掉。
```cpp
void WorkerManager::save()
{
	ofstream ofs;
	ofs.open(FILENAME, ios::out);


	for (int i = 0; i < this->m_EmpNum; i++)
	{
		ofs << this->m_EmpArray[i]->m_Id << " " 
			<< this->m_EmpArray[i]->m_Name << " " 
			<< this->m_EmpArray[i]->m_DeptId << endl;
	}

	ofs.close();
}
```
ofs 是一个 std::ofstream 类型的对象。std::ofstream 是 C++ 标准库的一部分，它提供了一种方式来将数据输出到文件中。std::ofstream 继承自 std::ostream，因此你可以使用所有的输出流操作符（如 <<）来写入到文件。

在这段代码中：
```cpp
ofstream ofs;
ofs.open(FILENAME, ios::out);

首先创建了一个 std::ofstream 对象 ofs，然后使用 open 方法打开一个文件，文件名由 FILENAME 宏指定。ios::out 是一个打开模式，表示文件应当以写入模式打开。如果你试图打开一个不存在的文件进行写操作（使用 std::ofstream 或者 std::fstream 的 ios::out 模式），C++ 会尝试创建一个新的文件。如果文件创建成功，你就可以正常地向这个新文件写入数据。
```
一旦文件被打开，你就可以使用流插入运算符 << 来将数据写入文件：
```cpp
ofs << "Hello, World!";
```
以上代码会将字符串 "Hello, World!" 写入到文件中。

在完成写操作后，你应该关闭文件：
```cpp
ofs.close();
```
这个操作会关闭文件，并释放所有与之相关的资源。如果你忘记关闭文件，那么当 ofs 对象被析构时（例如，当它离开其声明的作用域时），文件会被自动关闭。但是，显式地关闭文件是一种好的编程习惯，因为这样可以在完成操作后立即释放资源，而不是等待对象被析构。

### 08 ifstream 保存文件
在C++中，std::ifstream是一个输入文件流类，它提供了从文件中读取数据的功能。

以下是一些基本用法：
```cpp
std::ifstream ifs("file.txt"); // 创建一个输入文件流对象并打开文件 "file.txt"

if (!ifs) { // 检查文件是否成功打开
    std::cerr << "Failed to open the file.\n";
    return;
}

std::string line;
while (std::getline(ifs, line)) { // 使用 getline 函数读取文件的每一行
    std::cout << line << "\n";
}

ifs.close(); // 关闭文件
```
一种检查文件是否为空的做法:它通过尝试从文件中读取一个字符来检查文件是否为空。如果文件为空（或者只包含空白字符），那么这个读取操作会立即到达文件末尾，此时ifs.eof()会返回true。
```cpp
//文件存在，并且没有记录
char ch;
ifs >> ch; // 尝试从文件中读取一个字符
if (ifs.eof()) // 检查是否已经到达文件末尾
{
	cout << "文件为空!" << endl;
	this->m_EmpNum = 0; // 将员工数量设置为0
	this->m_FileIsEmpty = true; // 标记文件为空
	this->m_EmpArray = NULL; // 将员工数组设置为NULL
	ifs.close(); // 关闭文件
	return; // 退出函数
}
```

### 09 清空文件
在C++中，ios::trunc 是 std::ofstream 的一种打开模式，用于在打开文件进行写操作时清空该文件的内容。

当你以 ios::trunc 模式打开一个文件时，如果该文件已经存在，其原有内容将被删除（即清空），然后文件被打开以供写入。如果文件不存在，那么一个新文件将被创建。
```cpp
void WorkerManager::Clean_File()
{
	cout << "确认清空？" << endl; // 提示用户确认清空
	cout << "1、确认" << endl; // 1代表确认清空
	cout << "2、返回" << endl; // 2代表返回，不清空

	int select = 0; // 用于保存用户的选择
	cin >> select; // 接收用户的输入

	if (select == 1) // 如果用户选择了1，即确认清空
	{
		//打开模式 ios::trunc 如果存在删除文件并重新创建
		ofstream ofs(FILENAME, ios::trunc); // 以ios::trunc模式打开文件，这将清空文件
		ofs.close(); // 关闭文件流

		if (this->m_EmpArray != NULL) // 如果员工数组不为空
		{
			for (int i = 0; i < this->m_EmpNum; i++) // 遍历员工数组
			{
				if (this->m_EmpArray[i] != NULL) // 如果当前员工不为空
				{
					delete this->m_EmpArray[i]; // 删除当前员工
				}
			}
			this->m_EmpNum = 0; // 将员工数量设置为0
			delete[] this->m_EmpArray; // 删除员工数组
			this->m_EmpArray = NULL; // 将员工数组指针设为空
			this->m_FileIsEmpty = true; // 标记文件为空
		}
		cout << "清空成功！" << endl; // 提示用户清空成功
	}

	system("pause"); // 暂停程序，等待用户按任意键继续
	system("cls"); // 清空控制台屏幕
}
```

























