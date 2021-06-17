# Button

> 文档：[https://developer.apple.com/documentation/swiftui/button](https://developer.apple.com/documentation/swiftui/button)

在触发时执行操作的控件。

```swift
Button(
    action: {
        // 点击事件
    },
    label: { Text("Click Me") }
)
```

如果按钮的标签只有 `Text`，则可以通过下面这种简单的方式进行初始化：

```swift
Button("Click Me") {
    // 点击事件
}
```

您可以像这样给按钮添加属性：

```swift
Button(action: {
                
}, label: {
    Image(systemName: "clock")
    Text("Click Me")
    Text("Subtitle")
})
.foregroundColor(Color.white)
.padding()
.background(Color.blue)
.cornerRadius(5)
```