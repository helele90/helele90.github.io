# Swift特性

## 基础部分

#### typealias
类型别名(typealias)用于给现有类型定义另一个名字。
```swift
typealias AudioSample = UInt16
var maxAmplitudeFound = AudioSample.min 
// maxAmplitudeFound == Unit16.min
```

#### 元组
元组把多个值组合成一个复合值。
```swift
let http404Error = (404, "Not Found")
print(http404Error.$0) // 打印404
// 给单个元素命名
let http200Status = (statusCode: 200, description: "OK")
print(http200Status.statusCode) // 打印200
```
> 提示：不适合用于复杂数据类型

## 基本运算符

#### 区间运算符
```swift
// 闭区间
for index in 1...5 {
  // 打印 1,2,3,4,5
}
// 半开区间
for index in 1..<5 {
  // 打印 1,2,3,4
}
// 单侧区间
let nums = [1, 2, 3, 4, 5]
for index in nums[...2] {
  // 打印 1,2,3
}
```

## 可选值
```swift
enum Optional<Wrapped> {
    case none
    case some(Wrapped)
}

var a: Int? // 可选值
var b: Int! // 隐式可选值
var c: Int = 1
if let a = a {
	print(a)
}
print(a) // error
print(a!) // 强解包
print(b)
a = nil // error
c = nil // error
```

#### ??
`(a ?? b)`将对可选类型 a 进行空判断，如果 a 包含一个值就进行解包，否则就返回一个默认值 b。
```swift
let a: Int?
let b = a ?? 1 // 当a = nil， b = 1
```

#### 强解包
应该尽可能避免使用强解包，当没有值的时，会触发运行时错误。

#### 隐式可选值
- 为了避免每次都要判断和解析可选值
- 使用时会强制解包
- 如果你在隐式解析可选类型没有值的时候尝试取值，会触发运行时错误。
###### 使用场景
 - 当可选类型被第一次赋值之后就可以确定之后一直有值的时候。
> 比如`@IBOutlet`属性，初始化以后一定有值
 - 如果一个变量之后可能变成 nil 的话请不要使用隐式解析可选类型。如果你需要在变量的生命周期中判断是否是 nil 的话，请使用普通可选类型。

#### 可选链
```swift
var testScores = ["Dave": [86, 82, 84], "Bev": [79, 94, 81]]
testScores["Dave"]?[0] = 91

if let johnsStreet = john.residence?.address?.street {
    print("John's street name is \(johnsStreet).")
}
```
- 当遇到nil值时，会自动中断停止执行
> 可选值的价值：可选值可以让开发者编写更安全的代码。他最大的价值是区分出`nil`和`非nil`. 只要代码编译通过，我们编写的代码就是安全的。不需要做大量的非空对象的判断。

## 字符串
```swift
let string = ""
```

#### 切片

## 集合
`Swift`标准库中的集合类型`Array/Dictionary/Set`都是`值类型`.
> 提示：使用`let`创建的集合，编译器会优化创建集合性能

#### map/filter/reduce
使用`map/filter/reduce `方法可以减少模板代码，代码条理更加清晰。
```swift
var nums: [Int] = [1, 2, 3]
var result = ""
nums.forEach { (num) in
    if num > 1 {
        result += String(num)
    }
}

let result4 = nums.filter { (num) -> Bool in
    return num > 1
}.map { (num) -> String in
    return String(num)
}.reduce("") { (result, text) -> String in
    return result + text
}

let result2 = nums.filter { $0 > 1 }.map { String($0) }.reduce("", { $0 + $1 })
```

##### lazy
- 不会创建保存中间结果的数组。提升运行速度，减少内存消耗
- 数组的实际创建推迟到使用前的最后一刻
```swift
let result = nums.lazy.filter { $0 > 1 }.map { String($0) }.reduce("", { $0 + $1 })
```
> 提示：使用filter的场景，优先执行filter，可以减少循环次数

## 控制流

#### for循环
```swift
// 遍历值
let names = ["Anna", "Alex", "Brian", "Jack"]
for name in names {
    print("Hello, \(name)!")// 打印 Anna
}

// 遍历索引和值
for (index, name) in names.enumerated() {
    print("Hello, \(index)-\(name)!") // 打印 0-Anna
}
```

