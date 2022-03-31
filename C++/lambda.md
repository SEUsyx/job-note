# lambda
```c++
auto f= [ 捕获列表 ] ( 参数列表 ){函数体};
```
- 捕获分为值捕获和引用捕获
  - 值捕获,不允许函数体修改值,要想修改,必须加mutable关键字:`auto f = [val]() mutable {return ++val;};`
  - 引用捕获,允许修改值:`auto f = [&val]() {return ++val;};`
- 指定返回类型
  - `[](){if(i<0) return -i;else return i;};`编译器无法推断返回类型,需要使用尾置返回类型`[]()->int {if(i<0) return -i;else return i;};` 
- lambda的主要应用场景
  - 一两个地方使用的简单操作
  - 某些库函数只接受一元谓词和二元谓词,lambda的捕获列表使得它能使用更多的参数
    - 例子: 从find_if()说起:
        ```c++
        InputIterator find_if ( InputIterator first, InputIterator last, Predicate pred )它在区间[first,end)  //中搜寻使一元判断式pred为true的第一个元素。如果没找到，返回end。但是pred只接受一元谓词(只接受一个参数),

        int sz=6;

        find_if(str.begin(),str.end(),[sz](const string& a){return a.size()>=sz;}) //此时可以使用lamba

        bool chack_size(const string& s, string::size_type sz){
            return s.size()>=sz;
        }
        find_if(words.begin(), words.end(), bind(chack_size(),_1, sz)) //也可以使用bind函数
        ```
# bind函数
调用的一般形式:
```c++
auto newCallable= bind(callable, arg_list)
```
通常配合占位符一起使用:_1,_2,_3,......,_n
_n通常定义在placeholders的命名空间中, 通常使用`using namespace std::placeholders `声明
bind()传参默认使用拷贝(值传递),若想要使用引用,则需要使用std::ref()函数
其他详细使用说明见C++ primer p357