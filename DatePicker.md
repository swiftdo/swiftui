# DatePicker

> 文档：[https://developer.apple.com/documentation/swiftui/datepicker](https://developer.apple.com/documentation/swiftui/datepicker)

选择日期的控件。

日期选择器样式也会根据其父视图进行更改，在表单或列表下作为一个列表行显示，点击可以扩展到日期选择器（就像日历 App 一样）

```swift
@State var selectedDate = Date()

var dateClosedRange: ClosedRange<Date> {
    let min = Calendar.current.date(byAdding: .day, value: -1, to: Date())!
    let max = Calendar.current.date(byAdding: .day, value: 1, to: Date())!
    return min...max
}

NavigationView {
    Form {
        Section {
            DatePicker(
                selection: $selectedDate,
                in: dateClosedRange,
                displayedComponents: .date,
                label: { Text("Due Date") }
            )
        }
    }
}
```

不在表单或列表里，它就可以作为普通的旋转选择器。

```swift
@State var selectedDate = Date()

var dateClosedRange: ClosedRange<Date> {
    let min = Calendar.current.date(byAdding: .day, value: -1, to: Date())!
    let max = Calendar.current.date(byAdding: .day, value: 1, to: Date())!
    return min...max
}

DatePicker(
    selection: $selectedDate,
    in: dateClosedRange,
    displayedComponents: [.hourAndMinute, .date],
    label: { Text("Due Date") }
)
```
