# Toggle

> 文档：[https://developer.apple.com/documentation/swiftui/toggle](https://developer.apple.com/documentation/swiftui/toggle)

在开/关状态之间切换的控件。

```swift
@State var isShowing = true // toggle 状态值

Toggle(isOn: $isShowing) {
    Text("Hello World")
}
```