# **📘 Day 2：C# 专项练习（2025-05-18）**

## **✅ 选择题（每题 1 分，共 15 分）**

请从以下选项中选出一个最合适的答案。

1. C# 中，用于终止整个程序的语句是： D

   - A. break

   - B. continue

   - C. return

   - D. Environment.Exit()

     

2. 若一个类中的字段被声明为 static，它的意义是：C

   - A. 每个实例都有一份
   
   - B. 每次调用都会新建一份
   
   - C. 所有实例共享一份
   
   - D. 只能在构造函数中使用
   
     
3. params 关键字用于：C

   - A. 使方法必须传参数
   - B. 指定参数默认值
   - C. 接收不定数量参数
   - D. 设置方法返回值



4. 以下关于接口的说法正确的是：D

   - A. 接口可以包含字段
   - B. 接口中不能有任何方法
   - C. 接口中的方法必须是静态方法
   - D. 接口定义了一组必须实现的方法



5. 在C#中，int? 表示：C

   - A. 只读整数
   - B. 引用类型整数
   - C. 可空类型
   - D. 数组类型



6. 在 Unity 中，Update 方法的调用频率是：B

   - A. 每秒一次
   - B. 每帧一次
   - C. 每次事件触发
   - D. 每秒多次（与硬件无关）



7. 以下哪项是继承的正确声明方式？A

   - A. class A : B
   - B. class A inherits B
   - C. class A -> B
   - D. class B extends A



8. 判断以下变量的作用域属于哪一类：for (int i = 0; i < 10; i++) B

   - A. 全局作用域
   - B. 块级作用域
   - C. 类级作用域
   - D. 函数作用域



9. 用于将整数类型强制转换为浮点型的语法是：D

   - A. (float)myInt
   - B. myInt.toFloat()
   - C. Convert.ToSingle(myInt)
   - D. A 和 C 都对



10. 若有代码 List<int> nums = new List<int>();，其中 List<T> 属于：D

    - A. System.List
    - B. System.Array
    - C. System.Generic
    - D. System.Collections.Generic



11. 在 Unity 中，检测触发器进入的回调函数是：A

    - A. OnTriggerEnter
    - B. OnTriggerExit
    - C. OnCollisionEnter
    - D. OnTriggerStay



12. readonly 和 const 的主要区别是：A

    - A. readonly 运行时赋值，const 编译时
    - B. readonly 编译时赋值，const 运行时
    - C. 两者无区别
    - D. const 只能修饰字段，readonly 可以修饰方法



13. 当你想从一个类派生一个不能再被继承的类时使用：B

    - A. abstract
    - B. sealed
    - C. final
    - D. override



14. 什么关键字可以防止基类方法被子类重写？B 

    - A. static
    - B. sealed
    - C. private
    - D. readonly



15. 以下哪个 Unity 组件用于播放动画？A

    - A. Animator
    - B. Rigidbody
    - C. Collider
    - D. AudioSource
    
    







## **💻 编程题（每题 2 分，共 10 分）**

请用 C# 编写以下程序，并尽可能在 Unity 中测试运行效果。

### **编程题 1**

编写一个方法 IsPrime(int n)，判断输入的整数是否为素数，并返回布尔值。





### **编程题 2**

写一个程序，接收用户输入的一个字符串，输出其中所有出现次数最多的字符。



### **编程题 3**

创建一个 Unity 脚本，实现以下功能：当用户点击空格键时，使角色在 Y 轴上移动 1 个单位。



### **编程题 4**

写一个类 Person，包含 Name 和 Age 两个属性，并实现一个构造函数初始化这两个字段。



### **编程题 5**

在 Unity 中写一个脚本，每 3 秒输出一次当前的时间（使用 Debug.Log）。