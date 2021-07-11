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

![-w1060](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/87728e8f02aa4886aaed9cb29813fca8~tplv-k3u1fbpfcp-zoom-1.image)


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
![-w1065](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2fabd5119bbb452aa58c23c821d928a6~tplv-k3u1fbpfcp-zoom-1.image)


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

![-w1069](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b21b0575220c4a3983c2ae492f5d830a~tplv-k3u1fbpfcp-zoom-1.image)

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

![-w1088](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d2eb03cd7bd4986af9088aa7d047cc9~tplv-k3u1fbpfcp-zoom-1.image)

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

![-w1091](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0803632df5e84324b12c0e7364ddc40c~tplv-k3u1fbpfcp-zoom-1.image)

## 让列表可编辑

为了进一步体现列表的动态性，打算添加待办清单的侧滑删除、移动功能。那么如何实现？

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

![-w1048](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e829891bb9b847c1bdd1f4fc77ab54c5~tplv-k3u1fbpfcp-zoom-1.image)

* 将 listData 添加 @State 装饰，这样数据变化，会驱动视图更新
* 添加 `.onDelete`，收到已删除项目的索引，从数组中删除相应的元素
* 添加 `.onMove`，收到移动项目的源和目标索引，相应地重新排序数组

运行代码，我们会发现一个问题，侧滑可以删除，但是项目移动无法触发。那是因为只有当用户进入**编辑模式**时才能移动项目。所以我们需要在导航栏中添加一个 `EditButton`。当用户点击 `Edit` 时，才可以继续移动项目：

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
              .onDelete(perform: deleteItem)
              .onMove(perform: moveItem)
          }
          Section(header: Text("其他内容")) {
              Text("Hello World")
          }
      }
      .toolbar { EditButton() } // 这里实现添加
      .listStyle(GroupedListStyle())
      .navigationTitle(Text("待办清单"))
  }
}
```

![111ee0710](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42186b81c8354617935ad56d81d9bc8f~tplv-k3u1fbpfcp-zoom-1.image)

