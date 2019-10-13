---
layout: post
title:  "從神經元感知器到函數式到現實"
date:   2017-12-13 13:00:00 +0800
categories: Others
---
# 從神經元感知器到函數式到現實

> 本文是發散式隨筆與感想，并不是諸如 “如何學習神經網絡” 或 “函數式概念指導” 的文章，如果沒有興趣就可以離開了。


----------

今天偶然看到阮一峰老師的[「神經網絡入門」](http://www.ruanyifeng.com/blog/2017/07/neural-network.html)一文，讓我這個對神經網絡（neural network）一竅不通的人對這塊領域有了一窺之見，隨後又發現神經網絡模型和函數式的思路有異曲同工之妙，便有了這篇文章。


## 神經網絡與感知器

感知器（perceptron）是人工神經元最早的模型，其表達大概是這樣：

![感知器](http://upload-images.jianshu.io/upload_images/339176-7d37baa46e3c260a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


上圖基本表達為：x1, x2, x3 作為輸入參數，通過圓圈內的運算方法，輸出某種結果。這和人體中神經元接受刺激傳遞信號給神經中樞，再由神經中樞傳遞給人體讓人來做出反應的過程非常相似。
使用 Swift 來表達這一過程大致為：

    func execute<A, B, C, Output>(x1: A, x2: B, x3: C) -> Output {
      // handle x1, x2, x3
      ...
      return output
    }


A, B, C 對象都可以看成是某種值，通過其中的運算邏輯，輸出 Output。
那麼這個概念可以聯想到什麼呢？我的答案是函數式。

## Factor, Applicative, Monad

在接下來的說明當中，我會以 Swift 中的 Optional 對象為例來做解釋。

    enum Optional<T> {
      case some(T)
      case none
    }

什麼叫 Factor？對於一個對象來說，如果其內部實現了一個 map 函數，那麼我們可以稱其為 Factor。
因此，我們要對 Optional 來實現一個 map 函數：

    // Factor
    extension Optional {
      func map<U>(_ transform: (T) -> U) -> Optional<U> {
        switch self {
        case .some(let value):
          return .some(transform(value))
        case .none
          return .none
        }
      }
    }

也就是說，Optional 對象中的 some 狀態，取出 value 值（感知器參數 x1），經過 transform（感知器運算邏輯），輸出 Optional<U>（Output）。在這裡，不要把 transform  當成是參數對象，因為 map 函數是一個高階函數，不要被表象所欺騙。之後的 apply，flatMap 也都是這樣的情況。

鑒於 Applicative 與 Monad 的過程與 Factor 類似，同時本文目的並不是對於 Swift 函數式的介紹，因此不再做多解釋，只通過實現來思考一下相似度：

    // Applicative
    extension Optional {
      func apply<U>(_ transform: Optional<T -> U>) -> Optional<U> {
        switch transform {
        case .some(let method):
          return map(method)
        case .none
          return .none
        }
      }
    }
    
    // Monad
    extension Optional<U> {
      func flatMap<U>(_ transform: T -> Optional<U>) -> Optional<U> {
        switch self {
        case .some(value):
          return transform(value)
        case .none
          return .none
        }
      }
    }

我們可以發現，這些函數式概念的過程，和感知器概念是相似的，這真是很有意思。

當然，感知器不止那麼簡單，裡面還包含有權重（weight）和閾值（threshold）等其他概念，這裡並沒有說概念上絕對一致。


## 輸出與值類型

複雜的神經網絡會有多個感知器組成，比如深度學習（Deep Learning）：

![](http://upload-images.jianshu.io/upload_images/339176-2bb935d1654b36fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


遞歸神經網路（recurrent neural network）：

![](http://upload-images.jianshu.io/upload_images/339176-637e9de2e4dc3a64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


但是，和函數式的概念類似，多個感知器輸出的結果并不是共享（類似 reference type），而是向多個對象輸入值，這意味著參數不會在處理過程中發生改變（類似 value type 的 immutable）。


## 假如你有這樣一個產品思路

最近在做 100 Days of SICP 的實踐，腦洞一開，如果以 100 Days of Something 做一個 App，Model 的設計是不是其實可以按這樣的思路去實現呢？

假設每一天你都有可能做或不做這個實踐，而最終目的是依據你每天執行這個實踐所花的的時間和心力，計算出你的滿意程度和成果值，那麼可以實現為：

    struct DayDoSomething {
      let spentTime: Int
      let energy: Int // 0...100
      ...
    }
    
    struct DayCell {
      let dayDoSomething: DayDoSomething
      let weight: Weight
      let threshold: Threshold
      ...
    }
    
    func evaluate(dayCells: DayCell...) -> Result {
      // evaluating...
      ...
      return result
    }

## Curry

在函數式編程世界中是沒有物件的概念，一切對象皆是函數。

那麼使用函數式，如何處理感知器多參數輸入的問題呢？

我的想法是柯里化（Curry），也就是說，原本 x1, x2, x3 → Output 的過程就會變為：

    x1 -> x2 -> x3 -> Output

因此對於 x1 → x2,  x2 -> x3, x3 -> Output 來說，都可以把他們當成各種感知器所需要的實現，而這，就是函數。在 Curry 的過程，不僅解決了沒有物件概念問題，還使得人們面對每一步，都需要作出謹慎的思考，拆分思路。

## 面對實際生活

童話故事的美好結局總是過於簡單，不切實際：

    主人公 -> 遇到苦難 -> 征服苦難 -> 完美結局

同樣，普通人幻想愛情的過程也很類似：

    邀請吃飯看電影 -> 提高好感度 -> 英雄救美 -> 抱得美人歸 -> 美好生活

幻想成為富翁的過程：

    白手起家 ->勤奮工作 -> 獲得上司或市場的認可 -> 成為千萬富翁

為什麼這些想法都沒辦法實現呢？我們從本文得到啟發來看，至少可以得到兩個原因：

- 對於每一個細小的感知器實現，都沒有正確的參數、閾值和權重，從而得到錯誤的輸出，自然最終結果就不會正確
- 感知器具體實現不明確，沒有不斷拆分細節，達到小階段目標等等

另外還有一點想說的是，即使都有避免上面的原因，也可能依舊沒有成功。這其中原因還包含了正態分佈和對數正態曲線分佈問題，詳情請繼續參考阮老師的[「正態分佈為什麼常見？」](http://www.ruanyifeng.com/blog/2017/08/normal-distribution.html)一文。

*而點進文章看到現在的你，是不是也希望（觀看一篇文章） → （處理知識）→ （現實圓滿）呢？這和文章標題「從神經元感知器到函數式到現實」是不是有相似之處？*


## 題外話

最近打算買 Apple Pencil + iPad Pro → Drawing || Coding → Success! 

而我已經能預見結果了。


> 文章三個圖片均來自阮老師的文章，如侵權則刪。
----------
## 相關鏈接
- [「神經網絡入門」](http://www.ruanyifeng.com/blog/2017/07/neural-network.html)
- [「正態分佈為什麼常見？」](http://www.ruanyifeng.com/blog/2017/08/normal-distribution.html)