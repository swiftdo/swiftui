# Image

> 文档：[https://developer.apple.com/documentation/swiftui/image](https://developer.apple.com/documentation/swiftui/image)

## 显示图像的视图

```swift
var body: some View {
    VStack {
        Image("space11-people")
    }
} //图像名字为 space11-people
```
![](http://blog.loveli.site/tuc/20210617210720.png ':size=300')


## 使用 SF Symbols：

```swift
var body: some View {
    VStack {
        Image(systemName: "clock.fill")
    }
}
```

![](http://blog.loveli.site/tuc/20210617213409.png ':size=300')


您可以通过为系统图标添加样式，来匹配您使用的字体：

```swift
var body: some View {
    VStack {
        Image(systemName: "cloud.heavyrain.fill")
            .foregroundColor(.red)
            .font(.title)
            .padding()
        
        Image(systemName: "clock")
            .foregroundColor(.red)
            .font(Font.system(.largeTitle).bold())
            .padding()
    }
}
```

![](http://blog.loveli.site/tuc/20210617213703.png ':size=240')


## 调整大小

Image 视图具有以不同方式缩放的能力，默认情况下，图像视图会自动调整其大小以适应其内容，这可能会使它们超出屏幕。如果添加`resizable()`修改器，则图像将自动调整大小，以使其充满所有可用空间：

```swift
var body: some View {
    VStack {
        Image("space11-people")
            .resizable() // 调整大小，以便填充所有可用空间
    }
} 
```

![](http://blog.loveli.site/tuc/20210617210951.png ':size=300')

这也可能导致图像的原始宽高比失真，因为它将在所有尺寸上拉伸所需的任何量以使其填充空间。

如果要保持宽高比，则应 `aspectRatio` 使用 `.fill` 或来添加修饰符 `.fit`。

将图片设置为 `.fit`：

```swift
var body: some View {
    VStack {
        Image("space11-people")
            .resizable()
            .aspectRatio(contentMode: .fit)
    }
}
```
![](http://blog.loveli.site/tuc/20210617211431.png ':size=300')

将图片设置为 `.fill`：

```swift
var body: some View {
    VStack {
        Image("space11-people")
            .resizable()
            .aspectRatio(contentMode: .fill)
    }
}
```

![](http://blog.loveli.site/tuc/20210617211719.png ':size=300')

如果想自定义 Image 大小，可以添加 `frame`，`clipped()` 相当于 UIKit 里的 `clipsToBounds`，与`aspectRatio(contentMode: .fill)`搭配使用：

```swift
var body: some View {
    VStack {
        Image("space11-people")
            .resizable()
            .aspectRatio(contentMode: .fill)
            .frame(width: 360, height: 200)
            .clipped()
    }
}
```

![](http://blog.loveli.site/tuc/20210617212138.png ':size=300')





