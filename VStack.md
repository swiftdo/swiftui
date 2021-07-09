# VStack

> 文档：[https://developer.apple.com/documentation/swiftui/vstack](https://developer.apple.com/documentation/swiftui/vstack)

垂直排列子元素的视图。

创建一个垂直排列的静态列表：

```swift
VStack (alignment: .center, spacing: 20){
    Text("Hello")
    Divider()
    Text("World")
}
```

![](http://blog.loveli.site/mweb/16255374287716.jpg)


VStack 的定义：

```swift
@frozen public struct VStack<Content> : View where Content : View {
    @inlinable public init(alignment: HorizontalAlignment = .center, spacing: CGFloat? = nil, @ViewBuilder content: () -> Content)
    public typealias Body = Never
}
```

HStack 构造函数的参数说明：

* `spacing: CGFloat?`，设置子视图之间垂直间距
* `alignment: HorizontalAlignment`, 子视图对齐方式

## HorizontalAlignment

> An alignment position along the horizontal axis.

它是个结构体，有以下几个静态属性：

* leading 
* center
* trailing

为了更深刻的理解这几种布局差役，首先创建个通用的 `VStackCaseItem`, 便于代码复用：

```swift
struct VStackCaseItem: View {
    let alignment: HorizontalAlignment
    
    var body: some View {
         VStack(alignment: alignment, spacing: 20){
            Text("A\nB")
                .frame(width: 50)
                .background(Color.yellow)
            
            Text(alignment.name)
                .foregroundColor(.white)
                .frame(width:150)
                .background(Color.red)
                
               
            Text("OldBirds")
                .background(Color.green)
                .foregroundColor(.white)
            
         }.background(Color.gray)
    }
}

extension HorizontalAlignment {
    var name: String {
        switch self {
        case .leading:
            return "leading"
        case .trailing:
            return "trailing"
        case .center:
            return "center"
        default:
            return "other"
        }
    }
}
```

然后分别初始化不同的 alignment：

```swift

struct ContentView: View {
    var body: some View {
        VStack(spacing: 20) {
            VStackCaseItem(alignment: .leading)
            VStackCaseItem(alignment: .trailing)
            VStackCaseItem(alignment: .center)
        }
    }
}
```

![](https://blog.loveli.site/mweb/Screen%20Shot%202021-07-07%20at%209.22.08%20AM.png)

通过预览效果，可以比较容易发现 `alignment` 间的差异。

* `.leading`: 子视图局左对齐
* `.center`: 子视图局中对齐
* `.trailing`: 子视图局右对齐

## 使 VStack 填充屏幕的宽度

有这么个案例：

```swift
struct ContentView: View {
    var body: some View {
        VStack(alignment: .leading) {
          Text("标题")
            .font(.title)

          Text("内容")
            .lineLimit(nil)
            .font(.body)

          Spacer()
        }
        .background(Color.red)
    }
}
```

显示效果：

![](http://blog.loveli.site/mweb/16256254805070.jpg)

标签/文本组件不需要全宽，如何让 VStack 填充屏幕的宽度？

### 方案一：设置 frame

```swift
struct ContentView: View {
    var body: some View {
        VStack(alignment: .leading) {
          Text("标题")
            .font(.title)
            .background(Color.yellow)

          Text("内容")
            .lineLimit(nil)
            .font(.body)
            .background(Color.blue)
        }
        .frame(
          maxWidth: .infinity,
          maxHeight: .infinity,
          alignment: .topLeading
        )
        .background(Color.red)
    }
}
```

![](http://blog.loveli.site/mweb/16256261587910.jpg)

除了通过设置尺寸为 `.infinity`, 还可以通过 `GeometryReader` 设置尺寸：

```swift
struct ContentView: View {
    var body: some View {
        GeometryReader { geometryProxy in
            VStack(alignment: .leading) {
              Text("标题")
                .font(.title)
                .background(Color.yellow)

              Text("内容")
                .lineLimit(nil)
                .font(.body)
                .background(Color.blue)
            }
            .frame(
                maxWidth: geometryProxy.size.width,
                maxHeight: geometryProxy.size.height,
                alignment: .topLeading
            )
        }
        .background(Color.red)
    }
}
```

![](http://blog.loveli.site/mweb/16256264169659.jpg)

### 方案二：结合 HStack 和 Spacer

```swift
struct ContentView: View {

    var body: some View {
        HStack {
            VStack(alignment: .leading) {
              Text("标题")
                .font(.title)
                .background(Color.yellow)

              Text("内容")
                .lineLimit(nil)
                .font(.body)
                .background(Color.blue)
                
                Spacer()
            }

            Spacer()
        }.background(Color.red)
    }
}
```

![](http://blog.loveli.site/mweb/16256266738969.jpg)