#### stride
```swift
let minuteInterval = 5
for tickMark in stride(from: 0, to: minutes, by: minuteInterval) {
    // 打印 0, 5, 10, 15 ... 45, 50, 55
}
```

#### Switch
```swift
let someCharacter: Character = "z"
switch someCharacter {
case "a":
    print("The first letter of the alphabet")
case "z":
    print("The last letter of the alphabet")
default:
    print("Some other character")
}
// 输出“The last letter of the alphabet”
```
- 不需要使用`break`
- 当`case`包含所有匹配时不能使用`default`

#### Guard
使用`guard`语句来要求条件必须为真时，以执行 `guard` 语句后的代码。不同于` if` 语句，一个 `guard` 语句总是有一个 `else` 从句，如果条件不为真则执行` else` 从句中的代码。
```swift
func greet(person: [String: String]) {
    guard let name = person["name"] else {
        return
    }

    print("Hello \(name)!")

    guard let location = person["location"] else {
        print("I hope the weather is nice near you.")
        return
    }

    print("I hope the weather is nice in \(location).")
}

greet(person: ["name": "John"])
// 输出“Hello John!”
// 输出“I hope the weather is nice near you.”
greet(person: ["name": "Jane", "location": "Cupertino"])
// 输出“Hello Jane!”
// 输出“I hope the weather is nice in Cupertino.”
```
> 提示：使用`guard`语句可以提升代码的可读性。当不满足条件时，提前退出，不会执行之后的语句。并且可以避免多层的`if`嵌套语句。

## 函数

#### 忽略参数标签
如果你不希望为某个参数添加一个标签，可以使用一个下划线（_）来代替一个明确的参数标签。
```swift
func someFunction(_ firstParameterName: Int, secondParameterName: Int) {
     // 在函数体内，firstParameterName 和 secondParameterName 代表参数中的第一个和第二个参数值
}
someFunction(1, secondParameterName: 2)
```

#### 可变参数
可变参数可以接受零个或多个值。
```swift
func arithmeticMean(_ numbers: Double...) -> Double {
    var total: Double = 0
    for number in numbers {
        total += number
    }
    return total / Double(numbers.count)
}
arithmeticMean(1, 2, 3, 4, 5)
// 返回 3.0, 是这 5 个数的平均数。
arithmeticMean(3, 8.25, 18.75)
// 返回 10.0, 是这 3 个数的平均数。
```
> 提示： 一个函数最多只能拥有一个可变参数。

#### 默认参数值
当默认值被定义后，调用这个函数时可以忽略这个参数。
```swift
func test(num: Int = 1) {
    print(num)
}
test() // 1
test(num: 2) // 2

func test(num: Int?) {

}
test()
```
> 提示：尽可能将默认参数放在函数最后
> 
#### 输入输出参数
```swift
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}

var someInt = 3
var anotherInt = 107
swapTwoInts(&someInt, &anotherInt)
print("someInt is now \(someInt), and anotherInt is now \(anotherInt)")
// 打印“someInt is now 107, and anotherInt is now 3”
```
`inout`使用拷入拷出内存模式。在函数内部使用的是参数的 copy，函数结束后，对参数重新赋值
> 函数参数使用值类型进行传递，默认不能修改
> 
#### 嵌套函数
```swift
func test() -> Int {
	func add(num: Int) -> Int {
		return num + 1
	}
	
	return add(1)
}
```
> 嵌套函数可以对外部隐藏
---- 

## 闭包
- 闭包是引用类型。
#### 值捕获
```swift
func test(block: () -> Void) {
    
}

var num = 1
test {
    num += 1
}
```
- 正常情况，闭包捕获值是捕获一个值的引用。所以闭包也会对类实例强引用。
- 为了优化，如果一个值不会被闭包改变，或者在闭包创建后不会改变，Swift 可能会改为捕获并保存一份对值的拷贝。
- Swift 也会负责被捕获变量的所有内存管理工作，包括释放不再需要的变量。
#### @escaping
当一个闭包作为参数传到一个函数中，但是这个闭包在函数返回之后才被执行，我们称该闭包从函数中逃逸。
```swift
func method(block: @escaping () -> Void) {
    DispatchQueue.main.async {
        block()
    }
}
method {
    print("")
}
```
> @escaping闭包内捕获值会强制标注self
#### @autoclosures
自动闭包是一种自动创建的闭包，用于包装传递给函数作为参数的表达式。这种闭包不接受任何参数，当它被调用的时候，会返回被包装在其中的表达式的值。
```swift
func method(block: @autoclosure () -> Int) {
    block()
}
method(block: 3)
```
- 这种便利语法让你能够省略闭包的花括号
> 提示：过度使用会让代码难以理解


