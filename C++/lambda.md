# lambda

auto f=[捕获列表](参数列表){函数体};
- 捕获分为值捕获和引用捕获
  - 值捕获,不允许函数体修改值,要想修改,必须加mutable关键字:`auto f = [val]() mutable {return ++val;};`
  - 引用捕获,允许修改值:`auto f = [&val]() {return ++val;};`
- 指定返回类型
  - `[](){if(i<0) return -i;else return i;};`编译器无法推断返回类型,需要使用尾置返回类型`[]()->int {if(i<0) return -i;else return i;};` 
- lambda的主要应用场景
  - 一两个地方使用的简单操作
  - 某些库函数只接受一元谓词和二元谓词,lambda的捕获列表使得它能使用更多的参数