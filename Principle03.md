# 关于 some View

在 Swift 5.0 之前我们如果想返回抽象类型一般使用 `Generic Type` 或者 `Protocol`, 使用泛型会显示的暴露一些信息给 `API` 使用者，不是完整的类型抽象

但是使用 `Protocol` 也有几个限制: 泛型返回值在运行时都是一个容器，效率较差，返回值不能调用自身类型的方法，协议不允许拥有关联类型，由于编译时丢失了类型信息，编译器无法推断类型，导致无法使用 `==` 运算符。

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

这种带有 `associatedtype` 的协议不能作为类型来使用，而只能作为`类型约束`使用：

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

这是一个`编译期间的特性`，在保证 `associatedtype protocol` 的功能的前提下，使用 `some` 可以抹消具体的类型。这个特性用在 `SwiftUI` 上简化了书写难度，让不同 `View` 声明的语法上更加统一。