## Enum

#### 关联值
每个枚举成员可以存储任意类型的关联值
```swift
enum Barcode {
    case none
    case upc(Int)
    case qrCode(String)
}

let a: Barcode = .none
let b: Barcode = .upc(1)
let c: Barcode = .qrCode("2")

let type: Barcode = .none
switch type {
case .none:
    break
case .upc(let value):
    break
case .qrCode(let code):
    break
}
```

## Struct
Swift中的结构体包含大部分类包含的功能。
```swift
struct Test {
let num: Int
}
extension Test {
	func test() {}
}
extension Test: SomeProtocol {}
```


## Class

#### ===
使用`===`来比较类是同一个实例

#### 继承
`Swift`中的类没有通用的基类

#### final
- 类不能被继承
- 方法不能被重写
> 提示：编译器可能会将final方法优化为静态派发提高性能。

#### deinit
- 当一个类的实例被释放之前，`deinit`会被立即调用。 
- 一般在`deinit`方法中进行释放资源操作。

## 下标
下标提供了设置和获取值快捷方式。
```swift
struct Test {
    private var name: String?
    
    subscript(key: String) -> String? {
        get {
            if key == "name" {
                return name
            } else {
                return nil
            }
        }
        set(newValue) {
            if key == "name" {
                name = newValue
            }
        }
    }
}

var test = Test()
test["name"] = "he"
let name = test["name"]
```


## 错误处理
#### Error
```swift
protocol Error {
}
```
 Swift中`Error`是一个空协议，需要自己实现。
```swift
enum VendingMachineError: Error {
    case invalidSelection                     //选择无效
    case insufficientFunds(coinsNeeded: Int) //金额不足
    case outOfStock                             //缺货
}
```
#### throwing
用 throws函数抛出错误
```swift
func canThrowErrors() throws -> String {
	throw outOfStock
}
```
#### do-catch
```swift
do {
    let result = try contents(ofFile: "input.txt")
    print(result)
} catch FileError.fileDoesNotExist {
    print("File not found")
} catch {
    print(error)
    // 处理其它错误。
}
```
> 提示：Swift 中的错误处理和其他语言中用 `try`，`catch` 和 `throw` 进行异常处理很像。和其他语言中（包括 Objective-C ）的异常处理不同的是，Swift 中的错误处理并不涉及解除调用栈，这是一个计算代价高昂的过程。就此而言，`throw` 语句的性能特性是可以和 `return` 语句相媲美的。
#### try
```swift
let result = try? contents(ofFile: "input.txt")
let result = try! contents(ofFile: "input.txt")
```
- `try?`语句将错误转换为可选值。如果发生异常，会返回一个可选值
- `try!`语句，如果发生异常，会产生运行时错误。尽量避免使用，除非方法内部可以抛出异常
#### Result
```swift
enum Result<Success, Failure: Error> {
    case success(Success)
    case failure(Failure)
}

enum EmptyError: Error {
    case none
}

func test(block: (Result<Int, EmptyError>) -> Void) {
    block(.success(1))
    block(.failure(.none))
}

test { (result) in
    switch result {
    case .success(let value):
        print(value)
    case .failure(let error):
        print(error)
    }
}
```
> `Result`非常适合用于类似网络请求这种包含错误场景的异步回调。而且枚举可以保证必须处理`success`和`failure`的场景。
####  assert
```swift
let age = -3
assert(age >= 0, "A person's age cannot be less than zero")
// 因为 age < 0，所以断言会触发
```
> 提示：断言仅在`Debug`环境执行，用于在开发环境检查错误。并且不会影响生产环境的性能。
####  precondition
`precondition`和`assert`功能一样，只不过`precondition`可以在Release环境执行。
```swift
preconditionFailure(true, "This url: \(value) is not invalid")
```
#### fatalError
抛出一个运行时错误
```swift
fatalError("运行时错误")
```
#### defer
Defer会在方法即将结束的最后异步执行
```swift
func processFile(filename: String) throws {
    print("1")
    defer {
    	print("3")
    }
	print("2")
}
// 打印：123
```
> defer非常适合用于方法执行结束后必须做某些清理操作，不管方法内部执行正确还是错误。

