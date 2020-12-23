---
banner: /css/images/tree_20.jpg
title: Swfit-02.闭包,属性,inout,下标,继承。。。
permalink: {{ title }}
date: 2019-12-30 10:31:30
tags: Swift
thumbnail: /css/images/tree_20.jpg
categories:
- iOS开发
---

- ##### 11. 闭包表达式(Closure Expression) ：`一种函数的定义方式`
    - 在Swift中，可以通过func定义一个函数，也可以通过闭包表达式定义一个函数
 ```
func sum(_ v1: Int, _ v2: Int) -> Int { v1 + v2 }

var fn = {
(v1: Int, v2: Int) -> Int in 
    return v1 + v2
}
fn(10, 20)

{
(v1: Int, v2: Int) -> Int in 
    return v1 + v2
}(10, 20)

{
(参数列表) -> 返回值类型 in 函数体代码
}
```

<!--more-->

- ##### 12. 闭包
    * 一个函数和它所捕获的变量\常量环境组合起来，称为闭包 
      1. 一般指定义在函数内部的函数
      2. 一般它捕获的是外层函数的局部变量\常量
```
typealias Fn = (Int) -> Int 
func getFn() -> Fn {
    var num = 0
    func plus(_ i: Int) -> Int {
        num += i
        return num 
    }
    return plus
} // 返回的plus和num形成了闭包
```
 辅助理解： 
 - 可以把闭包想象成是一个类的实例对象
     - 内存在堆空间
     -  捕获的局部变量\常量就是对象的成员(存储属性) 
     - 组成闭包的函数就是类内部定义的方法
```
class Closure {
    var num = 0
    func plus(_ i: Int) -> Int { num += i
      return num 
    }
}
```
- ##### 13. 自动闭包`@autoclosure`
```
// 如果第1个数大于0，返回第一个数。否则返回第2个数
func getFirstPositive(_ v1: Int, _ v2: Int) -> Int {
    return v1 > 0 ? v1 : v2
}
getFirstPositive(10, 20) // 10 
getFirstPositive(-2, 20) // 20 
getFirstPositive(0, -4) // -4
// 改成函数类型的参数，可以让v2延迟加载
func getFirstPositive(_ v1: Int, _ v2: () -> Int) -> Int? {
    return v1 > 0 ? v1 : v2() 
}
getFirstPositive(-4) { 20 }
func getFirstPositive(_ v1: Int, _ v2: @autoclosure () -> Int) -> Int? {
     return v1 > 0 ? v1 : v2()
}
getFirstPositive(-4, 20)
```
- @autoclosure 会自动将 20 封装成闭包 { 20 }
- @autoclosure 只支持 () -> T 格式的参数 
- @autoclosure 并非只支持最后1个参数
- 空合并运算符 ?? 使用了 @autoclosure 技术
- 有@autoclosure、无@autoclosure，构成了函数重载
` 为了避免与期望冲突，使用了@autoclosure的地方最好明确注释清楚:这个值会被推迟执行`
 - ##### 14. 属性
   - ######存储属性(Stored Property)
      - 类似于成员变量这个概念
      - 存储在实例的内存中
      - 结构体、类可以定义存储属性 
      - 枚举***不可以***定义存储属性
      - 在创建类 或 结构体的实例时，必须为所有的存储属性设置一个合适的初始值
         - 可以在初始化器里为存储属性设置一个初始值
         - 可以分配一个默认的属性值作为属性定义的一部分
      - **延迟存储属性(Lazy Stored Property):**
         - 使用lazy可以定义一个延迟存储属性，在第一次用到属性的时候才会进行初始化
         - lazy属性必须是var，不能是let(存储类型属性除外)
         - let必须在实例的初始化方法完成之前就拥有值
         - 如果多条线程同时第一次访问lazy属性 `无法保证属性只被初始化1次`
         - 当结构体包含一个延迟存储属性时，只有var才能访问延迟存储属性, 因为延迟属性初始化时需要改变结构体的内存
   - ######计算属性(Computed Property)
      - ***本质就是方法(函数)***
      - 不占用实例的内存
      - 枚举、结构体、类都可以定义计算属性
      - set传入的新值默认叫做newValue，也可以自定义
      - 只读计算属性:只有get，没有set(有set，必须有get)
      - 定义计算属性只能用var，不能用let。let代表常量:值是一成不变的 ，而计算属性的值是可能发生变化的(即使是只读计算属性)
      - `枚举原始值rawValue的本质是:只读计算属性`
    - ######类型属性(Type Property):
      - 属性可以分为:
        - 实例属性(Instance Property):只能通过实例去访问
           - 存储实例属性(Stored Instance Property):存储在实例的内存中，每个实例都有1份 
           - 计算实例属性(Computed Instance Property)
        - 类型属性(Type Property):只能通过类型去访问
           - 存储类型属性(Stored Type Property):整个程序运行过程中，就只有1份内存(类似于全局变量) 
           - 计算类型属性(Computed Type Property)
           -  可以通过static定义类型属性 
           -  如果是类，也可以用关键字class

        - ps:类型属性细节
           - 不同于存储实例属性，你必须给存储类型属性设定初始值。 因为类型没有像实例那样的init初始化器来初始化存储属性
           - 存储类型属性默认就是lazy，会在第一次使用的时候才初始化。 就算被多个线程同时访问，`保证只会初始化一次`。 存储类型属性可以是let
           - 枚举类型也可以定义类型属性(存储类型属性、计算类型属性)
