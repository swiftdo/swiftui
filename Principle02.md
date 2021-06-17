# 关于 propertyWarpper

```swift
// before swift 5.0
struct User {
    static var usesTouchID: Bool {
        get {
            return UserDefaults.standard.bool(forKey: "USES_TOUCH_ID")
        }
        set {
            UserDefaults.standard.set(newValue, forKey: "USES_TOUCH_ID")
        }
    }
    static var isLoggedIn: Bool {
        get {
            return UserDefaults.standard.bool(forKey: "LOGGED_IN")
        }
        set {
            UserDefaults.standard.set(newValue, forKey: "LOGGED_IN")
        }
    }
}

// after switft 5.1
@propertyWrapper
struct UserDefault<T> {
    // 这里的属性 key 和 defaultValue 还有 init 方法都是实际业务中的业务代码
    let key: String
    let defaultValue: T

    init(_ key: String, defaultValue: T) {
        self.key = key
        self.defaultValue = defaultValue
        UserDefaults.standard.register(defaults: [key: defaultValue])
    }

    // wrappedValue 是 @propertyWrapper 必须实现的属性。
    // 当操作我们包裹的属性时，其具体的set get 方法走这里。
    var wrappedValue: T {
        get {
            return UserDefaults.standard.object(forKey: key) as? T ??  defaultValue
        }
        set {
            UserDefaults.standard.set(newValue, forKey: key)
        }
    }
}

struct User {
    @UserDefault("USES_TOUCH_ID", defaultValue: false)
    static var usesTouchID: Bool

    @UserDefault("LOGGED_IN", defaultValue: false)
    var isLoggedIn: Bool
}
let user = User()
User.usesTouchID = true
let usesTouchID = User.$usesTouchID
let isLoggedIn = user.$isLoggedIn
```

实际上属性包装器是在编译时期翻译为一下的代码, 并且编译器禁止使用 `$` 开头的标识符

```swift
struct User {
    static var $usesTouchID = UserDefault<Bool>("USES_TOUCH_ID", defaultValue: false)
    static var usesTouchID: Bool {
        set {
            $usesTouchID.value = newValue
        }
        get {
            $usesTouchID.value
        }
    }

    @UserDefault("LOGGED_IN", defaultValue: false)
    var isLoggedIn: Bool
}
```