### 枚举

原始值：

原始值rawValue的本质是：只读计算属性



关联值：

- 1个字节存储成员值
- N个字节存储关联值（N取占用内存最大的关联值），任何一个case的关联值都共用这N个字节



### 可选项 (Optional)

可选项，一般也叫可选类型，它允许将值置为nil

在类型名称后面加个问号？来定义一个可选项



### 强制解包 （Forced Unwrapping）

可选项是对其他类型的一层包装，可以将它理解为一个盒子

- 如果为nil，那么它是个空盒子
- 如果不为nil，那么盒子里装的是: 被包装类型的数据

如果要从可选项中取出被包装的数据（将盒子里装的东西取出来），需要使用感叹号！强制解包

如果对值为nil的可选项（空盒子）进行强制解包，将会产生运行时错误

#### 可选项绑定（Optional Binding）

可以使用可选项绑定来判断可选项是否包含值

如果包含就自动解包，把值赋给一个临时的常量（let）或者变量（var），并返回true，否则返回false



### 空合并运算符 ?? (Nil Coalescing Operator)



### guard语句

```swift
guard 条件 else {
  // do something...
  退出当前作用域
  // return、break、continue、throw error
}	
```

- 当guard语句的条件为false时，就会执行大括号里边的代码

- 当guard语句的条件为true时，就会跳过guard语句
- guard语句特别适合用来做 提前退出

当使用guard语句进行可选项绑定时，绑定的常量（let）、变量（var）也能在外层作用域中使用

```swift
func login(_ info: [String: String]) {
  guard let username = info["username"] else {
    print("请输入用户名")
    return
  }
  guard let password = info["password"] else {
    print("请输入密码")
    return
  }
  
  print("用户名: \(username)", "密码: \(password)", "登录中...")
}
```



### 隐式解包

在某些情况下，可选项一旦被设定值之后，就会一直拥有值

在这种情况下，可以去掉检查，也不必每次访问的时候都进行解包，因为它能确定每次访问的时候都有值

可以在类型后面加个感叹号! 定义一个隐式解包的可选项



### 多重可选项

```swift
var num1: Int? = 10
var num2: Int?? = num1
var num3: Int?? = 10
```

![](./swift_img/swift_1.png)

``` swift
var num1: Int? = nil
var num2: Int?? = num1
var num3: Int?? = nil
```



### 结构体

在Swift标准库中，绝大多数的公开类型都是结构体，而枚举和类只占很小一部分

比如Bool、Int、Double、String、Array、Dictionary等常见类型都是结构体

```swift
struct Date {
  var year: Int
  var month: Int
  var day: Int
}
var date = Date(year: 2019, month:6, day:23)
```

所有的结构体都有一个编译器自动生成的初始化器（initializer，初始化方法、构造器、构造方法）

在最后一行调用的，可以传入所有成员值，用以初始化所有成员（存储属性，Stored Property）

编译器会根据情况，可能会为结构体生成多个初始化器，宗旨是：保证所有成员都有初始值



一旦在定义结构体时自定义了初始化器，编译器就不会再帮它自动生成其他初始化器

``` swift
struct Point {
  var x: Int = 0
  var y: Int = 0
  
  init(x: Int, y: Int) {
    self.x = x
    self.y = y
  }
}
```



 结构体内存结构

``` swift
struct Point {
  var x: Int = 0 // 8
  var y: Int = 0 // 8
  var origin: Bool = false // 1
}
print(MemoryLayout<Point>.size) // 17
print(MemoryLayout<Point>.stride) // 24
print(MemoryLayout<Point>.alignment) // 8
```



### 类

类的定义和结构体类似，但编译器并没有为类自动生成可以传入成员值得初始化器

``` swift
class Point {
    var x: Int = 0
    var y: Int = 0
}
let p1 = Point()  						// ✔️
let p2 = Point(x:10, y:20) 		// ❌
```

如果类的所有成员都在定义的时候指定了初始值，编译器会为类生成无参的初始化器

