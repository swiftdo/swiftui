# 关于 ViewBuilder

SwiftUI DSL 的需要：

* 从表达方式上从简：尽量省略不必要的逗号，`return`，中括号等等。
* 支持简单的逻辑控制，比如 `if` 控制语句。
* 强类型：`some View` 代表了一个复合的强类型，在 `View` 发生改变的时候，复合的强类型有助于做 `View diff` 优化。
* 与 Swift 已有的语法不冲突

```swift
// 定义
struct VStack<Content> where Content : View {
  init(alignment: HorizontalAlignment = .center, spacing: Length? = nil,
  @ViewBuilder content: () -> Content)
}

// 使用
struct ContentView : View {
  var body: some View {
    VStack(alignment: .leading) {
      Text("Hello, World")
      Text("Leon Lu")
    }
  }
}
```

如何在一个容器类型 `VStack` 的构造函数的闭包中平铺其包含的两个 `Text`；另一方面，在闭包的函数声明中，我们看到了 `@ViewBuidler` 的修饰；
其实不难推测，为了编译通过， `ViewBuidler` 对于这个闭包中的代码在编译阶段 “动了手脚”，那么这是如何做到的呢？来看 `ViewBuilder` 中的关键方法：

```swift
static func buildBlock() -> EmptyView
static func buildBlock<Content>(Content) -> Content
static func buildBlock<C0, C1>(C0, C1) -> TupleView<(C0, C1)>
static func buildBlock<C0, C1, C2>(C0, C1, C2) -> TupleView<(C0, C1, C2)>
static func buildBlock<C0, C1, C2, C3>(C0, C1, C2, C3) -> TupleView<(C0, C1, C2, C3)>
static func buildBlock<C0, C1, C2, C3, C4>(C0, C1, C2, C3, C4) -> TupleView<(C0, C1, C2, C3, C4)>
...
```

我们的两个 `Text` 的例子中，编译器自动（根据名称的约定）使用了 

```swift
static func buildBlock<C0, C1>(C0, C1) -> TupleView<(C0, C1)>
```

方法，这时候 `VStack` 的类型就成为了 `VStack<TupleView<(Text,Text)>>` 了。经过 `ViewBuilder` 转换后的代码：

```swift
struct ContentView : View {
  var body: some View {
    VStack(alignment: .leading) {
      ViewBuilder.buildBlock(Text("Hello, World"), Text("Leon Lu"))
    }
  }
}
```

值得一提的是，由于 `buildBlock` 的 overload 版本最多泛型参数是 10 个。所以当超过 10 个的时候可以使用 `Group` 包一下； 如果有循环可以展开，则可以使用 `ForEach`。

**分支条件的情况**

`ViewBuilder` 中还有两个函数被用来构建含分支条件时候的类型

```swift
static func buildEither<TrueContent, FalseContent>(first: TrueContent) ->
 ConditionalContent<TrueContent, FalseContent>

static func buildEither<TrueContent, FalseContent>(second: FalseContent) -> 
 ConditionalContent<TrueContent, FalseContent>
```

如果根据不同条件返回不同的视图，那么生成的类型中包含两个类型。

```swift
struct SlideViewer: View {
  @State private var isEditing = false
  @Binding var slide: Slide

  var body: some View {
    VStack {
      Text("Slide #\(slide.number)")

      if isEditing {
        TextFiled($slide.title)
      } else {
        Text(slide.title)
      }
    }
  }
}
```

此时，`VStack` 的类型变成了

```swift
VStack<TupleView<(Text, ConditionalContent<TextField,Text>)>>
```