## 类型转换
#### is
用来检查一个实例是否属于特定子类型。若实例属于那个子类型，类型检查操作符返回 true，否则返回 false。
```swift
class MediaItem {}
class Movie: MediaItem {}
class Song: MediaItem {}
let library: [MediaItem] = []
for item in library {
    if item is Movie {
        print("movie")
    } else if item is Song {
        print("song")
    }
}
```
#### as
用于转换到某个子类型
```swift
for item in library {
    if let movie = item as? Movie {
        print(movie)
    } else if let song = item as? Song {
        print(song)
    }
}
```
- `as?`转换失败时会返回nil
- `as!`转换失败会触发运行时错误
#### Any/AnyObject
- `Any` 可以表示任何类型，包括函数类型。
- `AnyObject` 可以表示任何类类型的实例。

## Extension
```swift
extension SomeType: SomeProtocol, AnotherProtocol {
  // 协议所需要的实现写在这里
}
extension SomeType {
	func test() {
    }
	var m: Double {
		return 1
	}
	var c: Double // error
}
```
- 只能添加方法，不可以替换现有方法
- 扩展可以添加新的`计算属性`，但是它们不能添加`存储属性`，或向现有的属性添加属性观察者。

## Protocol
#### 属性
用 `var`关键字来声明变量属性，在类型声明后加上 `{ set get }` 来表示属性是可读可写的，可读属性则用`{ get }`来表示
```swift
protocol SomeProtocol {
	var num: Int { set get }
    var name: String { get }
}
```
#### AnyObject协议
添加`AnyObject`关键字限制协议只能被类类型实现
```swift
protocol SomeClassOnlyProtocol: AnyObject {

}
```
#### 协议合成
```swift
protocol SomeProtocol {}
protocol AnotherProtocol {}
SomeProtocol & AnotherProtocol
```
#### optional
协议可以定义可选要求，遵循协议的类型可以选择是否实现这些要求。在协议中使用` optional` 关键字作为前缀来定义可选要求。
```swift
@objc protocol CounterDataSource {
    @objc optional func increment(forCount count: Int) -> Int
    @objc optional var fixedIncrement: Int { get }
}
```
#### 提供默认实现
```swift
protocol SomeProtocol {
	func test()
}
extension SomeProtocol {
	func test() {
		
	}
}
```

## 泛型
泛型编程主要是为了代码复用，抽象出公共的代码逻辑，减少模板代码。
```swift
func swapTwoValues<T>(_ a: inout T, _ b: inout T) {
    let temporaryA = a
    a = b
    b = temporaryA
}

struct A<T> {
    var a: T
	func add(value: T) {

	}
}

let a: A<Int> = A(a: 1)
let b: A<Int> = A(a: "111") // error
let c: A<String> = A(a: "122")
```
#### 类型约束
在一个类型参数名后面放置一个类名或者协议名，并用冒号进行分隔，来定义类型约束。下面将展示泛型函数约束的基本语法（与泛型类型的语法相同）：
```swift
func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
    // 这里是泛型函数的函数体部分
}
```
上面这个函数有两个类型参数。第一个类型参数 T 必须是 SomeClass 子类；第二个类型参数 U 必须符合 SomeProtocol 协议。
#### 关联类型
关联类型为协议中的某个类型提供了一个占位符名称，其代表的实际类型在协议被遵循时才会被指定。关联类型通过 `associatedtype` 关键字来指定。
```swift
protocol IView {
    associatedtype IProps
    func bind(props: IProps)
}

struct CustomViewProps {
	let title: String
}

class CustomView: UIView, IView {

	typealias Props = CustomViewProps

	func bind(props: CustomViewProps) {
		//
	}

}
```
#### 泛型 Where 语句
```swift
protocol ReuseIdentifier: class {
    static var identifier: String { get }
}

// 实现协议的类需要是`UITableViewCell`的子类
extension ReuseIdentifier where Self: UITableViewCell {
    static var identifier: String {
        return String(describing: self)
    }
}

extension UITableView {

    final func register<T: UITableViewCell>(cellClass: T.Type) where T: ReuseIdentifier {
        register(cellClass, forCellReuseIdentifier: cellClass.identifier)
    }
    
    final func dequeueReusableCell<T: UITableViewCell>(for indexPath: IndexPath) -> T where T: ReuseIdentifier {
        guard let cell = dequeueReusableCell(withIdentifier: T.identifier, for: indexPath) as? T else {
            fatalError("错误的identifier")
        }
        
        return cell
    }

}

class TestCell: UITableViewCell, ReuseIdentifier {}
let cell: TestCell = tableView.dequeueReusableCell(for: indexPath)

class TestCell2: UITableViewCell {}
// error
let cell: TestCell2 = tableView.dequeueReusableCell(for: indexPath)
```

