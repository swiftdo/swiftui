# 介绍

> A modifier that you apply to a view or another view modifier, producing a different version of the original value.

属性修改器可以用于多个视图之中，生成与原始值不同的版本。

## 样例

首先我们创建一个自定义的 `ViewModifier` 结构体来实现我们想做的事情：

```swift
struct Title: ViewModifier {
    func body(content: Content) -> some View {
        content
            .font(.largeTitle)
            .foregroundColor(.white)
            .padding()
            .background(Color.blue)
            .clipShape(RoundedRectangle(cornerRadius: 10))
    }
}
```

现在我们可以通过 `modifier()` 方法来使用这个 modifier:

```swift
Text("Hello World")
    .modifier(Title())
```

使用自定义 `modifier` 的时候，基于 `View` 创建 `extension` 是个好主意：

```swift
extension View {
    func ob_titleStyle() -> some View {
        self.modifier(Title())
    }
}
```

## 让所有标题拥有一个特别的样式

可以根据需要创建新的视图结构。`modifiers` 是返回新的对象，而不是修改已经存在的对象，因此我们可以把视图嵌到一个`ZStack`里，并且添加其他视图：

```swift
struct Watermark: ViewModifier {
    var text: String

    func body(content: Content) -> some View {
        ZStack(alignment: .bottomTrailing) {
            content
            Text(text)
                .font(.caption)
                .foregroundColor(.white)
                .padding(5)
                .background(Color.black)
        }
    }
}

extension View {
    func od_watermarked(with text: String) -> some View {
        self.modifier(Watermark(text: text))
    }
}
```

上面的代码就位后，我们就可以像下面这样给任何视图添加水印了：

```swift
Color.blue
    .frame(width: 300, height: 200)
    .od_watermarked(with: "Oldbirds with Swiftui")
```

这块可以比较方便抽出产品的品牌样式，比如：

```swift
extension Text {
    enum Kind {
        case primary
        case secondary
    }

    func style(_ kind: Kind) -> some View {
        switch kind {
        case .primary:
            return self
                .padding()
                .background(Color.black)
                .foregroundColor(Color.white)
                .font(.largeTitle)
                .cornerRadius(10)

        case .secondary:
            return self
                .padding()
                .background(Color.blue)
                .foregroundColor(Color.red)
                .font(.largeTitle)
                .cornerRadius(20)
        }
    }
}

struct ContentView: View {
    @State var kind = Text.Kind.primary

    var body: some View {
        VStack {
        Text("Primary")
            .style(kind)
            Button(action: {
                self.kind = .secondary
            }) {
                Text("Change me to secondary")
            }
        }
    }
}
```