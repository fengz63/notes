### 1. const关键字

+ 1.1 不考虑类的情况
    + const常量在定义时必须被初始化，之后无法更改
    + const形参可以接受const和非const类型的实参，如：
        ```cpp
        void fun(const int& i){}    //可以是int类型或者const int类型
        ```
+ 1.2 考虑类
    + const成员变量：不能再类定义外部初始化，只能通过构造函数初始化列表进行初始化，并且必须有构造函数；不同类对其const数据成员的值可以不同，所以不能再类中声明时初始化。
    + const成员函数：放在函数名后面，大括号前面。表明这个函数是不能改变类的成员变量的（加了mutable修饰的除外）。详细如下图所示：
    ![](https://raw.githubusercontent.com/fengz63/picture/main/20210326152237.png)

### 2. 顶层const和底层const

+ 2.1  顶层const