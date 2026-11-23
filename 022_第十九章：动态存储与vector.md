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

### 19.1.1 清单有几行，和清单纸有多大，是两件事

`size()`回答“已经写了多少项”，`capacity()`回答“这次分配的空间，在需要重新分配之前最多能容纳多少项”。例如你撕下一张能写十行的纸，在第一行写了“葱”。家里现在让你买一样，不是十样。只要`size()`还是1，可以用下标访问的就只有第0项。

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

int main()
{
    vector<string> list;
    list.reserve(3);
    cout << "预留后已有" << list.size() << "项" << endl;
    cout << "至少能装3项：" << (list.capacity() >= 3 ? "能" : "不能") << endl;
    list.push_back("葱");
    cout << "现在：" << list.size() << "项，第一项是" << list[0] << endl;
    return 0;
}
```

【运行结果】：

```plain
预留后已有0项
至少能装3项：能
现在：1项，第一项是葱
```

`reserve(3)`请求预留至少三项所需的空间，没有创建任何商品，因此它后面仍不能立刻写`list[0] = "葱";`。示例刻意检查`>= 3`，不要求容量恰好等于3；扩容时分几格由实现决定。第十九章最后提到“搬家”，说的就是原先预留空间不够，需要换一块更大的存储。一般写小练习时，直接`push_back`已经能完成任务，不必先猜要预留几格。

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

### 19.2.1 清单临时多一位，已有的东西怎么办？

`resize(n)`会直接改变**元素个数**。我们拿三只水果袋做个实验：先在第2袋放4个苹果，再把袋子数缩到1，然后重新要回第2袋。你猜那4个苹果会不会也跟着回来？

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main()
{
    vector<int> bags{2, 4, 1};
    bags.resize(1);
    cout << "缩短后：" << bags.size() << "袋，第一袋有" << bags[0] << "个" << endl;
    bags.resize(2);
    cout << "再加一袋：" << bags[0] << " " << bags[1] << endl;
    bags.push_back(7);
    cout << "最后一袋：" << bags.back() << endl;
    bags.pop_back();
    cout << "拿走最后一袋，还剩" << bags.size() << "袋" << endl;
    return 0;
}
```

【运行结果】：

```plain
缩短后：1袋，第一袋有2个
再加一袋：2 0
最后一袋：7
拿走最后一袋，还剩2袋
```

按顺序画四张小清单：`[2,4,1]` → `[2]` → `[2,0]` → `[2,0,7]`。从3缩到1时，尾部两个元素已经被移除；从1扩到2时，尾部的`int`是新创建的，值从0开始。`back()`查看最后一个元素，`pop_back()`删除最后一个元素但不返回它。如果想知道拿走了什么，先读`back()`再删；两个操作都要求容器非空。

| 你想做什么 | 对应操作 | 对元素个数的影响 |
| :--- | :--- | :--- |
| 提前备纸 | `reserve(n)` | 个数不变 |
| 直接要n个袋子 | `resize(n)` | 变为n |
| 添上一项已知内容 | `push_back(value)` | 多1项 |
| 划掉最后一项 | `pop_back()` | 少1项，原来必须有东西 |

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

### 19.4.1 谁记了柜号，谁负责还钥匙？

`int *items = new int[count]{};`做了两件事：申请一段动态存储，并把首元素地址交给本地指针`items`。例子先检查`count >= 1`，正是因为接下来要访问`items[0]`。结尾写`delete[] items;`会释放那段存储，但指针变量`items`本身仍在作用域内，所以还有下一句把它设置为`nullptr`。

如果在申请成功之后、释放之前又加上一条提前`return`的分支，那条路径可能漏掉归还。这是“能写对一次”和“以后改动还容易写对”之间的区别。改用`vector<int> items(count);`时，容器的生命周期负责回收它自己的存储。写普通购物清单时，选`vector`就少操一份借钥匙的心。

## 19.5 常见易错点提醒

- **先看长度，再取下标**。空`vector`里没有第0项，读最后一项之前也要先保证非空。
- **给已有格子填内容，用下标；需要新行，用`push_back`**。三个空袋子再追加一个，最后会有四个袋子。
- **清单变长，原来的地址可能会失效**。`vector`扩展时有时会把元素搬到一块更大的空间。如果保存过元素的指针或引用，发生这种搬家后就不能沿用旧地址。可以在追加完成后用下标重新取元素。
- **手动借来的空间只还一次**。给`items`设成`nullptr`能清空这一个指针，别的指针如果也保存过旧地址，仍然不能再拿来读写。

## 19.6 小试牛刀：动手练习

1. 在清单末尾再加“豆腐”，输出第几样是葱。想清楚“第三样”和“下标3”各指哪一项。
2. 输入三行商品，其中两行都写`牛奶`。用循环数一数家里发过几次“牛奶”，空清单也要能输出0。
3. 已知有三袋苹果，每袋分别装2、4、1个。用`vector<int>`保存数量，遍历求和。再加一袋3个，答案应从7变成10。

练习2先别碰复杂输入，画三行：`牛奶`、`鸡蛋`、`牛奶`。每轮让`i`前进一步，只有`list[i] == "牛奶"`才把计数加1。预期计数依次是1、1、2。把`count = 0`错放到循环里，会在走到下一行时把之前的消息忘掉。

## 19.7 本章小结

`size()`能告诉你清单有几行，`push_back`往末尾添一行，下标负责读写已有的行。程序退出时，购物清单也随着容器离开了。下章我们把清单写到文件里，省得明天又打电话问“葱到底买没买”。
