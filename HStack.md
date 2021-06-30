# HStack

> 文档：[https://developer.apple.com/documentation/swiftui/hstack](https://developer.apple.com/documentation/swiftui/hstack)

水平排列子元素的视图。

创建一个水平排列的静态列表：

```swift
struct HStackPage: View {
    var body: some View {
        HStack (spacing: 10){
            Spacer()
                .frame(width: 50, height: 50)
                .background(Color.red)
                .cornerRadius(25)
                .overlay(
                    RoundedRectangle(cornerRadius: 25)
                            .stroke(lineWidth: 4)
                            .foregroundColor(.blue)
                )
            
            Text("Hello, World!")
            Spacer()
        }.padding()
      
    }
}
```

![](http://blog.loveli.site/tuc/20210630212004.png)

HStack 的定义：

```swift
@frozen public struct HStack<Content> : View where Content : View {
    @inlinable public init(alignment: VerticalAlignment = .center, spacing: CGFloat? = nil, @ViewBuilder content: () -> Content)

    public typealias Body = Never
}
```

HStack 构造函数的参数说明：

* `spacing: CGFloat?`，设置子视图之间的空隙
* `alignment: VerticalAlignment`, 子视图的垂直对齐方式

spacing 比较容易设置和理解，下面着重介绍下 alignment。

## VerticalAlignment

> An alignment position along the horizontal axis.(沿水平轴的对齐位置。)

它是个结构体，有以下几个静态属性：

* static let bottom: VerticalAlignment
* static let center: VerticalAlignment
* static let firstTextBaseline: VerticalAlignment
* static let lastTextBaseline: VerticalAlignment
* static let top: VerticalAlignment

为了更深刻的理解这几种布局的差异，首先创建个通用组件 `HStackCaseItem`，便于代码的复用：

```swift
struct HStackCaseItem: View {
    let alignment: VerticalAlignment
    var body: some View {
         HStack(alignment: alignment, spacing: 20){
            Text("A\nB")
                .frame(width: 50)
                .background(Color.yellow)
            
            Text("\(alignment.name)\nhello\nworld\nOldBirds")
                .foregroundColor(.white)
                .frame(width:150)
                .background(Color.red)
                .padding([.bottom, .top])
                .background(Color.red.opacity(0.5))
                
            Text("OldBirds")
                .background(Color.green)
                .foregroundColor(.white)
            
         }.background(Color.gray)
    }
}

extension VerticalAlignment {
    var name: String {
        switch self {
        case .top:
            return "top"
        case .bottom:
            return "bottom"
        case .center:
            return "center"
        case .firstTextBaseline:
            return "firstTextBaseline"
        case .lastTextBaseline:
            return "lastTextBaseline"
        default:
            return "other"
        }
    }
}
```

然后分别初始化不同的 alignment：

```swift
struct CasePage: View {
    var body: some View {
        VStack {
            HStackCaseItem(alignment: .top)
            Spacer().frame(height: 20)
            HStackCaseItem(alignment: .bottom)
            Spacer().frame(height: 20)
            HStackCaseItem(alignment: .center)
            Spacer().frame(height: 20)
            HStackCaseItem(alignment: .firstTextBaseline)
            Spacer().frame(height: 20)
            HStackCaseItem(alignment: .lastTextBaseline)
        }
    }
}
```

![](http://blog.loveli.site/tuc/20210630220746.png)

通过预览效果，可以比较容易发现 `alignment` 间的差异。

* `.top`: 子视图局顶部对齐
* `.center`: 子视图局中心对齐
* `.bottom`: 子视图局底部对齐
* `.firstTextBaseline`: 表示所有 text 的以各自最上边的那一行的 base line对齐
* `.lastTextBaseline`: 表示所有text的以最下边的那一行的 base line 对齐。

## 利用 padding 让 HStack 的视图重叠效果

```swift
struct CircleRow:View {
    let colors = [Color.red, Color.yellow, Color.purple]
    var body: some View {
        HStack {
            ForEach(0..<3) { (index) in
                Circle()
                    .strokeBorder(Color.white, lineWidth: 2)
                    .shadow(color: .black, radius: 5, x: 1, y: 1)
                    .background(Circle().foregroundColor(colors[index]))
                    .frame(width: 100, height: 100)
                    .padding(.leading, -30)
            }
        }
    }
}

```

![](http://blog.loveli.site/tuc/20210630222516.png)









