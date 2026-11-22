# 第十九章：动态存储与vector

老师问：“这次有多少人参加活动？”你说：“等同学们报完名才知道。”如果现在就用`int scores[50];`定好位置，第51个人来了该坐哪儿？如果先画上500个格子，报名表又得翻好几页。

C++标准库有一个可以按需要调整长度的容器叫`vector`。我们先用它处理报名名单，再认识运行时申请空间的另一种写法。

## 19.1 先搞懂：vector和普通数组用起来有什么区别？

`vector<int> scores;`定义了一个存整数的容器。尖括号里的`int`说明每个元素是什么类型。刚定义时它可以一个元素也没有，每调用一次`push_back`，就往末尾加一个值。它仍然支持从0开始的下标，写起来和数组很像。

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main()
{
    vector<int> scores;
    cout << "开始有" << scores.size() << "项。" << endl;
    scores.push_back(88);
    scores.push_back(92);

    cout << "现在有" << scores.size() << "项。" << endl;
    cout << "第一项：" << scores[0] << endl;
    return 0;
}
```

【运行结果】：

```plain
开始有0项。
现在有2项。
第一项：88
```

`push_back(88)`办了两件事：加入88，让元素个数从0变成1。要是开头直接写`scores[0] = 88;`，第0格还不存在，就已经越界了。

## 19.2 已经知道长度时，先准备好位置

如果程序先读到报名人数，可以用`vector<int> scores(count);`一次准备`count`个整数，初始值都是0。既然位置已经存在，就能按下标逐个填进去。我们给练习规定人数为1到100，每份成绩为0到100。

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main()
{
    int count;
    if (!(cin >> count) || count < 1 || count > 100)
    {
        cout << "人数应为1到100之间的整数。" << endl;
        return 0;
    }

    vector<int> scores(count);  // 先放入count个0
    int total = 0;
    for (int i = 0; i < count; i++)
    {
        if (!(cin >> scores[i]) || scores[i] < 0 || scores[i] > 100)
        {
            cout << "成绩应为0到100之间的整数。" << endl;
            return 0;
        }
        total += scores[i];
    }

    cout << "平均分：" << static_cast<double>(total) / count << endl;
    return 0;
}
```

输入`3 70 80 90`，输出`平均分：80`。第一次循环把70放到第0格，总分变成70。第二次到第1格拿到80，总分变成150。第三次写完第2格，总分240，再除以3。

你猜`vector<int> a(3);`和`vector<int> b{3};`有什么区别？前者准备3个0，后者只放1个元素，值为3。圆括号表示准备几个，花括号在这个例子里列出要放进去的值。区别只有一对括号，格子数已经差了两个。

## 19.3 实战：不知道有几项，就边读边加

现在改一下要求：输入若干个分数，用`-1`表示录入结束。每读到一个合法分数，就`push_back`；最后按录入顺序回放。结束标记负责通知程序收工，不进成绩名单。

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main()
{
    vector<int> scores;
    int score;
    while (cin >> score && score != -1)
    {
        if (score < 0 || score > 100)
        {
            cout << "跳过无效分数：" << score << endl;
            continue;
        }
        scores.push_back(score);
    }

    cout << "共录入" << scores.size() << "项。" << endl;
    for (vector<int>::size_type i = 0; i < scores.size(); i++)
    {
        cout << "第" << i + 1 << "项：" << scores[i] << endl;
    }
    return 0;
}
```

输入`86 120 95 -1`，输出如下：

```plain
跳过无效分数：120
共录入2项。
第1项：86
第2项：95
```

输入结束，或遇到不能读取的文字，`cin >> score`也会让循环停下来；这个小程序会回放之前已成功录入的分数。直接输入`-1`时，`scores.size()`为0，下面的循环一次也不执行。

## 19.4 拓展：认识new[]和delete[]

有时你会在旧代码里看到`new int[count]{}`。它会在运行时申请一段空间，里面有`count`个初始化为0的整数，并给你首元素的地址。谁负责申请，谁就要安排释放。`new[]`申请的空间用`delete[]`释放。

```cpp
#include <iostream>
using namespace std;

int main()
{
    int count;
    if (!(cin >> count) || count < 1 || count > 100)
    {
        cout << "请输入1到100之间的整数。" << endl;
        return 0;
    }

    int *scores = new int[count]{};
    scores[0] = 86;
    cout << "第一项：" << scores[0] << endl;

    delete[] scores;
    scores = nullptr;  // 这个指针现在没有可用的目标
    return 0;
}
```

输入`3`，输出`第一项：86`。申请后可以像数组一样通过下标访问，范围是0到`count - 1`。`delete[]`后这段空间就不能再访问。若还有别的指针指向那里，把`scores`设为`nullptr`也不会替它们清空地址。手动管理时，多一条提前返回的路，就得多想一次有没有释放。上面的成绩程序用`vector`来管理存储，离开作用域时容器会自己收拾。

## 19.5 常见易错点提醒

- **`size()`表示已经有多少个元素**。要访问第0项，先确定容器不空。下标仍然要求小于`size()`。
- **需要追加时才用`push_back`**。已用`vector<int> scores(3);`准备好三个位置时，填值用下标。再调用一次`push_back`会多出第四个元素。
- **增加元素后重新取位置**。`vector`有时需要把数据搬到一块更大的空间，之前取出的元素地址、引用就可能失效。刚开始写程序时，添加后重新按下标访问就好。
- **手动申请的数组要成对释放**。`new[]`对应`delete[]`。释放后别继续读写，也别重复释放。

## 19.6 小试牛刀：动手练习

1. **练习1**：定义一个空的`vector<int>`，依次加入`4、6、9`，再输出元素个数和所有元素。
2. **练习2**：修改录入分数的程序，添加最高分。直接输入`-1`时提示“没有可统计的分数”；用`0 70 70 -1`测试重复最高分。
3. **练习3**：写一个操作`vector<string>`的程序，循环用`getline`收集报名姓名，读到空行结束。用一个带空格的姓名测试它有没有被读完整。

## 19.7 本章小结

`vector`能在运行时决定长度，也能在末尾增加元素，适合人数未定的名单。普通下标规则依然有效。学习`new[]`和`delete[]`能帮助我们读懂手动管理的代码，日常练习可以先请`vector`管理这部分工作。

不过，程序一关，容器里的成绩也就没了。下一章我们把成绩写进文件，明天开电脑还能接着看。