> 有兴趣可以看看Swift标准库`集合`相关API， 使用了大量的泛型编程。
#### 泛型和Any
通常，泛型和 Any 的用途是类似的，但它们有截然不同的表现。在没有泛型的编程语言里，Any 通常用来实现和泛型同样的效果，但是却缺少了类型安全性。这通常意味着要使用一些运行时特性，例如内省 (introspection) 或动态类型转换，把 Any 这种不确定的类型变成一个确定的具体类型。而泛型不仅能解决绝大部分同样的问题，还能带来编译期类型检查以及提高运行时性能等额外的好处。
> 总的来讲，Any需要在运行时做各种转换处理。而泛型只要编译通过，代码就是安全的，而且不需要运行时转换，性能更高。
#### 泛型特化（待更新）


## 不透明类型

## 自动引用计数
Swift 使用`自动引用计数（ARC）`机制来跟踪和管理你的应用程序的内存。
> 提示：Swift没有`AutoRelease`机制。
#### 处理循环引用
##### weak
- 当捕获的引用对象被释放时，会自动置为nil
#### unowned
- 如果对象被释放后使用，会触发运行时错误。
- 性能比weak更好，因为运行时不需要进行弱引用处理
- 如果捕获的引用永远不会为nil, 可以考虑使用unowned
> 提示：一般建议使用`weak`， 主要是`unowned`的场景判断不好会有风险。性能上的差异可以忽略。


## 访问控制
##### 访问级别
- `open` 模块外可以导入使用。只能作用于类和类的成员，它和 public 的区别主要在于 open 限定的类和成员能够在模块外能被继承和重写
- `public` 模块外可以导入使用
- `internal` 限制只能在模块内访问
- `fileprivate` 限制只能在定义的文件内部访问
- `private` 限制实体只能在其定义的作用域，以及同一文件内的 extension 访问
> `private(set)` 外部可读，内部可读写
> 提示：尽可能最小化访问控制级别是一个好的编码习惯。


## 高级运算符
##### 自定义运算符
- `默认`中缀
- `prefix` 前缀
- `postfix` 后缀
```swift

struct Model {
    var num: Int
}

extension Model {
    static func + (left: Self, right: Self) -> Self {
        return Model(num: left.num + right.num)
    }
    static prefix func -- (vector: inout Self) -> Self {
        vector.num -= 1
        return vector
    }
    static postfix func ++ (vector: inout Self) -> Self {
        vector.num += 1
        return vector
    }
}

let a = Model(num: 1)
let b = Model(num: 1)
var c = a + b
print(c) // c.num = 2
--c
print(c) // c.num = 1
c++
print(c) // c.num = 2
```


## 其他特性
### KeyPath
KeyPath提供了一种`类型安全`的`KVC/KVO`特性。
- `key`直接使用属性名，不需要使用字符串
- `value`可以根据属性推断出类型
###### KVO
```swift
// KVO
view.observe(\.tag, options: [.new]) { (object, change) in
    let newValue = change.newValue
	// newValue为Int类型
}
```
###### KVC
```swift
struct Model {
    let num: Int
    let name: String
}

struct Model2 {
    let num2: Int
    let name2: String
}

func test<Model>(name: KeyPath<Model, String>, num: KeyPath<Model, Int>, model: Model) {
	let name = model[keyPath: name]
	let num = model[keyPath: num]
	print("\(name)-\(num)")
}

test(name: \.name, num: \.num, model: Model(num: 1, name: "222"))
test(name: \.name2, num: \.num2, model: Model2(num2: 1, name2: "222"))
```

