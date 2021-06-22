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

## 推荐阅读

* [A Beginner’s Guide to SwiftUI Buttons](https://www.appcoda.com/swiftui-buttons/#button-full-width)
* [SwiftUI 的按鈕 — button](https://medium.com/%E5%BD%BC%E5%BE%97%E6%BD%98%E7%9A%84-swift-ios-app-%E9%96%8B%E7%99%BC%E5%95%8F%E9%A1%8C%E8%A7%A3%E7%AD%94%E9%9B%86/swiftui-%E7%9A%84%E6%8C%89%E9%88%95-button-89d1c35d99dc)
* [Build your own button component library in SwiftUI from scratch](https://www.calincrist.com/blog/2020-05-12-step-up-your-button-theme-in-swiftui/)
* [Create and Customize a Button with SwiftUI](https://programmingwithswift.com/create-and-customize-a-button-with-swiftui/)
* [SwiftUI: Buttons](https://whatdidilearn.info/2020/05/16/swiftui-buttons.html)