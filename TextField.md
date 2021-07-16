# TextField

> 官方文档: [https://developer.apple.com/documentation/swiftui/textfield](https://developer.apple.com/documentation/swiftui/textfield)

`TextField`类似于`UIKit`中的`UITextField`，用于实现用户的文字内容的输入。

示例代码：

```swift
import SwiftUI

struct TextFieldPage: View {
    @State var content: String
    var body: some View {
        VStack {
            Text("你说：\(content)")
            
            TextField("输入",
                      text: $content,
                      onEditingChanged: { isEditing in
                        print("onEditingChanged::\(content)")
                      },
                      onCommit: {
                        print("onCommit::\(content)")
                      })
                .textFieldStyle(.roundedBorder)
                .padding()
        
        }
    }
}

struct TextFieldPage_Previews: PreviewProvider {
    static var previews: some View {
        TextFieldPage(content: "")
    }
}
```

![hello-swiftui-textfield2](http://blog.loveli.site/mweb/hello-swiftui-textfield2.gif)


 `TextField`的`init`参数说明：
 
 * **onEditingChanged**：在编辑状态改变时触发，在游标出现在 TextFiled，或游标离开 TextFiled 时，onEditingChanged 将触发执行。
 * **onCommit**：在结束编辑时执行，当我们按下 enter 键收键盘后，onCommit 将触发执行。


## 待办清单创建功能

