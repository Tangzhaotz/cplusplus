## c++易混淆知识点及难点

### 1、其他地方补充的

#### 1、stoi（）函数

stoi（字符串，起始位置，n进制），将 n 进制的字符串转化为十进制  示例：  stoi(str, 0, 2); //将字符串 str 从 0 位置开始到末尾的 2 进制转换为十进制。stoi()会做范围检查，默认范围是在int的范围内的，如果超出范围的话则会runtime error！

#### 2、const

**（1）、const对象默认为文件局部变量**

非const变量默认为extern。要使const变量能够在其他文件中访问，必须在文件中显式地指定它为extern。

未被const修饰的变量在不同文件下的访问

```c++
// file1.cpp
int ext
// file2.cpp
#include<iostream>
/**
 * by 光城
 * compile: g++ -o file file2.cpp file1.cpp
 * execute: ./file
 */
extern int ext;
int main(){
    std::cout<<(ext+10)<<std::endl;
}
```

const常量在不同文件的访问

```c++
//extern_file1.cpp
extern const int ext=12;
//extern_file2.cpp
#include<iostream>
/**
 * by 光城
 * compile: g++ -o file const_file2.cpp const_file1.cpp
 * execute: ./file
 */
extern const int ext;
int main(){
    std::cout<<ext<<std::endl;
}
```

**（2）指针与const**

```
const char * a; //指向const对象的指针或者说指向常量的指针。
char const * a; //同上
char * const a; //指向类型对象的const指针。或者说常指针、const指针。
const char * const a; //指向const对象的const指针。
```

如果*const*位于`*`的左侧，则const就是用来修饰指针所指向的变量，即指针指向为常量；如果const位于`*`的右侧，*const*就是修饰指针本身，即指针本身是常量。

**指向常量的指针常量的值不可以修改（指向常量的指针）**

```c++
const int * a =10;
*a=20;  //错误，不能更改常量的值，但是指针的指向可以修改
```

**常量指针指向不可以修改，且必须初始化（常指针）**

```c++
i#include<iostream>
using namespace std;
int main(){

    int num=0;
    int * const ptr=&num; //const指针必须初始化！且const指针的值不能修改
    int * t = &num;
    *t = 1;
    cout<<*ptr<<endl;
}
```

当把一个const常量的地址赋值给ptr时候，**由于ptr指向的是一个变量，而不是const常量**，所以会报错，出现：const int`*` -> int `*`错误！

```c++
#include<iostream>
using namespace std;
int main(){
    const int num=0;  //const常量
    int * const ptr=&num; //error! const int* -> int*
    cout<<*ptr<<endl;
}
```



除此之外，也不能使用void`*`指针保存const对象的地址，必须使用const void`*`类型的指针保存const对象的地址。

```c++
const int p = 10;
const void * vp = &p;
void *vp = &p; //error
```

**允许把非const对象的地址赋值给指向const对象的指针**

```c++
const int *ptr;
int val = 3;
ptr = &val; //ok
```

**指向常量的常指针**

```c++
const int p = 3;
const int * const ptr = &p; 
```

**（3）const修饰函数的形参列表**

1、对于非内部数据类型的参数而言，象void func(A a) 这样声明的函数注定效率比较低。因为函数体内将产生A 类型的临时对象用于复制参数a，而临时对象的构造、复制、析构过程都将消耗时间。

2、对于内部数据类型来说，即常见的数据类型int，double、char等

内部数据类型的参数不存在构造、析构的过程，而复制也非常快，“值传递”和“引用传递”的效率几乎相当。

总结：**对于非内部数据类型的输入参数，应该将“值传递”的方式改为“const 引用传递”，目的是提高效率。例如将void func(A a) 改为void func(const A &a)。
对于内部数据类型的输入参数，不要将“值传递”的方式改为“const 引用传递”。否则既达不到提高效率的目的，又降低了函数的可理解性。例如void func(int x) 不应该改为void func(const int &x)。**

**（4）类中使用const**

在一个类中，任何不会修改数据成员的函数都应该声明为const类型。如果在编写const成员函数时，不慎修改数据成员，或者调用了其它非const成员函数，编译器将指出错误，这无疑会提高程序的健壮性。使用const关字进行说明的成员函数，称为常成员函数。只有常成员函数才有资格操作常量或常对象，没有使用const关键字明的成员函数不能用来操作常对象。

对于类中的const成员变量必须通过初始化列表进行初始化，如下所示：

```c++
class Apple
{
private:
    int people[100];
public:
    Apple(int i); 
    const int apple_number;
};

Apple::Apple(int i):apple_number(i)
{

}
```

const对象只能访问const成员函数,而非const对象可以访问任意的成员函数,包括const成员函数.

```c++
//apple.cpp
class Apple
{
private:
    int people[100];
public:
    Apple(int i);   //构造函数
    const int apple_number;  
    void take(int num) const;  //const成员函数
    int add(int num);
    int add(int num) const;  //const成员函数
    int getCount() const;  //const成员函数

};
//main.cpp
#include<iostream>
#include"apple.cpp"
using namespace std;

Apple::Apple(int i):apple_number(i)  //初始化
{

}
int Apple::add(int num){
    take(num);
}
int Apple::add(int num) const{
    take(num);
}
void Apple::take(int num) const
{
    cout<<"take func "<<num<<endl;
}
int Apple::getCount() const
{
    take(1);
//    add(); //error
    return apple_number;
}
int main(){
    Apple a(2);
    cout<<a.getCount()<<endl;
    a.add(10);
    const Apple b(3);
    b.add(100);
    return 0;
}
//编译： g++ -o main main.cpp apple.cpp
//结果
take func 1
2
take func 10
take func 100
```

#### 3、static相关

#### 3.1、静态变量

**静态变量：** 函数中的变量，类中的变量

**静态类的成员：** 类对象和类中的函数

**1、静态变量**

**函数中的静态变量**

当变量声明为static时，空间**将在程序的生命周期内分配**。即使多次调用该函数，静态变量的空间也**只分配一次**，前一次调用中的变量值通过下一次函数调用传递

```c++
#include <iostream> 
#include <string> 
using namespace std; 

void demo() 
{ 
    // static variable 
    static int count = 0; 
    cout << count << " "; 

    // value is updated and 
    // will be carried to next 
    // function calls 
    count++; 
} 

int main() 
{ 
    for (int i=0; i<5; i++)  
        demo(); 
    return 0; 
} 
```

输出：0 1 2 3 4 

**类中的静态变量**

由于声明为static的变量只被初始化一次，因为它们在单独的静态存储中分配了空间，因此类中的静态变量**由对象共享**

**类中的静态变量应由用户使用类外的类名和范围解析运算符显式初始化**，在类外初始化

```c++
#include<iostream> 
using namespace std; 

class Apple 
{ 
public: 
    static int i; 

    Apple() 
    { 
        // Do nothing 
    }; 
}; 

int Apple::i = 1; 

int main() 
{ 
    Apple obj; 
    // prints value of i 
    cout << obj.i; 
} 
```

```c++
class Person
{
	
public:

	static int m_A; //静态成员变量

	//静态成员变量特点：
	//1 在编译阶段分配内存
	//2 类内声明，类外初始化
	//3 所有对象共享同一份数据

private:
	static int m_B; //静态成员变量也是有访问权限的
};
int Person::m_A = 10;
int Person::m_B = 10;

void test01()
{
	//静态成员变量两种访问方式

	//1、通过对象
	Person p1;
	p1.m_A = 100;
	cout << "p1.m_A = " << p1.m_A << endl;

	Person p2;
	p2.m_A = 200;
	cout << "p1.m_A = " << p1.m_A << endl; //共享同一份数据
	cout << "p2.m_A = " << p2.m_A << endl;

	//2、通过类名
	cout << "m_A = " << Person::m_A << endl;


	//cout << "m_B = " << Person::m_B << endl; //私有权限访问不到
}

int main() {

	test01();

	system("pause");

	return 0;
}
```

**2、静态成员**

允许静态成员函数仅访问静态数据成员或其他静态成员函数，它们无法访问类的非静态数据成员或成员函数。

