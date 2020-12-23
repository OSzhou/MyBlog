---
banner: /css/images/tree_22.jpg
title: Swfit-04.类,元类,错误处理,泛型,拓展。。。
permalink: {{ title }}
date: 2020-01-15 10:31:30
tags: Swift
thumbnail: /css/images/tree_22.jpg
categories:
- iOS开发
---

- ##### 31. X.self、X.Type、AnyClass
   - X.self(对应OC中的类对象)是一个元类型(metadata)的指针，metadata存放着类型相关信息 
   - X.self属于X.Type类型(对应OC中的元类对象) 

<!--more-->

```
class Person {}
class Student : Person {}
var perType: Person.Type = Person.self 
var stuType: Student.Type = Student.self 
perType = Student.self

var anyType: AnyObject.Type = Person.self 
anyType = Student.self

public typealias AnyClass = AnyObject.Type 
var anyType2: AnyClass = Person.self 
anyType2 = Student.self

var per = Person()
var perType = type(of: per) // Person.self 
print(Person.self == type(of: per)) // true

  
//元类型的应用
class Animal { required init() {} } 
class Cat : Animal {}
class Dog : Animal {}
class Pig : Animal {}
func create(_ clses: [Animal.Type]) -> [Animal] { 
	var arr = [Animal]()
	for cls in clses {
		arr.append(cls.init()) 
	}
	return arr 
}
print(create([Cat.self, Dog.self, Pig.self]))
```
   - Swift还有个隐藏的基类:Swift._SwiftObject [源码连接](https://github.com/apple/swift/blob/master/stdlib/public/runtime/SwiftObject.h)
- ##### 32.大写Self
```
Self
// Self代表当前类型
class Person {
    var age = 1
    static var count = 2
    func run() {
		print(self.age) // 1
		print(Self.count) // 2 等价于Person.count
	}
}
// Self一般用作返回值类型，限定返回值跟方法调用者必须是同一类型(也可以作为参数类型)
protocol Runnable { 
  	func test() -> Self
}
class Person : Runnable {
    required init() {}
	func test() -> Self { 
		type(of: self).init() 
	} 
}
class Student : Person {}
var p = Person() 

// Person 
print(p.test())

var stu = Student() 
// Student 
print(stu.test())
```
- ##### 33.错误处理
    - 自定义错误:
```
// Swift中可以通过Error协议自定义运行时的错误信息
enum SomeError : Error {
	case illegalArg(String) 
	case outOfBounds(Int, Int) 
	case outOfMemory
}
// 函数内部通过throw抛出自定义Error，可能会抛出Error的函数必须加上throws声明
 func divide(_ num1: Int, _ num2: Int) throws -> Int { 
 	if num2 == 0 {
		throw SomeError.illegalArg("0不能作为除数")
	}
    return num1 / num2
}
// 需要使用try调用可能会抛出Error的函数
var result = try divide(20, 10)
```
 - 处理Error的3种方式：
>1 通过do-catch捕捉Error
2 不捕捉Error，在当前函数增加throws声明，Error将自动抛给上层函数， 如果最顶层函数(main函数)依然没有捕捉Error，那么程序将终止
3 可以使用`try?、try!`调用可能会抛出Error的函数，这样就不用去处理Error
- 抛出Error后，try下一句直到作用域结束的代码都将停止运行

- `rethrows`
rethrows表明:函数本身不会抛出错误，但调用***闭包参数抛出错误***，那么它会将错误向上抛
```
func exec(_ fn: (Int, Int) throws -> Int, _ num1: Int, _ num2: Int) rethrows { 
 	print(try fn(num1, num2))
}
// Fatal error: Error raised at top level
try exec(divide, 20, 0)
```
- `defer`
1. defer语句:用来定义以任何方式(抛错误、return等)离开代码块前必须要执行的代码 
 2. defer语句将延迟至当前作用域结束之前执行
```
// defer语句的执行顺序与定义顺序相反
func test() {
    defer { fn1() }
    defer { fn2() }
}
test()
// fn2
// fn1
```
- `assert`(断言)
1. 很多编程语言都有断言机制:不符合指定条件就抛出运行时错误，常用于调试(Debug)阶段的条件判断 
2. 默认情况下，Swift的断言只会在Debug模式下生效，Release模式下会忽略

 >增加Swift Flags修改断言的默认行为 
 p-assert-config Release:强制关闭断言 
 p-assert-config Debug:强制开启断言
- `fatalError`
1. 如果遇到严重问题，希望结束程序运行时，可以直接使用fatalError函数抛出错误(`这是无法通过do-catch捕捉的错误`) 
2. 使用了fatalError函数，就不需要再写return
3. 在某些不得不实现、但不希望别人调用的方法，可以考虑内部使用fatalError函数
- ##### 34.泛型
   - 泛型可以将类型参数化，提高代码复用率，减少代码量 
   ```
   // 泛型函数赋值给变量
   func swapValues<T>(_ a: inout T, _ b: inout T) { (a, b) = (b, a) }
   ```
   - 关联类型(Associated Type)
 关联类型的作用:给协议中用到的类型定义一个占位名称。`就是协议中泛型的实现` 
 协议中可以拥有多个关联类型
   ```
   	protocol Stackable {
		associatedtype Element // 关联类型 
		mutating func push(_ element: Element) 
		mutating func pop() -> Element
		func top() -> Element
		func size() -> Int
	}
	class StringStack : Stackable {
		// 给关联类型设定真实类型
		// typealias Element = String
		var elements = [String]()
		func push(_ element: String) { elements.append(element) } 
		func pop() -> String { elements.removeLast() }
		func top() -> String { elements.last! }
		func size() -> Int { elements.count } 
	}
	var ss = StringStack() 
	ss.push("Jack") 
	ss.push("Rose")
   ```
   - `where`类型约束
   ```
  protocol Stackable {
	associatedtype Element: Equatable
  }
   class Stack<E : Equatable> : Stackable { typealias Element = E }
   
   func equal<S1: Stackable, S2: Stackable>(_ s1: S1, _ s2: S2) -> Bool 
    where S1.Element == S2.Element, S1.Element : Hashable {
	return false
   }
  ```
   - 协议类型的注意点
   - `some` 不透明类型应用
   ```
    泛型解决
    // 解决方案1:使用泛型
	func get<T : Runnable>(_ type: Int) -> T { 
		if type == 0 {
	        return Person() as! T
	    }
	    return Car() as! T
	}
	var r1: Person = get(0)
	var r2: Car = get(1)

	不透明类型(Opaque Type): 屏蔽内部实现细节
	// 解决方案2:使用some关键字声明一个不透明类型
    // some限制只能返回一种类型
 	func get(_ type: Int) -> some Runnable { 
 		Car() 
 	} 
 	var r1 = get(0)
	var r2 = get(1)
       
	// some
	// some除了用在返回值类型上，一般还可以用在属性类型上
	protocol Runnable { associatedtype Speed }
	class Dog : Runnable { typealias Speed = Double }
	class Person {
	    var pet: some Runnable {
	        return Dog()
		} 
	}
   ```
   - 可选项的本质是`enum`类型
   ```
	public enum Optional<Wrapped> : ExpressibleByNilLiteral { 
		case none // nil
	    case some(Wrapped) // 关联值
		public init(_ some: Wrapped) 
	}

	var age: Int? = .none 
	age = 10
	age = .some(20)
	age = nil

	var age: Int? = 10
	var age0: Optional<Int> = Optional<Int>.some(10)
	var age1: Optional = .some(10)
	var age2 = Optional.some(10)
	var age3 = Optional(10)

	age = nil
	age3 = .none

	// 两种判断nil的写法

	switch age {
		// 默认会解包
		case let v?:
	    	print("some", v)
		case nil:
	    	print("none")
	}

	switch age {
		case let .some(v):
		print("some", v) 

		case .none:
	    print("none")
	}
   ```
- ##### 35. 关于String的本质探索
```
  // 字符串长度 <= 0xF(15)，字符串内容直接存放在str1变量的内存中。（类似OC中的tagger pointer 技术）
 var str1 = "0123456789"
 
 // 字符串长度 > 0xF，字符串内容存放在__TEXT.cstring中（常量区）
 // 字符串的地址值信息存放在str2变量的后8个字节中
 var str2 = "0123456789ABCDEF"
 
 // 由于字符串长度 <= 0xF，所以字符串内容依然存放在str1变量的内存中
 str1.append("ABCDE")
 // 开辟堆空间
 str1.append("F")
 
 // 开辟堆空间
 str2.append("G")
```
- ##### 36.call，jmp指令区别
- call会把他的下一条指令的地址压入堆栈，然后跳转到他调用的开始处，同时ret会自动弹出返回地址。 
- JMP只是简单的跳转
call的本质相当于push+jmp  ret的本质相当于pop+jmp

- ##### 37.高级运算符
   -  溢出运算符(Overflow Operator)：
1. Swift的算数运算符出现溢出时会抛出运行时错误
2. Swift有溢出运算符`(&+、&-、&*)`，用来支持溢出运算
   ```
	var min = UInt8.min
	print(min &- 1) // 255, Int8.max
	var max = UInt8.max
	print(max &+ 1) // 0, Int8.min 
	print(max &* 2) // 254, 等价于 max &+ max
	// 逐渐递增
	UInt8.min 0
	UInt8.max 255
   ```
   - 运算符重载(Operator Overload):
       - 类、结构体、枚举可以为现有的运算符提供自定义的实现，这个操作叫做:运算符重载
   ```
       struct Point : Equatable {
        var x = 0, y = 0
        static func + (p1: Point, p2: Point) -> Point {
            Point(x: p1.x + p2.x, y: p1.y + p2.y)
        }
        static func - (p1: Point, p2: Point) -> Point {
            Point(x: p1.x - p2.x, y: p1.y - p2.y)
        }
        static prefix func - (p: Point) -> Point {
            Point(x: -p.x, y: -p.y)
        }
        static func += (p1: inout Point, p2: Point) {
            p1 = p1 + p2
        }
        static prefix func ++ (p: inout Point) -> Point {
            p += Point(x: 1, y: 1)
            return p
        }
        static postfix func ++ (p: inout Point) -> Point {
            let tmp = p
            p += Point(x: 1, y: 1)
            return tmp
        }
    }
   ```
   - Equatable:
      - 要想得知2个实例是否等价，一般做法是遵守Equatable 协议，重载== 运算符 
      - 与此同时，等价于重载了 != 运算符
      - Swift为以下类型提供默认的Equatable 实现 
      > 1. 没有关联类型的枚举
      > 2. 只拥有遵守 Equatable 协议关联类型的枚举 
      > 3. 只拥有遵守 Equatable 协议存储属性的结构体
      - 引用类型比较存储的地址值是否相等(是否引用着同一个对象)，使用恒等运算符=== 、!==
   -  Comparable
      -  要想比较2个实例的大小，一般做法是: 1.遵守 Comparable 协议 2.重载相应的运算符
   ```
   	 // score大的比较大，若score相等，age小的比较大 
	 struct Student : Comparable {
	    var age: Int
	    var score: Int
	    init(score: Int, age: Int) {
		self.score = score
		self.age = age 
		}
		static func < (lhs: Student, rhs: Student) -> Bool { 
			(lhs.score < rhs.score)
			|| (lhs.score == rhs.score && lhs.age > rhs.age)
		}
		static func > (lhs: Student, rhs: Student) -> Bool {
			(lhs.score > rhs.score)
			|| (lhs.score == rhs.score && lhs.age < rhs.age)
		}
		static func <= (lhs: Student, rhs: Student) -> Bool {
			!(lhs > rhs) 
		}
		static func >= (lhs: Student, rhs: Student) -> Bool { 
			!(lhs < rhs)
		} 
	}

	var stu1 = Student(score: 100, age: 20) 
	var stu2 = Student(score: 98, age: 18) 
	var stu3 = Student(score: 100, age: 20) 
	print(stu1 > stu2) // true
	print(stu1 >= stu2) // true
	print(stu1 >= stu3) // true
	print(stu1 <= stu3) // true
	print(stu2 < stu1) // true
	print(stu2 <= stu1) // true

   ```
   - 自定义运算符(Custom Operator)
      - 可以自定义新的运算符:在全局作用域使用operator进行声明
   ```
   	prefix operator 前缀运算符
	postfix operator 后缀运算符
	infix operator 中缀运算符 : 优先级组
	precedencegroup 优先级组 {
		associativity: 结合性(left\right\none)
		higherThan: 比谁的优先级高
		lowerThan: 比谁的优先级低
		assignment: true代表在可选链操作中拥有跟赋值运算符一样的优先级
	}

	prefix operator +++
	infix operator +- : PlusMinusPrecedence 
	precedencegroup PlusMinusPrecedence {
		associativity: none
		higherThan: AdditionPrecedence 
		lowerThan: MultiplicationPrecedence 
		assignment: true
	}
   ```
   [Apple文档参考1](:https://developer.apple.com/documentation/swift/swift_standard_library/operator_declarations)
   [Apple文档参考2]( https://docs.swift.org/swift-book/ReferenceManual/Declarations.html#ID380)
   - 示例：
   ```
   	struct Point {
		var x: Int, y: Int
		static prefix func +++ (point: inout Point) -> Point {
			point = Point(x: point.x + point.x, y: point.y + point.y)
	        return point
	    }
		static func +- (left: Point, right: Point) -> Point { 
			return Point(x: left.x + right.x, y: left.y - right.y)
		}
		static func +- (left: Point?, right: Point) -> Point {
			print("+-")
			return Point(x: left?.x ?? 0 + right.x, y: left?.y ?? 0 - right.y) 
		}
	}
	struct Person {
	    var point: Point
	}
	var person: Person? = nil 
	person?.point +- Point(x: 10, y: 20)
   ```
- ##### 38. 扩展(Extension)
   - Swift中的扩展，有点类似于OC中的分类(Category) 
   - 扩展可以为枚举、结构体、类、协议添加新功能
   - 可以添加方法、计算属性、下标、(便捷)初始化器、嵌套类型、协议等等

   - 扩展不能办到的事情:
   > 不能覆盖原有的功能 
   > 不能添加存储属性，不能向已有的属性添加属性观察器 
   > 不能添加父类 
   > 不能添加指定初始化器，不能添加反初始化器
   > ...
   - 协议拓展
 扩展可以给协议提供默认实现，也间接实现『可选协议』的效果 
 扩展可以给协议扩充『协议中从未声明过的方法』
    - 注意下面代码的打印信息：
   ```
    protocol TestProtocol {
        func test1()
    }
   	extension TestProtocol {
		func test1() { 
			print("TestProtocol test1")
	    }
	    func test2() {
			print("TestProtocol test2")
        }
    }
	class TestClass : TestProtocol {
		func test1() { 
			print("TestClass test1") 
		} 
		func test2() { 
			print("TestClass test2") 
		}
	}
	var cls = TestClass()
	cls.test1() // TestClass test1 
	cls.test2() // TestClass test2

	var cls2: TestProtocol = TestClass() 
	cls2.test1() // TestClass test1 
	cls2.test2() // TestProtocol test2
   ```  
   - 泛型拓展：
      ```
     	// 符合条件才扩展(只有Stack中的元素遵守Equatable协议，拓展才有效)
	 extension Stack : Equatable where E : Equatable {
		static func == (left: Stack, right: Stack) -> Bool { 
			left.elements == right.elements
		} 
	 }
     ```
- ##### 39.访问控制(Access Control)
   - 在访问权限控制这块，Swift提供了5个不同的访问级别(以下是从高到低排列， 实体指***被访问级别修饰的内容***)  
> open:允许在定义实体的模块、其他模块中访问，允许其他模块进行继承、重写(open只能用在类、类成员上)  
> public:允许在定义实体的模块、其他模块中访问，不允许其他模块进行继承、重写 
> internal:只允许在定义实体的模块中访问，不允许在其他模块中访问  
> fileprivate:只允许在定义实体的源文件中访问
> private:只允许在定义实体的封闭声明中访问 
`绝大部分实体默认都是internal 级别`
   - 访问级别的使用准则:`一个实体不可以被更低访问级别的实体定义`
>1.元祖，泛型都以最低那个为准
>2.协议实现不得小于协议和类中的较小的那个
>3.枚举不能每项单独定义
>4.public 限定作用域内部默认是internal
>5.required初始化器 ≥ 它的默认访问级别

- `直接在全局作用域下定义的private等价于fileprivate`
- ##### 40.内存管理
- 跟OC一样，Swift也是采取基于引用计数的ARC内存管理方案(针对堆空间) 
- Swift的ARC中有3种引用:
   - 1.强引用(`strong reference`):默认情况下，引用都是强引用

   - 2.弱引用(`weak reference`):通过weak定义弱引用
     - 必须是可选类型的var，因为实例销毁后，***ARC会自动将弱引用设置为nil*** 
     - ARC自动给弱引用设置nil时，不会触发属性观察器
   - 3.无主引用(`unowned reference`):通过unowned定义无主引用
     - ***不会产生强引用，实例销毁后仍然存储着实例的内存地址***(类似于OC中的unsafe_unretained)
     - 试图在实例销毁后访问无主引用，会产生运行时错误(野指针)
• Fatal error: Attempted to read an unowned reference but object 0x0 was already deallocated
   - 用`inout`输入输出参数时，注意内存访问冲突问题
   - 内存访问冲突(Conflicting Access to Memory)
 >内存访问冲突会在两个访问满足下列条件时发生: 
 至少一个是写入操作
 它们访问的是同一块内存
 它们的访问时间重叠(比如在同一个函数内)
```
	// 不存在内存访问冲突
	func plus(_ num: inout Int) -> Int { num + 1 } var number = 1
	number = plus(&number)

	// 存在内存访问冲突
	// Simultaneous accesses to 0x0, but modification requires exclusive access var step = 1
	func increment(_ num: inout Int) { num += step }
	increment(&step)

	// 解决内存访问冲突
	var copyOfStep = step 
	increment(&copyOfStep) 
	step = copyOfStep
```

***以上均根据CoderMJLee老师的视频及资料整理，祝小码哥教育越来越好，特此感谢！！！***

如有侵权，请联系~~删除~~：fm939071955@163.com