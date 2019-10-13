---
layout: post
title:  "答卓同學 Swift 面試題"
date:   2017-12-13 22:00:00 +0800
categories: interview
---
附上[面試鏈接](http://www.jianshu.com/p/7c7f4b4e4efe)。

下班時候看到這個，突然很心癢，決定當成一次實際面試中可能的真正面試題，在不通過網絡搜索的情況下來回答。希望看看自己目前的水平大概是一個什麼程度。

### 1. class 和 struct 的区别

`class` 是引用類型，通過引用計數來處理記憶體回收，可以被繼承，多態。順帶一提，閉包是引用類型。

`struct` 是值類型，通過複製的方式來進行傳遞，因此沒有引用計數，就沒有循環引用問題。不能被繼承，但可以遵循各個協議。順帶一提，`enum` 也是值類型。

### 2. 不通过继承，代码复用（共享）的方式有哪些

- `protocol` + `extension: protocol where Self: ...` 。
- `extension SomeClass / SomeStruct / SomeEnum`。
- 如果繼承自 NSObject，還可以用 runtime，不過如果和原 Foundation 庫命名重複的話，則不確定會先執行那個，遇到 bug 會無語。

### 3. Set 独有的方法有哪些？

Set 可以有交集並集等數學方法，同時如果使用類似 `let set = Set([1, 1, 1, 2])` 的方式可以過濾掉重複的項目，也就是說 `set` 中最終只會有 `1, 2`，使用的前提條件是 `Element` 要遵循 `Hashable`。

### 4. 实现一个 min 函数，返回两个元素较小的元素

```swift

func myMin<T: Comparable>(a: T, b: T) -> T {
	return a < b ? a : b
}

```

### 5. map、filter、reduce 的作用

使用函數式的方法來避免宣告 `var` 來進行 `for in` 迴圈時，進行操作時數值發生改變，可以鏈式操作。

`map` 是傳入 `f: (T) -> V` 返回 `Box<V>` 的函數。

`filter` 是傳入 `f: (T) -> Bool` 返回 `T` 的函數。

`reduce` 是傳入一個初始值與下一個結果值和 `Element` 值的函數（具體有點類型有點記不住了），寫法大致為：
```swift
func reduce<T>(_ initValue: T, method: (T, Element) -> Void) {
}
```

### 6. map 与 flatmap 的区别

`map` 函數定義大致為 `func map<T>(f: T -> V) -> Box<V>`，它會嘗試取出 `Box` 中的值，如果成功則進行操作，如果不成功則維持原樣再傳遞下去。

`flatMap` 函數定義大致為 `func flatMap<T>(f: T -> Box<V>) -> Box<V>`，他可以避免比如 `map` 中傳入的是一個 `Optional<T>` 後返回了一個 `Optional<Optional<T>>` 的糾結情況，簡單來說就是攤平結果，使之更好處理。`Array` 的 `flatMap` 還有另外一種用法：如果 `Array` 中的 `Element` 也是一個 `Array<V>`，則可以將其攤平為 `Array<V>`。

### 7. 什么是 copy on write

大致上來說就是蘋果針對值類型對象的一種優化行為，避免值類型在傳入時總要進行深拷貝，具體不是很清楚。

### 8. 如何获取当前代码的函数名和行号

```swift
func callMethod(name: String = #function, line: Int = #line) { }

callMethod()
```

### 9. 如何声明一个只能被类 conform 的 protocol

```swift
protocol MyProtocol: class { }
```

### 10. guard 使用场景

用於提前退出并要求你先進行 else 情況，可以避免 `if else` 的循環嵌套，最常見在 `guard let ... else ...`。

### 11. defer 使用场景

被觸發在函數將要返回時。需要注意的是多重 `defer` 時候會遇到的問題：
```swift
func myDefer() {
	defer { print("3") }
	defer { print("2") }
	defer { print("1") }
	print("0")
}

myDefer()
// Result:
// 0
// 1
// 2
// 3
```

### 12. String 与 NSString 的关系与区别

`String` 是值類型（在 Swift 4 中內部 API 會再重做一次），`NSString` 是引用類型，繼承 `NSObject`。對於一些特別的字串兩者取得長度不同。`String` 通過 `characters.count` 來取得長度，而 `NSString` 似乎是通過 `length` 取得？

### 13. 怎么获取一个 String 的长度

```swift
let length1 = "string".characters.count
let length2 = "string".data(using: .utf8).count
let length3 = ("string" as NSString).length
```

### 14. 如何截取 String 的某段字符串

```swift
let a = "string".characters[0..1]
```

還可以通過 Range 和 Index 來取得，具體要翻翻文檔。。

### 15. throws 和 rethrows 的用法与作用

- `throws` 用來表明該函數會拋出錯誤，必須要用 `do try catch` / `try?` / `try!` 來執行。
- `rethrows` 會用來表明該函數內部會拋出錯誤，印象中在 Swift `Sequence` 協議的 `map` `flatMap` 等函數中有見到過。

### 16. try？ 和 try！是什么意思

- `try?` 用來修飾一個會拋出錯誤的函數，其返回結果會被包在 `Optional` 中，用來表示“我不想要管這次拋出的錯誤是什麼”，所以有說會拋錯的函數就不要返回 `Optional` 對象。
- `try!` 用來表示“我還是不想管拋出的錯誤，但是如果拋錯了，就崩潰程序好了”，其結果會被表示為 `T!`。

### 17. associatedtype 的作用

用來在 `protocol` 中聲明泛型，主要是因為 Swift 3.0 中還不支持協議時泛型寫法吧？不是很肯定。Swift 3.1 中協議似乎可以聲明泛型了。

### 18. 什么时候使用 final

- 聲明該 `class` 不可被繼承。
- 聲明該函數或屬性不可被 `override`。
- 聲明為 `final` 時可以讓編譯器知道該類型不會被繼承，因此可以做一小點優化。

### 19. public 和 open 的区别

`public` 用來聲明該函數或屬性只有在這個 `module` 中可以被覆寫和繼承，而 `open`的則權限最高，不僅可以在其他 `module` 中被訪問，而且可以被繼承和覆寫。

### 20. 声明一个只有一个参数没有返回值闭包的别名
```
Typealias Callback = (Int) -> Void
```

### 21. Self 的使用场景
`Self` 與 `self` 的差異在於，一個表明類，一個表明實例。

協議中聲明：
```swift
procotol MyProtocol: class {
	func then() -> Self
}
```

擴展協議：
```swift
protocol MyProtocol { }

extension MyProtocol where Self: UIView { }
```

### 22. dynamic 的作用
宣告其屬性可被 KVO 的，該類必須繼承 `NSObject`，最經常用到的就是繼承 Realm.Object 的時候。

### 23. 什么时候使用 @objc
宣告某個協議方法是可以不被實現的，比如 `UITableViewDelegate` 中的所有方法都是。

另外，使用 `Target-Action` 時為了不暴露類中 `selector` 所指的方法而將該方法宣告為 `fileprivate` 或 `private` 時，因為需要被 runtime 時可以觸發，因此加上 `@objc`：
```swift

class MyViewController: UIViewController {
	func myMethod() {
		let _ = UIMenuItem(target: self, action: #selector(theAction))
	}
	
	@objc private func theAction(sender: Any) { 
	}
}

```

### 24. Optional（可选型） 是用什么实现的
`Optional` 本身是一個 enum 對象，裡面只有兩個枚舉成員，一個是 `.some(Wrapped)`，一個是 `.none`。另外還符合了 `Nil....` 什麼的協議表明可以被指向 `nil`。

### 25. 如何自定义下标获取
```swift
class MyClass {
	private let a = ["a", "b"]
	
	subscript(index: Int) -> String {
		return a[index]
	}
}

```

### 26. ?? 的作用
語法糖。用於嘗試取得 `Optional` 中的值時，可以聲明當 `Optional` 為 `.none` 時，使用 `??` 右邊的值。另外右邊值為閉包時，有被 `@autoclosure` 優化，確保需要時才計算得到結果。

### 27. lazy 的作用
- 延時計算，常見用於類似 `lazy var session: URLSession = URLSession(...)`。
- `Sequence` 對象似乎都有 `lazy` 屬性，意圖也是延時計算，具體使用不確定，不常用。

### 28. 一个类型表示选项，可以同时表示有几个选项选中（类似 UIViewAnimationOptions ），用什么类型表示
`OptionSet` 好像是這樣拼？

### 29. inout 的作用
聲明函數傳入的參數可以被修改，用法為：
```swift
func addOne(_ a: inout Int) {
	a += 1
}

var a = 1
addOne(&a)
```

### 30. Error 如果要兼容 NSError 需要做什么操作
`error as NSError`

或者，擴展：
```swift
extension Error {
	func toNSError(
		domain: String = "myDomain",
		code: Int = 0, 
		userInfo: [String: Any]? = nil) 
		-> NSError 
	{
		return NSError(domain: domain, code: code, userInfo: userInfo)
	}
}

enum MyError: Error {
	case some
}

let _ = MyError.some.toNSError()
let error = MyError.some.toNSError(domain: "myError", code: 0)
```

### 31. 下面的代码都用了哪些语法糖 / [1, 2, 3].map{ $0 * 2 }
1. `[1, 2, 3]` 是陣列的語法糖，意義為字面上表示為 `Array<Int>` 的陣列。
2. `Trailing Closures`。如果一個函數的最後一個參數是閉包時，則可以不寫 `()`。
3. 省略閉包參數的類型聲明，其被推斷為 `Int`。
4. `$0` 表明是陣列中的每一個元素，省略了閉包參數的命名。
5. 省略了閉包回傳值類型的聲明，其被推斷為 `Int`。
6. 若閉包只有一行，則可以省略 `return`。

### 32. 什么是高阶函数
基本上可以定義為 `map`、`flatMap`、`filter`、`reduce` 等函數式的方法。

### 33. 如何解决引用循环
引用循環問題主要發生為：引用類型對象和另外一個引用類型對象相互引用，解決方法為：
1. 其中一個引用對象的屬性聲明為 `weak` 或 `unowned`。
2. 閉包的捕獲列表中聲明 `[weak someObject, weak self, unowned self]` 等，再次論證閉包是引用類型。

### 34. 下面的代码会不会崩溃，说出原因
```swift
// 題目：
var mutableArray = [1,2,3]
for _ in mutableArray {
  mutableArray.removeLast()
}
```
不會。原因：
大致上來說是因為 `Sequece` 中的迭代器吧，記得有在《Swift 進階》中提到，詳細原因說不清。


### 35. 给集合中元素是字符串的类型增加一个扩展方法，应该怎么声明
```swift
protocol MyProtocol { }

extension Array where Element: MyProcotol { }
```

### 36. 定义静态方法时关键字 static 和 class 有什么区别
`class` 關鍵字對象可以被 `override`，`static` 則不行。

### 37. 一个 Sequence 的索引是不是一定从 0 开始？
不一定。比如：
```swift
let array = [0...100]
let arraySlice = array[50..<80] // arraySlice 的 startIndex 應該是50
```

### 38. 数组都实现了哪些协议
。。。
`Collection`, `RandomAddress...(忘記了)`。

### 39. 如何自定义模式匹配
不太理解題目，是說 `if case .some(let value) = 3?` 之類的嗎？

### 40. autoclosure 的作用
額，恰好前面有提到，用來進行一些優化，可以當必要時候再計算閉包中的數值，因此要求類型為 `() -> T`。

### 41. 编译选项 whole module optmization 优化了什么
不確定，看了一堆資料，忘光了。反正這個優化設定在混編時候遇到了很多坑。

### 42. 下面代码中 mutating 的作用是什么
```swift
// 題目
struct Person {
  var name: String {
      mutating get {
          return store
      }
  }
}
```
不曉得，沒看出 `store` 從哪裡來。

### 43. 如何让自定义对象支持字面量初始化
只要讓某個類或結構體遵循 `ExpressibleByStringLiteral` 協議就行。

### 44. dynamic framework 和 static framework 的区别是什么
`static framework` 是 `.a` 檔，需要在運行時就被載入，無法被共用記憶體。
`dynamic framework` 是 iOS 8 之後提出的，本質上是個 `Bundle`，如果已經被加載過，則可以共享使用。

### 45. 为什么数组索引越界会崩溃，而字典用下标取值时 key 没有对应值的话返回的是 nil 不会崩溃。
使用 `Index` Swift 認為你很自信并肯定取值的過程是不會有異常的，所以越界就給你崩；而 `Dictionary` 的 `key` 需要遵循 `Hashable` 協議才可以使用，因此在查找值的時候進行了算法提高，但也表明取值結果的不一定？總覺得這話自己說的都怪怪的。

### 46. 一个函数的参数类型只要是数字（Int、Float）都可以，要怎么表示。
```swift
func myMethod<T>(_ value: T) where T: ExpressibleByIntegerLiteral, T: ExpressibleByFloatLiteral {

}
```

## 後記

本來自信60分鐘搞定，沒想到題目那麼多，結果花了1小時50分鐘。

有些知識點以前有看過但是記不太清楚了，有的則完全不曉得。如果能再配合一下文檔和以前收集的一些資料或許能答的更好。裡面一定有各個錯誤，希望能多改正。

---

來自台灣的一枚 iOS 開發者。

歡迎關注我的微博：[Arco-vv](http://www.weibo.com/1679063565/profile?rightmod=1&wvr=6&mod=personinfo)。

歡迎關注我維護的簡書專題：[iOS Swift && Objective-C](http://www.jianshu.com/c/b6cc50b537d5?utm_source=desktop&utm_medium=notes-included-collection)。