```c++
class Person
{

public:

	//静态成员函数特点：
	//1 程序共享一个函数
	//2 静态成员函数只能访问静态成员变量
	
	static void func()
	{
		cout << "func调用" << endl;
		m_A = 100;
		//m_B = 100; //错误，不可以访问非静态成员变量
	}

	static int m_A; //静态成员变量
	int m_B; // 
private:

	//静态成员函数也是有访问权限的
	static void func2()
	{
		cout << "func2调用" << endl;
	}
};
int Person::m_A = 10;


void test01()
{
	//静态成员变量两种访问方式

	//1、通过对象
	Person p1;
	p1.func();

	//2、通过类名
	Person::func();


	//Person::func2(); //私有权限访问不到
}

int main() {

	test01();

	system("pause");

	return 0;
}
```

#### 4、this指针

this指针的用处：

（1）一个对象的this指针并不是对象本身的一部分，不会影响sizeof(对象)的结果。

（2）this作用域是在类内部，当在类的非静态成员函数中访问类的非静态成员的时候，编译器会自动将对象本身的地址作为一个隐含参数传递给函数。也就是说，即使你没有写上this指针，编译器在编译的时候也是加上this的，它作为非静态成员函数的隐含形参，对各成员的访问均通过this进行。

this指针的使用：

（1）在类的非静态成员函数中返回类对象本身的时候，直接使用 return *this。

（2）当参数与成员变量名相同时，如this->n = n （不能写成n = n)。

#### 5、inline内联函数

1、内联函数

在高级程序语言编程中，我们经常使用函数，使用函数的好处就是修改方便，便于我们代码的设计，但是函数存在的缺点就是：

调用函数速度很慢：调用前要先保存到寄存器，并在返回时恢复，复制实参，程序还必须转向一个新的位置执行

```c++
#include<iostream>
using namespace std;

int func(int a,int b)
{
    return a + b;
}

int main()
{
    int x = 10;
    int y =20;
    int c = func(x,y);  //这里调用函数，需要进行实参的复制，还有程序会跑到上面函数的位置进行计算，计算完的返回值，又要返回到这里，速度会很慢
    cout <<"c=" << c << endl;  //30
    system("pause");

    return 0;
}
```

![image-20210515202011360](/home/tz/Pictures/image-20210515202011360.png)

c++中提出使用内联函数来提高函数执行的效率，用关键字inline放在函数定义之前（注意是定义之前，不是声明）

采用内联函数速度变快了（604和647对比）

![image-20210515202222684](/home/tz/Pictures/image-20210515202222684.png)

定义在类内部的成员函数将自动地成为内联函数

```c++
class A
{
    public:
    void Foo(int x,int y){};  //成员函数，自动地成为内联函数
}
```

内联函数应该在头文件中定义，这一点不同于其他函数，编译器在调用点内联地展开，必须能够找到inline函数的定义菜鸟呢个将调用函数替换为函数代码，而对于在头文件中仅有声明是不够的。当然，也可以放在源文件中，但此时只有定义那个源文件可以使用它，而且必须为每一份源文件拷贝一份定义，头文件也需要拷贝，只是编译器代替你完成，所以最好放在头文件中比较方便。

**谨慎使用内联函数**

内联函数

```c++
#include <iostream>
#include "inline.h"

using namespace std;

/**
 * @brief inline要起作用,inline要与函数定义放在一起,inline是一种“用于实现的关键字,而不是用于声明的关键字”
 *
 * @param x
 * @param y
 *
 * @return 
 */
int Foo(int x,int y);  // 函数声明
inline int Foo(int x,int y) // 函数定义
{
    return x+y;
}

// 定义处加inline关键字，推荐这种写法！
inline void A::f1(int x){

}

int main()
{


    cout<<Foo(1,2)<<endl;

}
/**
 * 编译器对 inline 函数的处理步骤
 * 将 inline 函数体复制到 inline 函数调用点处；
 * 为所用 inline 函数中的局部变量分配内存空间；
 * 将 inline 函数的的输入参数和返回值映射到调用方法的局部变量空间中；
 * 如果 inline 函数有多个返回点，将其转变为 inline 函数代码块末尾的分支（使用 GOTO）。
 */
```

以下情况不宜用内联：

（1）如果函数体内的代码比较长，使得内联将导致内存消耗代价比较高。

（2）如果函数体内出现循环，那么执行函数体内代码的时间要比函数调用的开销大。

（3）、递归尽量不要使用内联，递归因为效率本来就低，使用内联函数也不会有多快。

**虚函数（virtual）可以是内联函数（inline）吗？**

- 虚函数可以是内联函数，内联是可以修饰虚函数的，但是当虚函数表现多态性的时候不能内联。

- 内联是在编译器进行编译期内联，而虚函数的多态性在运行期，编译器无法知道运行期调用哪个代码，因此虚函数表现为多态性时（运行期）不可以内联。

- `inline virtual` 唯一可以内联的时候是：编译器知道所调用的对象是哪个类（如 `Base::who()`），这只有在编译器具有实际对象而不是对象的指针或引用时才会发生。

  ```c++
  /*
  内联函数
  */
  #include<iostream>
  using namespace std;
  #include<time.h>
  
  class Base
  {
      public:
      inline virtual void who()
      {
          cout << "i am Base" << endl;
      }
  
      virtual ~Base(){};  //虚析构函数
  };
  
  class Derived:public Base
  {
      public:
      inline void who()
      {
          cout << " i am Derived" << endl;
      }
  };
  
  int main()
  {
      //此处虚函数who（）是通过Base的具体对象b来调用的，所以确定了编译时间，所以可以使内联的，但是最终内联还是要取决于编译器
      Base b;
      b.who();  //这里调用是内联的
  
      Base *ptr = new Derived();  //这里涉及到向上转型，虽然是子类new了对象，但是传递给左边的父类，是多态调用，需要在运行期间才能确定具体的对象调用，所以不能为内联
      ptr->who();
  
      delete ptr;  //程序员自己申请的内存需要自己释放
      ptr=nullptr;  //避免出现野指针，将指针置空
  
      return 0;
  }
  ```

  

#### 6、c++prime

含有无符号类型的表达式

1、与无符号数字做运算时，先要将数值转换为无符号数值再计算

2、当从无符号减去一个数时，不管这个值是不是无符号，必须确保结果不为负

```c++
#include<iostream>`

`using namespace std;`



`int main()`

`{`

​    `unsigned u=10;  //定义无符号整数`

​    `int i=-42;  //定义整数`

​    `cout << i+i << endl;  //-84`

​    `cout << u+i << endl;  //4294967264  ,这里先把-42转换为无符号数，再计算`



​    `unsigned u1=42,u2=10;`

​    `cout << u1-u2 << endl;  //32`

​    `cout << u2-u1 << endl;  //4294967264`



​    `system("pause");`



​    `return 0;`

`}
```



初始化不是赋值，初始化的含义是创建变量时赋予其一个初值，赋值的含义是把对象当前的值擦出，而以一个新值来代替



声明与定义的区别：

声明使得名字为程序所知，定义负责创建与名字相关的实体，变量声明规定了变量的类型和名字，定义除了上述的操作外，还申请了存储空间

如果要声明一个变量而不是定义需要使用extern关键字

```c++
extern int i;  //声明

int j;  //定义


```

```c++
extern int i =10; //extern被忽视，直接变为定义
```



引用：

引用必须初始化，而且一旦初始化就不可以更改

一般定义程序的时候，程序把引用它的初始值绑定在一起，而不是将初始值拷贝给引用

引用只能绑定在对象上，不能与字面值或者某个表达式的计算结果绑定



```c++
int a=10;

int &b = a;  //引用绑定到a

int &c;  //错误，没有初始化



int &c = 10; //错误，不能将引用直接绑定字面值






```



指针：

指针是指向另外一种类型的符合数据类型，指针本身也是对象，而引用本身不是对象

指针可以改变指向的位置

指针无须在定义时赋值



 void * 指针：

 是一种特殊类型的指针，可以用于存放任何对象的地址

 不能直接操作void×所指的对象，因为我们不知道是什么类型



 指向指针的指针

 

```c++
int val = 1024;

 int *p= &val;  //p是一个指针，存放的是val的地址

 int **p = &p;  //指向指针的指针，存放的是指针p的地址
```



 指向指针的引用：

 因为指针是一个对象，所以可以定义指向指针的引用，表示给指针起别名

 

```c++
int i =43;

 int *p = &i;  //定义指针指向i

 int *&r = p;  //给指针起别名
```



const限定符：

const对象一旦创建完成以后其值就不能再改变，所以const对象必须初始化

如果利用一个对象去初始化另外一个对象，则他们是不是const对象都可以

```c++
int i =42;

const int ci =i;  //正确

int j = ci;  //正确


```

