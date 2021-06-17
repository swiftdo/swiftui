# ActionSheet

> 文档：[https://developer.apple.com/documentation/swiftui/actionsheet](https://developer.apple.com/documentation/swiftui/actionsheet)

操作表视图的存储类型。

我们可以根据布尔值显示 `ActionSheet`。

```swift
@State var isSheet: Bool = false

var actionSheet: ActionSheet {
    ActionSheet(title: Text("Action"),
                message: Text("Description"),
                buttons: [
                    .default(Text("OK"), action: {
                        
                    }),
                    .destructive(Text("Delete"), action: {
                        
                    })
                ]
    )
}

Button("Action Sheet") {
    self.isSheet = true
}.actionSheet(isPresented: $isSheet, content: {
    self.actionSheet
})
```

它也可与 `Identifiable` 项目绑定。

```swift
@State var sheetDetail: SheetDetail?

var body: some View {
    Button("Action Sheet") {
        self.sheetDetail = ModSheetDetail(body: "Detail")
    }.actionSheet(item: $sheetDetail, content: { detail in
        self.sheet(detail: detail.body)
    })
}

func sheet(detail: String) -> ActionSheet {
    ActionSheet(title: Text("Action"),
                message: Text(detail),
                buttons: [
                    .default(Text("OK"), action: {
                        
                    }),
                    .destructive(Text("Delete"), action: {
                        
                    })
                ]
    )
}

struct SheetDetail: Identifiable {
    var id: String {
        return body
    }
    
    let body: String
}
```