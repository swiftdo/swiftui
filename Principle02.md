# 关于 propertyWarpper

每次属性更改时，将其新值打印到 `Xcode` 控制台:

```swift
struct Bar {
    private var _x = 0
    
    var x: Int {
        get { _x }
        set {
            _x = newValue
            print("New value is \(newValue)") 
        }
    }
}

var bar = Bar()
bar.x = 1 // Prints 'New value is 1'
```

如果我们继续记录更多这样的属性，需要每个属性都一遍遍复制相同的代码。那么通过如下的做法，就会减少很多拷贝的代码。

```swift
@propertyWrapper
struct ConsoleLogged<Value> {
    private var value: Value
    
    init(wrappedValue: Value) {
        self.value = wrappedValue
    }

    var wrappedValue: Value {
        get { value }
        set { 
            value = newValue
            print("New value is \(newValue)") 
        }
    }
}

struct Bar {
    @ConsoleLogged var x = 0
}

var bar = Bar()
bar.x = 1 // Prints 'New value is 1'
```

被`property wrapper`声明的属性，实际上在存储时的类型是 `ConsoleLogged`，只不过编译器施了一些魔法，让它对外暴露的类型依然是被包装的原来的类型。
上面的 `Bar` 等同于下方这种写法：

```swift
struct Bar {
    private var _x = ConsoleLogged(wrappedValue: 0)
    var x: Int {
        get { return _x.wrapperValue }
        set { _x.wrapperValue = newValue }
    }
}
```

## Property Wrapper 使用

有两个要求：

* 必须使用属性 `@propertyWrapper` 进行定义。
* 它必须具有 `wrappedValue` 属性。

下面就是最简单的包装器:

```swift
@propertyWrapper
struct Wrapper<Value>{
    var wrappedValue: Value
}

struct Text {
    @Wrapper var x: Int = 2
    @Wrapper(wrappedValue: 10) var y
}

var t = Text()
print(t.x) 
print(t.y)
```

以上两种声明之间有区别：

* `@Wrapper var x: Int = 2` 编译器隐式地调用 `init(wrappedValue:)` 用2初始化x。
* `@Wrapper(wrappedValue: 10) var y` 初始化方法被明确指定为属性的一部分。


## 访问属性包装器

在属性包装器中提供额外的行为:

```swift
@propertyWrapper
struct Wrapper<Value>{
    var wrappedValue: Value
    
    func log() {
        print("\(wrappedValue)")
    }
}
```

但如何才能去调用 `log`?

通过定义 `projectedValue` 属性，属性包装器可以公开更多API

> `projectedValue` 的类型没有任何限制
> `$属性名` 是访问包装器属性

```swift
@propertyWrapper
struct Wrapper<Value>{
    var wrappedValue: Value
    
    var projectedValue: Wrapper<Value> { return self }
    
    func log() {
        print("\(wrappedValue)")
    }
}

struct Text {
    @Wrapper var x: Int = 2
    
    func foo() {
        print(x) // 'wrappedValue'
        print(_x) // wrapper type itself, 私有的
        print($x) // projectedValue
    }
}

var t = Text()
print(t.x) // 2
print(t.$x) // Wrapper<Int>(wrappedValue: 2)
t.$x.log() // 2
```

## 再举个例子：

```swift
@propertyWrapper
struct SmallNumber {
    private var number: Int
    
    var projectedValue: Bool
    
    init() {
        self.number = 0
        self.projectedValue = false
    }
    
    var wrappedValue: Int {
        get {return number}
        set {
            if newValue > 12 {
                number = 12
                projectedValue = true
            } else {
                number = newValue
                projectedValue = false
            }
        }
    }

}

struct SomeStructure {
    @SmallNumber var someNumber: Int
}

var somes = SomeStructure()
print(somes.someNumber) // 0
print(somes.$someNumber) // false

somes.someNumber = 14
print(somes.someNumber) // 12
print(somes.$someNumber) // true
```

## 推荐阅读

* [官方文档: Property Wrappers](https://docs.swift.org/swift-book/LanguageGuide/Properties.html#ID617)
* [相关提案](https://github.com/apple/swift-evolution/blob/master/proposals/0258-property-wrappers.md)
* [Property wrappers in Swift](https://www.swiftbysundell.com/articles/property-wrappers-in-swift/)