```
// 单例模式
public class FileManager {
    public static let shared = FileManager()
    private init() { }
}
public class FileManager {
    public static let shared = {
       // ....
       // ....
       return FileManager()
     }()
    private init() { }
}
```

- ##### 15. 属性观察器(Property Observer)
   - 可以为非lazy的var存储属性设置属性观察器
   - willSet会传递新值，默认叫newValu
   -  didSet会传递旧值，默认叫oldValue
   - ***在初始化器中设置属性值不会触发willSet和didSet*** 
   - ***在属性定义时设置初始值也不会触发willSet和didSet***
   - ***父类的属性在它自己的初始化器中赋值不会触发属性观察器，但在子类的初始化器中赋值会触发属性观察器***
   - 属性观察器、计算属性的功能，同样可以应用在全局变量、局部变量身上
```
struct Circle {
  var radius: Double {
      willSet {
         print("willSet", newValue)
      } 
      didSet {
            print("didSet", oldValue, radius)  
      }
    } 
    init() {
        self.radius = 1.0
        print("Circle init!")
    }
}
```
- ##### 16. inout的本质

   - 如果实参有物理内存地址，且没有设置属性观察器 
      - 直接将实参的内存地址传入函数(实参进行引用传递)
   - 如果实参是计算属性 或者 设置了属性观察器
       - 采取了Copy In Copy Out的做法
       - 调用该函数时，先复制实参的值，产生副本【get】
       - 将副本的内存地址传入函数(副本进行引用传递)，在函数内部可以修改副本的值   
       - 函数返回后，再将副本的值覆盖实参的值【set】
  - 总结:inout的本质就是引用传递(地址传递)
- ##### 17. 方法
   - `mutating`
    结构体和枚举是值类型，默认情况下，值类型的属性不能被自身的实例方法修改。
 在func关键字前加mutating可以允许这种修改行为
   - `@discardableResult`
      在func前面加个@discardableResult，可以消除:***函数调用后返回值未被使用***的警告
⚠
- ##### 18. 下标(subscript)
   - 使用subscript可以给任意类型(枚举、结构体、类)增加下标功能，有些地方也翻译为:下标脚本 
   - subscript的语法类似于实例方法、计算属性，***本质就是方法(函数)***
   - subscript中定义的返回值类型决定了 
       get方法的返回值类型 
       set方法中newValue的类型
   - subscript可以接受多个参数，并且类型任意
   - subscript可以没有set方法，但必须要有get方法。如果只有get方法，可以省略。
- ##### 19.继承(Inheritance)
  > - ***值类型(枚举、结构体)不支持继承***，只有类支持继承。 
  > - 没有父类的类，称为:基类
  > - Swift并没有像OC、Java那样的规定:任何类最终都要继承自某个基类
  > - 子类可以重写父类的下标、方法、属性，重写***必须加上override关键字***
   - 重写类型方法、下标：
  1.被class修饰的类型方法、下标，允许被子类重写
  2.被static修饰的类型方法、下标，不允许被子类重写
   - 重写属性：
       - 子类可以将父类的属性(存储、计算)重写为`计算属性 `
       - 子类不可以将父类属性重写为存储属性
       - 只能重写var属性，不能重写let属性
       - 重写时，属性名、类型要一致
       - 子类重写后的属性权限 不能小于 父类属性的权限 
           - 如果父类属性是只读的，那么子类重写后的属性可以是只读的、也可以是可读写的 
           - 如果父类属性是可读写的，那么子类重写后的属性也必须是可读写的
   - 重写类型属性:
      - 被class修饰的计算类型属性，可以被子类重写
      - 被static修饰的类型属性(存储、计算)，不可以被子类重写
   - 属性观察器:
      - 可以在子类中为父类属性(除了只读计算属性、let属性)增加属性观察器
    - `final`:
      - 被final修饰的方法、下标、属性，禁止被重写 
      - 被final修饰的类，禁止被继承
   - 多态（继承）原理图示：
![多态](https://upload-images.jianshu.io/upload_images/2149459-0e5f84a301094e9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)