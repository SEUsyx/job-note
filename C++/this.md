# this 指针


### 用处
- 一个对象的this指针并不是对象本身的一部分，不会影响sizeof(对象)的结果。
- this作用域是在类内部，当在类的非静态成员函数中访问类的非静态成员的时候，编译器会自动将对象本身的地址作为一个隐含参数传递给函数。也就是说，即使你没有写上this指针，编译器在编译的时候也是加上this的，它作为非静态成员函数的隐含形参，对各成员的访问均通过this进行。

### 使用方法
- 在类的非静态成员函数中返回类对象本身的时候，直接使用 `return *this`
- 当参数与成员变量名相同时，如`this->n = n` （不能写成n = n）

注：this指针会被解析成 `<类型名>* const`，即this指着是个const指针（指针不可变，指针内容可变）如果是const函数，则会被解析成`const <类型名>* const`

```C++
#include<iostream>
#include<cstring>


using namespace std;
class Person{
public:
    typedef enum {
        BOY = 0, 
        GIRL 
    }SexType;
    Person(char *n, int a,SexType s){
        name=new char[strlen(n)+1];
        strcpy(name,n);
        age=a;
        sex=s;
    }
    int get_age() const{
    
        return this->age;  // this指针是 const Person* const
    }
    Person& add_age(int a){
        age+=a;
        return *this;  // this指针是 Person* const
    }
    ~Person(){
        delete [] name;
    }
private:
    char * name;
    int age;
    SexType sex;
};


int main(){
    Person p("zhangsan",20,Person::BOY); 
    cout<<p.get_age()<<endl;
	cout<<p.add_age(10).get_age()<<endl;
    return 0;
}
```