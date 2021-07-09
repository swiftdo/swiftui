# List

> 文档：[https://developer.apple.com/documentation/swiftui/list](https://developer.apple.com/documentation/swiftui/list)

用于显示排列一系列数据行的容器。

## 创建一个静态列表

```swift
struct LearnPage: View {
    var body: some View {
        List {
            Text("写一篇SwiftUI文章")
            Text("看WWDC视频")
            Text("定外卖")
            Text("关注OldBirds公众号")
            Text("6点半跑步2公里")
        }
    }
}
```

![-w1060](http://blog.loveli.site/mweb/16258380288707.jpg)


List 可以是单个组件，也可以是组合组件，表单里的内容是可以混搭的。以下示例中列表的前几行都由图像和文本组件组成的 `HStack`，最后一行是单个 `Text`：

```swift
struct LearnPage: View {
    var body: some View {
        List {
            HStack {
                Image(systemName: "pencil.circle")
                Text("写一篇SwiftUI文章")
            }
            HStack {
                Image(systemName: "square.and.pencil")
                Text("看WWDC视频")
            }
            HStack {
                Image(systemName: "folder")
                Text("定外卖")
            }
            HStack {
                Image(systemName: "link")
                Text("关注OldBirds公众号")
            }
            HStack {
                Image(systemName: "moon")
                Text("6点半跑步2公里")
            }
            Text("Hello World")
        }
    }
}
```
![-w1065](http://blog.loveli.site/mweb/16258391049764.jpg)


## 创建一个动态列表

动态列表可以根据数据的元素进行展示，当数据进行添加、编辑、删除时，列表会动态更新以反应这些变化。

我们将上面的静态列表中的数据，使用一组 `TodoItem` 对象来模拟向列表提供数据。

```swift
/// 需要实现 Identifiable 协议
struct TodoItem: Identifiable {
    var id: UUID = UUID()
    var task: String
    var imgName: String
}

struct LearnPage: View {
    
    var listData: [TodoItem] = [
        TodoItem(task: "写一篇SwiftUI文章", imgName: "pencil.circle"),
        TodoItem(task: "看WWDC视频", imgName: "square.and.pencil"),
        TodoItem(task: "定外卖", imgName: "folder"),
        TodoItem(task: "关注OldBirds公众号", imgName: "link"),
        TodoItem(task: "6点半跑步2公里", imgName: "moon"),
    ]
    
    var body: some View {
        List(listData) { item in
            HStack{
                Image(systemName: item.imgName)
                Text(item.task)
            }
        }
    }
}
```

通过上面的改动，代码变得更加清晰了，不会出现重复的代码。但是 `Text("Hello World")` 该如何加入到列表中么？

我们可以借助 `ForEach` 去实现：

```swift
var body: some View {
    List {
        ForEach(listData) { item in
            HStack{
                Image(systemName: item.imgName)
                Text(item.task)
            }
        }
        Text("Hello World")
    }
}
```

![-w1069](http://blog.loveli.site/mweb/16258424295348.jpg)

## 加入分区

写到这里，我们已经完成了一个动态列表了，但是略显丑陋。我们可以对它进行美化一下，首先将其进行分组，将同类为一组，这种需求还是比较常见的。

结合 `Section`，我们的代码有如下变化：

```swift
var body: some View {
    List {
        Section(header: Text("待办事项")) {
            ForEach(listData) { item in
                HStack{
                    Image(systemName: item.imgName)
                    Text(item.task)
                }
            }
        }
        
        Section(header: Text("其他内容")) {
            Text("Hello World")
        }
    }.listStyle(GroupedListStyle())
}
```

![-w1088](http://blog.loveli.site/mweb/16258429083232.jpg)

如果还想进一步美化页面，我们可以给页面添加标题：

```swift
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

![-w1091](http://blog.loveli.site/mweb/16258431849466.jpg)

## 让列表可编辑

为了进一步提现列表的动态性，打算添加待办清单的侧滑删除、移动功能。那么如何实现？

SwiftUI 提供了非常便利的方案：

```swift
struct LearnPage: View {
    
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
    
    /// 移动
    func moveItem(from source: IndexSet, to destination: Int) {
        listData.move(fromOffsets: source, toOffset: destination)
    }
    
    /// 删除
    func deleteItem(at offsets: IndexSet) {
        listData.remove(atOffsets: offsets)
    }
}
```

![-w1048](http://blog.loveli.site/mweb/16258445771255.jpg)

* 将 listData 添加 @State 装饰，这样数据变化，会驱动视图更新
* 添加 `.onDelete`，收到已删除项目的索引，从数组中删除相应的元素
* 添加 `.onMove`，收到移动项目的源和目标索引，相应地重新排序数组