### Literals
`Literal`允许您为一些类型添加文字初始化方法
###### ExpressibleByIntegerLiteral
```swift
extension Bool: ExpressibleByIntegerLiteral {
    
    public init(integerLiteral value: Int) {
        self = value == 1 ? true : false
    }

}

let a: Bool = 1
```
###### ExpressibleByStringLiteral
```swift
extension URL: ExpressibleByStringLiteral {
    
    public init(stringLiteral value: String) {
        guard let url = URL(string: "\(value)") else {
            preconditionFailure("This url: \(value) is not invalid")
        }
        
        self = url
    }
}

let url: URL = "https://www.jd.com"
```
以下是目前支持的类型：
![](%E6%88%AA%E5%B1%8F2020-03-2210.51.33.png)
> 提示：`Literal`不能进行`nil`初始化


## 标准库
### ??
```swift
public func ?? <T>(optional: T?, defaultValue: @autoclosure () throws -> T) rethrows -> T

var num: Int?
let num2 = num ?? 1
```
###  Bool




## 动态性
### Mirror
`Mirror`提供了一个简单的反射机制用来获取某种类型的类型和属性信息。
```swift
struct Point {
    let x: Int, y: Int
}

let p = Point(x: 21, y: 30)
print(String(reflecting: p))
// Prints "▿ Point
//           - x: 21
//           - y: 30"
```
- 只能读取值，不能修改
- `debugger`和`Playground`就是使用`Mirror`来打印类型的值表示
> 提示：`Mirror`目前性能比较差，尽量避免在`Release`环境大量使用。

### @dynamicMemberLookup
```swift
@dynamicMemberLookup
struct Person {
    
    private var dic: [String: Any] = ["num": 1]
    
    subscript(dynamicMember member: String) -> Any? {
        return dic[member]
    }
    
}

let person = Person()
print(person.num)
print(person.name)

@dynamicMemberLookup
enum JSON {
    case int(Int)
    case string(String)
    case array([JSON])
    case dictionary(Dictionary<String, JSON>)
    
    subscript(dynamicMember member: String) -> JSON? {
        if case .dictionary(let dict) = self {
            return dict[member]
        }
        return nil
    }
    
}

extension JSON {
    
    var stringValue: String? {
        if case .string(let str) = self {
            return str
        }
        return nil
    }
    
    var intValue: Int? {
        if case .int(let int) = self {
            return int
        }
        return nil
    }
    
    subscript(index: Int) -> JSON? {
        if case .array(let arr) = self {
            return index < arr.count ? arr[index] : nil
        }
        return nil
    }
    
}

extension JSON {
    
    init?(_ value: Any) {
        if let value = value as? Int {
            self = .int(value)
        } else if let value = value as? String {
            self = .string(value)
        } else if let value = value as? [Any] {
            self = JSON(value)
        } else if let value = value as? [String: Any] {
            self = JSON(value)
        } else {
            return nil
        }
    }
    
    init(_ array: [Any]) {
        self = .array(array.compactMap { JSON($0) })
    }
    
    init(_ dic: [String: Any]) {
        var values: [String: JSON] = [:]
        for (key, value) in dic {
            if let a = JSON(value) {
                values[key] = a
            }
        }
        self = .dictionary(values)
    }
    
}

let value = ["num": 1, "name": "hexiao", "nums": [1, 2], "dic": ["nums": [1, 2], "dic": ["name": "hexiao"]]] as [String : Any]
var json: JSON = JSON(value)
let num = json.dic?.nums?[0]?.intValue
let name = json.dic?.dic?.name?.stringValue
print(name)
print(num)
```

### @dynamicCallable

### type(of:)
使用`type(of:)`方法获取运行时的真实类型。
```swift
func printInfo(_ value: Any) {
    let t = type(of: value)
    print("'\(value)' of type '\(t)'")
}

let count: Int = 5
printInfo(count)
// '5' of type 'Int'
```