成员的初始化是在这个初始化器中完成的

``` swift
class Point {
    var x: Int = 10
    var y: Int = 10
}

class Point {
    var x: Int
    var y: Int
    init() {
        x = 10
        y = 10
    }
}
// 以上两段代码等效
```



结构体与类的本质区别

- 结构体是值类型（枚举也是值类型），类是引用类型（指针类型）

``` swift
class Size {
    var width = 1
    var height = 2
}

struct Point {
    var x = 3
    var y = 4
}

var size = Size()
var point = Point()
```

![](./swift_img/swift_2.png)



### 值类型

值类型赋值给var、let或者给函数传参，是直接将所有内容拷贝一份

类似于对文件进行copy、paste操作，产生了全新的文件副本。属于深拷贝（deep copy）

在Swift标准库中，为了提升性能，String、Array、Dictionary、Set采取了Copy On Write的技术

比如仅当有 写 操作时，才会真正执行拷贝操作

对于标准库值类型的赋值操作，Swift能确保最佳性能，所以没必要为了保证最佳性能来避免赋值。 不需要修改的，尽量定义为let



### 引用类型

引用赋值给var、let或者给函数传参，是将内存地址拷贝一份

类似于制作一个文件的替身（快捷方式、链接），指向的是同一个文件，属于浅拷贝（shallow copy）



### 闭包表达式（Closure Expression）

在swift中，可以通过func定义一个函数，也可以通过闭包表达式定义一个函数

```swift
func sun(_ v1: Int, _ v2: Int) -> Int {
	v1 + v2 
}
```

``` swift
var fn = {
    (v1: Int, v2: Int) -> Int in
    return v1 + v2
}
print(fn(10, 20))

{
  (参数列表)-> 返回值类型 in
  函数体代码
}
```

如果将一个很长的闭包表达式作为函数的最后一个实参，使用尾随闭包可以增强函数的可读性

尾随闭包是一个被书写在函数调用括号外面（后面）的闭包表达式

``` swift
func exec(v1: Int, v2: Int, fn:(Int, Int) -> Int) {
  print(fn(v1, v2))
}

exec(v1: 10, v2: 20) {
  $0 + $1
}
```



### 闭包（Closure）

一个函数和它所捕获的 变量/常量 环境组合起来，称为闭包

- 一般指定义在函数内部的函数
- 一般它捕获的是外层函数的局部变量/常量

 ``` swift
typealias Fn = (Int) -> Int

func getFn() -> Fn {
    var num = 0
    func plus(_ i: Int) -> Int {
        num += i
        return num
    }
    return plus
} // 返回的plus和num形成了闭包

var fn1 = getFn()
print(fn1(1))
print(fn1(2))

// 一般涉及到外部变量，会生成堆空间，并把num拷贝到堆空间
 ```

可以把闭包想象成是一个类的实例对象

- 内存在堆空间
- 捕获的局部变量/常量就是对象的成员（存储属性）
- 组成闭包的函数就是类内部定义的方法

``` swift
class Closure {
    var num = 0
    func plus(_ i: Int) -> Int {
        num += i
        return num
    }
}

var cs1 = Closure()
var cs2 = Closure()
cs1.plus(1) // 1
cs1.plus(3) // 4
cs2.plus(2) // 2
cs2.plus(4) // 6
```



**自动闭包**

@autoclosure 会自动将20封装成闭包 { 20 }

@autoclosure 只支持 () -> T 格式的参数

@autoclosure 并非只支持最后一个参数

空合并运算符 ?? 使用了 @autoclosure技术

有@autoclosure、无@autoclosure 构成了函数重载



### 属性

swift中跟实例相关的属性可以分为2大类

- 存储属性（Stored ）
  - 类似于成员变量这个概念
  - 存在实例的内存中
  - 结构体、类可以定义存储属性
  - 枚举不可以定义存储属性
- 计算属性（Computed Property）
  - 本质就是方法(函数)
  - 不占用实例的内存
  - 枚举、结构体、类 都可以定义计算属性

