
## @State

通过使用 `@State` 修饰器我们可以关联出 `View` 的状态. SwiftUI 将会把使用过 @State 修饰器的属性存储到一个特殊的内存区域，并且这个区域和 View struct 是隔离的. 当 @State 装饰过的属性发生了变化，SwiftUI 会根据新的属性值重新创建视图

> 被设计用于存储当前视图的本地数据，支持对值类型的修饰，不适用于复杂的引用类型。
> 
> `@State` 用于 `View` 中的私有状态值，一般来说它所修饰的都应该是 `struct` 值，并且不应该被其他的 `view` 看到。它代表了 `SwiftUI` 中作用范围最小，本身也最简单的状态，比如一个 Bool，一个 Int 或者一个 String。简单说，如果一个状态能够被标记为 private 并且它是值类型，那么 `@State` 是适合的

```swift
struct ProductsView: View {
    let products: [Product]
    @State private var showFavorited: Bool = false
    var body: some View {
        List {
            Button("授权") {
                showFavorited.toggle()
            }

            ForEach(products) { product in
                if !showFavorited || product.isFavorited {
                    Text(product.title)
                }
            }
        }
    }
}
```

这个例子里我们创建了一个列表，点击按钮 `showFavorited` 会发生值的取反操作，然后 `SwiftUI` 会通过最新的值更新值。

## @Binding

有时候我们会把一个视图的属性传至子节点中，但是又不能直接的传递给子节点，因为在 Swift 中值的传递形式是值类型传递方式，也就是传递给子节点的是一个拷贝过的值。

> 如果我们有两个 SwiftUI 的视图，并且我们用相同的结构体实例赋给它们，它们实际上是各自拥有一份唯一的结构体拷贝；如果其中一个改变，另外一个并不会随着改变。另一方面，如果我们创建一个类实例，赋给两个视图，它们会共享改变。
> 对于 SwiftUI 开发者，这意味着如果我们想在多个视图之间共享数据，或者说让两个或者更多视图引用相同的数据，以便一个改变，全部跟随改变 —— 这种情况下我们需要用类而不是结构体。
> ```swift
> struct User {
>     var firstName = "Bilbo"
>     var lastName = "Baggins"
> }
> struct ContentView: View {
>     @State private var user = User()
>     var body: some View {
>         VStack {
>             Text("Your name is \(user.firstName) \(user.lastName).")
>             TextField("First name", text: $user.firstName)
>             TextField("Last name", text: $user.lastName)
>         }
>     }
> }
> ```
> 所以，我们是不是可以把 User 结构体改成一个类，把下面的代码：
> ```swift
> struct User {
> ```
> 改成这样：
> ```swift
> class User {
> ```
> 现在运行代码，看看会发生什么？
> app 无法正常工作。 当我们像之前那样往文本框里输入字符串时，文本视图不再改变了。这是为什么？
> 当我们使用`@State`的时候，我们是在要求`SwiftUI`为我们监视某个属性的变化。这样让我们改变一个字符串，反转一个布尔型，或者往数组里加东西的时候，属性会变化，而 `SwiftUI` 会重新调用视图的`body`属性。
> 当`User`还是一个结构体的时候，每当我们修改它的属性时，`Swift` 实际上创建了一个新的结构体实例。`@State`能够看穿这种变化，并自动重新载入视图。现在我们把它改成类，这种行为不再发生：因为`Swift`能够直接修改目标对象的值 —— 没有新实例产生。


但是通过 `@Binding` 修饰器修饰后，属性变成了一个引用类型，传递变成了引用传递，这样父子视图的状态就能关联起来了。

