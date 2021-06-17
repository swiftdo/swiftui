# List

> 文档：[https://developer.apple.com/documentation/swiftui/list](https://developer.apple.com/documentation/swiftui/list)

用于显示排列一系列数据行的容器。

创建一个静态可滚动列表：

```swift
List {
    Text("Hello world")
    Text("Hello world")
    Text("Hello world")
}
```

表单里的内容可以混搭：

```swift
List {
    Text("Hello world")
    Image(systemName: "clock")
}
```

创建一个动态列表：

```swift
let names = ["John", "Apple", "Seed"]
List(names) { name in
    Text(name)
}
```

加入分区：

```swift
List {
    Section(header: Text("UIKit"), footer: Text("We will miss you")) {
        Text("UITableView")
    }

    Section(header: Text("SwiftUI"), footer: Text("A lot to learn")) {
        Text("List")
    }
}
```

要使其成为分组列表，请添加 `.listStyle(GroupedListStyle())`：

```swift
List {
    Section(header: Text("UIKit"), footer: Text("We will miss you")) {
        Text("UITableView")
    }

    Section(header: Text("SwiftUI"), footer: Text("A lot to learn")) {
        Text("List")
    }
}.listStyle(GroupedListStyle())

```