``` swift
struct Circle {
    // 存储属性
    var radius: Double
    // 计算属性
    var diameter: Double {
        set {
            radius = newValue / 2
        }
        get {
            radius * 2
        }
    }
}

var circle = Circle(radius: 5)
print(circle.radius)    // 5.0
print(circle.diameter)  // 10.0

circle.diameter = 12
print(circle.radius)    // 6.0
print(circle.diameter)  // 12.0
```



#### 存储属性

在创建类 或 结构体的实例时，必须为所有的存储属性设置一个合适的初始值

- 可以在初始化器里为存储属性设置一个初始值
- 可以分配一个默认的属性值作为属性定义的一部分

``` swift
struct Circle {
    // 存储属性
    var radius: Double = 5.0
}

struct Circle {
    // 存储属性
    var radius: Double
    init() {
        radius = 5.0
    }
}
```



**延迟存储属性（Lazy Stored Property）**

使用lazy可以定义一个延迟存储属性，在第一次用到属性的时候才会进行初始化

``` swift
class Person {
  lazy var car = Car()
}
```

lazy属性必须是var，不能是let

当结构体包含一个延迟存储属性时，只有var才能访问延迟存储属性



#### 计算属性

set传入的新值默认叫做newValue，也可以自定义

只读计算属性：只有get，没有set

``` swift
struct Circle {
    var radius: Double
    var diameter: Double {
        get {
            radius * 2
        }
    }
}

struct Circle {
    var radius: Double
    var diameter: Double {
        radius * 2
    }
}
```

定义计算属性只能用var，不能用let

- let代表常量，值是一成不变的
- 计算属性的值是可能发生变化的（即使是只读计算属性）



#### 属性观察器(Property Observer)

可以为非lazy的var存储属性设置属性观察器

``` swift
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

willSet 会传递新值，默认叫newValue

didSet 会传递旧值，默认叫oldValue

在初始化器或属性定义时设置属性值不会触发willSet和didSet

在属性定义时设置初始值也不会触发willSet和didSet



属性观察器、计算属性的功能，同样可以应用在全局变量、局部变量上

``` swift
var num: Int {
  get {
    return 10
  }
  set {
    print("setNum", newValue)
  }
}
num = 11 // setNum 11
print(num) // 10
```

``` swift
func test() {
	var age = 10 {
    willSet {
      print("willSet", newValue)
    }
    didSet {
      print("didSet", oldValue, age)
    }
  }
  
  // willSet 11
  // didSet 10 11
  age = 11 
}
```



###  类型属性 (Type Property)

严格来说，属性可以分为

- 实例属性（Instance Property）：只能通过实例去访问
  - 存储实例属性（Stored Instance Property）：存储在实例的内存中，每个实例都有1份
  - 计算实例属性（Computed Instance Property）
- 类型属性（Type Property）：只能通过类型去访问
  - 存储类型属性（Stored Type Property）：整个程序运行过程中，就只有1份内存（类似于全局变量）
  - 计算类型属性（Computed Type Property）

可以通过static定义类型属性

如果是类，也可以用关键字class

``` swift
struct Car {
  static var count: Int = 0
  init() {
    Car.count += 1
  }
}

let c1 = Car()
let c2 = Car()
let c3 = Car()
print(Car.count) // 3
```

不同于存储属性，你必须给存储类型属性设定初始值。因为类型没有像实例那样的init初始化器来初始化存储属性

存储类型属性默认就是lazy，会在第一次使用的时候才初始化，就算被多个线程同时访问，保证只会初始化一次。底层使用gcd的dispatch_once



**单例模式**

``` swift
public class FileManager {
    public static let shared = FileManager()
    private init() {
        
    }
}

