---
layout: post
title:  "Using Swift Packager Manager for Xcode 11"
date:   2019-10-13 13:00:00 +0800
categories: iOS
---
一直以來，我都很想嘗試在 iOS 開發中使用 Swift Package Manager，現在在 Xcode 11 中這個願望實現了。

同時，將 App 中的代碼按照職責與依賴的關係分離，可以讓整個 App 的框架層級以一個清晰的方式呈現出來，也會利於我們去寫 unit tests。我也會在以下內容中說明和解釋這個。

## 為什麼不使用 CocoaPods 或 Carthage
CocoaPods 是幾乎每一個 iOS 開發都會入門使用的第三方包管理系統，它對於我們進行整合 remote (like Github) 中的框架非常方便。

不過，寫一個簡單的本地 pod 並不是一件特別容易的事情，你需要學會一些 Ruby 語法和 podspec 的說明才能很好完成這個任務。

Carthage 使用者也許並沒有很多，但它對於加快 App build 速度有些良好的表現。我在過去使用時與 Xcode project 整合需要很多人工的方式來處理，不過現在它的發展怎麼樣了，我就不做其他評論。

## 為什麼不使用建立 Local Framework 來處理
在我一開始實驗時，我也確實是使用 local framework 來分離與組織代碼。

它一開始易於幫助你簡單分離 App 內部的複雜邏輯，而缺點在於每當你修改一個 framework 邏輯，你就必須重新 build 它一次。如果 framework 之間還有依賴關於，你還得依據它們之間的關於按照順序一個一個 build 起來。

另外，一旦你想切換不同的 device，如從真機和模擬器中切換，你也需要重頭按照 frameworks 之間的依賴關係一個一個 build 過去，這對我來說有一點煩人，這也是這篇文章會出現的一個原因。

我也有嘗試從 xcbuild 中找到一點解決方式，不過並沒有很順利，如果你們有遇到類似的問題且有良好的解決手段，歡迎和我交流聯繫。

## Get Started
我會寫一些演示代碼，你可以按照下面的步驟重現出來。