默认情况下，const对象仅在文件内有效，当多个文件中出现了同名的const变量时，其实等同于在不同的文件中分别设立了独立的变量

可以用extern来将const的范围不仅仅限制在文件内，可以在全部文件有效



const引用：

const可以绑定到对象的上，称谓对常量的引用，对常量的引用不能用来改变它所绑定的对象

```c++
const int i=10;

const int &r1=i;

i=42;  //错误，不能改变
```



允许为一个常量的引用绑定到非常量的对象、字面值，甚至是一个一般表达式

```c++
int i =42;

const int &r1=i;  //true

const int &r2 = 42; //true

const int &r3 = r1 * 2;  //true

int &r4 = r1 * 2; //  false
```



对const的引用可能引用一个并非const的对象

```c++
int i =42;

int &r1=i;  //r1引用i

const int &r2=i;  //r2也绑定i

r1=0;  //修改值

r2=0;  //错误，但是上一步，i的值已经被改变
```



指针和const

指向常量的指针是指绑定的对象的值不能改变，而不是指向的是一个常量，const在*左侧

const指针：指针本身是常量,const 在*右侧



顶层const：指针本身是常量

底层const：指针怎所指的值是一个常量



constexpr和常量表达式

一个对象（表达式）是不是常量表达式由他的数据类型和初始值决定

```c++
const int max_files=20;  //是常量表达式

int staff_size=27;  //不是常量表达式

const int sz = get-size();  //不是常量表达式
```



允许将变量声明为constexpr类型，以便由编译器来验证变量值是否为常量表达式，声明为constexpr变量的一定是一个常量，而且必须用常量表达式初始化

```c++
constexpr int mf = 20;  //是常量表达式

constexpr int limit = mf+1;  //是常量表达式

constexpr int sz = get_size();  //不确定
```



指针和constexpr

在constexpr声明中定义了指针，则限定符constexpr仅对指针有效，与指针所指的对象无关

```c++
const int *p=nullptr;  //指针指向空

constexpr int ×q =nullptr;
```



decltype类型指示符

作用：选择并返回操作数的数据类型



string

直接初始化和拷贝初始化：

直接初始化没有=，拷贝初始化利用=

```c++
string s5 = "hiya";  //拷贝初始化

string s6("hello ");  //直接初始化


```

当把string对象和字符串字面值混在一条语句中时，确保每个+号运算符的两侧的运算对象至少一个为string

```c++
string s4 = s1 + ",";  //正确

string s5 = "hello" + ",";  //错误
```

#### 7、sizeof

###### 1、空类的大小为一个字节

```c++
#include<iostream>
using namespace std;

class A{};  //建立空类

int main()
{
    cout << "空类的大小为："<<sizeof(A) << endl;

    system("pause");

    return 0;
}
```

![image-20210515205053728](/home/tz/.config/Typora/typora-user-images/image-20210515205053728.png)



###### 2、含有虚函数的类的大小

一个类中，虚函数本身、成员函数（包括静态与非静态）和静态数据成员都是不占用类对象的存储空间

```c++
#include<iostream>
using namespace std;


class A
{
    public:
    char b;
    virtual void fun(){};  //虚函数
    static int c;  //静态成员函数
    static int d;
    static int f;
};

int main()
{
    cout << "类的大小为：" << sizeof(A) << endl;
    cout << sizeof(char) << endl;
    int a = 10;
    int *p = &a;
    cout << sizeof(p) << endl;
    return 0;
}
```

![image-20210515210641243](/home/tz/.config/Typora/typora-user-images/image-20210515210641243.png)

这里是16个字节，原因下面解释展开。

###### 3、字节对齐

3.1、什么是字节对齐

现代计算机中内存空间都是按照byte来划分的，理论上讲对任何类型的变量的访问可以从任何地址开始，但是在实际情况中访问特定类型的变量的时候经常在特定的内存地址访问，这就需要各种数据类型按照一定的规则在空间上排列，而不是顺序的一个接着一个。

3.2、字节对齐的作用和原因

各个硬件平台对存储空间的处理上有很大的不同，一些平台对某些数据类型只能从某些特定的地址开始存取，例如一个int类型的数据，占4个字节的大小，如果我们取地址的时候都是从它的倍数开始，那么就只需要进行一次内存读取，如下图所示，如果存储的位置为上面的0x000,那么我们只需要读取0x000到0x004之间就可以存储一个int类型，但是如果我们是从下面的0x002开始，那么我们需要首先取出0x002-0x003的一个short，再取0x004到0x005的一个short，再将两者组合起来才有一个int大小，所以需要访问两次内存，效率比较低。有些平台每次读都是从偶地址开始，如果一个int型（假设为32位系统）如果存放在偶地址开始的地方，那么一个读周期就可以读出这32bit，而如果存放在奇地址开始的地方，就需要2个读周期，并对两次读出的结果的高低字节进行拼凑才能得到该32bit数据。显然在读取效率上下降很多。

![image-20210515215834849](/home/tz/.config/Typora/typora-user-images/image-20210515215834849.png)

3.3、各种数据类型的字节对齐情况

![image-20210515221843908](/home/tz/.config/Typora/typora-user-images/image-20210515221843908.png)

详细的如下：

（1）、空类

```c++
class A
{
};
sizeof(A); //1

```

解析：类的实例化就是为每个实例在内存中分配一块地址；每个类在内存中都有唯一的标识，因此空类被实例化时，编译器会隐含地为其添加一个字节，以作区分。

（2）、虚函数类

```c++
class A
{
    virtual void Fun();
};
sizeof(A); //4

```

解析：当一个类中包含虚函数时，会有一个指向其虚函数表的指针vptr，**系统为类指针分配大小为4个字节(即使有多个虚函数)。**

（3）、普通数据成员

```c++
class A
{
    int a;  //4
    char b;  //2,但是需要进行字节对齐，所以为4
};

sizeof(A); //8

```

解析：普通数据成员，按照其数据类型分配大小，由于字节对齐（4），所以a+b=8字节。

（4）、静态数据成员

```
class A
{
    int a;  //大小为4
    static int b;  //不占对象的大小，为0
};

sizeof(A); //4

```

解析：**静态数据成员存放的是全局数据段，即使它是类的一个成员，但不影响类的大小；**不管类产生多少实例或者派生多少子类，静态成员数据在类中永远只有一个实体存在。而类的非静态数据成员只有被实例化时，才存在，但类的静态数据成员一旦被声明，无论类是否被实例化，它都已存在，类的静态数据成员可以说是一种特殊的全局变量。
（5）、普通成员函数

```c++
class A
{
    void Fun();  //不占类的大小，所以和空类一样
};

sizeof(A); //1


```

解析：**类的大小与它的构造函数、析构函数以及其他成员函数无关，只与它的数据成员相关。**

（6）、普通继承

```c++
class A
{
    int a;  //大小为4
};

class B:public A
{
    int b;  //大小为4，
};

sizeof(B); //8，子类继承包括父类的成员一起计算

```

解析：**普通类的继承，类的大小为本身数据成员大小+基类数据成员大小。**

（7）、**虚函数继承**

```c++
virtual class A
{
    int a;  //4
};

class B:virtual public A
{
    int b;  //4
};

sizeof(B); //12，一个是基类的a大小为4,一个是子类b大小为4,同时虚函数还有一个指针4字节大小，总共12字节

```

解析：**虚函数类的继承，派生类大小=派生类自身成员大小+基类数据成员大小+虚拟指针大小（即使继承多个虚基类，也只有一个指向其虚函数表的指针vptr，大小为4字节）。**

###### 4、结构体的字节对齐

```c++
#include<iostream>
#include<stdlib.h>
using namespace std;

struct bb
{
    int i;  //4字节
    double w;  //8字节
    float h;  //4字节
};

int main()
{
    cout << "sizeof bb" << sizeof(bb) << endl;  //结果：24字节
    return 0;
}
```

解释：结构体字节对齐，假设上述的三个变量为iiii wwwwwwww hhhh，因为最大的变量的字节数为8,所以要进行字节对齐，八个为一组，因为w已经是八个了，不能将其拆分，所以，i要补足八个，需要补5,同理h要补4个，所以总的字节大小为24

```c++
#include<iostream>
#include<stdlib.h>
using namespace std;

struct s1
{
    char c;  //1个字节
    int i;  //4字节
    short f;  //2字节
    double v;  //8字节
};

int main()
{
    cout << "sizeof s1" << sizeof(s1) << endl;  //24字节
    return 0;
}
```