```swift

struct FilterView: View {
    @Binding var showFavorited: Bool
    
    var body: some View {
        Toggle(isOn: $showFavorited, label: {
            Text("toggle")
        }).padding(30)
    }
}

struct Learn: View {
    @State private var showFavorited: Bool = false
    
    var body: some View {
        NavigationView(content: {
            VStack(content: {
                FilterView(showFavorited: $showFavorited)
                    .navigationTitle(Text("Learn"))
                
                if showFavorited {
                    Text("OK").font(.title)
                } else {
                    Text("Bad").font(.title)
                }
            })
        })
    }
}
```
在 `FilterView` 视图里用 `@Binding` 修饰 `showFavorited` 属性, 在传递属性是使用 `$` 来传递 `showFavorited` 属性的引用，这样 `FilterView` 视图就能读写父视图 `ProductsView` 里的状态值了，并且值发生了修改 `SwiftUI` 会更新 `ProductsView` 和 `FilterView` 视图。


## @ObservedObject

> 在应用开发过程中，很多数据其实并不是在`View`内部产生的，这些数据有可能是一些本地存储的数据，也有可能是网络请求的数据，这些数据默认是与`SwiftUI`没有依赖关系的，要想建立依赖关系可以使用`ObservableObject`

对于更复杂的一组状态，我们可以将它组织在一个`class`中，并让其实现 `ObservableObject` 协议。对于这样的`class`类型，其中被标记为`@Published`的属性，将会在变更时自动发出事件，通知对它有依赖的`View`进行更新。`View`中如果需要依赖这样的`ObservableObject`对象，在声明时则使用`@ObservedObject`来订阅。

```swift
class BookingStore: ObservableObject {
    @Published var bookingName = ""
    @Published var seats: Int = 1
}

struct Mine: View {
    @ObservedObject var model = BookingStore()
    var body: some View {
        NavigationView(content: {
            VStack{
                Text(model.bookingName)
                TextField("Your Name", text: $model.bookingName)
                Stepper("Seats: \(model.seats)", value: $model.seats, in: 1...5)
            }.navigationTitle(Text("Mine"))
        })
    }
}
```

`@Published` 是`Xcode11 beta5`之后新增的代理属性，此属性如果用在 `ObservableObject`内，一旦修饰的属性发送了变化，会自动触发 `ObservableObject` 的 `objectWillChange` 的 `send`方法，刷新页面，`SwiftUI`已经默认帮我实现好了，但也可以自己手动出发这发这个行为。

```swift
class BookingStore: ObservableObject {
    var objectWillChange = PassthroughSubject<Void, Never>()
    
    var bookingName: String = "" {
        didSet { updateUI() }
    }
    
    var seats: Int = 1 {
        didSet {
            updateUI()
        }
    }
    
    func updateUI() {
        objectWillChange.send()
    }
}

struct Mine: View {
    @ObservedObject var model = BookingStore()
    var body: some View {
        NavigationView(content: {
            VStack{
                Text(model.bookingName)
                TextField("Your Name", text: $model.bookingName)
                Stepper("Seats: \(model.seats)", value: $model.seats, in: 1...5)
            }.navigationTitle(Text("Mine"))
        })
    }
}
```

> 是不是跟 Flutter 的 Provider 中，定义模型`ChangeNotifier`比较相像。

## @EnvironmentObject

这个修饰器是针对全局环境的。通过它，我们可以避免在初始`View`时创建 `ObservableObject`, 而是从 `Environment` 中获取 `ObservableObject`。

```swift
class User: ObservableObject {
    @Published var name = "OldBirds"
}


struct EditView: View {
    
    @EnvironmentObject var user: User
    
    var body: some View {
        TextField("Name", text: $user.name)
    }
}

struct DisplayView: View {
    @EnvironmentObject var user: User
    
    var body: some View {
        Text(user.name)
    }
}

struct LearnEnvironmentObject: View {
    let user = User()
    
    var body: some View {
        VStack {
            EditView()
            DisplayView()
        }.environmentObject(user)
    }
}
```

### @Environment

我们的确开一个从 `Environment` 拿到用户自定义的 `object`，但是`SwiftUI`本身就有很多系统级别的设定，我们开一个通过 `@Environment` 来获取到它们

```swift
struct CalendarView: View {
    @Environment(\.calendar) var calendar: Calendar
    @Environment(\.locale) var locale: Locale
    @Environment(\.colorScheme) var colorScheme: ColorScheme

    var body: some View {
        return Text(locale.identifier)
    }
}
```

