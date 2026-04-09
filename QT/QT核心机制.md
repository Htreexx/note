### QT元对象系统

### QT信号与槽

信号与槽是用来QT对象之间通信的，它们是QT的核心特性。它底层是依靠QT元对象系统，当我们进行connect时，元对象系统会在内部维护一张连接映射表，记录发送者和接受者的对应关系。当信号真正被触发时，它会去查这张表，并根据发送者和接收者所在的线程环境决定调用策略：如果双方在同一线程，则通过函数指针直接同步调用槽函数；如果跨线程，则将该调用请求打包成一个元调用事件（MetaCallEvent），放入接收者的事件队列中等待异步处理。

- 信号

  - 在 Qt 中，信号（Signal） 是一种特殊的成员函数声明。
  - 信号函数只能在类中声明，并且总是公有的，不论是在private或protect修饰符下。
  - 信号函数只有声明，我们不能够手动定义它，它的具体实现是由Qt中的`moc`(元对象编译器)自动生成
  - 信号函数返回值总是`void`

- 槽

  槽：响应特定信号而被调用的函数。所以槽又可以称为槽函数。Qt 控件有很多预定义槽函数，槽函数也可以自己自定义。

  槽函数的特点

  - 槽是普通的 C++ 函数，只要是可调用对用都可以作为槽（例如：普通函数，成员函数，静态函数，仿函数，lambda表达式，function对象）

  - 通过信号-槽连接，任何组件都可以调用槽，不受访问级别限制
    ```C++
    MainWindow::MainWindow(QWidget *parent)
        : QMainWindow(parent)
        , ui(new Ui::MainWindow)
    {
        ui->setupUi(this);
        //槽函数为成员函数
        QPushButton *bt1 = new QPushButton(this);
        bt1->setText("close");
        connect(bt1, &QPushButton::clicked,this, &MainWindow::close);
    	//槽函数为普通函数
        QPushButton *bt2 = new QPushButton(this);
        bt2->setText("print");
        bt2->move(0,50);
        connect(bt2, &QPushButton::clicked,myPrint);
    	//槽函数为lambda表达式
        QPushButton *bt3 = new QPushButton(this);
        bt3->setText("Lambda");
        bt3->move(0,100);
        connect(bt3, &QPushButton::clicked,[](){std::cout<<"Lambda enter"<<std::endl;});
    	//槽函数为静态成员函数
        QPushButton *bt4 = new QPushButton(this);
        bt4->setText("static");
        bt4->move(0,150);
        connect(bt4, &QPushButton::clicked,&MainWindow::test);
    	//成员函数为仿函数
        QPushButton *bt5 = new QPushButton(this);
        bt5->setText("functor");
        bt5->move(0,200);
        function f;
        connect(bt5, &QPushButton::clicked, f);
    }
    ```

    

- 连接信号与槽
  `connect(发送者，信号，接收者，槽函数)`

  - 发送者是Object对象指针
  - 信号是成员函数的函数指针
  - 接收者可以有，也可以没有
  - 槽函数是任何可调用对象
  - 信号函数的参数个数<=槽函数的参数个数（并且从左到右要依次匹配）

- 连接规则
  - 可以将多个信号连接到一个槽
  - 一个信号可以连接到多个槽（当触发时，槽会按连接顺序依次执行）
  - 信号可以与信号进行连接

- 信号与槽的连接在以下几种情况下会被破坏（断开）

  1. 对象销毁（最常见）：

  这是 Qt 信号槽机制最核心的安全特性。当发送者（sender）或接收者（receiver）对象被销毁时，所有与该对象相关的连接都会被自动断开。

  2. 显式调用 `QObject::disconnect()`


​	你可以手动断开连接，`disconnect()` 函数有多种重载形式，可以实现不同粒度的断开操作

- 补充：函数存在重载时如何确定某个函数的指针

你可以先定义一个具有确切签名的函数指针变量，然后将重载函数名赋给它。编译器会根据变量的类型自动推断出你想要的是哪个版本的函数。

```C++
#include <iostream>

struct Printer {
    void print(int i) { std::cout << "int: " << i << std::endl; }
    void print(double d) { std::cout << "double: " << d << std::endl; }
};

int main() {
    // 声明一个明确类型的函数指针变量
    void (Printer::*int_print_ptr)(int) = &Printer::print;
    void (Printer::*double_print_ptr)(double) = &Printer::print;

    Printer p;
    (p.*int_print_ptr)(10);
    (p.*double_print_ptr)(3.14);
    
    return 0;
}
```

