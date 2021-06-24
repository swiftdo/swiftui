# Button

> 文档：[https://developer.apple.com/documentation/swiftui/button](https://developer.apple.com/documentation/swiftui/button)
>
> 环境：Xcode 12.5.1，Swift5.4

## 概览

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

为了更好的运用 Button，下面我们将一起来实现一些常见的效果：

* 如何创建一个简单的按钮并处理用户的选择？
* 如何自定义按钮的背景、填充和字体？
* 如何给按钮添加边框？
* 如何创建同时包含图像和文本的按钮？
* 如何创建具有渐变背景和阴影的按钮？
* 如何创建圆角按钮且添加外圆角边框？


## 如何创建一个简单的按钮并处理用户的选择？

使用 SwiftUI 创建按钮非常容易。基本上，您可以使用下面的代码片段来创建一个按钮：

```swift
struct CasePage: View {
    @State var change = false;

    var body: some View {
        VStack {
            Text(change ? "欢迎关注 OldBirds 公众号": "OldBirds")
            .font(.title)
            
            Button("welcome") {
                self.change.toggle()
            }.padding()
        }
    }
}
```

![](http://blog.loveli.site/tuc/111ee.gif ':size=300')


## 如何自定义按钮的背景、填充和字体？

```swift
Button(action: {
    self.change.toggle()
    
}, label: {
    Text("Hello World")
        .padding()
        .background(Color.purple)
        .foregroundColor(.white)
        .font(.title)
        
}).padding()
```

![](http://blog.loveli.site/tuc/111ee002.gif ':size=300')

特别强调：modifier 的顺序非常重要，假设我们把 `padding` 移动，不在 `background` 的前面，效果是不一样的。

```swift
Button(action: {
    self.change.toggle()
    
}, label: {
    Text("Hello World")
        .background(Color.purple)
        .foregroundColor(.white)
        .font(.title)
        .padding()
}).padding()
```

![](http://blog.loveli.site/tuc/111ee003.gif ':size=300')


## 如何给按钮添加边框？

```swift
Button(action: {
    self.change.toggle()
}, label: {
    Text("Hello World")
        .padding()
        .background(Color.purple)
        .foregroundColor(.white)
        .font(.title)
        .border(Color.red, width: 5)
})
```

![](http://blog.loveli.site/tuc/111ee004.gif ':size=300')



## 如何创建同时包含图像和文本的按钮？

```swift
Button(action: {
    self.change.toggle()
}, label: {
    HStack(content: { 
        Image(systemName: "trash")
        Text("Hello World")
            .foregroundColor(.black)
            .font(.title)
    })
    .padding()
    .background(Color.purple)
    .border(Color.red, width: 5)
    
})
```

![](http://blog.loveli.site/tuc/111ee005.gif ':size=300')

## 如何创建具有渐变背景和阴影的按钮？

```swift
Button(action: {
    self.change.toggle()
}, label: {
    HStack(content: {
        Image(systemName: "trash")
            .font(.title)
        Text("Hello World")
            .font(.title)
    })
    .padding()
    .foregroundColor(.white)
    .background(LinearGradient(gradient: Gradient(colors: [Color.green, Color.blue]), startPoint: .leading, endPoint: .trailing))
    .cornerRadius(40)
})
```

![](http://blog.loveli.site/tuc/111ee006.gif ':size=300')

## 如何创建圆角按钮且添加外圆角边框？

```swift
 
Button(action: {
    self.change.toggle()
}, label: {
    HStack(content: {
        Image(systemName: "trash")
            .font(.title)
        Text("Hello World")
            .font(.title)
    })
    .padding()
    .foregroundColor(.white)
    .background(LinearGradient(gradient: Gradient(colors: [Color.green, Color.blue]), startPoint: .leading, endPoint: .trailing))
//  .border(Color.red, width: 5) // 不行
    .cornerRadius(40)
    .overlay(RoundedRectangle(cornerRadius: 40).stroke(Color.red, lineWidth: 5))
})
```

![](http://blog.loveli.site/tuc/111ee007.gif ':size=300')

## 推荐阅读

* [A Beginner’s Guide to SwiftUI Buttons](https://www.appcoda.com/swiftui-buttons/#button-full-width)
* [SwiftUI 的按鈕 — button](https://medium.com/%E5%BD%BC%E5%BE%97%E6%BD%98%E7%9A%84-swift-ios-app-%E9%96%8B%E7%99%BC%E5%95%8F%E9%A1%8C%E8%A7%A3%E7%AD%94%E9%9B%86/swiftui-%E7%9A%84%E6%8C%89%E9%88%95-button-89d1c35d99dc)
* [Build your own button component library in SwiftUI from scratch](https://www.calincrist.com/blog/2020-05-12-step-up-your-button-theme-in-swiftui/)
* [Create and Customize a Button with SwiftUI](https://programmingwithswift.com/create-and-customize-a-button-with-swiftui/)
* [SwiftUI: Buttons](https://whatdidilearn.info/2020/05/16/swiftui-buttons.html)