# SecureField

> 文档：[https://developer.apple.com/documentation/swiftui/securefield](https://developer.apple.com/documentation/swiftui/securefield)

用户安全地输入私人文本的控件。

```swift
@State var password: String = "1234"    
var body: some View {
    SecureField($password)
        .textFieldStyle(RoundedBorderTextFieldStyle())
        .padding()
}
```