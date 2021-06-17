# NavigationView

> 文档：[https://developer.apple.com/documentation/swiftui/navigationview](https://developer.apple.com/documentation/swiftui/navigationview)

用于渲染视图堆栈的视图，这些视图会展示导航层次结构中的可见路径。

```swift
NavigationView {
    List {
        Text("Hello World")
    }
    .navigationBarTitle(Text("Navigation Title")) // 默认使用大标题样式
}
```

对于旧样式标题：

```swift
NavigationView {
    List {
        Text("Hello World")
    }
    .navigationBarTitle(Text("Navigation Title"), displayMode: .inline)
}
```

增加 UIBarButtonItem：

```swift
NavigationView {
    List {
        Text("Hello World")
    }
    .navigationBarItems(trailing:
        Button(action: {
            // Add action
        }, label: {
            Text("Add")
        })
    )
    .navigationBarTitle(Text("Navigation Title"))
}
```

作为 `UISplitViewController`：

```swift
NavigationView {
    List {
        NavigationLink("Go to detail", destination: Text("New Detail"))
    }.navigationBarTitle("Master")
    Text("Placeholder for Detail")
}
```