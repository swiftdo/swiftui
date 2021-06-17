# Picker


> 文档：[https://developer.apple.com/documentation/swiftui/picker](https://developer.apple.com/documentation/swiftui/picker)

从一组互斥值中进行选择的控件。

选择器样式根据其被父视图进行更改，在表单或列表下作为一个列表行显示，点击可以推出新界面展示所有的选项卡。

```swift
NavigationView {
    Form {
        Section {
            Picker(selection: $selection, label:
                Text("Picker Name")
                , content: {
                    Text("Value 1").tag(0)
                    Text("Value 2").tag(1)
                    Text("Value 3").tag(2)
                    Text("Value 4").tag(3)
            })
        }
    }
}
```
