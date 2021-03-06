# const关键字

### const作用

- 定义常量 `const int a=100` 必须在定义时初始化！
- 类型检查： const常量具有类型，编译器可以对其进行安全检查（这也是const与#define的区别，#define宏定义没有数据类型，只是简单的字符串替换，不能进行安全检查）
- 防止修改，起保护作用
- 节省空间，避免不必要的内存分配（const定义常量从汇编的角度来看，只是给出了对应的内存地址，而不是像#define一样给出的是立即数。const定义的常量在程序运行过程中只有一份拷贝，而#define定义的常量在内存中有若干个拷贝）

注：const定义的变量只有类型为整数或枚举，且以常量表达式初始化时才能作为常量表达式。其他情况下它只是一个 const 限定的变量，不要将与常量混淆。


### const 默认为内部链接性（具体见变量的存储模型）

### const与指针
```C++
const char * a; //指向const对象的指针或者说指向常量的指针。 常量不可变
char const * a; //同上

char * const a; //指向类型对象的const指针。或者说常指针、const指针。指针不可变
const char * const a; //指向const对象的const指针

```

tips:  
1.如果const位于 * 的左侧，则const就是用来修饰指针所指向的变量，即指针指向为常量；可以不用初始化，允许把非const对象的地址赋给指向const对象的指针,不能通过ptr指针来修改val的值，即使它指向的是非const对象!（但可以通过其他方式修改）  
2.如果const位于 * 的右侧，const就是修饰指针本身，即指针本身是常量。const指针必须初始化！且const指针的值不能修改

### cosnt与函数

如果函数需要传入一个指针，是否需要为该指针加上const，把const加在指针不同的位置有什么区别；
- const修饰函数参数：

```C++
void func(const int var); // 传递过来的参数不可变
void func(int *const var); // 指针本身不可变
```
传递过来的参数及指针本身在函数内不可变，无意义！var本身就是形参，在函数内不会改变。包括传入的形参是指针也是一样。

- 参数指针所指内容为常量不可变
```C++
void StringCopy(char *dst, const char *src);
```
其中src 是输入参数，dst 是输出参数。给src加上const修饰后，如果函数体内的语句试图改动src的内容，编译器将指出错误,这就是加了const的作用之一。

- 参数为引用，为了增加效率同时防止修改
```C++
void func(const A &a)
```
引用传递仅借用一下参数的别名而已，不需要产生临时对象。如果不希望改变参数的值，加上const限定

### const与类
- 对于类中的const成员变量必须通过初始化列表进行初始化，也可以在定义时就进行初始化
  
- 任何不会修改数据成员的函数都应该声明为const类型。如果在编写const成员函数时，不慎修改 数据成员，或者调用了其它非const成员函数，编译器将指出错误，这无疑会提高程序的健壮性。

- 使用const关键字进行说明的成员函数，称为常成员函数。只有常成员函数才有资格操作常对象，没有使用const关键字进行说明的成员函数不能用来操作常对象。

- const对象只能访问const成员函数；非const对象可以访问任意的成员函数,包括const成员函数

```C++
#include <iostream>
using namespace std;
class Apple
{
private:
    int people[100];
    const int apple_number;
public:
    Apple(int i):apple_number(i){}

    void take(int num) const{
        cout<<"take func "<<num<<endl;
    }

    void add(int num){
        cout<<"add: ";
        take(num); //非const函数可以访问任意的成员函数,包括const成员函数
    }

    void subtract(int num){
        take(num);
    }

    void add(int num) const{
        cout<<"add-const: ";
        take(num);
    }

    int getCount() const{
        cout<<"get count: ";
        take(1);
        //subtract(1); //error! const成员函数只能访问const成员函数!
        return apple_number;
    }

};


int main(){
    Apple a(2);
    cout<<a.getCount()<<endl;
    a.add(10); //说明非const对象默认调用非const成员函数
    const Apple b(3); //说明const对象默认调用const成员函数
    b.add(100);
    //b.subtract(10); //error! 说明const对象只能调用const成员函数
    return 0;
}
```
```
get count: take func 1
2
add: take func 10
add-const: take func 100
```

### const变量如何保证不被修改

- 分2种变量。一种是在函数内部定义的const，一种是定义在全局的const。
  - 如果是函数内部定义的const，可以通过指针修改
```C++
const int const_value = 100;
int * ptr = (int *)&const_value;
*ptr = 200
cout<<*ptr<<"  "<<const_value<<endl;
//结果输出为200，100，其实已经通过指针将const_value变量地址内的值已经变成了200，但是由于编译器的优化，将const类型的const_value放在编译器的符号表内，计算时编译器直接从表中取值，所以输出为100
```
  - 如果是全局const变量，通过指针修改会报错runtime memory violation
    - 原因：函数级变量是在函数的帧里的，程序拥有对这个存储区写的权限。而全局性的const变量是放在另一个存储区里的，程序默认不拥有写权限
```C++
#include <stdio.h>
#include <stdlib.h>

static const char const_data[16] = "I'm Const Data";

int main(int args,char ** argv)
{   
    // 注意这里用了跟上面一样的trick来修改
    char * pc = (char *)const_data;
    *pc = 'X';
    return 0;
}

//编译通过，运行会报错，在生成ELF的过程中，代码的各部分变量或者数据不是全部无脑放在一起的。他们会被放在不同的地方。可以上网搜一下怎么分析ELF文件。用readelf -S obj看一下ELF静态的目标分区。嗯，你会看见很多东西。简单说一下重点分区，其中.rodata 分区会存放全局常量（也就是我们这个例子中的const_data），.text分区存放源码编译的机器指令。你可以查到.rodata分区会和.text分区会加载到一个段中，并且可操作权限只有R + E，而没有W，所以你write的时候会报错----执行的时候发现你往一块没有写权限的内存写东西了。
```