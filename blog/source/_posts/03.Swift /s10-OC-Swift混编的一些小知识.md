---
banner: /css/images/tree_01.jpg
title: OC/Swift混编的一些小知识
permalink: {{ title }}
date: 2019-04-15 20:31:57
tags: Swift
thumbnail: /css/images/tree_01.jpg
categories:
- iOS开发
---

- ### Swift调用OC
桥接头文件:
>1.新建1个桥接头文件，文件名格式默认为:`{targetName}-Bridging-Header.h`
2.在`{targetName}-Bridging-Header.h` 文件中#import OC需要暴露给Swift的内容
(可通过编译器自动生成)

`@_silgen_name`关键字：
> 1.如果C语言暴露给Swift的函数名跟Swift中的其他函数名冲突了 
2.可以在Swift中使用 @_silgen_name 修改C函数名（可以用此方法调用系统底层函数）
```
// C语言
int sum(int a, int b) {
return a + b; }

// Swift
@_silgen_name("sum") 
func swift_sum(_ v1: Int32, _ v2: Int32) -> Int32

print(swift_sum(10, 20)) // 30
print(sum(10, 20)) // 30
```
<!--more-->
- ### OC调用Swift
>Xcode已经默认生成一个用于OC调用Swift的头文件，文件名格式是: `{targetName}-Swift.h`

- Swift暴露给OC的类最终继承自NSObject 
- 使用`@objc`修饰需要暴露给OC的成员
- 使用`@objcMembers`修饰类 
    - 代表默认所有成员都会暴露给OC(包括扩展中定义的成员) 
    - 最终是否成功暴露，还需要考虑成员自身的访问级别

`@objc`:
```
// 可以通过@objc 重命名Swift暴露给OC的符号名(类名、属性名、函数名等)
@objc(MJCar)
@objcMembers class Car: NSObject {
	var price: Double
	@objc(name)
	var band: String
	init(price: Double, band: String) {
		self.price = price
		self.band = band 
	}
	@objc(drive)
	func run() { print(price, band, "run") } 
	static func run() { print("Car run") }
}
extension Car {
    @objc(exec:v2:)
	func test() { print(price, band, "test") } 
}
```
选择器(`Selector`):
```
// Swift中依然可以使用选择器，使用#selector(name)定义一个选择器 
// 必须是被@objcMembers或@objc修饰的方法才可以定义选择器
@objcMembers class Person: NSObject {
	func test1(v1: Int) { print("test1") }
	func test2(v1: Int, v2: Int) { print("test2(v1:v2:)") }
	func test2(_ v1: Double, _ v2: Double) { print("test2(_:_:)") } 
	func run() {
		perform(#selector(test1)) 
		perform(#selector(test1(v1:))) 
		perform(#selector(test2(v1:v2:))) 
		perform(#selector(test2(_:_:))) 
		perform(#selector(test2 as (Double, Double) -> Void))
	} 
}

```
`dynamic`
 被 @objc dynamic 修饰的内容会具有动态性，比如调用方法会走runtime那一套流程
```
class Dog: NSObject {
    @objc dynamic func test1() {}
    func test2() {}
}
var d = Dog() 
d.test1()
d.test2()
```
- ### Swift语言中的一些编译器特性：
代码注释相关：
>// MARK: 类似于OC中的 #pragma mark
// MARK: - 类似于OC中的 #pragma mark - 
// TODO: 用于标记未完成的任务
// FIXME: 用于标记待修复的问题
`#warning("undo")`

条件编译：
```
// 操作系统:macOS\iOS\tvOS\watchOS\Linux\Android\Windows\FreeBSD #if os(macOS) || os(iOS)
// CPU架构:i386\x86_64\arm\arm64
#elseif arch(x86_64) || arch(arm64)
// swift版本
#elseif swift(<5) && swift(>=3)
// 模拟器
#elseif targetEnvironment(simulator) 
// 可以导入某模块
#elseif canImport(Foundation)
#else
#endif	
```
系统版本检测:
```
if #available(iOS 10, macOS 10.12, *) {
	// 对于iOS平台，只在iOS10及以上版本执行
	// 对于macOS平台，只在macOS 10.12及以上版本执行 // 最后的*表示在其他所有平台都执行
}
```
API可用性说明:
```
@available(iOS 10, macOS 10.15, *) 
class Person {}
struct Student {
    @available(*, unavailable, renamed: "study")
    func study_() {}
    func study() {}
	@available(iOS, deprecated: 11) 
	@available(macOS, deprecated: 10.12) 
	func run() {}
}
```
[更多用法参考](https://docs.swift.org/swift-book/ReferenceManual/Attributes.html)
自定义打印:
```
func logger<T>(_ msg: T,
			    file: NSString = #file,
                line: Int = #line,
                  fn: String = #function) {
    #if DEBUG
	let prefix = "\(file.lastPathComponent)_\(line)_\(fn):" 
	print(prefix, msg)
	#endif
}
```