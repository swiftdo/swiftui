# TextField

> 文档：[https://developer.apple.com/documentation/swiftui/textfield](https://developer.apple.com/documentation/swiftui/textfield)

显示可编辑文本界面的控件。

```swift
@State var name: String = "John"
var body: some View {
    TextField("Name's placeholder", text: $name)
        .textFieldStyle(RoundedBorderTextFieldStyle())
        .padding()
}
```

