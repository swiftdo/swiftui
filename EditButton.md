# EditButton

> 文档：[https://developer.apple.com/documentation/swiftui/editbutton](https://developer.apple.com/documentation/swiftui/editbutton)

负责控制打开和关闭编辑模式。这个编辑模式需要和[NavigationView](NavigationView.md)相结合。

在 [List](List.md) 的文章中，我们有写一个待完成清单的案例，本文将继续沿用。

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

在这个案例中，我们发现 EditButton 显示的是英文，那么按钮有办法显示为中文标题么？

首先我们看下 `EditButton` 的代码定义：

```swift
public struct EditButton : View {
    public init()
    public var body: some View { get }
    public typealias Body = some View
}
```

从它构造方法中，我们知道它是不需要传递参数的。同时也没有任何可以修改 title的方法提供。

> EditButton 的标题和外观由系统决定，不能被覆盖。

那么还有哪些操作可以控制 `EditButton`?

## 控制显示位置

使用 `.navigationBarItems` 可以控制`EditButton`是在左还是右：

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
        .navigationBarItems(leading: EditButton())
        .listStyle(GroupedListStyle())
        .navigationTitle(Text("待办清单"))
    }
}
```











