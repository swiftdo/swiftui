# 单表达式隐式返回值

```swift
// before swift 5.0
struct Rectangle {
    var width = 0.0, height = 0.0
    var area1: Double {
        return width * height
    }

    func area2() -> Double {
        return width * height
    }
}

// after switft 5.1
struct Rectangle {
    var width = 0.0, height = 0.0
    var area1: Double { width * height }

    func area2() -> Double { width * height }
}
```