var mgr = FileManager.shared
```



### inout

如果实参有物理内存地址，且没有设置属性观察器

- 直接将实参的内存地址传入函数（实参进行引用传递）

如果实参是计算属性或者设置了属性观察器

- 采取了copy in copy out的做法
- 调用该函数时，先复制实参的值，产生副本【get】
- 将副本的内存地址传入函数（副本进行引用传递），在函数内部可以修改副本的值
- 函数返回后，再将副本的值覆盖实参的值【set】

inout的本质就是引用传递（地址传递）



### 方法（Method）

枚举、结构体、类都可以定义实例方法、类型方法

- 实例方法（Instance Method）：通过实例对象调用
- 类型方法（Type Method）：通过类型调用，用static或者class关键字定义

``` swift
class Car {
  static var count = 0
  init () {
    Car.count = 1
  }
  static func getCount() -> Int {
    count
  }
}
let c0 = Car()
let c1 = Car()
let c2 = Car()
print(Car.getCount()) // 3
```



#### mutating

结构体和枚举是值类型，默认情况下，值类型的属性不能被自身的实例方法修改

在func关键字前加mutating可以允许这种修改行为

``` swift
struct Point {
  var x = 0, y = 0
  mutating func moveBy(deltaX: Int, deltaY: Int) {
    x += deltaX
    y += deltaY
  }
}
```

``` swift
enum StateSwitch {
  case low, middle, high
  mutating func next() {
    switch self {
      case .low:
      	self = .middle
     	case .middle:
      	self = .high
      case .high:
      	self = .low
    }
  }
}
```

#### @discardableResult

在func前面加个@discardableResult，可以消除：函数调用后返回值未被使用的警告



### 下标（subscript）

使用subscript可以给任意类型（枚举、结构体、类）增加下标功能，有些地方也翻译为：下标脚本

subscript的语法类似于 实例方法、计算属性、本质就是方法（函数）

``` swift
class Point {
  var x = 0, y = 0
  subscript(index: Int) -> Int {
    set {
      if index == 0 {
        x = newValue
      } else if index == 1 {
        y = newValue
      }
    }
    get {
      if index == 0 {
        return x
      } else if index == 1 {
        return y
      }
      return 0
    }
  }
}

var p = Point()
p[0] = 11
p[1] = 12
print(p.x) // 11
print(p.y) // 22
print(p[0]) // 11
print(p[1]) // 22
```

subscript中定义的返回值类型决定了 get方法的返回值类型 set方法中的newValue类型



### 继承（Inheritance）

值类型（枚举、结构体）不支持继承，只有类支持继承

没有父类的类，称为：基类

swift没有像oc、java那样规定：任何类最终都要继承自某个基类

子类可以重写父类的下标、方法、属性，重写必须加上override关键字

#### 重写实例方法、下标

``` swift
class Animal {
  func speak() {
    print("Animal speak")
  }
  subscript(index: Int) -> Int {
    return index
  }
}

class Cat: Animal {
  override func speak() {
    super.speak()
    print("Cat speak")
  }
  
  override subscript(index: Int) -> Int {
    return super[index] + 1
  }
}
```

#### 重写类型方法、下标

- 被class修饰的类型方法、下标，允许被子类重写
- 被static修饰的类型方法、下标，不允许被子类重写

``` swift
class Animal {
  class func speak() {
    print("Animal speak")
  }
  
  class subscript(index: Int) -> Int {
    return index
  }
}

class Cat: Animal {
  override class func speak() {
    super.speak()
    print("Cat speak")
  }
  
  override class subscript(index: Int) -> Int {
    return super[index] + 1
  }
}
```

#### 重写属性

- 子类可以将 父类的属性（存储、计算）重写为 计算属性

- 子类不可用将 父类属性 重写为 存储属性

- 只能重写var属性，不能重写let属性
- 重写时，属性名、类型要一致
- 子类重写后的属性权限，不能小于 父类属性的权限

#### final

被final修饰的方法、下标、属性，禁止被重写

被final修饰的类，禁止被继承



### 多态

从堆空间取前8个字节，获取类的类型信息，从类型信息内获取到方法地址



### 初始化 

类、结构体、枚举都可以定义初始化器

类有2种初始化器：指定初始化器（designated initializer）、便携初始化器（convenience initializer）

``` swift
// 指定初始化器
init(parameters) {
  
}