你也可以在 [https://github.com/Arcovv/SPMSample](https://github.com/Arcovv/SPMSample) 中找到最終成果。

以下版本是建立在 Xcode 11.1(11A1027) 中，如果在其他版本有其他出路麻煩自行處理。

首先建立一個空白專案，命名為 `SPMExample`，無論你選擇 SwiftUI 或者 Storyboard 都可以，不過我們還是拿 ViewController 簡單操作比較快一點，所以選擇 Storyboard。

在這篇文章中，我們會簡單建立一個 Utility framework，一個 Model framework，以及將他們與協作至 App 中。

## Implement ViewController
一直以來我都喜歡研究著函數式，所以我們也來寫一個簡單的函數，用於檢查 optional 是否為 nil，如果是就傳出設定的默認值，否則就傳出 unwrapped 值。

新建一個 Swift File 並命名為 Optional+Extension 並寫下：

```swift
func coalesce<T>(_ default: T) -> (T?) -> T {
  return { maybe in
    maybe ?? `default`
  }
}
```

以及一個新的文件命名為 Dog，裡面有一個 Dog struct 與 DogFactory：

```swift
struct Dog {
  let name: String
  let age: Int
  
  static let sample = Dog(name: "Bill", age: 10)
}

final class DogFactory {
  var timer: Timer!
  var generateCallback: ((Dog) -> Void)?
  
  deinit {
    self.timer.invalidate()
    self.timer = nil
  }
  
  init() {
    self.timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
      self?.generateCallback?(.sample)
    }
    self.timer.fire()
  }
}
```

然後在 ViewController 中寫道：

```swift
import UIKit

class ViewController: UIViewController {
  var dogFactory: DogFactory?
  
  override func viewDidLoad() {
    super.viewDidLoad()
  
    self.dogFactory = coalesce(.init())(self.dogFactory)
    self.dogFactory?.generateCallback = { dog in
      print("Generate \(dog)")
    }
  }
}
```

當然你想寫成 `self.dogFactory = self.dogFactory ?? .init()` 也可以，不過有時我並不太喜歡 `??`。

嘗試 run 一下結果你就會發現 console 中會不斷 print 出 simple dog 的資料。

## Import Utility
由於 `coalesce(_:)` 是一級函數，也許會和我們的 App 其中的函數名發生衝突，當然有時候我們也可以使用函數重載技術來解決這個問題，不過有時候 Swift Type System 依舊沒有辦法很好判斷當前應該是什麼函數。

選擇 File - Net - Swift Package… 並命名為 Utility，儲存在和 App 同層級的文件夾中：

![Folder](https://github.com/Arcovv/ImageAssets/blob/master/2019-10-13-Swift-Package-Manager-Architecture/Utility%20Dest.png?raw=true)

然後拉進我們的專案中，並放在 SPMExample xcodeproj 的上方：
![Project](https://github.com/Arcovv/ImageAssets/blob/master/2019-10-13-Swift-Package-Manager-Architecture/Utility%20Folder.png?raw=true)

Xcode 會詢問你要不要新建一個 workspace，我們選擇 Save：
![Workspace](https://github.com/Arcovv/ImageAssets/blob/master/2019-10-13-Swift-Package-Manager-Architecture/Workspace.png?raw=true)

這樣 framework 的分離雛形就逐漸形成了。

將 Optional+Extension 文件移動到 Utility - Sources - Utility 底下，並刪除掉原來的 Utility.swift：
![](https://github.com/Arcovv/ImageAssets/blob/master/2019-10-13-Swift-Package-Manager-Architecture/Opional%20Dest.png?raw=true)

嘗試寫下 `import Utility` 會發現 Xcode 報錯說沒有這個 module，這是因為我們還沒有這個框架和 App 接起來。

>  No such module ‘Utility’  

進入以下區域並按下 + 選擇 Utility 並 Add 進來，然後執行 build。

![](https://github.com/Arcovv/ImageAssets/blob/master/2019-10-13-Swift-Package-Manager-Architecture/Add%20SPM.png?raw=true)

你會發現 Xcode 不再提示找不到這個 module 而是換了一個 Error 提示。

> Use of unresolved identifier ‘coalesce’  

這是因為 `coalesce(_:)` 函數的訪問權限是 internal 的，我們回到 Optional+Extension 中，將它加上 public 的前綴字：

```swift
public func coalesce<T>(_ default: T) -> (T?) -> T {
  return { maybe in
    maybe ?? `default`
  }
}
```

好了你再 build 一次，會發現世界沒有了錯誤，我們已經成功取得 Utility 中的函數了！

另外這裡我也想讓 `Dog.swift`  中也和 Utility 有依賴性關係，所以改了一下 `DogFactory`，算一個不是太恰當的例子：

```swift
final class DogFactory {
  var timer: Timer!
  var generateCallback: ((Dog) -> Void)?
  
  deinit {
    self.timer.invalidate()
    self.timer = nil
  }
  
  init() {
    self.timer = coalesce(getTimer())(self.timer)
    self.timer.fire()
  }
  
  func getTimer() -> Timer {
    Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
      self?.generateCallback?(.sample)
    }
  }
}
```

## Import Model
我們如法炮製，也建立一個 Model Framework，一樣是新建一個 Swift Package，結果如下：
![](https://github.com/Arcovv/ImageAssets/blob/master/2019-10-13-Swift-Package-Manager-Architecture/Package%20Final.png?raw=true)

然後也將 Dog.swift 移動到 Sources/Model 底下，build 之後會發現 Xcode 發出的 Error 提示：

>  No such module ‘Utility’  

這裡我們的 Model framework 就和 Utility framework 發生依賴性關係了。

找到 Model framework 底下的 Package.swift，將內容修改為：

```swift
// swift-tools-version:5.1
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
  name: "Model",
  platforms: [
    .iOS(.v11) // 1
  ],
  products: [
    .library(
      name: "Model",
      targets: ["Model"]),
  ],
  dependencies: [
    .package(path: "../Utility") // 2
  ],
  targets: [
    .target(
      name: "Model",
      dependencies: ["Utility"]), // 3
    .testTarget(
      name: "ModelTests",
      dependencies: ["Model"]),
  ]
)
```

**About // 1**

由於 `Timer.scheduledTimer(withTimeInterval:repeats:block:)` 是 iOS 10 以上的 API，而 Package 默認的最低版本是 iOS 8，於是我們就簡單寫一下說明這個 Package 最低要求 iOS 11 以上。

**About // 2**

這裡我們說明了這個 Package 需要的依賴框架是什麼以及在哪裡，最後也需要在 // 3 寫上對於的依賴框架名稱。

重新再 build 之後，會發現 DogFactory 找不到，也是同樣的訪問權限問題，將 Dog.swift 修改成如下：

```swift
import Foundation
import Utility

public struct Dog {
  public let name: String
  public let age: Int
  
  public static let sample = Dog(name: "Bill", age: 10)
}

public final class DogFactory {
  var timer: Timer!
  public var generateCallback: ((Dog) -> Void)?
  
  deinit {
    self.timer.invalidate()
    self.timer = nil
  }
  
  public init() {
    self.timer = coalesce(getTimer())(self.timer)
    self.timer.fire()
  }
  
  func getTimer() -> Timer {
    Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
      self?.generateCallback?(.sample)
    }
  }
}
```

現在再重新 build 一次 App，會發現沒有其他問題啦。

## Test Utility Framework
對於一個良好的工具類框架，有完整的 Unit Test 才能說明這個框架的強壯，我們也來為 `coalesce(_:)` 寫一些測試吧。

打開 Utility / Tests / UtilityTests / UtilityTests.swift，改成如下內容：

```swift
import XCTest
@testable import Utility

final class UtilityTests: XCTestCase {
  func testUseCoalesceDefaultValue() {
    let target: Int? = nil
    let result = coalesce(1)(target)
    XCTAssertNotNil(result)
  }
  
  func testNotUseCoalesceDefaultValue() {
    let target: Int? = 3
    let result = coalesce(1)(target)
    XCTAssertEqual(target, result)
  }
}
```

將 Target 切換成 Utility 然後開始測試，OK 函數並沒有問題。

同樣你也可以使用 Terminal 來測試這個 package。在 Utility 目錄下，執行 `swift test` 即可。

```
$ swift test
[6/6] Linking UtilityPackageTests
Test Suite 'All tests' started at 2019-10-13 12:19:02.874
Test Suite 'UtilityPackageTests.xctest' started at 2019-10-13 12:19:02.875
Test Suite 'UtilityTests' started at 2019-10-13 12:19:02.875
Test Case '-[UtilityTests.UtilityTests testNotUseCoalesceDefaultValue]' started.
Test Case '-[UtilityTests.UtilityTests testNotUseCoalesceDefaultValue]' passed (0.261 seconds).
Test Case '-[UtilityTests.UtilityTests testUseCoalesceDefaultValue]' started.
Test Case '-[UtilityTests.UtilityTests testUseCoalesceDefaultValue]' passed (0.001 seconds).
Test Suite 'UtilityTests' passed at 2019-10-13 12:19:03.137.
	 Executed 2 tests, with 0 failures (0 unexpected) in 0.262 (0.262) seconds
Test Suite 'UtilityPackageTests.xctest' passed at 2019-10-13 12:19:03.137.
	 Executed 2 tests, with 0 failures (0 unexpected) in 0.262 (0.262) seconds
Test Suite 'All tests' passed at 2019-10-13 12:19:03.137.
	 Executed 2 tests, with 0 failures (0 unexpected) in 0.262 (0.263) seconds
```

## Test Model Framework
我們也需要測試我們的 Model package，同樣打開 Model / Tests / ModelTests / ModelTests.swift，寫成如下內容：

```swift
import XCTest
@testable import Model

final class ModelTests: XCTestCase {
  func testDogModel() {
    let dog = Dog(name: "Jack", age: 8)
    XCTAssertEqual(dog.name, "Jack")
    XCTAssertEqual(dog.age, 8)
  }
  
  func testDogFactoryGenerate5DogCount() {
    var dogs = [Dog]()
    var index = 0
    let goalCount = 2
    
    let expect = expectation(description: #function)
    let factory = DogFactory()
    factory.generateCallback = { dog in
      dogs.append(dog)
      
      if index == goalCount - 1 {
        XCTAssertEqual(dogs.count, goalCount)
        expect.fulfill()
      }
      
      index += 1
    }

    wait(for: [expect], timeout: Double(goalCount))
  }
}
```

測試也沒問題，這樣我們就完成了 Unit Test。

## Final
將 App 的組織架構拆分成多個，不僅是大公司需要做的事情，對於一些小的專案並維護，都是一件很好且輕鬆的作法。

這個我並沒有使用 Remote pods，我們其實也可以將我們分離出來的 package 放在如 Github 上面託管。如果我們要這樣做的事情，得為每一個託管在 Github 的 package 編著標註上 tag，這樣 Swift Package Manager 才能取得到 package 內容。

```
git init
git add .
git commit -m "First Inital"
git tag 1.0.0
git push <OriginalURL>
```

然後修改對應需要依賴的 package 內容：

```swift
// 從這個
.package(path: "Local Path")
// 改為
.package(url: "Original URL", from: 1.0.0)
```

這樣不論你的 package 你 local 的還是 remote 的，都能有效把你的 App 組織架構整理得很乾淨，也能讓你知道你的相關類（如 ViewModel） 是否很好與 ViewController / Database / Network 等良好分離（如使用 Dependency Injection 技術）。