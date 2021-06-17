# Text

代码：

```swift
Text("Tap me !") 
    .font(.largeTitle)
    .fontWeight(.bold)
    .foregroundColor(.white)
    .padding(30)
    .frame(alignment: .center)
    .background(Color.red)
    .cornerRadius(100)
```

效果：

![效果](http://blog.loveli.site/tuc/20210616195437.png ':size=240')

## 用法：

### 设置文本

```swift
Text("Hello, World!")
```

### 设置文字颜色

通过 `foregroundColor` 设置文本颜色

```swift
Text("Hello, World!")
    .foregroundColor(.red)
```

### 设置字体大小

可以通过 `font` 设置字体大小，指定字形

```swift
Text("Hello, World!")
    .foregroundColor(.red)
    .font(.system(size: 30))
```

修成largeTitle的样式而且是圆弧的

```swift
Text("Hello, World!")
    .foregroundColor(.red)
    .font(.system(.largeTitle, design: .rounded))
```

更改字体的字型

```swift
Text("Hello, World!")
    .foregroundColor(.red)
    .font(.font(.custom("Courier", size: 30)))
```

### 设置字重

可以使用 `fontWeight` 设置字重

```swift
Text("Hello, World!")
    .foregroundColor(.red)
    .font(.system(size: 30))
    .fontWeight(.regular)
```

### 设置阴影

可以使用`shadow`设置阴影

```swift
Text("Hello, World!")
    .foregroundColor(.red)
    .font(.custom("Courier", size: 30))
    .fontWeight(.regular)
    .shadow(color: .black, radius: 2, x: 0, y: 15)
```

![](http://blog.loveli.site/tuc/20210616203147.png ':size=240')

### 设置文本对齐方式

通过 `multilineTextAlignment` 指定文本的对齐方式。

```swift
Text("Hello, World! nice to miss you!")
    .foregroundColor(.red)
    .font(.custom("Courier", size: 30))
    .fontWeight(.regular)
    .shadow(color: .black, radius: 2, x: 0, y: 1)
    .multilineTextAlignment(.center)
```

### 限制显示行数

通过 `lineLimit` 将行数限制为一定数量

```swift
Text("Hello, World! welcome to OldBirds ,nice to miss you! see you again ~~ ")
    .foregroundColor(.black)
    .font(.system(size: 50))
    .fontWeight(.regular)
    .lineLimit(3)
```

![](http://blog.loveli.site/tuc/20210616204427.png ':size=240')


### 设置行间距

通过 `lineSpacing` 设置行间距

```swift
Text("Hello, World! welcome to OldBirds ,nice to miss you! see you again ~~ ")
    .foregroundColor(.black)
    .font(.system(size: 50))
    .fontWeight(.regular)
    .lineSpacing(20)
```

![](http://blog.loveli.site/tuc/20210616204807.png ':size=240')


### 旋转文字

* 2D 旋转

    ```swift
    Text("Hello, World! welcome to OldBirds ,nice to miss you! see you again ~~ ")
        .foregroundColor(.blue)
        .font(.system(size: 25))
        .rotationEffect(.degrees(20), anchor: UnitPoint(x: 0, y: 0))
    ```

    ![](http://blog.loveli.site/tuc/Screen%20Shot%202021-06-16%20at%209.05.02%20PM.png ':size=240')


* 3D 旋转

    ```swift
    Text("Hello, World! welcome to OldBirds ,nice to miss you! see you again ~~ ")
        .foregroundColor(.blue)
        .font(.system(size: 25))
        .rotation3DEffect(.degrees(60), axis: (x: 1, y: 0, z: 0))
    ```

    ![](http://blog.loveli.site/tuc/20210616210549.png ':size=240')