解释：与上述一样，假设变量分别为，c iiii ff vvvvvvvv，最大的字节数为v，占八个字节，所以按八个wei一组，c需要补3,f需要补6,最后的字节数为24



#### 8、纯虚函数和抽象类

1、纯虚函数是指没有实现的函数,并用0来赋值

包含纯虚函数的类称为抽象类

抽象类只能作为基类派生新类使用，不能创建抽象类的对象，抽象类的指针和引用只能指向派生类生出来的对象

2、实现抽象类

抽象类中，在成员函数内可以调用纯虚函数，在构造函数和析够函数内部不能使用纯虚函数;

如果一个类从抽象类派生而来，那么它必须实现基类中的所有纯虚函数，才能成为非抽象类

```c++
#include<iostream>
using namespace std;


class A
{
    public:
    virtual void f() =0;  //纯虚函数
    void g(){this->f();}  //成员函数调用纯虚函数
    A(){};  //构造函数，不能调用纯虚函数
};

class B:public A
{
    void f()
    {
        cout << "B::f()" << enndl;
    }
};

int main()
{
    B b;  //创建对象
    b.g();  //调用成员函数
    return 0;    
}
```

3、重要点

（1）、纯虚函数使得一个类变为抽象类

```c++
#include<iostream>
using namespace std;


class test
{
    int x;
    public:
    virtual void show()=0;  //纯虚函数
    int getX()
    {
        return x;
    }
};
int main()
{
    Test t;  //抽象类不能创建对象
    return 0;
}
```

（2）、抽象类类型的指针和引用

```c++
#include<iostream>
using namespace std;

class Base
{
    int x;
    public:
    virtual void show()=0;
    int getX()
    {
        return x;
    }
};

class Derived:public Base
{
    public:
    void show()
    {
        cout << "i'm derived" << endl;
    }
    Derived(){}  //构造函数
};

int main()
{
    //Base b;  //错误，不能创建对象
    //Base *b = new Base();  //不能创建抽象类对象;

    Base *bp = new Derived();  // 抽象类的指针和引用指向派生类的对象
    bp->show();
    
    return 0;
}
```

（3）、如果不能覆盖抽象类的纯虚函数，那么派生类也会变成抽象类，不能创建抽象类对象

```c++
#include<iostream>
using namespace std;

class Base
{
    int x;
    public:
    virtual void show()=0;
    int getX()
    {
        return x;
    }
};

class Derived:public Base
{
    public:
};

int main()
{
    Derived d;  //不能创建对象
    return 0;
}
```

(4)、抽象类可以有构造函数

（5）、构造函数不能是虚函数，析够函数可以是虚析构函数

虚函数表：当一个类中有虚函数时，编译器会为这个表专门分配一个虚函数表记录这些虚函数，在类中会隐藏一个指针成员来指向这张表。而且一个类中只有一张虚函数表，所有对象共享一张虚函数表。

**构造函数为什么不可以是虚函数？**

因为如果构造函数是虚函数，通过父类指针或者引用去指向子类对象时，会先调用父类的构造函数，但是父类的构造函数是虚函数，被子类中的构造函数覆盖，调用子类的构造函数，指针和引用又会返回去子类的地方，会造成矛盾和死循环。

**析构函数设置为虚函数的原因**

```c++
#include<iostream>
using namespace std;

class Animal
{
    public:
    Animal()
    {
        cout << "animal的构造函数" << endl;
    }

    virtual void speak()=0;  //纯虚函数

    ~Animal()
    {
        cout << "Animal的析构函数调用" << endl;
    }
};

class Cat:public Animal
{
    public:
    Cat(string name)
    {
        cout << "cat的构造函数调用" << endl;
        m_name = new string(name);  //在堆区开辟数据
    }

    virtual void speak()
    {
        cout << *m_name << "小猫在说话" << endl;
    }

    ~Cat()
    {
        cout << "cat的析构函数调用" << endl;
        if(this->m_name !=NULL)
        {
            delete m_name;
            m_name = NULL;
        }
    }
    public:
    string *m_name;
};

void test01()
{
    Animal * animal = new Cat("tom");
    animal->speak();

    delete animal;
}

int main()
{
    test01();
    return 0;
}
```

![image-20210516184727063](/home/tz/.config/Typora/typora-user-images/image-20210516184727063.png)

上述执行结果可以看出，Cat的析构函数并没有调用，所以在堆区申请的内存没有释放掉，会造成内存泄漏。

解决办法就是将基类析构函数设置为虚析构函数：

```c++
#include<iostream>
using namespace std;

class Animal
{
    public:
    Animal()
    {
        cout << "animal的构造函数" << endl;
    }

    virtual void speak()=0;  //纯虚函数

    virtual ~Animal()  //设置为虚析构，就可以避免内存泄漏
    {
        cout << "Animal的析构函数调用" << endl;
    }
};

class Cat:public Animal
{
    public:
    Cat(string name)
    {
        cout << "cat的构造函数调用" << endl;
        m_name = new string(name);  //在堆区开辟数据
    }

    virtual void speak()
    {
        cout << *m_name << "小猫在说话" << endl;
    }

    ~Cat()
    {
        cout << "cat的析构函数调用" << endl;
        if(this->m_name !=NULL)
        {
            delete m_name;
            m_name = NULL;
        }
    }
    public:
    string *m_name;
};

void test01()
{
    Animal * animal = new Cat("tom");
    animal->speak();

    delete animal;
}

int main()
{
    test01();
    return 0;
}
```

![image-20210516185344348](/home/tz/.config/Typora/typora-user-images/image-20210516185344348.png)

也可以设置为纯虚析构，但是需要在类外定义。

#### 9、c++虚函数的vptr与vtable

虚函数表：每个使用虚函数的类或者从虚函数类的派生类都有自己的虚函数表，该表只是在编译时期设置的静态数组，虚拟表包含可由类的对象调用的每一个虚函数的条目。此表中的每一个条目是一个函数指针，指向该类可访问的派生类函数。

其次，编译器还会添加一个以藏的指向基类的指针，我们称为vptr，vptr在创建类实例时自动设置，以便指向该类的虚拟表。与this指针不同，this指针实际上是编译器用来解析自引用的函数参数，vptr是一个真正的指针。

![image-20210517155957193](/home/tz/.config/Typora/typora-user-images/image-20210517155957193.png)

#### 10、virtual

（1）、虚函数与运行多态

虚函数的调用取决于指向或者引用的对象的类型，而不是指针或者自身类型

```c++
#include<iostream>
using namespace std;


class Employee
{
    public:
    virtual void raiseSalary()
    {
        cout << 0 << endl;
    }
    virtual void promote()
    {

    }
};

class Manager:public Employee
{
    virtual void raiseSalary()
    {
        cout << 100 << endl;
    }

    virtual void promote()
    {

    }
};

class Engineer:public Employee
{
    virtual void raiseSalary()
    {
        cout << 200<< endl;
    }

    virtual void promote()
    {

    }
};

void globalRaiseSalary(Employee *emp[],int n)
{
    for(int i=0;i<n;i++)
    {
        emp[i]->raiseSalary();
    }
}

int main()
{
    Employee* emp[]={new Manager(),new Engineer};
    globalRaiseSalary(emp,2);
    return 0;
}
```

（2）、虚函数中默认参数

默认参数是静态绑定的，虚函数是动态绑定的，默认参数的使用需要看引用本身的类型，而不是对象类型

```c++
#include<iostream>
using namespace std;

class Base
{
    public:
    virtual void func(int x=10)
    {
        cout << "Base::func(),x=" << x << endl;
    }
};

class Derived:public Base
{
    public:
    virtual void func(int x=20)
    {
        cout << "Derived::func(),x=" << x << endl;
    }
};


int main()
{
    Derived d1;  //创建对象
    Base *bp = &d1;  //基类指针指向派生类的实例对象
    bp->func();  //Derived::func(),x=10
    return 0;
}
```

上述的代码，对象d1是派生类，bp指向的是派生类的指针，但是在调用函数时，得到的是基类的结果

（3）、可不可以

a、 **静态函数可以声明为虚函数吗？**

原因主要有两方面：

**静态函数不可以声明为虚函数，同时也不能被const 和 volatile关键字修饰**

static成员函数不属于任何类对象或类实例，所以即使给此函数加上virutal也是没有任何意义

虚函数依靠vptr和vtable来处理。vptr是一个指针，在类的构造函数中创建生成，并且只能用this指针来访问它，静态成员函数没有this指针，所以无法访问vptr。

