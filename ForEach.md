# ForEach

在待办清单案例中，我们有用到`ForEach`这个视图。ForEach 将集合的东西生成一个个 View，然后再组合成一个 View。大大简化了相似代码，同时也解决了`ViewBuilder`参数个数限制问题。它在`SwiftUI`开发中非常常见，所以今天一起深入挖掘下 `ForEach`。

> 官方文档：[https://developer.apple.com/documentation/swiftui/foreach](https://developer.apple.com/documentation/swiftui/foreach)


在待办清单案例中，涉及`ForEach`使用的代码如下：

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

1. `ForEach` 有哪些使用方法？
2. `TodoItem`为啥要实现`Identifiable` 协议？

## `ForEach` 的使用方法

## `TodoItem`为啥要实现 Identifiable?
