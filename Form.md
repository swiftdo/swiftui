# Form

> 文档：[https://developer.apple.com/documentation/swiftui/form](https://developer.apple.com/documentation/swiftui/form)

对数据输入的控件进行分组的容器，例如在设置或检查器中。

您可以往表单中插入任何内容，它将为表单渲染适当的样式。

```swift
NavigationView {
    Form {
        Section {
            Text("Plain Text")
            Stepper(value: $quantity, in: 0...10, label: { Text("Quantity") })
        }
        Section {
            DatePicker($date, label: { Text("Due Date") })
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
