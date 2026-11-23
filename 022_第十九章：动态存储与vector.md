# 第十九章：动态存储与vector

出门前你列了一张购物清单，只写了两样：“鸡蛋、牛奶”。刚到超市，家里来消息：“顺手带根葱。”拐进蔬菜区，又来一条：“既然都去了，再买块豆腐。”你开始怀疑这句“顺手”究竟有几只手。

普通数组定义后，格子数就定下来了。`vector`是C++标准库里的容器，元素可以一个一个往末尾加。今天用它来接住这张随时可能多一行的购物清单。

## 19.1 先搞懂：空清单怎么填第一行？

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

int main()
{
    vector<string> list;
    cout << "进超市前：" << list.size() << "样" << endl;
    list.push_back("鸡蛋");
    list.push_back("牛奶");
    list.push_back("葱");
    cout << "最新消息：" << list[2] << endl;
    cout << "现在要买：" << list.size() << "样" << endl;
    return 0;
}
```

【运行结果】：

```plain
进超市前：0样
最新消息：葱
现在要买：3样
```

`<vector>`是这个工具所在的头文件。`vector<string>`里尖括号指定每一项都是字符串。刚开始`size()`为0，`push_back("鸡蛋")`在末尾增加第一项，这时才有`list[0]`。如果一开始就给`list[0]`赋值，那是在一张还没画出第一格的表里填东西，下标已经越界了。

## 19.2 先要三个袋子，还是要装一个“3”？

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main()
{
    vector<int> bags(3);
    vector<int> order{3};
    bags[1] = 4;
    cout << "三个袋子：" << bags[0] << " " << bags[1] << " " << bags[2] << endl;
    cout << "一条订单：" << order.size() << "项，数量是" << order[0] << endl;
    return 0;
}
```

【运行结果】：

```plain
三个袋子：0 4 0
一条订单：1项，数量是3
```

`bags(3)`准备了3个`int`元素，本例里它们从0开始，所以可以直接给第二格赋值。`order{3}`列出了要装的值，清单上只有一项，内容正好是3。“发我三个袋子”和“在袋子里放一张写着3的纸条”，收银员听到后做的动作可不一样。

## 19.3 实战：家里还发来了什么？

这一回每行读一种商品，空行表示“买齐了”。名字里允许空格，所以用第十三章的`getline`。最后按接到消息的顺序念一遍清单。

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

int main()
{
    vector<string> list;
    string item;
    while (getline(cin, item) && !item.empty())
    {
        list.push_back(item);
    }

    cout << "共" << list.size() << "样。" << endl;
    for (vector<string>::size_type i = 0; i < list.size(); i++)
    {
        cout << i + 1 << ". " << list[i] << endl;
    }
    return 0;
}
```

依次输入`鸡蛋`、`ice cream`，然后空一行：

```plain
共2样。
1. 鸡蛋
2. ice cream
```

第一次读到“鸡蛋”，下标0有了内容。第二次读到整行`ice cream`，放进下标1。第三次读到空行，条件不成立，程序收笔。输出时用`i + 1`给人看序号，用`list[i]`给程序取元素。直接按回车也能收工，输出`共0样。`。读到输入结束，同样能把之前的内容列出来。

## 19.4 拓展：自己申请一排储物格

超市也提供按需借用的储物柜。把柜子借来、用完归还，是理解`new[]`和`delete[]`的一种办法。下面让用户输入要借的格子数，在第一格记下3件东西。

```cpp
#include <iostream>
using namespace std;

int main()
{
    int count;
    if (!(cin >> count) || count < 1 || count > 20)
    {
        cout << "请借1到20格。" << endl;
        return 0;
    }
    int *items = new int[count]{};
    items[0] = 3;
    cout << "第一格：" << items[0] << "件" << endl;
    delete[] items;
    items = nullptr;
    return 0;
}
```

输入`2`，输出`第一格：3件`。`new int[count]{}`在运行时准备`count`个值为0的整数，返回第一个整数的地址。`delete[] items`释放整段空间。即使`count`最后等于1，也要用和`new[]`配套的`delete[]`。归还柜子后，就不能再拿旧柜号去取东西了。前面用`vector`的例子里，容器离开作用域时会自行释放它管理的存储。

## 19.5 常见易错点提醒

- **先看长度，再取下标**。空`vector`里没有第0项，读最后一项之前也要先保证非空。
- **给已有格子填内容，用下标；需要新行，用`push_back`**。三个空袋子再追加一个，最后会有四个袋子。
- **清单变长，原来的地址可能会失效**。`vector`扩展时有时会把元素搬到一块更大的空间。如果保存过元素的指针或引用，发生这种搬家后就不能沿用旧地址。可以在追加完成后用下标重新取元素。
- **手动借来的空间只还一次**。给`items`设成`nullptr`能清空这一个指针，别的指针如果也保存过旧地址，仍然不能再拿来读写。

## 19.6 小试牛刀：动手练习

1. 在清单末尾再加“豆腐”，输出第几样是葱。想清楚“第三样”和“下标3”各指哪一项。
2. 输入三行商品，其中两行都写`牛奶`。用循环数一数家里发过几次“牛奶”，空清单也要能输出0。
3. 已知有三袋苹果，每袋分别装2、4、1个。用`vector<int>`保存数量，遍历求和。再加一袋3个，答案应从7变成10。

## 19.7 本章小结

`size()`能告诉你清单有几行，`push_back`往末尾添一行，下标负责读写已有的行。程序退出时，购物清单也随着容器离开了。下章我们把清单写到文件里，省得明天又打电话问“葱到底买没买”。