通过 `@Environment` 修饰的属性，我们开一个监听系统级别信息的变换，这个例子里一旦 Calendar, Locale, ColorScheme 发生了变换，我们定义的 CalendarView 就会刷新


## @StateObject 

`@StateObject` 是 `@State` 的升级版。`@State`这种底层存储被 SwiftUI “全面接管” 的状态不同，`@ObservedObject` 只是在 `View` 和 `Model` 之间添加订阅关系，而不影响存储。`SwiftUI` 将为 `@State` 创建额外的存储空间，来保证在`View`刷新 (也就是重新创建时)，状态能够保持。但这对`@ObservedObject`并不适用。`@StateObject` 则是针对 `ObservableObject class` 的存储。它保证这个 `class` 实例不会随着 View 被重新创建。

如果不希望 `model` 状态在 `View` 刷新时丢失，那确实可以进行替换，这 (虽然可能会对性能有一些影响，但) 不会影响整体的行为。但是，如果 `View` 本身就期望每次刷新时获得一个全新的状态，那么对于那些不是自己创建的，而是从外界接受的 `ObservableObject` 来说，`@StateObject` 反而是不合适的。

```swift
class StateObjectClass:ObservableObject{
    let type:String
    let id:Int
    @Published var count = 0
    init(type:String){
        self.type = type
        self.id = Int.random(in: 0...1000)
        print("type:\(type) id:\(id) init")
    }
    deinit {
        print("type:\(type) id:\(id) deinit")
    }
}

struct CountViewState:View{
    @StateObject var state = StateObjectClass(type:"StateObject")
    var body: some View{
        VStack{
            Text("@StateObject count :\(state.count)")
            Button("+1"){
                state.count += 1
            }
        }
    }
}

struct CountViewObserved:View{
    @ObservedObject var state = StateObjectClass(type:"Observed")
    var body: some View{
        VStack{
            Text("@Observed count :\(state.count)")
            Button("+1"){
                state.count += 1
            }
        }
    }
}
```

测试：

```swift
struct Test1: View {
    @State var count = 0
    var body: some View {
        VStack{
            Text("刷新CounterView计数 :\(count)")
            Button("刷新"){
                count += 1
            }

            CountViewState()
                .padding()

            CountViewObserved()
                .padding()

        }
    }
}
```

当进点击+1按钮时，无论是`@StateObject`或是`@ObservedObject`其都表现出一致的状态，两个`View`都可以正常的显示当前按钮的点击次数，不过当点击刷新按钮时，`CountViewState` 中的数值仍然正常，不过`CountViewObserved`中的计数值被清零了。
可以看出，当点击刷新时，`CountViewObserved`中的实例被重新创建了，并销毁了之前的实例（`CountViewObserved`视图并没有被重新创建，仅是重新求了body的值）。

```sh
type:Observed id:443 init
type:Observed id:103 deinit
```

## 对比

![-w975](http://blog.loveli.site/mweb/16028977358443.png)

### @State vs @ObservedObject

`@State`在视图本地，值或数据在视图本地保存。它由框架管理，因为它存储在本地，因此它是一种值类型。

`@ObservableObject`在视图外部，并且不存储在视图中。它是一种引用类型，不在本地存储，而仅具有对该值的引用。这不是框架自动管理的，而是由开发人员。这适用于外部数据，例如数据库或由代码管理的模型。


### @State VS @Binding

`@State`只能在当前修饰的属性改变时会触发UI刷新，所以很适合值类型，因为对值类型里面属性的更新，也会触发整个值类型的重新设置。不过值类型在传递时会发生复制操作，所以给传递后的值类型即使属性更新了也不会触发最初的传过来的值类型的重新赋值，所以界面并不会刷新，此时需要用`@Binding`，因为它可以将值类型转为引用类型，这样在传递时，其实是一个引用，任何一方修改属性都会触发值类型的重新设置，UI界面也随之更新。


## 总结：

![-w716](http://blog.loveli.site/mweb/16029023986453.png)