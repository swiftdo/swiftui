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


## 待办清单添加创建功能

下面我们将一起来实现点击创建按钮的时候，弹出一个页面进行新增待办事项。

首先我们在代码清单项目底部添加个添加按钮：

```swift
HStack {
    Button("添加待办事项") {
        popoverIsShow = true
    }
    .padding(.leading)    
    Spacer()
}
.frame(height: 44)
```

![](http://blog.loveli.site/mweb/16267701052795.jpg)

然后我们创建个 `AddPage` 页面：

```swift
import SwiftUI

struct AddPage: View {
    var body: some View {
        Text("Hello, OldBirds!")
    }
}

struct AddPage_Previews: PreviewProvider {
    static var previews: some View {
        AddPage()
    }
}
```

我们给`添加待办事项`按钮添加点击事件，点击的时候弹出`AddPage`：

![](http://blog.loveli.site/mweb/16268265284520.jpg)

通过 `popoverIsShow` 控制页面的弹出与否，我们一起看下跳转效果：

![hello-swiftui-pop-view](http://blog.loveli.site/mweb/hello-swiftui-pop-view.gif)

实现跳转后，我们需要完善 `AddPage` 页面，这里将简单的实现一个输入框用来输入 task。

```swift
struct AddPage: View {
    @Binding var isPresented: Bool
    @State var task: String = ""
    var addItem: (String)->Void

    var body: some View {
        NavigationView {
            VStack {
                HStack (alignment: .center){
                    Button("取消") {
                        isPresented = false
                    }.padding()

                    Spacer()

                    Text("添加待办事项")
                        .padding()

                    Spacer()

                    Button("添加") {
                        if task.count > 0 {
                            isPresented = false
                            addItem(task)
                        }
                    }
                    .foregroundColor(task.count > 0 ? Color.blue: Color.gray )
                    .padding()

                }.frame(height: 44)

                VStack {
                    Group {
                        TextField("标题", text: $task).padding()
                    }
                    .background(Color.white)
                    .cornerRadius(10)
                    .padding()
                }
                Spacer()
            }
            .edgesIgnoringSafeArea([.bottom])
            .background(Color("pageBgColor").edgesIgnoringSafeArea(.all))
            .navigationBarHidden(true)
        }
    }
}
```

![](http://blog.loveli.site/mweb/16268273594679.jpg)

在 AddPage 中，点击取消，我们修改 `isPresented = false` 即可，需要外部传递。可以在输入按钮中输入待办事项的名字。当待办事项的名字的长度大于0，我们将添加按钮进行高亮，点击高亮关闭页面，且回调 `addItem`，将 task 回传给调用方。

在 `MainPage` 中，我们修改代码如下：

![](http://blog.loveli.site/mweb/16268276601714.jpg)

将 addItem 中我们创建了一个新的 `TodoItem`，然后追加到 listData 中。

我们一起来看下最终的运行效果。

![hello-swiftui-add2-view](http://blog.loveli.site/mweb/hello-swiftui-add2-view.gif)

## 最终的代码如下

### MainPage.swift

```swift

import SwiftUI

struct TodoItem: Identifiable {
    var id: UUID = UUID()
    var task: String
    var imgName: String
}

struct MainPage: View {
    @State var isEditMode: EditMode = .inactive
    @State private var popoverIsShow: Bool = false
    
    @State var listData: [TodoItem] = [
        TodoItem(task: "写一篇SwiftUI文章", imgName: "pencil.circle"),
        TodoItem(task: "看WWDC视频", imgName: "square.and.pencil"),
        TodoItem(task: "订外卖", imgName: "folder"),
        TodoItem(task: "关注OldBirds公众号", imgName: "link"),
        TodoItem(task: "6点半跑步2公里", imgName: "moon"),
    ]
    
    var body: some View {
        NavigationView {
            VStack {
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
                .toolbar {
                    Button(isEditMode.isEditing ? "完成": "编辑") {
                        switch isEditMode {
                        case .active:
                            self.isEditMode = .inactive
                        case .inactive:
                            self.isEditMode = .active
                        default:
                            break
                        }
                    }
                }
                .environment(\.editMode, $isEditMode)
                .listStyle(GroupedListStyle())
                
                HStack {
                    Button("添加待办事项") {
                        popoverIsShow = true
                    }
                    .padding(.leading)
                    .sheet(isPresented: $popoverIsShow) {
                    } content: {
                        AddPage(isPresented: $popoverIsShow, addItem: addItem)
                    }
                    
                    Spacer()
                }
                .frame(height: 44)
            }
            .navigationBarTitle(Text("待办清单"))
        }
        .navigationViewStyle(.stack)
    }
    
    /// 添加
    func addItem(task: String) {
        listData.append(TodoItem(task: task, imgName: "circle"))
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


struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        MainPage()
    }
}
```

### AddPage.swift

```swift
//
//  AddPage.swift
//  swiftui-demo
//
//  Created by mac on 2021/7/20.
//

import SwiftUI

struct AddPage: View {
    
    @Binding var isPresented: Bool
    
    @State var task: String = ""

    var addItem: (String)->Void

    var body: some View {
        NavigationView {
            VStack {
                HStack (alignment: .center){
                    Button("取消") {
                        isPresented = false
                    }.padding()

                    Spacer()

                    Text("添加待办事项")
                        .padding()

                    Spacer()

                    Button("添加") {
                        if task.count > 0 {
                            isPresented = false
                            addItem(task)
                        }
                    }
                    .foregroundColor(task.count > 0 ? Color.blue: Color.gray )
                    .padding()

                }.frame(height: 44)

                VStack {
                    Group {
                        TextField("标题", text: $task).padding()
                    }
                    .background(Color.white)
                    .cornerRadius(10)
                    .padding()
                }
                Spacer()
            }
            .edgesIgnoringSafeArea([.bottom])
            .background(Color("pageBgColor").edgesIgnoringSafeArea(.all))
            .navigationBarHidden(true)
        }
    }
}

struct AddPage_Previews: PreviewProvider {
    static var previews: some View {
        AddPage(isPresented: .constant(true)) { value in }
    }
}
```
