# @State

> The views are a function of state, not a sequence of events
>
> 视图是状态的函数，而不是事件的序列

在 SwiftUI 中，视图是由数据(状态)驱动的。每当视图在创建或解析时，都会为该视图和该视图中使用的状态数据之间创建一个依赖关系，每当状态的信息发生变化，有依赖关系的视图会马上翻译出这些变化并重绘。

```swift
struct CasePage: View {
    @State var name = "OldBirds"
    var body: some View {
        VStack {
            Text(name)
                .font(.title)
                .bold()
            
            Button("welcome") {
                self.name = "欢迎关注 OldBirds 公众号"
            }.padding()
        }
    }
}
```

![](http://blog.loveli.site/tuc/a.gif ':size=300')

`@State`只能用于当前视图，通过执行上面代码，我们可以发现：

* 通过使用`@State`，我们可以在未使用 mutating 的情况下修改结构中的值
* 当状态值发生变化后，视图会自动重绘以反应状态的变化。

## @State 如何工作?

`State`的定义：

```swift
@frozen @propertyWrapper public struct State<Value> : DynamicProperty {

    /// Creates the state with an initial wrapped value.
    public init(wrappedValue value: Value)

    /// Creates the state with an initial value.
    public init(initialValue value: Value)

    /// The underlying value referenced by the state variable.
    public var wrappedValue: Value { get nonmutating set }

    /// A binding to the state value.
    public var projectedValue: Binding<Value> { get }
}
```


`Binding` 的定义：

```swift
@frozen @propertyWrapper @dynamicMemberLookup struct Binding<Value>
```

`DynamicProperty` 的定义：

```swift
public protocol DynamicProperty {

    /// Called immediately before the view's body() function is
    /// executed, after updating the values of any dynamic properties
    /// stored in `self`.
    mutating func update()
}
```

* `@propertyWrapper`,意味着他是一个[属性包装器](Principle02.md)
* 它的`projectedValue`（投射值）为`Binding`类型。`Binding`是数据的一级引用，在 SwiftUI 中作为数据（状态）双向绑定的桥梁，允许在不拥有数据的情况下对数据进行读写操作。
* 遵循`DynamicProperty`协议，该协议完成了创建数据（状态）和视图的依赖操作所需接口。现在只暴露了很少的接口，我们暂时无法完全使用它。

读到这里，我们对 @State 的有一个初步的了解，但好像还停留在一个表象，无法体会到具体的运转原理。那么我们就仿写 State，更细化分析它的实现。

## 仿写 State
让我们先定义我们的状态如下:

```swift
@propertyWrapper
struct OBState<Val> {
    var value: Val
    var wrappedValue: Val {
        get {
          value
        }
        set {
          value = newValue
        }
    }
    init(wrappedValue value: Val) {
        self.value = value
    }
}
```

有了这个定义，我们现在可以将`@State` 替换为 `@OBState`：

```swift
struct CasePage: View {
    @OBState var name = "OldBirds"
    var body: some View {
        VStack {
            Text(name)
                .font(.title)
                .bold()
            
            Button("welcome") {
                self.name = "欢迎关注 OldBirds 公众号"
            }.padding()
        }
    }
}
```

替换后，出现报错：

![](http://blog.loveli.site/tuc/20210621210741.png ':size=300')

使用结构体，我们可以声明`mutating`的方法，但不能声明`mutating`的计算属性(如body)，也不能在计算属性中调用`mutating`的方法。

为了不改变结构体，我们需要给 `OBState` 的 wrappedValue 的 set 方法中声明 `nonmutating`。


```swift
@propertyWrapper
struct OBState<Val> {
    var value: Val
    var wrappedValue: Val {
        get {
          value
        }
        nonmutating set {
          value = newValue
        }
    }
    init(wrappedValue value: Val) {
        self.value = value
    }
}
```

很不幸，产生错误提示：

![](http://blog.loveli.site/tuc/20210621211631.png ':size=300')

`'self' is immutable` 从name转移到`OBState`的`wrappedValue`的`setter`方法中了。

因为我们承诺不改变 struct 实例，但我们设置`value = newValue`，这是可变的。所以对 `OBState` 的 value, 我们应该设计为**引用类型**。

```swift
final class Box<Val> {
  var value: Val
  init(_ value: Val) {
    self.value = value
  }
}

@propertyWrapper
struct OBState<Val> {
    var box: Box<Val>
    var wrappedValue: Val {
        get { box.value }
        nonmutating set { box.value = newValue }
    }
    init(wrappedValue value: Val) {
        self.box = Box(value)
    }
}
```

完成上述修改后，我们消灭了报错。

![](http://blog.loveli.site/tuc/bb.gif ':size=300')

点击按钮，但没有看到任何变化。我们更新数据，但 SwiftUI 并不知道它应该监听这些变化，随意没有重新绘制 body。

SwiftUI 会监听连接到每个属性包装器的所有发布者，然后这些发布者会告诉SwiftUI 什么时候重新绘制。

> 因为无法访问实际的SwiftUI源码，代码均为模仿，所以非最优实现

我们使用`@StateObject`来匹配`Box`的`ObservableObject`：

```swift
final class Box<Val>: ObservableObject {
  var value: Val {
    willSet {
      objectWillChange.send()
    }
  }
  init(_ value: Val) {
    self.value = value
  }
}

@propertyWrapper
struct OBState<Val> {
    @StateObject var box: Box<Val>
    
    var wrappedValue: Val {
        get { box.value }
        nonmutating set { box.value = newValue }
    }
    init(wrappedValue value: Val) {
        self._box = StateObject(wrappedValue: Box(value))
    }
}
```

由于每次更新box的值变化:

* objectWillChange事件被触发
* box的publisher将会监听到

再次运行：

![](http://blog.loveli.site/tuc/cc.gif ':size=300')

当我们的值发生变化时，新的发布者确实会发送事件。但这一变化 SwiftUI 并未接收到。

要改变这一点，我们需要`OBState`遵守`DynamicProperty`协议。

> An interface for a stored variable that updates an external property of a view.

```swift
@propertyWrapper
struct OBState<V>: DynamicProperty {
  ...
}
```

最后运行效果：

![](http://blog.loveli.site/tuc/1ee.gif ':size=300')

终于可以了！尽管最后成功实现点击效果。但这仅仅是个探究，具体实现肯定不会如此粗糙。但是经过这样去剖解分析，也对一些知识有更深刻的理解。

对于 Binding, 也可以用此分析方式去解剖，这里就不过多细说了。建议读者尝试一下。

## 推荐阅读
* [@State 研究](https://zhuanlan.zhihu.com/p/141229504)
* [SwiftUI:@State原理解析](https://www.jianshu.com/p/d6443da77f3e)
