### **📘 C# 专项练习：数据类型与类型转换（Day 1）**

#### **🕒 建议完成时间：45～60 分钟**

#### **🎯 今日重点：**

- 值类型 vs 引用类型
- 类型转换（隐式 / 显式 / Convert / Parse）
- 装箱与拆箱
- 字符串与数值的转换
- 数组基础（声明与初始化）

## **✅ 一、选择题（每题 1 分，共 15 分）**

**1.** 以下哪个是 C# 中的值类型？

A. string

B. object

C. int

D. dynamic

<font color="red"> 答：C </font>

**2.** 下列关于隐式类型转换的说法正确的是：

A. double 可以自动转换为 int

B. long 可以自动转换为 int

C. int 可以自动转换为 double

D. string 可以自动转换为 int

<font color="red">答：C  （int → double 是隐式转换；string 不能隐式转换为 int）</font>

**3.** 关于装箱（Boxing）说法正确的是：

A. 将值类型赋值给 object 是装箱

B. 装箱过程是编译期行为

C. 装箱不会创建新对象

D. string 装箱为 object 会抛出异常

<font color="red">答：A (装箱是值类型 → object，过程创建堆对象)</font>

**4.** 关于拆箱（Unboxing）下列语句中正确的是：

A. int i = (int)"123";

B. int i = (int)obj; （obj 为 object 类型，存储一个 int）

C. int i = Convert.ToInt32(obj); （obj = “123”）

D. B 和 C 正确

<font color="red"> 答：D </font>

**5.** 以下关于 Convert.ToInt32("abc") 的执行结果是：

A. 返回 0

B. 抛出 FormatException

C. 返回 null

D. 返回 -1

<font color="red">答：B</font>

**6.** 下列哪个类型不属于 C# 的数值类型？

A. decimal

B. float

C. char

D. double

<font color="red"> 答：C </font>

**7.** 以下代码的输出结果为：

```
int x = 5;
double y = x;
Console.WriteLine(y);
```

A. 编译错误

B. 5

C. 5.0

D. 抛出异常

<font color="red"> 答：C </font>

**8.** 以下声明哪个数组是合法的：

A. int[] a = new int[5];

B. int a[] = int[5];

C. int a = new int[5];

D. array<int> a = new array<int>(5);

<font color="red"> 答：A </font>

**9.** 下列关于 Parse 与 TryParse 区别描述错误的是：

A. Parse 出错会抛异常

B. TryParse 返回 true/false

C. TryParse 不会修改参数

D. TryParse 更适合用于不确定字符串合法性的情况

<font color="red">答：C </font>

**10.** 以下关于 C# 中 string 类型的说法正确的是：

A. string 是值类型

B. string 是引用类型但不可变

C. string 是可变类型

D. string 是 struct 类型

<font color="red"> 答：B (string是引用类型，但不可变) </font>

**11.** bool b = Convert.ToBoolean("false"); 结果为？

A. true

B. false

C. 抛出异常

D. null

<font color="red"> 答：B </font>

**12.** 对于 object obj = 123; 执行 (short)obj 是否安全？

A. 安全

B. 编译错误

C. 运行时抛出 InvalidCastException

D. 会返回 123 的 short 类型

<font color="red"> 答：C </font>

**13.** C# 中 byte 类型的最大值是？

A. 127

B. 256

C. 255

D. 2^16

<font color="red"> 答：C </font>

**14.** C# 中数组的下标默认从哪开始？

A. -1

B. 0

C. 1

D. 自定义起始值

<font color="red"> 答：B </font>

**15.** 以下哪种方法不能将字符串 “123” 转为 int 类型？

A. int.Parse(“123”)

B. Convert.ToInt32(“123”)

C. (int)“123”

D. int.TryParse(“123”, out int x)

<font color="red"> 答：C </font>

------





## **💻 二、编程题（共 5 题）**





**1. 类型转换练习**

请编写一个方法，接收 string input，如果是合法整数，返回其平方；否则返回 -1。

```
int GetSquareOrMinusOne(string input)
```

**2. 装箱与拆箱练习**

请写一个示例展示：



- 将 int 类型变量装箱为 object
- 再将 object 拆箱为 int 并打印





------



**3. 字符串与数值转换器**

写一个函数，接收一个 double 值，返回它保留两位小数的字符串。例如输入 3.1415，返回 "3.14"。



------



**4. 多维数组初始化并打印**

定义一个 2 行 3 列的 int 二维数组，并将其内容设置为如下：

```
1 2 3
4 5 6
```

要求使用嵌套 for 循环打印每个元素。



------



**5. Convert 工具类封装**

写一个静态类 MyConverter，封装一个方法 ToIntOrDefault(string input, int defaultValue)，如果字符串无法转换为整数，则返回默认值。