// 便携初始化器
convenience init(parameters) {
  
}
```

每个类至少有一个指定初始化器，指定初始化器是类的主要初始化器

默认初始化器总是类的指定初始化器

类偏向于少量指定初始化器，一个类通常只有一个指定初始化器



初始化器的相互调用规则

- 指定初始化器必须从它的直系父类调用指定初始化器
- 便捷初始化器必须从相同的类里调用另一个初始化器
- 便捷初始化器最终必须调用一个指定初始化器



#### 两段式初始化

- 第一阶段：初始化所有存储属性
  1. 外层调用指定、便捷初始化器
  2. 分配内存给实例，但未初始化
  3. 指定初始化器确保当前类定义的存储属性都初始化
  4. 指定初始化器调用父类的初始化器，不断向上调用，形成初始化器链
- 第二阶段：设置新的存储属性值
  1. 从顶部初始化器往下，链中的每一个指定初始化器都有机会进一步定制实例
  2. 初始化器现在能够使用self（访问、修改它的属性，调用它的实例方法等）
  3. 最终，链中任何便捷初始化器都有机会定制实例以及使用self



#### 安全检查

- 指定初始化器必须保证在调用父类初始化器之前，其所在类定义的所有存储属性都要初始化完成
- 指定初始化器必须先调用父类初始化器，然后才能为继承的属性设置新值
- 便捷初始化器必须先调用同类中的其它初始化器，然后再为任意属性设置新值
- 初始化器在第一阶段初始化完成之前，不能调用任何实例方法，不能读取任何实例属性的值，也不能引用self
- 直到第一阶段结束，实例才算完全合法



#### 重写

当重写父类的指定初始化器时，必须加上override（即使子类的实现是便捷初始化器）

如果子类写了一个匹配父类便捷初始化器的初始化器，不用加上override

因为父类的便捷初始化器永远不会通过子类直接调用，因此，严格来说，子类无法重写父类的便捷初始化器



#### 自动继承

- 如果子类没有自定义任何指定初始化器，它会自动继承父类所有的指定初始化器
- 如果子类提供了父类所有指定初始化器的实现，子类自动继承所有的父类便捷初始化器



#### required

用required修饰指定初始化器，表明其所有子类都必须实现该初始化器（通过继承或者重写实现）

如果子类重写了required初始化器，也必须加上required，不用加override

``` swift
class Person {
  required init() {}
  init(age: Int) {}
}

class Student: Person {
  required init() {
    super.init()
  }
}
```



#### 可失败初始化器

类、结构体、枚举都可以使用init？定义可失败初始化器

``` swift
class Person {
  var name: String
  init?(name: String) {
    if name.isEmpty {
      return nil
    }
    self.name = name
  }
}
```

- 不允许同时定义参数标签、参数个数、参数类型相同的可失败初始化器和非可失败初始化器
- 可以用init！定义隐式解包的可失败初始化器
- 可失败初始化器可以调用非可失败初始化器，非可失败初始化器调用可失败初始化器需要进行解包
- 如果初始化器调用一个可失败初始化器导致初始化失败，那么整个初始化过程都失败，并且之后的代码都停止执行
- 可以用一个非可失败初始化器重写一个可失败初始化器，但反过来是不行的



#### 反初始化器（deinit）

deinit叫做反初始化器，类似于C++的析构函数，OC中的dealloc方法

当类的实例对象被释放内存时，就会调用实例对象的deinit方法

```swift
class Person {
	deinit {
    
  }
}
```

deinit不接受任何参数，不能写小括号，不能自行调用

父类的deinit能被子类继承

子类的deinit实现执行完毕后会调用父类的deinit



### 可选链(Optional Chaining)

 

### 协议（Protocol）



### Error处理



### 泛型（Generics）



### 关联类型（Associated Type）







