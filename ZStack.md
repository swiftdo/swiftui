# ZStack

> 文档：[https://developer.apple.com/documentation/swiftui/zstack](https://developer.apple.com/documentation/swiftui/zstack)

子元素会在 z轴方向上叠加，同时在垂直/水平轴上对齐的视图。

```swift
ZStack {
    Text("Hello")
        .padding(10)
        .background(Color.red)
        .opacity(0.8)
    Text("World")
        .padding(20)
        .background(Color.red)
        .offset(x: 0, y: 40)
}
```