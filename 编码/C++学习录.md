# C++学习录
- [C++学习录](#c学习录)
  - [一、大小端](#一大小端)
  - [二、结构体对齐](#二结构体对齐)
  - [三、NULL与nullptr](#三null与nullptr)
    - [NULL](#null)
    - [**nullptr**](#nullptr)
  - [四、转义字符](#四转义字符)
    - [**一般转义字符**](#一般转义字符)
    - [**八进制转义字符**](#八进制转义字符)
    - [**十六进制转义字符**](#十六进制转义字符)
## 一、大小端

**大端模式（Big Endian）**：是指数据的高字节保存在内存的低地址中，而数据的低字节保存在内存的高地址中，这样的存储模式有点儿类似于把数据当作字符串顺序处理：地址由小向大增加，而数据从高位往低位放；这和我们的阅读习惯一致。

**小端模式（Little Endian）**：是指数据的高字节保存在内存的高地址中，而数据的低字节保存在内存的低地址中，这种存储模式将地址的高低和数据位权有效地结合起来，高地址部分权值高，低地址部分权值低。

小端模式便于机器处理, 大端模式方便人阅读。

C++怎么判断大小端模式：https://segmentfault.com/a/1190000039289938

## 二、结构体对齐

```c++
#include <iostream>
#include <string>
#include <string.h>
using namespace std;

//#pragma pack(1)
typedef struct
{
    char acChannelName[2];
    char   acChannelPath[4];
    int a;
}S_FAULT_NO_FETCH_FILE;
//#pragma pack()

main()
{
    S_FAULT_NO_FETCH_FILE str = {0};
    char* buff = (char*)&str;
    memcpy(buff, "1234567890", 10);

    cout << "acChannelName: " << (char)str.acChannelName[0] << (char)str.acChannelName[1] << endl;
    cout << "acChannelPath: " << (char)str.acChannelPath[0] << (char)str.acChannelPath[1] << (char)str.acChannelPath[2] << (char)str.acChannelPath[3] << endl;
    cout << "acChannelPath: " << (char)str.acChannelPath[4] << (char)str.acChannelPath[5] << endl;
    cout << "str.a " << str.a << endl;
    cout << "S_FAULT_NO_FETCH_FILE: " << sizeof(S_FAULT_NO_FETCH_FILE) << endl;
}
```

```
$ g++ -g byte_alignment.cpp -o byte_alignment
$ ./byte_alignment
acChannelName: 12
acChannelPath: 3456
acChannelPath: 78
str.a 12345 // 虚机为小端，字符0对应的十六进制（0x30） + 字符9对应的十六进制（0x39） = 0x3039(十进制为123456)
S_FAULT_NO_FETCH_FILE: 12
```

添加pragma pack(1)，使结构体按1个字节对齐。

```c++
#include <iostream>
#include <string>
#include <string.h>
using namespace std;

#pragma pack(1)
typedef struct
{
    char acChannelName[2];
    char   acChannelPath[4];
    int a;
}S_FAULT_NO_FETCH_FILE;
#pragma pack()

main()
{
    S_FAULT_NO_FETCH_FILE str = {0};
    char* buff = (char*)&str;
    memcpy(buff, "1234567890", 10);

    cout << "acChannelName: " << (char)str.acChannelName[0] << (char)str.acChannelName[1] << endl;
    cout << "acChannelPath: " << (char)str.acChannelPath[0] << (char)str.acChannelPath[1] << (char)str.acChannelPath[2] << (char)str.acChannelPath[3] << endl;
    cout << "acChannelPath: " << (char)str.acChannelPath[4] << (char)str.acChannelPath[5] << endl;
    cout << "str.a " << str.a << endl;
    cout << "S_FAULT_NO_FETCH_FILE: " << sizeof(S_FAULT_NO_FETCH_FILE) << endl;
}

```

```c++
$ g++ -g byte_alignment.cpp -o byte_alignment
$ ./byte_alignment
acChannelName: 12
acChannelPath: 3456
acChannelPath: 78
str.a 809056311 // 虚机为小端，字符0（0x30） + 字符9（0x39）+ 字符8（0x38） + 字符7（0x37） = 0x30393837(十进制为809056311)
S_FAULT_NO_FETCH_FILE: 10
```

## 三、NULL与nullptr

### NULL

NULL在C++中，定义如下：

```c++
#undef NULL
#if defined(__cplusplus)
#define NULL 0
#else
#define NULL ((void *)0)
#endif
```

为什么NULL在C++和C中**不一样**呢？

```c++
// null.cpp
#include<iostream>
int main(void)
{
    char p[] = "12345";
    int *a = (void*)p;
    return 0;
}
```

编译报错如下：

```c++
$ g++ -g null.cpp -o null
null.cpp: In function int main():
null.cpp:5:14: error: invalid conversion from void* to int* [-fpermissive]
     int *a = (void*)p;
```

### **nullptr**

nullptr的实际类型是std:nullptr_t。nullptr既非整型类别，也非指针类型，但是能转换成任意指针类型。

```c++
//来源：公众号【编程珠玑】，https://www.yanbinghu.com
#include<iostream>
using namespace std;
template<typename Type1,typename ptrType>
void test(Type1 fun,ptrType ptr)
{
    /*do something*/
    fun(ptr);
    return;
}
void fun(int *val)
{
    cout<<"fun"<<endl;
}
int main(void)
{
    test(fun,NULL);
    return 0;
}
```

编译报错如下：

```c++
main.cpp:8:8: error: invalid conversion from ‘long int’ to ‘int*’ [-fpermissive]
     fun(ptr);
```

显然，NULL被推导为long int，而不是空指针，因而导致函数类型不匹配而报错。

## 四、转义字符

| **ASCII值** | **16进制** | **控制字符** |
| ----------- | ---------- | ------------ |
| **34**      | **22H**    | **”**        |
| **49**      | **31H**    | **1**        |
| **97**      | **61H**    | **a**        |

### **一般转义字符**

形式上由两个字符组成，代表一个字符。常用的一般转义字符为：\’、 \＂。

```c++
int main() {
    char a = '\“'; // 6*8^1+1*8^0=49
    printf("a_d = (%d), a_c = (%c)", a, a); // a_d = (34), a_c = (")
}
```

### **八进制转义字符**

八进制整数，且是有符号整数（-128~127）（补码存储）。超过八位时，异常情况与编译器有关。

```c++
int main() {
    char a = '\61'; // 6*8^1+1*8^0=49
    printf("a_d = (%d), a_c = (%c)", a, a); // a_d = (49), a_c = (1)
}
```

### **十六进制转义字符**

由反斜杠'\'和字母x(或X)及随后的1～2个十六进制数字构成的字符序列。例如，'\x30'、'\x41'、'\X61'分别表示字符'0'、'A'和'a'。因为字符'0'、'A'和'a'的ASCII码的十六进制值分别为0x30、0x41和0x61。

```c++
int main() {
    char a = '\x61'; // 6*16^1+1*16^0=97
    printf("a_d = (%d), a_c = (%c)", a, a); // a_d = (97), a_c = (a)
}
```

