
# ForEach

> 官方文档：[https://developer.apple.com/documentation/swiftui/foreach](https://developer.apple.com/documentation/swiftui/foreach)

```swift
public struct ForEach<Data, ID, Content> where Data : RandomAccessCollection, ID : Hashable {
    public var data: Data
    public var content: (Data.Element) -> Content
}
```

在待办清单案例中，我们有用到`ForEach`这个视图。ForEach 将集合的东西生成一个个 View，然后再组合成一个 View。大大简化了相似代码，同时也解决了`ViewBuilder`参数个数限制问题。它在`SwiftUI`开发中非常常见，所以今天一起深入挖掘下 `ForEach` 相关知识。

待办清单案例中涉及`ForEach`使用的代码如下：

```swift
/// 需要实现 Identifiable 协议
struct TodoItem: Identifiable {
    var id: UUID = UUID()
    var task: String
    var imgName: String
}

@State var listData: [TodoItem] = [
    TodoItem(task: "写一篇SwiftUI文章", imgName: "pencil.circle"),
    TodoItem(task: "看WWDC视频", imgName: "square.and.pencil"),
    TodoItem(task: "定外卖", imgName: "folder"),
    TodoItem(task: "关注OldBirds公众号", imgName: "link"),
    TodoItem(task: "6点半跑步2公里", imgName: "moon"),
]
    
var body: some View {
    NavigationView {
        List {
            Section(header: Text("待办事项")) {
                ForEach(listData) { item in
                    HStack{
                        Image(systemName: item.imgName)
                        Text(item.task)
                    }
                }
                .onDelete(perform: deleteItem)
                .onMove(perform: moveItem)
            }
            Section(header: Text("其他内容")) {
                Text("Hello World")
            }
        }
        .listStyle(GroupedListStyle())
        .navigationTitle(Text("待办清单"))
    }
}
```

我们将围绕下面两个问题，进行对 `ForEach` 的深入了解：

1. `ForEach` 有哪些初始化方法？
2. `TodoItem` 为什么要实现`Identifiable` 协议？

## `ForEach` 的初始化方法

ForEach 是一个从元素可识别的数据集合中创建视图。它支持三种类型初始化方式：
* 通过 range 迭代
* 对由实现 `Identifiable` 的元素组成的集合迭代
* 对由可被标志，但没有实现 `Identifiable` 的元素组成的集合迭代

### range

这个初始化器是 `ForEach` 中最简单的。它类似于常见的`for`循环：

```swift
struct ContentView: View {
    
    let data = ["写一篇SwiftUI文章",
                "看WWDC视频",
                "订外卖",
                "关注OldBirds公众号",
                "6点半跑步2公里",]

    var body: some View {
        List {
            ForEach(0..<data.count) { index in
                Text(data[index])
            }
        }
    }
}
```

![-w1200](http://blog.loveli.site/mweb/16262493140464.jpg)

但是请注意，此视图仅呈现一次。因此，如果`data.count`更改，视图将不会更新。例如，单击以下代码中的`添加待办事项`会返回错误：

```swift
struct ContentView: View {
    
    @State var data = ["写一篇SwiftUI文章",
                "看WWDC视频",
                "订外卖",
                "关注OldBirds公众号",
                "6点半跑步2公里",]

    var body: some View {
        VStack {
            List {
                ForEach(0..<data.count) { index in
                    Text(data[index])
                }
            }
            Button("添加待办事项") {
                addTodo()
            }.padding()
        }
    }
    
    func addTodo() {
        data.append("Hello SwiftUI")
    }
}
```

![-w1283](http://blog.loveli.site/mweb/16262505795215.jpg)

点击按钮，列表没有反应。我们然后将代码运行到模拟器，点击按钮，在控制台输出如下错误：

```sh
ForEach<Range<Int>, Int, Text> count (6) != its initial count (5). `ForEach(_:content:)` should only be used for *constant* data. Instead conform data to `Identifiable` or use `ForEach(_:id:content:)` and provide an explicit `id`!
```

从报错中，我们知道**`ForEach(_:content:)`只能使用常量数据**。

### Identifiable

在待办清单案例中，我们直接传递数组 listData 就可以了

![](http://blog.loveli.site/mweb/16262510185884.jpg)

如果我们将`TodoItem`不实现`Identifiable`，看看会发生什么？

![-w1208](http://blog.loveli.site/mweb/16262511281419.jpg)

从报错信息可知，我们必须要将数组的元素实现`Identifiable`协议。

数组的元素如果是普通类型，比如`String`也会报错：

![-w628](http://blog.loveli.site/mweb/16262515237081.jpg)

需要通过指定`id: \.self`即可，因为它们本身可以作为可识别的对象。

![](http://blog.loveli.site/mweb/16262527504186.jpg)

下面我们详细说下`id`。

### id & Hashable

```swift
public init(_ data: Data, id: KeyPath<Data.Element, ID>, @ViewBuilder content: @escaping (Data.Element) -> Content)
```

id 要求我们传入的是一个 `KeyPath`，且`ID`必须实现`Hashable`。而我们常见的数据类型 `Int/String/...`都默认实现`Hashable`。

通过指定 `id` 的初始化方式适用于元素没有实现 `Identifiable` 的场景。

如果`TodoItem`没有实现`Identifiable`，我们也可以这样做：

![](http://blog.loveli.site/mweb/16262535194278.jpg)

但是我们得确保task的唯一性，为什么呢？

## `TodoItem` 为啥要实现 Identifiable?

实现`Identifiable`的目的是为了要区分数组的成员，这样可以更有效率地管理 List 里的视图。

我们简单通过样例来说明下其中的原理：

```swift
struct TodoItem {
    var task: String
    var imgName: String
}

struct ContentView: View {
    @State var listData: [TodoItem] = [
        TodoItem(task: "写一篇SwiftUI文章", imgName: "pencil.circle"),
        TodoItem(task: "看WWDC视频", imgName: "square.and.pencil"),
        TodoItem(task: "定外卖", imgName: "folder"),
        TodoItem(task: "关注OldBirds公众号", imgName: "link"),
        TodoItem(task: "6点半跑步2公里", imgName: "moon"),
    ]
    
    var body: some View {
        VStack {
            List {
                ForEach(listData, id: \.task) { item in
                    HStack{
                        Image(systemName: item.imgName)
                        Text(item.task)
                    }
                }
            }
            Button("添加待办事项") {
                addTodo()
            }.padding()
        }
    }
    
    func addTodo() {
        listData.append(TodoItem(task: "写一篇SwiftUI文章", imgName: "moon"))
    }
}
```

![-w1365](http://blog.loveli.site/mweb/16262557143716.jpg)


我们让`TodoItem`不实现`Identifiable`, 用`\.task`传递给 id。我们希望点击添加按钮的时候，列表会同步刷新，且新增的图标是个`月亮`。但是事情并没有顺从你的心意：

![hello-swiftui](http://blog.loveli.site/mweb/hello-swiftui.gif)

你应该也发现了，新增的视图的 icon 并不是月亮。而是用了第一个`TodoItem(task: "写一篇SwiftUI文章", imgName: "pencil.circle")`的 `pencil.circle`，其实不仅仅是图标，整个显示都跟第一个视图一样的。因为无法区分这两个 TodoItem，所以就以最开始出现的视图重复，发生了错乱。

## 总结

ForEach 的使用还是比较简单的，有三种初始化方式，通过range 迭代的时候，记得保证集合是个常量。第二种通过 Identifiable 保证元素唯一，通常做法用UUID 作为id。第三种，通过指定KeyPath，需要保证实现了 Hashable，且最好做到唯一，不然显示会错乱。











