# Image

> 文档：[https://developer.apple.com/documentation/swiftui/image](https://developer.apple.com/documentation/swiftui/image)

显示图像的视图。

```swift
Image("foo") //图像名字为 foo
```

我们可以使用新的 SF Symbols：

```swift
Image(systemName: "clock.fill")
```

您可以通过为系统图标添加样式，来匹配您使用的字体：

```swift
Image(systemName: "cloud.heavyrain.fill")
    .foregroundColor(.red)
    .font(.title)
Image(systemName: "clock")
    .foregroundColor(.red)
    .font(Font.system(.largeTitle).bold())
```

为图片增加样式：

```swift
Image("foo")
    .resizable() // 调整大小，以便填充所有可用空间
    .aspectRatio(contentMode: .fit)
```

