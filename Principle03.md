# 关于 some View

## 协议

Swift 协议的一个强大之处：

* 可以作为类型约束；
* associated type，让协议可以实现一定程度的范型。


但是这两个又是是互相矛盾的。

### 不能直接用作类型约束

如果协议内部有 associated type（或者协议引用了 Self 类型，因为这样其实也是一种 associated type 行为），这个协议就不能用于类型约束了

```swift
protocol Fuel { var name:String {get} }

struct EngineOil : Fuel {
    var name: String = "机油"
}


protocol Vehicle {
    associatedtype FuelType: Fuel
    var fuel: FuelType { get }
    func run()
}

extension Vehicle {
    func run() {
        print("燃烧\(self.fuel.name)跑起来了")
    }
}

struct Audi: Vehicle {
    var fuel: EngineOil = EngineOil()
}

/// ❌报错
func create() -> Vehicle {
    return Audi()
}
```

上例会报错：`Protocol 'Vehicle' can only be used as a generic constraint because it has Self or associated type requirements`

## 无法使用 == 运算符

由于编译时丢失了类型信息，编译器无法推断类型，导致无法使用 == 运算符

比如有一个协议：

```swift
protocol MySequence {
    associatedtype Element
    var storage: [Element] { get set }
}
```

假如我们希望实现一个「比较大小」的方法，我们会怎么定义呢？

```swift
// ❌
func are(_ s1: MySequence, lessThan s2: MySequence) -> Bool {
    // 返回 true / false
    return true
}
```

这应该是大多数人的第一反应，然而这样是通不过编译的。提示的错误就是上面说的：protocol can only be used as a generic constraint because it has Self or associated type requirements.

根据这个错误信息，我们只要这样实现即可：

```swift
func are<Sequence: MySequence>(_ s1: Sequence, lessThan s2: Sequence) -> Bool {
    // 返回 true / false
    return true
}
```

想了解更多协议抽象，不妨阅读 [《Protocol Oriented: Swift 协议陷阱之 Associated Type》](https://zhuanlan.zhihu.com/p/80672557)

## Opaque Type 

这个特性使用 `some` 修饰协议返回值，具有一下特性:

* 所有的条件分支只能返回一个特定类型，不同则会编译报错
* 方法使用者依旧无法知道类型，（使用方不透明）
* 编译器知情具体类型，因此可以使用类型推断。

```swift
struct ContentView: View {
    var body: some View {
        Text("Hello World")
    }
}
```

`View` 是 `SwiftUI` 的一个最核心的协议，代表一个屏幕上元素的描述。

```swift
public protocol View {
    associatedtype Body : View

    var body: Self.Body { get }
}
```

这种带有 `associatedtype` 的协议不能作为`类型`来使用，而只能作为`类型约束`使用：

```swift
// Error
func createView() -> View {

}

// OK
func createView<T: View>() -> T {

}
```

这样一来，其实我们是不能写类似这种代码的：

```swift
// Error，含有 associatedtype 的 protocol View 只能作为类型约束使用
struct ContentView: View {
    var body: View {
        Text("Hello World")
    }
}
```

想要 `Swift` 帮助自动推断出 `View.Body` 的类型的话，我们需要明确地指出 `body` 的真正的类型。在这里，`body` 的实际类型是 `Text`：

```swift
struct ContentView: View {
    var body: Text {
        Text("Hello World")
    }
}
```

当然我们可以明确指定出 `body` 的类型，但是这带来一些麻烦：

1. 每次修改 `body` 的返回时我们都需要手动去更改相应的类型。
2. 新建一个 `View` 的时候，我们都需要去考虑会是什么类型。
3. 其实我们只关心返回的是不是一个 `View`，而对实际上它是什么类型并不感兴趣。

`some View` 这种写法使用了 `Swift 5.1` 的 [Opaque return types特性](https://github.com/apple/swift-evolution/blob/master/proposals/0244-opaque-result-types.md)，它向编译器作出保证，每次 `body` 得到的一定是某一个确定的，遵守 `View` 协议的类型，但是请编译器“网开一面”，不要再细究具体的类型。返回类型`确定单一`这个条件十分重要，比如，下面的代码也是无法通过的：

```swift
let someCondition: Bool

// Error: Function declares an opaque return type, 
// but the return statements in its body do not have 
// matching underlying types.
var body: some View {
    if someCondition {
        // 这个分支返回 Text
        return Text("Hello World")
    } else {
        // 这个分支返回 Button，和 if 分支的类型不统一
        return Button(action: {}) {
            Text("Tap me")
        }
    }
}
```

这是一个`编译期间的特性`，在保证 `associatedtype protocol` 的功能的前提下，使用 `some` 可以抹消具体的类型。
这个特性用在 `SwiftUI` 上简化了书写难度，让不同 `View` 声明的语法上更加统一。

## Opaque Type 与 Generic 的关系 / 区别

Opaque Type 和 Generic 都解决了协议无法作为类型约束的问题，但它们解决问题的方式是截然相反的，或者说它们的思想是有区别的。

**Generic 让调用者决定参数或者返回值的类型**, 比如下面的 `drive(_:)` 方法，定义的时候只规定了参数必须遵从 `Vehicle`, 并没有具体安排那种类型。
具体是什么类型，完全由传入的变量决定：

```swift

func drive<V: Vehicle>(_ vehicle: V) {
    // ...
}

let car = Audi()
drive(car)
```

而 **Opaque Type 是让方法本身决定返回的类型**. 比如下面的 `getVehicle()` 方法，调用者完全不知道方法会返回什么类型的值，只知道它一定遵从 `Vehicle`。

```swift
func getVehicle() -> some Vehicle {
    someone.vehicle
}

let car = getVehicle()
```