b、**构造函数可以为虚函数吗？**

构造函数不可以声明为虚函数。同时除了inline|explicit之外，构造函数不允许使用其它任何关键字。

为什么构造函数不可以为虚函数？

尽管虚函数表vtable是在编译阶段就已经建立的，但指向虚函数表的指针vptr是在运行阶段实例化对象时才产生的。 如果类含有虚函数，编译器会在构造函数中添加代码来创建vptr。 问题来了，如果构造函数是虚的，那么它需要vptr来访问vtable，可这个时候vptr还没产生。 因此，构造函数不可以为虚函数。

我们之所以使用虚函数，是因为需要在信息不全的情况下进行多态运行。而构造函数是用来初始化实例的，实例的类型必须是明确的。 因此，构造函数没有必要被声明为虚函数。

c、**虚函数可以为私有函数吗？**

- 基类指针指向继承类对象，则调用继承类对象的函数；

- int main()必须声明为Base类的友元，否则编译失败。 编译器报错： ptr无法访问私有函数。 当然，把基类声明为public， 继承类为private，该问题就不存在了。

  ```c++
  #include<iostream> 
  using namespace std; 
  
  class Derived; 
  
  class Base { 
      private: 
          virtual void fun() { cout << "Base Fun"; } 
          friend int main(); 
  }; 
  
  class Derived: public Base { 
      public: 
          void fun() { cout << "Derived Fun"; } 
  }; 
  
  int main() 
  { 
      Base *ptr = new Derived; 
      ptr->fun(); 
      return 0; 
  }
  ```

  

```c++
#include<iostream> 
using namespace std; 

class Derived; 

class Base { 
    public: 
        virtual void fun() { cout << "Base Fun"; } 
     //   friend int main(); 
}; 

class Derived: public Base { 
    private: 
        void fun() { cout << "Derived Fun"; } 
}; 

int main() 
{ 
    Base *ptr = new Derived; 
    ptr->fun(); 
    return 0; 
}
```

d、**虚函数可以被内联吗？**

**通常类成员函数都会被编译器考虑是否进行内联。 但通过基类指针或者引用调用的虚函数必定不能被内联。 当然，实体对象调用虚函数或者静态调用时可以被内联，虚析构函数的静态调用也一定会被内联展开。**

- 虚函数可以是内联函数，内联是可以修饰虚函数的，但是当虚函数表现多态性的时候不能内联。
- 内联是在编译器建议编译器内联，而虚函数的多态性在运行期，编译器无法知道运行期调用哪个代码，因此虚函数表现为多态性时（运行期）不可以内联。
- `inline virtual` 唯一可以内联的时候是：编译器知道所调用的对象是哪个类（如 `Base::who()`），这只有在编译器具有实际对象而不是对象的指针或引用时才会发生。

#### 11、voliate

作用：被voliate修饰的变量，在对其进行读写操作时，会引发一些可观测的副作用，而这些可观测的副作用，是由程序之外的因素决定的

voliate提醒编译器后面的所定义的变量随时都可能改变，因此编译后的程序每次需要存储或者读取这个变量的时候，都会直接从变量地址中读取，如果没有voliate关键字，那么编译器可以优化读取和存储，可能暂时使用寄存器中的值，如果这个变量由别的程序更新了的话，将会出现不一致

```c++
short flag;
vpid test()
{
    do1();
    while(flag==0);
    do2();
}
```

上面这段程序，当flag==0时候将会一直执行循环，do2（）将不会执行，而因为flag不是voliate修饰的，所以计算之前会被先存储在一个寄存器中，但是寄存器不会改变（即使其他地方改变），所以会使得循环一直执行，解决方案就是加voliate修饰。

voliate主要用在这几个地方：

1、中断服务程序中修改的供其它程序检测的变量需要加voliate

2、多任务环境下各任务间共享的标志应该加voliate

3、寄存器映射的硬件寄存器通常也要加voliate，因为每次对它的读写可能有不同的意义

多线程应用中被多个任务共享的变量。 当多个线程共享某一个变量时，该变量的值会被某一个线程更改，应该用 volatile 声明。作用是防止编译器优化把变量从内存装入CPU寄存器中，当一个线程更改变量后，未及时同步到其它线程中导致程序出错。volatile的意思是让编译器每次操作该变量时一定要从内存中真正取出，而不是使用已经存在寄存器中的值。示例如下：

```c++
volatile  bool bStop=false;  //bStop 为共享全局变量  
//第一个线程
void threadFunc1()
{
    ...
    while(!bStop){...}
}
//第二个线程终止上面的线程循环
void threadFunc2()
{
    ...
    bStop = true;
}
```

要想通过第二个线程终止第一个线程循环，如果bStop不使用volatile定义，那么这个循环将是一个死循环，因为bStop已经读取到了寄存器中，寄存器中bStop的值永远不会变成FALSE，加上volatile，程序在执行时，每次均从内存中读出bStop的值，就不会死循环了。

1）一个参数既可以是const还可以是volatile吗？为什么？ 可以。一个例子是只读的状态寄存器。它是volatile因为它可能被意想不到地改变。它是const因为程序不应该试图去修改它。

一个指针可以是volatile吗？为什么？ 可以。尽管这并不常见。一个例子是当一个中断服务子程序修该一个指向一个buffer的指针时。

下面的函数有什么错误？

```c++
int square(volatile int *ptr) 
{ 
return *ptr * *ptr; 
} 
```

这段代码有点变态，其目的是用来返回指针ptr指向值的平方，但是，由于ptr指向一个volatile型参数，编译器将产生类似下面的代码：

```c++
int square(volatile int *ptr) 
{ 
int a,b; 
a = *ptr; 
b = *ptr; 
return a * b; 
} 
```

由于*ptr的值可能被意想不到地改变，因此a和b可能是不同的。结果，这段代码可能返回的不是你所期望的平方值！正确的代码如下：

```c++
long square(volatile int *ptr) 
{ 
int a=*ptr; 
return a * a; 
} 
```

总结：

- volatile 关键字是一种类型修饰符，用它声明的类型变量表示可以被某些编译器未知的因素（操作系统、硬件、其它线程等）更改。所以使用 volatile 告诉编译器不应对这样的对象进行优化。
- volatile 关键字声明的变量，每次访问时都必须从内存中取出值（没有被 volatile 修饰的变量，可能由于编译器的优化，从 CPU 寄存器中取值）
- const 可以是 volatile （如只读的状态寄存器）
- 指针可以是 volatile

#### 12、assert

断言：是宏，而不是函数，assert的作用就是如果它的条件返回错误，那么程序中止，可以通过NDEBUG来关闭assert，但是需要在源码的开头include之前

```c++
void assert(int expression);
```

```c++
#include<iostream>
#include<assert.h>

using namespace std;

int main()
{
    int x=7;  //定义变量x并向它赋予初值7

    x=9;  //将x的值改为9
    assert(x==7);  //断言，条件为假，返回一个错误

    return 0;
}
```

![image-20210521142002924](/home/tz/.config/Typora/typora-user-images/image-20210521142002924.png)

断言与正常错误处理：

断言主要用于检查逻辑上不可能的情况，例如，他们可以用于检查代码在开始运行前所期望的状态，或者在运行完成后检查状态，与正常的错误处理不同，断言通常在运行时被禁用。

忽略断言：

代码开头加上

```c++
#define NDEBUG;  //代码开头加上这句，则assert不可用
#include<assert.h>

int main()
{
    int x=7;
    assert(x==5);  //不可用
    return 0;
}
```

#### 13、位域



#### 14、extern

在C++中常在头文件见到extern "C"修饰函数，那有什么作用呢？ 是用于C++链接在C语言模块中定义的函数。

C++虽然兼容C，但C++文件中函数编译后生成的符号与C语言生成的不同。因为C++支持函数重载，C++函数编译后生成的符号带有函数参数类型的信息，而C则没有。

例如`int add(int a, int b)`函数经过C++编译器生成.o文件后，`add`会变成形如`add_int_int`之类的, 而C的话则会是形如`_add`, 就是说：相同的函数，在C和C++中，编译后生成的符号不同。

这就导致一个问题：如果C++中使用C语言实现的函数，在编译链接的时候，会出错，提示找不到对应的符号。此时`extern "C"`就起作用了：告诉链接器去寻找`_add`这类的C语言符号，而不是经过C++修饰的符号。

2、c++中调用c函数

C++调用C函数的例子: 引用C的头文件时，需要加`extern "C"`