## Void/Never
### Void
`Void`主要用于声明方法不返回任何值
```swift
// No return type declared:
func logMessage(_ s: String) {
    print("Message: \(s)")
}

let logger: (String) -> Void = logMessage
logger("This is a void function")
// Prints "Message: This is a void function"
```
标准库中的`Void`定义是一个空元组:
```swift
typealias Void = ()
```
### Never
Never用于方法

# 深入底层

# Objc兼容
## Nullability
默认情况下，Swift会将Objc中的`属性/方法参数/方法返回值`当做隐式解包类型。
- Objc
```swift
@interface MyList : NSObject
- (MyListItem *)itemWithName:(NSString *)name;
- (NSString *)nameForItem:(MyListItem *)item;
@property (copy) NSArray<MyListItem *> *allItems;
@end
```
- Swift
```swift
class MyList: NSObject {
    func item(withName name: String!) -> MyListItem!
    func name(for item: MyListItem!) -> String!
    var allItems: [MyListItem]!
}
```

可以在Objc代码中添加`Nullability`可空性申明
- Objc
```objc
NS_ASSUME_NONNULL_BEGIN

@interface MyList : NSObject
- (nullable MyListItem *)itemWithName:(NSString *)name;
- (nullable NSString *)nameForItem:(MyListItem *)item;
@property (nullable, copy) NSArray<MyListItem *> *allItems;
@end

NS_ASSUME_NONNULL_END
```
- Swift
```swift
class MyList: NSObject {
    func item(withName name: String) -> MyListItem?
    func name(for item: MyListItem) -> String?
    var allItems: [MyListItem]?
}
```
- `NS_ASSUME_NONNULL_BEGIN/NS_ASSUME_NONNULL_END`申明范围内默认都为非空，只需要添加可空类型申明。
> 提示：添加`Nullability`是一个好的编码习惯，即使不用于`Swift`。在Objc中，当给一个非空值传递`nil`时，Xcode也会进行警告。

## 轻量级泛型
为Objc代码添加轻量级泛型。
##### 集合泛型：
- Objc
```swift
@property NSArray<NSDate *> *dates;
@property NSCache<NSObject *, id<NSDiscardableContent>> *cachedData;
@property NSDictionary <NSString *, NSArray<NSLocale *> *> *supportedLocales;
```
- Swift
```swift
var dates: [Date]
var cachedData: NSCache<NSObject, NSDiscardableContent>
var supportedLocales: [String: [Locale]]
```
##### 类泛型：
- Objc：
```swift
@interface List<T: id<NSCopying>> : NSObject
- (List<T> *)listByAppendingItemsInList:(List<T> *)otherList;
@end
 
@interface ListContainer : NSObject
- (List<NSValue *> *)listOfValues;
@end

@interface ListContainer (ObjectList)
- (List *)listOfObjects;
@end
```
- Swift:
```swift
class List<T: NSCopying> : NSObject {
    func listByAppendingItemsInList(otherList: List<T>) -> List<T>
}
 
class ListContainer : NSObject {
    func listOfValues() -> List<NSValue>
}
 
extension ListContainer {
    func listOfObjects() -> List<NSCopying>
}
```

## NS_STRING_ENUM
使用`NS_STRING_ENUM `申明字符串枚举。
```objc
typedef NSString * OrderType NS_STRING_ENUM;
OrderType const OrderTypeJindong    = @"jindong";
OrderType const OrderTypePingou = @"pingou";
```

## NS_NOESCAPE
使用`NS_NOESCAPE`申明闭包为逃逸闭包。
```objc
@interface NSArray: NSObject
- (NSArray *)sortedArrayUsingComparator:(NSComparator NS_NOESCAPE)cmptr
@end
```

# Swift 6
## 异步
> 扩展： [ Swift 并发宣言 ](https://gist.github.com/yxztj/7744e97eaf8031d673338027d89eea76 "Swift 并发宣言") [ Async/Await for Swift ](https://gist.github.com/lattner/429b9070918248274f25b714dcfc7619 "Async/Await for Swift")[ All about Concurrency in Swift - Part 2: The Future ](https://www.uraimo.com/2017/07/22/all-about-concurrency-in-swift-2-the-future/ "All about Concurrency in Swift - Part 2: The Future")[https://onevcat.com/2016/12/concurrency/](https://onevcat.com/2016/12/concurrency/)
## 内存所有权
