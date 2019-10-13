---
layout: post
title:  "對 Swift 中 closure capture strong reference 導致 retain cycle 情況做的一些理解與研究"
date:   2017-07-28 20:00:00 +0800
categories: Swift
---
## 簡單概念區分

memory leak / retain cycle 的區別。memory leak 更多的應該是包含了 retain cycle 的概念，而 retain cycle 只是其中一種情況而已。例如 URLSession 在傳入自定義 delegate 對象後強引用 delegate 對象是刻意的設計方式，對此有疑問請往下看。

escaping / async / completionHanlder 這些關鍵字說明會持有這個 Closure，但不代表會 retain cycle，視情況而定。

weak / unowned 算是解決的方法，但不是原因。

retain cycle 相關情景：
- delegate 設計模式
- closure
- 更多詳細說明參考 <The Swift Programming Language> 一書中的 "Automatic Reference Counting" 一章，例子和圖片就不演示了。

---

## Scenarios

**URLSession**

*The session object keeps a strong reference to the delegate until your app exits or explicitly invalidates the session. If you do not invalidate the session, your app leaks memory until it exits.*

同樣，在 URLSession 文檔中，對於其 delegate 屬性也有相關說明：

*This delegate object is responsible for handling authentication challenges, for making caching decisions, and for handling other session-related events. The session object keeps a strong reference to this delegate until your app exits or explicitly invalidates the session. If you do not invalidate the session, your app leaks memory until it exits.*

進一步說明可以在[Life Cycle of a URL Session](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/NSURLSessionConcepts/NSURLSessionConcepts.html#//apple_ref/doc/uid/10000165i-CH2-SW1)找到。

**DispatchQueue**
不會造成 retain cycle，因此在類似 DispatchQueue.main.async(_:) 的函數中，是不用寫 weak 或 unowned 的。不僅是沒必要，也是對概念的理清。
另外對於實際情況來說，如果使用 asyncAfter 之類的方法，又希望 self 對象可以不被總是 retain 住，那麼就可以使用 [weak self] 來做。也就是要具體情況具體分析，不能寫死。
參考 [Understanding memory leaks in closures](https://medium.com/compileswift/understanding-memory-leaks-in-closures-48207214cba)。
參考 [Automatic Reference Counting](http://blog.stablekernel.com/how-to-prevent-memory-leaks-in-swift-closures)。

**UIView.animationWithDuration(_:)**
假設 retain 住的對象是 UIViewController，而因為 UIViewController 沒有直接持有 UIView.animationWithDuration(_:) 執行的閉包，所以不會有 retain cycle 現象。
參考 [Understanding memory leaks in closures](https://medium.com/compileswift/understanding-memory-leaks-in-closures-48207214cba)。

**RxSwift**
*Who owns the closure? The closure can be called many times and we don’t don’t know when, so RxSwift needs to keep a reference to it. In this case the closure is actually owned indirectly by searchBar. It makes sense because the closure is released when searchBar is. But wait, searchBar is owned by self when it’s added to the view hierarchy, and the closure references self. So in this case we have a cycle, and we need to break it in order to avoid a memory leak.*

參考 [Understanding memory leaks in closures](https://medium.com/compileswift/understanding-memory-leaks-in-closures-48207214cba)。

---

## unowned 與 weak

*In the specific case of a closure, you just need to realize that any variable that is referenced inside of it, gets "owned" by the closure.*
只要在閉包中使用那些 reference 類型對象，閉包就會擁有他們，因此是 strong reference。

這是現象：*So if a class owns a closure, and that closure captures a strong reference to that class, then you have a strong reference cycle between the closure and the class.*

具體原因應該可以細緻到記憶體的 retain count 上。

使用捕獲列表（capture list）後，會對該變量進行淺拷貝；否則在閉包中修改變量時，也會影響到外部的變量。而在捕獲列表上，可以使用 weak 或 unowned 詞語來修飾外部引用類型的變量、或乾脆不修飾。

unowned 的使用情景：
*The closure have the same lifetime of the variable, so the closure will be reachable only until the variable is reachable. The variable and the closure have the same lifetime. In this case you should declare the reference as unowned. A common example is the [unowned self] used in many example of small closures that do something in the context of their parent and that not being referenced anywhere else do not outlive their parents.*

weak 的使用情景：
*The closure lifetime is independent from the one of the variable, the closure could still be referenced when the variable is not reachable anymore. In this case you should declare the reference as weak and verify it's not nil before using it (don't force unwrap). A common example of this is the [weak delegate] you can see in some examples of closure referencing a completely unrelated (lifetime-wise) delegate object.*

參考 <The Swift Programming Language> 一書。

在效能方面，unowned 勝於 weak，[相關說明在此](https://twitter.com/jckarter/status/667364165057515521)，以及[Unowned or Weak? Lifetime and Performance](https://www.uraimo.com/2016/10/27/unowned-or-weak-lifetime-and-performance/)。

---

## 一些疑問

- 在 LLVM 的基礎上，是否有方法可以檢查出無效的 weak / unowned 寫法？
- Swift 依舊採用 ARC 是否是正確的選擇呢？對此這篇文章有些看法：[Memory Management In Swift: A Lost Opportunity For Apple](http://digitalleaves.com/blog/2014/07/memory-management-in-swift-a-lost-opportunity/)

--- 

## 相關參考

[Shall we always use [unowned self] inside closure in Swift](https://stackoverflow.com/questions/24320347/shall-we-always-use-unowned-self-inside-closure-in-swift)

[iOS blocks - 三個會造成retain cycle的anti patterns](http://popcornylu.blogspot.tw/2012/02/3-anti-patterns-which-lead-memory-leaks.html)

[Unowned or Weak? Lifetime and Performance](https://www.uraimo.com/2016/10/27/unowned-or-weak-lifetime-and-performance/)

[Retain Cycles, Weak and Unowned in Swift](http://www.thomashanning.com/retain-cycles-weak-unowned-swift/)

[Life Cycle of a URL Session](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/NSURLSessionConcepts/NSURLSessionConcepts.html#//apple_ref/doc/uid/10000165i-CH2-SW1)

[When to use weak/unowned in a closure to prevent memory leak?](https://www.reddit.com/r/swift/comments/3cz7n0/when_to_use_weakunowned_in_a_closure_to_prevent/)

[Understanding memory leaks in closures](https://medium.com/compileswift/understanding-memory-leaks-in-closures-48207214cba)

[Where does the weak self go?](https://stackoverflow.com/questions/41991467/where-does-the-weak-self-go)

[Automatic Reference Counting](http://blog.stablekernel.com/how-to-prevent-memory-leaks-in-swift-closures)