```c++
//add.h
#ifndef ADD_H
#define ADD_H
int add(int x,int y);  //函数声明
#endif

//add.c
#include "add.h"

int add(int x,int y) {  //函数的定义
    return x+y;
}

//add.cpp
#include <iostream>
#include "add.h"  //调用c函数的 文件
using namespace std;
int main() {
    add(2,3);
    return 0;
}
```

没有添加extern "C" 报错：

```c++
> g++ add.cpp add.o -o main                                   
add.o：在函数‘main’中：
add.cpp:(.text+0x0): `main'被多次定义
/tmp/ccH65yQF.o:add.cpp:(.text+0x0)：第一次在此定义
/tmp/ccH65yQF.o：在函数‘main’中：
add.cpp:(.text+0xf)：对‘add(int, int)’未定义的引用
add.o：在函数‘main’中：
add.cpp:(.text+0xf)：对‘add(int, int)’未定义的引用
collect2: error: ld returned 1 exit status
```

添加extern "C"后：

```c++
#include <iostream>
using namespace std;
extern "C" {
    #include "add.h"
}
int main() {
    add(2,3);
    return 0;
}
```

编译的时候一定要注意，先通过gcc生成中间文件add.o。

```c++
gcc -c add.c 
```

然后编译：

```c++
g++ add.cpp add.o -o main
```

3、c中调用c++函数

`extern "C"`在C中是语法错误，需要放在C++头文件中。

```c++
// add.h
#ifndef ADD_H
#define ADD_H
extern "C" {
    int add(int x,int y);  //放在c中，是错误的
}
#endif

// add.cpp
#include "add.h"

int add(int x,int y) {
    return x+y;
}

// add.c
extern int add(int x,int y);
int main() {
    add(2,3);
    return 0;
}
```

#### 15、struct

1、c中struct的用法

- 在C中struct只单纯的用作数据的复合类型，也就是说，在结构体声明中只能将数据成员放在里面，而不能将函数放在里面。
- 在C结构体声明中不能使用C++访问修饰符，如：public、protected、private 而在C++中可以使用。
- 在C中定义结构体变量，如果使用了下面定义必须加struct。
- C的结构体不能继承（没有这一概念）。
- 若结构体的名字与函数名相同，可以正常运行且正常的调用！例如：可以定义与 struct Base 不冲突的 void Base() {}。

2、c++中struct函数

与C对比如下：

- C++结构体中不仅可以定义数据，还可以定义函数。
- C++结构体中可以使用访问修饰符，如：public、protected、private 。
- C++结构体使用可以直接使用不带struct。
- C++继承
- 若结构体的名字与函数名相同，可以正常运行且正常的调用！但是定义结构体变量时候只能用带struct的！



情形1：不适用typedef定义结构体别名

未添加同名函数前：

```c++
struct Student {

};
Student(){}
Struct Student s; //ok
Student s;  //ok
```

添加同名函数后：

```c++
struct Student {

};
Student(){}
Struct Student s; //ok
Student s;  //error
```

情形二：使用typedef定义结构体别名

```
typedef struct Base1 {         
    int v1;
//    private:   //error!
        int v3;
    public:     //显示声明public
        int v2;
    void print(){       
        printf("%s\n","hello world");
    };    
}B;
//void B() {}  //error! 符号 "B" 已经被定义为一个 "struct Base1" 的别名
```

![image-20210521153901454](/home/tz/.config/Typora/typora-user-images/image-20210521153901454.png)

struct与class区别：

最本质的一个区别就是默认的访问控制

默认的继承访问权限。struct 是 public 的，class 是 private 的。

struct 作为数据结构的实现体，它默认的数据访问控制是 public 的，而 class 作为对象的实现体，它默认的成员变量访问控制是 private 的。

#### 16、union

联合（union）是一种节省空间的特殊的类，一个 union 可以有多个数据成员，但是在任意时刻只有一个数据成员可以有值。当某个成员被赋值后其他成员变为未定义状态。联合有如下特点：

- 默认访问控制符为 public
- 可以含有构造函数、析构函数
- 不能含有引用类型的成员
- 不能继承自其他类，不能作为基类
- 不能含有虚函数
- 匿名 union 在定义所在作用域可直接访问 union 成员
- 匿名 union 不能包含 protected 成员或 private 成员
- 全局匿名联合必须是静态（static）的

当多个基本数据类型或者复合数据结构要占用同一片内存的时候，我们要使用联合体;当多种数据类型、多个对象、多个事物只取一时，我们也可以使用联合体

```c++
#include<iostream>
using namespace std;

union test
{
    struct
    {
        int x;
        int y;
        int z;
    }s;

    int k;
}myUnion;

int main()
{
    myUnion.s.x=4;
    myUnion.s.y=5;
    myUnion.s.z=6;

    myUnion.k=0;

    cout << myUnion.s.x << endl;
    cout << myUnion.s.y << endl;
    cout << myUnion.s.z << endl;

    cout << myUnion.k << endl;

    return 0;
}
```

![image-20210521161939182](/home/tz/.config/Typora/typora-user-images/image-20210521161939182.png)

分析：上述的代码因为union是共享内存的，以size最大的结构作为自己的大小，每个数据成员在内存中的地址是相同的，所以myUnion大小等于struct的大小，内存中先将x赋值为3,y赋值为4,z赋值为5,然后对k赋值的时候，因为内存公共享，所以会从结构体的初始位置开始，将x替换为0.

```c++
#include<iostream>
using namespace std;

union test
{
    struct
    {
        int x;
        int y;
        int z;
    }s;

    int k;
    int l;
}myUnion;

int main()
{
    myUnion.s.x=4;
    myUnion.s.y=5;
    myUnion.s.z=6;

    myUnion.k=0;
    myUnion.l=9;

    cout << myUnion.s.x << endl;
    cout << myUnion.s.y << endl;
    cout << myUnion.s.z << endl;

    cout << myUnion.k << endl;
    cout<<myUnion.l << endl;

    return 0;
}

```

![image-20210521164409171](/home/tz/.config/Typora/typora-user-images/image-20210521164409171.png)

上述的代码可以看出，k，l和z的值都是一样的，看出联合体的共享

#### 17、c实现c++多态

C++中的多态:在C++中会维护一张虚函数表，根据赋值兼容规则，我们知道父类的指针或者引用是可以指向子类对象的。

如果一个父类的指针或者引用调用父类的虚函数则该父类的指针会在自己的虚函数表中查找自己的函数地址，如果该父类对象的指针或者引用指向的是子类的对象，而且该子类已经重写了父类的虚函数，则该指针会调用子类的已经重写的虚函数。

#### 18、友元

概述：友元提供了一种普通函数或者类成员函数访问另一个类中的私有或者保护成员的机制，也就是说有两种形式的友元

1、友元函数：普通函数对一个访问某个类中的私有或保护成员

2、友元类：类A中的成员函数访问类B中的私有成员函数或保护成员

优点：提高程序运行的效率

缺点：破坏了类的封装性和数据的透明性

###### 1、友元函数

在类的声明的任何区域中声明，而定义在类的外部

注意，友元函数只是一个普通函数，并不是该类的类成员函数，它可以在任何地方调用，友元函数中通过对象名来访问该类的私有或保护成员。

```c++
#include<iostream>
using namespace std;

class A
{
    public:
    A(int _a):a(_a){};   //构造函数
    friend int geta(A &ca);  //友元

    private:
    int a;
};

int geta(A &ca)
{
    return ca.a;  //通过对象调用
}



int main()
{
    A a(3);
    cout << geta(a) << endl;  //3

    return 0;
}
```

###### 2、友元类

友元类的声明在该类的声明中，而实现在该类外

类B是类A的友元，那么类B可以直接访问A的私有成员。

```c++
#include <iostream>

using namespace std;

class A
{
public:
    A(int _a):a(_a){};
    friend class B;
private:
    int a;
};

class B
{
public:
    int getb(A ca) {
        return  ca.a;   //直接访问类A的私有成员
    };
};

int main() 
{
    A a(3);
    B b;
    cout<<b.getb(a)<<endl;
    return 0;
}
```

###### 3、注意

- 友元关系没有继承性 假如类B是类A的友元，类C继承于类A，那么友元类B是没办法直接访问类C的私有或保护成员。
- 友元关系没有传递性 假如类B是类A的友元，类C是类B的友元，那么友元类C是没办法直接访问类A的私有或保护成员，也就是不存在“友元的友元”这种关系。

#### 19、using相关

基本使用

局部与全局

```c++
#include <iostream>
#define isNs1 1
//#define isGlobal 2
using namespace std;
void func()   //全局函数
{
    cout<<"::func"<<endl;
}

