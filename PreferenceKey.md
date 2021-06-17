# PreferenceKey

> [Inspecting the View Tree – Part 1: PreferenceKey](https://swiftui-lab.com/communicating-with-the-view-tree-part-1/)

在`SwiftUI`中我们一般不用关心子级视图内部发生了什么。不同的`View`各自管各自内部的事情。但总是会遇到一些特殊的需求。比较惨的是[文档](https://developer.apple.com/documentation/swiftui/view/3278633-preference)都讲的比较粗略。我们将要去了解 `PreferenceKey` 的协议和相关的修改器(modifier):

```swift
.preference(),
.transformPreference(),
.anchorPreference(),
.transformAnchorPreference(),
.onPreferenceChange(),
.backgroundPreferenceValue()
.overlayPreferenceValue().
```

SwiftUI 有一个让我们去给`View`添加很多属性的机制。这些属性我们叫做`Preferences`。 它们可以轻松的沿视图层次结构传递，甚至在这些些偏好设置更改时执行回调。

有没有想过`NavigationView`是如何通过`.navigationBarTitle()`来获取 title。请注意`.navigationBarTitle()`并没有直接修改`NavigationView`。而是在沿着`View`的层级去调用。那么它是怎么做到的呢？ 


## 独立的Views

为了更好的了解今天的话题，我们先用一个没有使用偏好的例子开始。在例子中，先创建一个显示月份名的`View`。当月份标签被点击的时候，会在月份标签上面慢慢的显示一个边框(从之前选中的月份标签移除)。

![ezgif.com-gif-make](http://blog.loveli.site/mweb/ezgif.com-gif-maker.gif)

代码:

```swift
import SwiftUI

struct EasyExample : View {
    @State private var activeIdx: Int = 0
    
    var body: some View {
        VStack {
            Spacer()
            
            HStack {
                MonthView(activeMonth: $activeIdx, label: "January", idx: 0)
                MonthView(activeMonth: $activeIdx, label: "February", idx: 1)
                MonthView(activeMonth: $activeIdx, label: "March", idx: 2)
                MonthView(activeMonth: $activeIdx, label: "April", idx: 3)
            }
            
            Spacer()
            
            HStack {
                MonthView(activeMonth: $activeIdx, label: "May", idx: 4)
                MonthView(activeMonth: $activeIdx, label: "June", idx: 5)
                MonthView(activeMonth: $activeIdx, label: "July", idx: 6)
                MonthView(activeMonth: $activeIdx, label: "August", idx: 7)
            }
            
            Spacer()
            
            HStack {
                MonthView(activeMonth: $activeIdx, label: "September", idx: 8)
                MonthView(activeMonth: $activeIdx, label: "October", idx: 9)
                MonthView(activeMonth: $activeIdx, label: "November", idx: 10)
                MonthView(activeMonth: $activeIdx, label: "December", idx: 11)
            }
            
            Spacer()
        }
    }
}

struct MonthView: View {
    @Binding var activeMonth: Int
    let label: String
    let idx: Int
    
    var body: some View {
        Text(label)
            .padding(10)
            .onTapGesture { self.activeMonth = self.idx }
            .background(MonthBorder(show: activeMonth == idx))
    }
}

struct MonthBorder: View {
    let show: Bool
    
    var body: some View {
        RoundedRectangle(cornerRadius: 15)
            .stroke(lineWidth: 3.0).foregroundColor(show ? Color.red : Color.clear)
            .animation(.easeInOut(duration: 0.6))
    }
}
```

## 相互协作的 Views

下面难度再升级一些，我们想让边框从一个月份移动到另外一个。

![ezgif.com-gif-maker-2](http://blog.loveli.site/mweb/ezgif.com-gif-maker-2.gif)


边框并不是月份的一部分，你需要创建一个单独的边框View，并相应的改变位置和大小，这意味着必须有一种方式去跟踪每个月份的大小和位置。

解决这个问题的一种方式就是： 每一个月份标签都通过`GeometryReader`去获得自身的大小和位置。每个月份标签依次更新父级视图中的存放位置的数组(通过 @Binding )。 一旦父级视图找到了每一个子视图的位置和大小，边框就可以很容易的替换了。这个方案还不错，但子级视图修改数组的时候可能会产生问题。

对于某些布局，如果在构建视图的时候，修改其某个变量，其父级视图也会受到影响，反过来子级视图也会受到影响。这使我们正在构建的视图失效，有时可能需要再重新开始构建视图。 还有时候会变成一个循环。好的是 SwiftUI 视乎可以检测到这种情况，也不会产生崩溃。它会给你一个运行时的警告: Modifying state during view update(当视图更新的时候修改视图). 快速修复这个问题的方法是延迟变量的改变，直到视图的更新完成：

```swift
DispatchQueue.main.async {
  self.rects[k] = rect
}
```

不过这好像有点取巧(hack), 虽然这起作用了，但只是一个暂时的解决方案。不确定以后会不会起作用。有点对框架底层的原理下赌注的意思了。幸运的是`PreferenceKey`可以解决。

## PreferenceKey

SwiftUI 提供给我们一个修改器让我们添加一些数据到某个具体的视图。我们可以通过顶级视图(ancestor view)查询这些数据。并且有多种方式去读取`PreferenceKey`。这取决于你的目的是怎样的。

我们先命名一个遵守`Equatable`协议的`MyTextPreferenceData`的结构体

```swift
struct MyTextPreferenceData: Equatable {
    let viewIdx: Int // 去标记一些view
    let rect: CGRect // 获取文本框的 CGRect
}
```

然后我们定义一个遵循`PreferenceKey`的结构体`MyTextPreferenceKey`

```swift
struct MyTextPreferenceKey: PreferenceKey {
    typealias Value = [MyTextPreferenceData]

    static var defaultValue: [MyTextPreferenceData] = []
    
    static func reduce(value: inout [MyTextPreferenceData], nextValue: () -> [MyTextPreferenceData]) {
        value.append(contentsOf: nextValue())
    }
}
```

现在有了`PreferenceKey`了，开始对之前的代码就行修改。

```swift
struct MonthView: View {
    @Binding var activeMonth: Int
    let label: String
    let idx: Int
    
    var body: some View {
        Text(label)
            .padding(10)
            .background(MyPreferenceViewSetter(idx: idx)).onTapGesture { self.activeMonth = self.idx }
    }
}

struct MyPreferenceViewSetter: View {
    let idx: Int
    
    var body: some View {
        GeometryReader { geometry in
            Rectangle()
                .fill(Color.clear)
                .preference(key: MyTextPreferenceKey.self,
                            value: [MyTextPreferenceData(viewIdx: self.idx, rect: geometry.frame(in: .named("myZstack")))])
        }
    }
}


struct ContentView : View {
    
    @State private var activeIdx: Int = 0
    @State private var rects: [CGRect] = Array<CGRect>(repeating: CGRect(), count: 12)
    
    var body: some View {
        ZStack(alignment: .topLeading) {
        /// 创建一个单独的边框视图，该视图将更改其偏移量和frame以匹配与最后点击的视图相对应的矩形
            RoundedRectangle(cornerRadius: 15).stroke(lineWidth: 3.0).foregroundColor(Color.green)
                .frame(width: rects[activeIdx].size.width, height: rects[activeIdx].size.height)
                .offset(x: rects[activeIdx].minX, y: rects[activeIdx].minY)
                .animation(.easeInOut(duration: 1.0))
            
            VStack {
                Spacer()
                
                HStack {
                    MonthView(activeMonth: $activeIdx, label: "January", idx: 0)
                    MonthView(activeMonth: $activeIdx, label: "February", idx: 1)
                    MonthView(activeMonth: $activeIdx, label: "March", idx: 2)
                    MonthView(activeMonth: $activeIdx, label: "April", idx: 3)
                }
                
                Spacer()
                
                HStack {
                    MonthView(activeMonth: $activeIdx, label: "May", idx: 4)
                    MonthView(activeMonth: $activeIdx, label: "June", idx: 5)
                    MonthView(activeMonth: $activeIdx, label: "July", idx: 6)
                    MonthView(activeMonth: $activeIdx, label: "August", idx: 7)
                }
                
                Spacer()
                
                HStack {
                    MonthView(activeMonth: $activeIdx, label: "September", idx: 8)
                    MonthView(activeMonth: $activeIdx, label: "October", idx: 9)
                    MonthView(activeMonth: $activeIdx, label: "November", idx: 10)
                    MonthView(activeMonth: $activeIdx, label: "December", idx: 11)
                }
                
                Spacer()
                }.onPreferenceChange(MyTextPreferenceKey.self) { preferences in
                    for p in preferences {
                        self.rects[p.viewIdx] = p.rect
                    }
            }
        }.coordinateSpace(name: "myZstack")
    }
}
```

当设备旋转，或者window的大小改变， 下面的代码都会被调用：

```swift
.onPreferenceChange(MyTextPreferenceKey.self) { preferences in
    for p in preferences {
        self.rects[p.viewIdx] = p.rect
    }
}
```

## 明智地使用 Preference

当我们使用preferences，可能会使用子级视图的几何信息来布局它们的一个顶层视图(ancestors)，如果是这样的话，你应该注意。 如果顶层视图影响了子级视图的布局，反过来子级视图也会影响顶层视图，就会陷入一个递归循环中。

可能有时候程序会卡死，或者屏幕会闪动来持续的重新绘制。或者CPU会达到一个峰值，这些都会暗示你错误的使用了preferences。