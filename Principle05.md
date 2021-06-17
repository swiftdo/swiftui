# 开始

这个语法特性叫 [Function Builders](https://link.zhihu.com/?target=https%3A//github.com/apple/swift-evolution/blob/9992cf3c11c2d5e0ea20bee98657d93902d5b174/proposals/XXXX-function-builders.md)，还没有正式添加到 Swift 语言中，只有草案。苹果直接修改了 Swift 编译器，SwiftUI 已经在用这个语法特性了。


`ViewBuilder` 结构使用了 `@_functionBuilder` 来修饰。而闭包使用 `@ViewBuilder` 来修饰，就会修改语法树，转调 `ViewBuilder` 的 `buildBlock` 函数。于是

```swift
VStack {
    Text("Hello World")
    Text("Hello SwiftUI")
    Text("Hello Friends")
}
```

就相当于:

```swift
VStack {
    return ViewBuilder.buildBlock(
        Text("Hello World"), 
        Text("Hello SwiftUI"),
        Text("Hello Friends")
    )
}
```

## 参阅

* [Function Builders in Swift and SwiftUI](https://www.vadimbulavin.com/swift-function-builders-swiftui-view-builder/)