namespace ns1 {
    void func()  //局部函数
    {
        cout<<"ns1::func"<<endl; 
    }
}

namespace ns2 {
#ifdef isNs1 
    using ns1::func;    /// ns1中的函数，调用局部函数
#elif isGlobal
    using ::func; /// 全局中的函数，调用全局函数
#else
    void func() 
    {
        cout<<"other::func"<<endl; 
    }
#endif
}

int main() 
{
    /**
     * 这就是为什么在c++中使用了cmath而不是math.h头文件
     */
    ns2::func(); // 会根据当前环境定义宏的不同来调用不同命名空间下的func()函数
    return 0;
}
```

###### 2、改变访问性

```c++
class Base{
public:
 std::size_t size() const { return n;  }  //公有的属性
protected:
 std::size_t n;  //受保护的权限
};
class Derived : private Base {  //私有继承
public:
 using Base::size;    //仍然可以访问私有权限
protected:
 using Base::n;  //子类的受保护权限
};
```

类Derived私有继承了Base，对于它来说成员变量n和成员函数size都是私有的，如果使用了using语句，可以改变他们的可访问性，如上述例子中，size可以按public的权限访问，n可以按protected的权限访问。

###### 3、函数重载

在继承过程中，派生类可以覆盖重载函数的0个或多个实例，一旦定义了一个重载版本，那么其他的重载版本都会变为不可见。

如果对于基类的重载函数，我们需要在派生类中修改一个，又要让其他的保持可见，必须要重载所有版本，这样十分的繁琐。

```c++
#include <iostream>
using namespace std;

class Base{
    public:
        void f(){ cout<<"f()"<<endl;
        }
        void f(int n){  //基类的函数重载
            cout<<"Base::f(int)"<<endl;
        }
};

class Derived : private Base {  //私有继承
    public:
        using Base::f;  //访问父类的成员函数
        void f(int n){
            cout<<"Derived::f(int)"<<endl;  //子类重载了父类的有参函数
        }
};

int main()
{
    Base b;
    Derived d;
    d.f();  //调用using那里的f
    d.f(1);  //调用了重载的函数版本
    return 0;
}
```

![image-20210528193224937](/home/tz/.config/Typora/typora-user-images/image-20210528193224937.png)

在派生类中使用using声明语句指定一个名字而不指定形参列表，所以一条基类成员函数的using声明语句就可以把该函数的所有重载实例添加到派生类的作用域中。此时，派生类只需要定义其特有的函数就行了，而无需为继承而来的其他函数重新定义。

###### 4、取代typedef

C中常用typedef A B这样的语法，将B定义为A类型，也就是给A类型一个别名B

对应typedef A B，使用using B=A可以进行同样的操作。

```c++
typedef vector<int> V1; 
using V2 = vector<int>;
```

#### 20、全局作用符

- 全局作用域符（::name）：用于类型名称（类、类成员、成员函数、变量等）前，表示作用域为全局命名空间
- 类作用域符（class::name）：用于表示指定类型的作用域范围是具体某个类的
- 命名空间作用域符（namespace::name）:用于表示指定类型的作用域范围是具体某个命名空间的

#### 21、enum（枚举类型）

如果一个变量你需要几种可能存在的值，那么就可以被定义成为枚举类型。之所以叫枚举就是说将变量或者叫对象可能存在的情况也可以说是可能的值一一例举出来。

举例说明：如果你的笔盒里面有两支笔，但是具体是铅笔还是钢笔不清楚，所以可以用一个枚举类型来表示它

```c++
enum box{pencil,pen};
```

定义了一个枚举类型的变量叫box，这个枚举变量内含有两个元素也称枚举元素在这里是pencil和pen，分别表示铅笔和钢笔。

如果你想定义两个具有同样特性枚举类型的变量那么你可以用如下的两种方式进行定义。

方式1：

```c++
enum box{pencil,pen};

enum box box2;//或者简写成box box2;
```

方式2：

```c++
enum {pencil,pen}box,box2; //在声明的同时进行定义！
```

枚举变量中的枚举元素系统是按照常量来处理的，故叫枚举常量，他们是不能进行普通的算术赋值的，(pencil=1;)这样的写发是错误的，但是你可以在声明的时候进行赋值操作！

```c++
enum box{pencil=1,pen=2};
```

如果你不进行元素赋值操作那么元素将会被系统自动从0开始自动递增的进行赋值操作，说到自动赋值，如果你只定义了第一个那么系统将对下一个元素进行前一个元素的值加1操作，例如

```c++
enum box{pencil=3,pen};//这里pen就是4系统将自动进行pen=4的定义赋值操作！
```

###### C++11中的枚举类型

特性：

- 新的enum的作用域不在是全局的
- 不能隐式转换成其他类型

```c++
/**
 * @brief C++11的枚举类
 * 下面等价于enum class Color2:int
 */
enum class Color2
{
    RED=2,
    YELLOW,
    BLUE
};
r2 c2 = Color2::RED;
cout << static_cast<int>(c2) << endl; //必须转！
```

- 可以指定用特定的类型来存储enum

```c++
enum class Color3:char;  // 前向声明

// 定义
enum class Color3:char 
{
    RED='r',
    BLUE
};
char c3 = static_cast<char>(Color3::RED);
```

###### 类中的枚举类型

有时我们希望某些常量只在类中有效。 由于#define 定义的宏常量是全局的，不能达到目的，于是想到实用const 修饰数据成员来实现。而const 数据成员的确是存在的，但其含义却不是我们所期望的。

const 数据成员只在某个对象生存期内是常量，而对于整个类而言却是可变的，因为类可以创建多个对象，不同的对象其 const 数据成员的值可以不同。

不能在类声明中初始化 const 数据成员。以下用法是错误的，因为类的对象未被创建时，编译器不知道 SIZE 的值是什么。(c++11标准前)

```c++
class A 
{
  const int SIZE = 100;   // 错误，企图在类声明中初始化 const 数据成员 
  int array[SIZE];  // 错误，未知的 SIZE 
}; 
```

正确应该在类的构造函数的初始化列表中进行：

```c++
class A 
{
  A(int size);  // 构造函数 
  const int SIZE ;    
}; 
A::A(int size) : SIZE(size)  // 构造函数的定义
{ 

} 
A  a(100); // 对象 a 的 SIZE 值为 100 
A  b(200); // 对象 b 的 SIZE 值为 200 
```

怎样才能建立在整个类中都恒定的常量呢？

别指望 const 数据成员了，应该用类中的枚举常量来实现。例如:

```c++
class Person{
public:
    typedef enum {
        BOY = 0,
        GIRL
    }SexType;
};
//访问的时候通过，Person::BOY或者Person::GIRL来进行访问。
```

枚举常量不会占用对象的存储空间，它们在编译时被全部求值。

枚举常量的缺点是：它的隐含数据类型是整数，其最大值有限，且不能表示浮点。

#### 22、decltype

###### 1、基本使用

decltype的语法是:

```c++
decltype (expression)
```

这里的括号是必不可少的,decltype的作用是“查询表达式的类型”，因此，上面语句的效果是，返回 expression 表达式的类型。注意，decltype 仅仅“查询”表达式的类型，并不会对表达式进行“求值”。

1.1、推导表达式

```c++
int i = 4;
decltype(i) a; //推导结果为int。a的类型为int。
```

1.2、与using/typedef合用，用于定义类型。

```C++
using size_t = decltype(sizeof(0));//sizeof(a)的返回值为size_t类型
using ptrdiff_t = decltype((int*)0 - (int*)0);
using nullptr_t = decltype(nullptr);
vector<int >vec;
typedef decltype(vec.begin()) vectype;
for (vectype i = vec.begin; i != vec.end(); i++)
{
//...
}
```

1.3、重用匿名函数

在C++中，我们有时候会遇上一些匿名类型，如:

```c++
struct 
{
    int d ;
    doubel b;
}anon_s;
```

而借助decltype，我们可以重新使用这个匿名的结构体：

```c++
decltype(anon_s) as ;//定义了一个上面匿名的结构体
```

1.4 泛型编程中结合auto，用于追踪函数的返回值类型

这也是decltype最大的用途了。

```c++
template <typename T>
auto multiply(T x, T y)->decltype(x*y)
{
    return x*y;
}
```

###### 2、判别规则

对于decltype(e)而言，其判别结果受以下条件的影响：

如果e是一个没有带括号的标记符表达式或者类成员访问表达式，那么的decltype（e）就是e所命名的实体的类型。此外，如果e是一个被重载的函数，则会导致编译错误。 否则 ，假设e的类型是T，如果e是一个将亡值，那么decltype（e）为T&& 否则，假设e的类型是T，如果e是一个左值，那么decltype（e）为T&。 否则，假设e的类型是T，则decltype（e）为T。

标记符指的是除去关键字、字面量等编译器需要使用的标记之外的程序员自己定义的标记，而单个标记符对应的表达式即为标记符表达式。例如：

```c++
int arr[4]
```

则arr为一个标记符表达式，而arr[3]+0不是。

```c++
int i = 4;
int arr[5] = { 0 };
int *ptr = arr;
struct S{ double d; }s ;
void Overloaded(int);
void Overloaded(char);//重载的函数
int && RvalRef();
const bool Func(int);

