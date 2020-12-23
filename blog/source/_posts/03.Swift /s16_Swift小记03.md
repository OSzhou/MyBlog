---
banner: /css/images/tree_21.jpg
title: Swfit-03.初始化器,可选链,协议。。。
permalink: {{ title }}
date: 2020-01-10 10:31:30
tags: Swift
thumbnail: /css/images/tree_21.jpg
categories:
- iOS开发
---

- ##### 20. 初始化器
   - 类、结构体、枚举都可以定义初始化器
   - 类有2种初始化器:指定初始化器(`纵向`)(designated initializer)、便捷初始化器(`横向`)(convenience initializer)
```
// 指定初始化器 
init(parameters) {
  statements 
}
// 便捷初始化器
convenience init(parameters) {
  statements 
}
```

<!--more-->

> 每个类至少有一个指定初始化器，指定初始化器是类的主要初始化器 
 默认初始化器总是类的指定初始化器
类偏向于少量指定初始化器，一个类通常只有一个指定初始化器

> 初始化器的相互调用规则:
指定初始化器必须从它的直系父类调用指定初始化器 
便捷初始化器必须从相同的类里调用另一个初始化器 
便捷初始化器最终必须调用一个指定初始化器
- ##### 21. 两段式初始化
   - Swift在编码安全方面是煞费苦心，为了保证初始化过程的安全，设定了两段式初始化、 安全检查
   - 两段式初始化：
       - ***第1阶段:初始化所有存储属性***:
     >1 外层调用指定\便捷初始化器
2 分配内存给实例，但未初始化
3 指定初始化器确保当前类定义的存储属性都初始化
4 指定初始化器调用父类的初始化器，不断向上调用，形成初始化器链
      
        - ***第2阶段:设置新的存储属性值***:
      >1 从顶部初始化器往下，链中的每一个指定初始化器都有机会进一步定制实例
2 初始化器现在能够使用self(访问、修改它的属性，调用它的实例方法等等) 
3 最终，链中任何便捷初始化器都有机会定制实例以及使用self
- ##### 22. 安全检查、重写、自动继承、required
    - 安全检查:
       > 1. 指定初始化器必须保证在调用父类初始化器之前，其所在类定义的所有存储属性都要初始化完成
       > 2. 指定初始化器必须先调用父类初始化器，然后才能为继承的属性设置新值
       > 3. 便捷初始化器必须先调用同类中的其它初始化器，然后再为任意属性设置新值
       > 4. 初始化器在第1阶段初始化完成之前，不能调用任何实例方法、不能读取任何实例属性的值，也不能引用self 
       > 5. `直到第1阶段结束，实例才算完全合法`
    - 重写：
    1.当重写父类的指定初始化器时，必须加上override(即使子类的实现是便捷初始化器)
    2.如果子类写了一个匹配父类便捷初始化器的初始化器，不用加上override，因为父类的便捷初始化器永远不会通过子类直接调用，因此，严格来说，`子类无法重写父类的便捷初始化`
   - 自动继承:
      > - 1 如果子类没有自定义任何指定初始化器，它会自动继承父类所有的指定初始化器 
      > - 2 如果子类提供了父类所有指定初始化器的实现(要么通过方式1继承，要么重写)
         >   - 子类自动继承所有的父类便捷初始化器
      > - 3 就算子类添加了更多的便捷初始化器，这些规则仍然适用
      > - 4 子类以便捷初始化器的形式重写父类的指定初始化器，也可以作为满足规则2的一部分
    - required:
1.用`required`修饰指定初始化器，表明其所有子类都必须实现该初始化器(通过继承或者重写实现)
2.如果子类重写了`required`初始化器，也必须加上`required`，不用加`override `

- ##### 23. 可失败初始化器
   - 类、结构体、枚举都可以使用init?定义可失败初始化器
```
class Person {
 var name: String
 init?(name: String) {
	if name.isEmpty { 
		return nil
	}
	self.name = name 
	}
}

1. 不允许同时定义参数标签、参数个数、参数类型相同的可失败初始化器和非可失败初始化器
2. 可以用init!定义隐式解包的可失败初始化器
3. 可失败初始化器可以调用非可失败初始化器，非可失败初始化器调用可失败初始化器需要进行解包
4. 如果初始化器调用一个可失败初始化器导致初始化失败，那么整个初始化过程都失败，并且之后的代码都停止执行 
5. 可以用一个非可失败初始化器重写一个可失败初始化器，但反过来是不行的
```
- ##### 24. 反初始化器(deinit)
   - deinit叫做反初始化器，类似于C++的析构函数、OC中的dealloc方法 
   - 当类的实例对象被释放内存时，就会调用实例对象的deinit方法 
