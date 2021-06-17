
# GeometryReader

> 原文：[GeometryReader to the Rescue](https://swiftui-lab.com/geometryreader-to-the-rescue/)

当我们创建自定义视图时，一般不用担心旁边视图的布局或`size`。如果你想要创建一个正方形。只要用一个`Rectangle`，就会以父级想要的`size`和`position`去画出一个正方形。

```swift
struct ContentView : View {
    var body: some View {
        VStack {
            Text("Hello There!")
            MyRectangle()
        }.frame(width: 150, height: 100).border(Color.black)

    }
}

struct MyRectangle: View {
    var body: some View {
        Rectangle().fill(Color.blue)
    }
}
```

![](http://blog.loveli.site/mweb/16032475574382.jpg)

正如你所看到的，`MyRectangle()`，不用去设置 size，它只有一个任务，就是画矩形。让SwiftUI 自己去管理好父级期望的子视图的大小和位置。 这个例子里 Vstack 就是父级视图。

> 如果你想要知道更多关于父级视图如何确定子视图的位置和大小：[Building Custom Views With SwiftUI](https://developer.apple.com/videos/play/wwdc2019/237/)

父级视图会自动为子视图找到合适的尺寸和位置。但是如果你想要自定义绘制一个矩形，大小是父级视图的一半。位置位于父级视图右边距里5像素的视图。其实也并不复杂，这个时候可以用`GeometryReader` 作为解决方案。


## 子视图做了什么？

> A container view that defines its content as a function of its own size and coordinate space.
> 一个容器视图，根据其自身大小和坐标空间定义其内容。

`GeometryReader`让你定义了它的`content`。 但是与其他`View`不同。你可以拿到一些你在其他`View`中拿不到的信息。

![](http://blog.loveli.site/mweb/16032478700155.jpg)


```swift
struct ContentView : View {
    var body: some View {
        VStack {
            Text("Hello There!")
            MyRectangle()
        }.frame(width: 150, height: 100).border(Color.black)
    }
}

struct MyRectangle: View {
    var body: some View {
        GeometryReader { geometry in
            Rectangle()
                .path(in: CGRect(x: geometry.size.width + 5,
                                 y: 0,
                                 width: geometry.size.width / 2.0,
                                 height: geometry.size.height / 2.0))
                .fill(Color.blue)
            
        }
    }
}
```

## GeometryProxy

上面的例子中，闭包中的`geometry`是一个`GeometryProxy`类的变量。

在 `GeometryProxy` 类中有两个计算型属性，一个方法，和一个下标取值。

```swift
// The size of the container view.
public var size: CGSize { get } // 父级视图建议的大小

// The safe area inset of the container view.
public var safeAreaInsets: EdgeInsets { get }

// Returns the container view’s bounds rectangle, converted to a defined coordinate space.
public func frame(in coordinateSpace: CoordinateSpace) -> CGRect


public subscript<T>(anchor: Anchor<T>) -> T where T : Equatable { get }
```

* **frame 方法**：暴露给我们了父级视图建议区域的大小位置，可以通过 `coordinateSpace`来获取不同的坐标空间。

    ```swift
    enum CoordinateSpace {
        case global
        case local
        case named(AnyHashable)
    }
    ```
    
    * `global`的意思是获取基于整块屏幕的坐标位置
    * `local`指的是容器视图，也就是`GeometryReader`的内部的相对位置。对于 `local` 怎么使用，这里有一张图，它比较清楚地解释了`local`中的位置参数。

        ![](http://blog.loveli.site/mweb/16033300134485.jpg)

    * `.name(AnyHashable)` 是获取相对坐标，比方下面的例子相对于蓝色 stack view 的坐标。
    
        ![](http://blog.loveli.site/mweb/16033307451728.jpg)

        ```swift
        struct ContentView: View {
            var body: some View {
                    VStack {
                        Image("tony")
                                .resizable()
                                .scaledToFit()
                                .frame(width: 300)
                                .overlay(
                                    GeometryReader {  geometry in
                                        Text(geometry.frame(in: .named("blue view")).origin.debugDescription)
                                            .background(Color.yellow)
                                    }
                        )
                    }
                    .frame(width: 400, height: 600)
                    .background(Color.blue)
                    .coordinateSpace(name: "blue view")
                }
        }
        ```
        
        给 `VStack` 设置 `.coordinateSpace(name: "blue view")`，之后即可找出相对于 `blue view`的坐标。
        
        ```swift
        geometry.frame(in: .named("blue view")).origin.debugDescription
        ```
    

* **subscript**: 通过下标取值来获取一个锚点。可以获取视图树中任何子级视图的 size 和 x，y


## 实战

`GeometryReader` 功能已经相当强大，但它如果与 **.background()** 或 **.overlay()** 的 modifier 相结合使用，功能就会更强大。

```swift
Text("hello").background(Color.red)
```

第一眼看，都会以为`Color.red`是一个颜色参数，它设置了背景色是红色，其实并不是，`Color.red`是一个`View`！它的功能就是把父级视图所建议的区域填充为红色。它的父级就是背景。而且背景修改了`Text`。所以建议`Color.red`所填充的区域就是`Text("Hello")`所在的区域。

`.overlay` 修改器也是同样的道理，只是它并不是绘制背景，而是绘制前景而已。

我们可以给任意一个`view`使用`.overlay()`方法还有`.background()`方法。下面我们将结合`GeometryReader`，画一个每个角指定不同的半径的矩形的例子来演示如何利用它们

![](http://blog.loveli.site/mweb/16032495562741.jpg)


```swift
struct ContentView : View {
    var body: some View {
        HStack {
            Text("SwiftUI")
                .foregroundColor(.black).font(.title).padding(15)
                .background(RoundedCorners(color: .green, tr: 30, bl: 30))
            
            Text("Lab")
                .foregroundColor(.black).font(.title).padding(15)
                .background(RoundedCorners(color: .blue, tl: 30, br: 30))
            
        }.padding(20).border(Color.gray).shadow(radius: 3)
    }
}

struct RoundedCorners: View {
    var color: Color = .black
    var tl: CGFloat = 0.0
    var tr: CGFloat = 0.0
    var bl: CGFloat = 0.0
    var br: CGFloat = 0.0
    
    var body: some View {
        GeometryReader { geometry in
            Path { path in
                
                let w = geometry.size.width
                let h = geometry.size.height
                
                // We make sure the redius does not exceed the bounds dimensions
                let tr = min(min(self.tr, h/2), w/2)
                let tl = min(min(self.tl, h/2), w/2)
                let bl = min(min(self.bl, h/2), w/2)
                let br = min(min(self.br, h/2), w/2)
                
                path.move(to: CGPoint(x: w / 2.0, y: 0))
                path.addLine(to: CGPoint(x: w - tr, y: 0))
                path.addArc(center: CGPoint(x: w - tr, y: tr), radius: tr, startAngle: Angle(degrees: -90), endAngle: Angle(degrees: 0), clockwise: false)
                path.addLine(to: CGPoint(x: w, y: h - br))
                path.addArc(center: CGPoint(x: w - br, y: h - br), radius: br, startAngle: Angle(degrees: 0), endAngle: Angle(degrees: 90), clockwise: false)
                path.addLine(to: CGPoint(x: bl, y: h))
                path.addArc(center: CGPoint(x: bl, y: h - bl), radius: bl, startAngle: Angle(degrees: 90), endAngle: Angle(degrees: 180), clockwise: false)
                path.addLine(to: CGPoint(x: 0, y: tl))
                path.addArc(center: CGPoint(x: tl, y: tl), radius: tl, startAngle: Angle(degrees: 180), endAngle: Angle(degrees: 270), clockwise: false)
                }
                .fill(self.color)
        }
    }
}
```

另外，我们对自定义的Overlay设置透明度为0.5，设置在Text上。 代码如下：

```swift
Text("SwiftUI")
    .foregroundColor(.black).font(.title).padding(15)
    .overlay(RoundedCorners(color: .green, tr: 30, bl: 30).opacity(0.5))

Text("Lab")
    .foregroundColor(.black).font(.title).padding(15)
    .overlay(RoundedCorners(color: .blue, tl: 30, br: 30).opacity(0.5))
```

## 关于鸡和鸡蛋的问题

因为`GeometryReader`需要给子级视图提供可用空间，它首先需要尽可能多的占用空间。但是子类可能会设置一个小的空间，这时候`GeometryReader`还是尽可能的保持大。一旦子级试图确定了其所需空间，你可能会被迫缩小`GeometryReader`。这时候子级试图就会`GeometryReader`计算出的新的大小做出反应。 一个循环就产生了。

所以 当遇到是子级视图依赖父级试图的大小，还是父级试图依赖于子级视图的大小。可能`GeometryReader`并不适合解决你的布局问题。由此，你可以看看我的下一篇文章 [Preferences and Anchor Preferences](https://swiftui-lab.com/communicating-with-the-view-tree-part-1/)。