//规则一：推导为其类型
decltype (arr) var1; //int 标记符表达式

decltype (ptr) var2;//int *  标记符表达式

decltype(s.d) var3;//doubel 成员访问表达式

//decltype(Overloaded) var4;//重载函数。编译错误。

//规则二：将亡值。推导为类型的右值引用。

decltype (RvalRef()) var5 = 1;

//规则三：左值，推导为类型的引用。

decltype ((i))var6 = i;     //int&

decltype (true ? i : i) var7 = i; //int&  条件表达式返回左值。

decltype (++i) var8 = i; //int&  ++i返回i的左值。

decltype(arr[5]) var9 = i;//int&. []操作返回左值

decltype(*ptr)var10 = i;//int& *操作返回左值

decltype("hello")var11 = "hello"; //const char(&)[9]  字符串字面常量为左值，且为const左值。


//规则四：以上都不是，则推导为本类型

decltype(1) var12;//const int

decltype(Func(1)) var13=true;//const bool

decltype(i++) var14 = i;//int i++返回右值
```

#### 22、引用与指针

1.引用与指针

总论：

| 引用         | 指针         |
| :----------- | :----------- |
| 必须初始化   | 可以不初始化 |
| 不能为空     | 可以为空     |
| 不能更换目标 | 可以更换目标 |

> 引用必须初始化，而指针可以不初始化。

我们在定义一个引用的时候必须为其指定一个初始值，但是指针却不需要。

```c++
int &r;    //不合法，没有初始化引用
int *p;    //合法，但p为野指针，使用需要小心
```

> 引用不能为空，而指针可以为空。

由于引用不能为空，所以我们在使用引用的时候不需要测试其合法性，而在使用指针的时候需要首先判断指针是否为空指针，否则可能会引起程序崩溃。

```c++
void test_p(int* p)
{
    if(p != null_ptr)    //对p所指对象赋值时需先判断p是否为空指针
        *p = 3;
    return;
}
void test_r(int& r)
{
    r = 3;    //由于引用不能为空，所以此处无需判断r的有效性就可以对r直接赋值
    return;
}
```

> 引用不能更换目标

指针可以随时改变指向，但是引用只能指向初始化时指向的对象，无法改变。

```c++
int a = 1;
int b = 2;

int &r = a;    //初始化引用r指向变量a
int *p = &a;   //初始化指针p指向变量a

p = &b;        //指针p指向了变量b
r = b;         //引用r依然指向a，但a的值变成了b
```

###### 2.引用

左值引用

常规引用，一般表示对象的身份。

右值引用

右值引用就是必须绑定到右值（一个临时对象、将要销毁的对象）的引用，一般表示对象的值。

右值引用可实现转移语义（Move Sementics）和精确传递（Perfect Forwarding），它的主要目的有两个方面：

- 消除两个对象交互时不必要的对象拷贝，节省运算存储资源，提高效率。
- 能够更简洁明确地定义泛型函数。

引用折叠

- `X& &`、`X& &&`、`X&& &` 可折叠成 `X&`
- `X&& &&` 可折叠成 `X&&`

C++的引用**在减少了程序员自由度的同时提升了内存操作的安全性和语义的优美性**。比如引用强制要求必须初始化，可以让我们在使用引用的时候不用再去判断引用是否为空，让代码更加简洁优美，避免了指针满天飞的情形。除了这种场景之外引用还用于如下两个场景：

> 引用型参数

一般我们使用const reference参数作为只读形参，这种情况下既可以避免参数拷贝还可以获得与传值参数一样的调用方式。

```c++
void test(const vector<int> &data)
{
    //...
}
int main()
{
    vector<int> data{1,2,3,4,5,6,7,8};
    test(data);
}
```

> 引用型返回值

C++提供了重载运算符的功能，我们在重载某些操作符的时候，使用引用型返回值可以获得跟该操作符原来语法相同的调用方式，保持了操作符语义的一致性。一个例子就是operator []操作符，这个操作符一般需要返回一个引用对象，才能正确的被修改。

```
vector<int> v(10);
v[5] = 10;    //[]操作符返回引用，然后vector对应元素才能被修改
              //如果[]操作符不返回引用而是指针的话，赋值语句则需要这样写
*v[5] = 10;   //这种书写方式，完全不符合我们对[]调用的认知，容易产生误解
```

3.指针与引用的性能差距

指针与引用之间有没有性能差距呢？这种问题就需要进入汇编层面去看一下。我们先写一个test1函数，参数传递使用指针：

```c++
void test1(int* p)
{
    *p = 3;    //此处应该首先判断p是否为空，为了测试的需要，此处我们没加。
    return;
}
```

该代码段对应的汇编代码如下：

```c++
(gdb) disassemble 
Dump of assembler code for function test1(int*):
   0x0000000000400886 <+0>:  push   %rbp
   0x0000000000400887 <+1>:  mov    %rsp,%rbp
   0x000000000040088a <+4>:  mov    %rdi,-0x8(%rbp)
=> 0x000000000040088e <+8>:  mov    -0x8(%rbp),%rax
   0x0000000000400892 <+12>: movl   $0x3,(%rax)
   0x0000000000400898 <+18>: nop
   0x0000000000400899 <+19>: pop    %rbp
   0x000000000040089a <+20>: retq   
End of assembler dump.
```

上述代码1、2行是参数调用保存现场操作；第3行是参数传递，函数调用第一个参数一般放在rdi寄存器，此行代码把rdi寄存器值（指针p的值）写入栈中；第4行是把栈中p的值写入rax寄存器；第5行是把立即数3写入到**rax寄存器值所指向的内存**中，此处要注意(%rax)两边的括号，这个括号并并不是可有可无的，(%rax)和%rax完全是两种意义，(%rax)代表rax寄存器中值所代表地址部分的内存，即相当于C++代码中的*p，而%rax代表rax寄存器，相当于C++代码中的p值，所以汇编这里使用了(%rax)而不是%rax。

我们再写出参数传递使用引用的C++代码段test2：

```c++
void test2(int& r)
{
    r = 3;    //赋值前无需判断reference是否为空
    return;
}
```

这段代码对应的汇编代码如下：

```c++
(gdb) disassemble 
Dump of assembler code for function test2(int&):
   0x000000000040089b <+0>:  push   %rbp
   0x000000000040089c <+1>:  mov    %rsp,%rbp
   0x000000000040089f <+4>:  mov    %rdi,-0x8(%rbp)
=> 0x00000000004008a3 <+8>:  mov    -0x8(%rbp),%rax
   0x00000000004008a7 <+12>: movl   $0x3,(%rax)
   0x00000000004008ad <+18>: nop
   0x00000000004008ae <+19>: pop    %rbp
   0x00000000004008af <+20>: retq   
End of assembler dump.
```

我们发现test2对应的汇编代码和test1对应的汇编代码完全相同，这说明C++编译器在编译程序的时候将指针和引用编译成了完全一样的机器码。所以C++中的引用只是C++对指针操作的一个“语法糖”，在底层实现时C++编译器实现这两种操作的方法完全相同。

3.总结

C++中引入了引用操作，在对引用的使用加了更多限制条件的情况下，保证了引用使用的安全性和便捷性，还可以保持代码的优雅性。在适合的情况使用适合的操作，引用的使用可以一定程度避免“指针满天飞”的情况，对于提升程序稳定性也有一定的积极意义。最后，指针与引用底层实现都是一样的，不用担心两者的性能差距。