```
class Person {
    deinit {
		print("Person对象销毁了") 
	}
}
// deinit不接受任何参数，不能写小括号，不能自行调用 
// 父类的deinit能被子类继承
// 子类的deinit实现执行完毕后会调用父类的deinit
```
- ##### 25. 可选链(Optional Chaining)
   >1. 如果可选项为nil，调用方法、下标、属性失败，结果为nil
   >2. 如果可选项不为nil，调用方法、下标、属性成功，结果会被包装成可选项 
   >3. 如果结果本来就是可选项，不会进行再次包装
```
// 多个?可以链接在一起
// 如果链中任何一个节点是nil，那么整个链就会调用失败
var dog = person?.dog // Dog?
var weight = person?.dog.weight // Int? 
var price = person?.car?.price // Int?

class Person {
	var name: String = ""
	var dog: Dog = Dog()
	var car: Car? = Car()
	func age() -> Int { 18 }
	func eat() { print("Person eat") } 
	subscript(index: Int) -> Int { index }
}

var person: Person? = Person()

if let _ = person?.eat() { // ()? 
	print("eat调用成功")
} else { 
	print("eat调用失败")
}
```
`person?.eat()`默认没有返回值，也可以写成`var p = person?eat()`的赋值形式。因为在Swift中函数其实默认返回`-> Void`， 而`Void = ()`，也就是空元祖。所以`p`也就是`()?`类型
- ##### 26. 协议(Protocol)
   - 协议中的属性：
      - 必须用var定义
      - 实现时权限不得小于协议中定义的权限
   - static、class：
      - 为了保证通用，协议中必须用static定义类型方法、类型属性、类型下标
   - 给枚举、结构体使用时注意加`mutating`关键字
   - init:
```
// 协议中还可以定义初始化器init 
// 非final类实现时必须加上required
protocol Drawable {
    init(x: Int, y: Int)
}
class Point : Drawable {
    required init(x: Int, y: Int) {}
}
final class Size : Drawable {
    init(x: Int, y: Int) {}
}
// 如果从协议实现的初始化器，刚好是重写了父类的指定初始化器  
// 那么这个初始化必须同时加required、override
protocol Livable {
    init(age: Int)
}
class Person {
    init(age: Int) {}
}
class Student : Person, Livable {
    required override init(age: Int) {
		super.init(age: age) 
	}
}
```
   - init、init?、init!:
      - 协议中定义的init?、init!，可以用init、init?、init!去实现 
     - 协议中定义的init，可以用init、init!去实现
- ##### 27. CaseIterable 协议
```
// 让枚举遵守CaseIterable协议，可以实现遍历枚举值
enum Season : CaseIterable {
    case spring, summer, autumn, winter
}
let seasons = Season.allCases print(seasons.count) // 4 
for season in seasons {
    print(season)
} // spring summer autumn winter
```
- ##### 28. CustomStringConvertible
   - 遵守CustomStringConvertible、 CustomDebugStringConvertible协议，都可以自定义实例的打印字符串
```
class Person : CustomStringConvertible, CustomDebugStringConvertible { 
 	var age = 0
	var description: String { "person_\(age)" }
	var debugDescription: String { "debug_person_\(age)" } 
}
var person = Person()
print(person) // person_0 
debugPrint(person) // debug_person_0
```
> print调用的是CustomStringConvertible协议的description
> debugPrint、po调用的是CustomDebugStringConvertible协议的debugDescription
- ##### 29. Any、AnyObject
   - Any:可以代表任意类型(枚举、结构体、类，也包括函数类型) 
   - AnyObject:可以代表任意类类型(在协议后面写上: AnyObject代表只有类能遵守这个协议) 
`在协议后面写上: class也代表只有类能遵守这个协议`
- ##### 30. is、as?、as!、as
   - is用来判断是否为某种类型，as用来做强制类型转换