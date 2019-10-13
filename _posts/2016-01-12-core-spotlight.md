---
layout: post
title:  "如何在iOS 9中使用Core Spotlight框架"
date:   2016-01-12 20:00:00 +0800
categories: CoreSpotlight
---
如何在iOS 9中使用Core Spotlight框架

每一次新iOS版本的到來都會帶給全世界所有的開發者們新“東西”并進階已存在的技術。無疑，最新的版本iOS 9，也不會置身于這個傳統之外。開發者們再一次有了新的框架與API以讓他們的App獲得更高水平。其中一個新東西就是 <strong>Core Spotlight</strong> 框架，它靜靜地等待在那邊讓開發者們使用它新的API。

Core Spotlight(CS)框架，是更大型API的集合，以 <em>Search APIs</em> 聞名，這給了開發者們機會去顯著提高他們App被訪問的可發現性，可見性和易用性，這是在以前iOS版本中完全不可能被做到的。Search APIs讓使用者與開發者的聯繫更加緊密，前者可以以更新更快的方式來到達後者的App，後者也可以從前者那獲得更多更迅速的響應。除了Core Spotlight，還有其他新的搜索特性包括在iOS 9中（這裡只是做個介紹）：

<ol>
<li>新的方法和屬性存在與 <em>NSUserActivity</em> 類中（這是用來負責存儲App狀態以為了能讓App在稍後的時間里恢復）。</li>
<li><em>web markup</em> 是用來讓網頁在裝置中被搜索到。</li>
<li><em>universal links</em> 是讓App在網頁鏈接中被直接啟動。</li>
</ol>

我們不會全部處理上述所說的三個方面，而我們將會看到對於Core Spotlight框架的細節。在那之前，我們需要介紹一下這個框架。

<img alt="core-spotlight-framework" src="http://www.appcoda.com/wp-content/uploads/2015/12/core-spotlight-framework.jpg"/>

<a href="https://developer.apple.com/library/ios/documentation/CoreSpotlight/Reference/CoreSpotlight_Framework/">Core Spotlight framework</a> 讓App中的數據 <em>可搜索化</em> ，以及隨後對於系統反饋的搜索結果回傳到App，并在其中打開。這讓使用者可以在第一時間從應用中搜尋到數據（而以前只能搜索到Apple原生的應用的資訊），並且相互作用，這是相當令人印象深刻的變革。按說用戶可以與相關的自定義應用的搜索結果進行互動，而我的意思是，不僅應用程序將因為搜索結果記錄而自動啟動，開發者也同時將用戶吸引到特定視圖控制器(specific view controllers)，這些最合適最恰當的數據就顯示在Spotlight中。

<!--more-->

其實以開發者的角度來看，結合Core Spotlight框架，並使用其提供的API并不是一個複雜的過程。你即將會發現在本教程中中，需要做到的代碼只有幾行。這其中的核心部份是讓開發者去“告知”iOS關於應用數據的 <em>索引(index)</em> 。

正因本次教程是Core Spotlight框架專屬的，我并不打算在本次介紹中深入其他細節。如果你樂於研究其中是如何實現一些東西的我個人是十分高興的，也請你繼續閱讀下去。我保證你在閱讀到最後會十分滿意這是如何簡單讓你的應用能夠在Spotlight中被識別。

<h2>關於範例App</h2>

我們將一如既往的通過使用一個範例應用來深入了解我們所提到的話題中的細節。在這裡，我們將放置一些數據集合到這個應用中，然後這些數據會被裝置（或模擬器）中的Spotlight所搜尋到。但是首先，我們必須要先來了解一下我們這個應用的一些細節。

這個範例是去展示一些 <em>電影</em> 資訊，比如其摘要，導演，明星陣容，評級等等。所有這些電影資訊會被放置在一個tableview中，當按下其中一列時，某部電影的細節資訊將會被展示在一個新的view controller中。就是這樣，這些東西就可以讓我們可以看到Core Spotlight的API是如何運作的。另外供你參考，我們數據中的這些資源是來自 <em><a href="http://www.imdb.com">International Movie Database (IMDB)</a></em> 。我是從這個地方取得樣品數據的。

你可以通過以下動畫來對我們的範例App有一個第一印象。

<img alt="Core Spotlight Demo" src="http://www.appcoda.com/wp-content/uploads/2015/12/t46_1_app_working-compressor.gif"/>

我們在這個教程中有兩個目標。最重要的一個就是讓所有包含在應用中的電影數據可以被Spotlight中搜尋到，然後，使用者會在Spotlight中輸入一些關鍵字來查詢到我們應用中的電影資訊並且展示給他們看。我們的任務與責任就是去決定并設置這些關鍵字。

通過按下一個電影的搜索結果，應用將會被打開，然後我們的第二個目標來了。如果我們沒有進行任何動作，默認的view controller會被加載並且顯示給使用者看，就是那個tableview包含了全部的電影列表的view controller。但這個使用者體驗聽起來並不太妙。一個更好的做法是，我們的應用應該展示那個在Spotlight中被選擇的電影細節，而這也是我們最終要做的。總之，我們不僅會讓我們的電影數據在Spotlight中被搜索到，同時也會當Spotlight中的項目被點擊時在應用中展示相關的細節。以下例子會讓這些更清楚一點：

<img alt="Core Spotlight Final Demo" src="http://www.appcoda.com/wp-content/uploads/2015/12/t46_2_final_sample-compressor.gif"/>

為了不要浪費更多的時間，你可以<a href="https://www.dropbox.com/s/2oge5z8q7u4r11m/SpotItStarter.zip?dl=0">在這裡下載一個project</a>。你會在裡面看到這些東西：

<ul>
<li>UI的部份通過所有必要的IBOutlet屬性已經被做好了。</li>
<li>對tableview的一個最小實現。</li>
<li>所有的電影數據都存在于一個.plist文件中。另外，還有一些圖片用來搭配對應的電影（一共5個）。</li>
</ul>

你可能想知道這個列表的電影數據屬性是屬於什麼類別的，你可以通過以下一個屏幕截圖來了解到這些信息：

<img alt="t46_3_movie_plist_sample" src="http://www.appcoda.com/wp-content/uploads/2015/12/t46_3_movie_plist_sample.png"/>

我們將會在看到Core Spotlight API的更多細節之前，先做兩個不同的任務：

<ol>
<li>我們先要載入和放置電影數據到tableview中。</li>
<li>我們將展示一個被選擇的電影資訊到detail view controller中。</li>
</ol>

我并沒有在起始項目中實現上述任務，即使這會讓我們更快進入這方面的話題，原因是：我非常自信通過了解這個範例應用和樣品的數據，會讓你有更多的對於具體數據是如何被Spotlight所搜索到有更多的認識。不用擔心，前期工作不多，可以做的很快。

<h2>加載和展示樣品的數據</h2>

如果你已經下載好了起始項目并看了關於電影數據的plist檔，我們就開始吧。在 <em>MoviesData.plist</em> 文件中你會發現一共有5個入口，每一個樣品電影都是隨機在IMDB網站上取得的。我們的第一個任務是從.plist檔中加載數據到一個array里，然後把array放進tableview

直接進入代碼里，打開 <em>ViewController.swift</em> 檔，這個我們主要運用到的文件，然後在class裡面的最上方宣告以下屬性：

<pre lang="swift">var moviesInfo: NSMutableArray!
</pre>

所有電影都會被加載進這個array中，每一個電影都會以存在于屬性列表文件中的屬性來代表成一個dictionary，裡面包含了keys和values.

讓我們先寫一個小型自定義方法，在該方法里，將會進行數據加載。我們首先確保這個屬性列表文件是存在的，如此之後，我們才在這個array中初始化文件中的內容：

<pre lang="swift">func loadMoviesInfo() {
    if let path = NSBundle.mainBundle().pathForResource("MoviesData", ofType: "plist") {
        moviesInfo = NSMutableArray(contentsOfFile: path)
    }
}
</pre>

接下來，我們必須在 <em>viewDidLoad()</em> 方法中呼叫它。你必須確保這個呼叫必須在 <em>configureTableView()</em> 方法發生之前：

<pre lang="swift">override func viewDidLoad() {
    super.viewDidLoad()

	// Load the movies data from the file.
	loadMoviesInfo()

	configureTableView()
	navigationItem.title = "Movies"
}
</pre>

注意以上創建的自定義方法當然可以寫在 <em>viewDidLoad()</em> 中，但是為了有整潔的代碼風格，我們這種方式可以讓代碼變得更輕更好。

你應該知道電影數據要在應用啟動前被加載，然後我們才能夠繼續將當前的tableview實例化去展示我們的電影數據。這裡不會再做其他事情了。我們將會依據電影來決定列的數量，然後將屬性數據展示在tableview的cell上。

很明顯的，列的數量應該是和電影的數量相等的。但是，我們應該不應該忘記首先需要去保證確實存在可展示的電影，不然整個應用將會瞬間崩潰，因為在這種情況下文件內容不會被加載到array中。

<pre lang="swift">func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    if moviesInfo != nil {
        return moviesInfo.count
    }

	return 0
}
</pre>

最後，讓我們將這些電影數據展示出來。在這個範例應用的起始項目上，你可以找到一個屬於 <em>UITableViewCell</em> 類的子類叫做 <em>MovieSummaryCell</em> ，以及對應的 <em>.xib</em> 檔去表示單個電影cell：

<img src="http://www.appcoda.com/wp-content/uploads/2015/12/t46_4_custom_cell_ib.png" alt="t46_4_custom_cell_ib" class="aligncenter size-full wp-image-6868" />

每一個cell都會展示出圖片，標題，描述的部份以及每一部電影的評價。所有的UI控制器都已經和IBOutlet屬性連接，你可以在 <em>MovieSummaryCell.swift</em> 檔中找到他們。

<pre lang="swift">@IBOutlet weak var imgMovieImage: UIImageView!

@IBOutlet weak var lblTitle: UILabel!

@IBOutlet weak var lblDescription: UILabel!

@IBOutlet weak var lblRating: UILabel!
</pre>

上述名稱都暗示了每個屬性的目的。既然我們已經看到了他們，那我們就開始讓他們與我們電影的細節關聯上吧。回到 <em>ViewController.swift</em> 中，通過接下來的程式碼來更新tableview中的方法：

<pre lang="swift">func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCellWithIdentifier("idCellMovieSummary", forIndexPath: indexPath) as! MovieSummaryCell

	let currentMovieInfo = moviesInfo[indexPath.row] as! [String: String]

	cell.lblTitle.text = currentMovieInfo["Title"]!
	cell.lblDescription.text = currentMovieInfo["Description"]!
	cell.lblRating.text = currentMovieInfo["Rating"]!
	cell.imgMovieImage.image = UIImage(named: 	currentMovieInfo["Image"]!)

	return cell

}
</pre>

對於 <em>currentMovieInfo</em> 字典的使用並非是必須的，然而這讓整個程式碼看起來清晰易讀。

你可以先執行一次這個應用，你會看到電影的細節都被tableview所監聽到。到目前為止我們都做了大家很熟悉的東西，現在讓我們直接進入第二預備步驟：將電影的細節部份展示出來。

<h2>展示數據細節</h2>

我們將在 <em>MovieDetailsViewController</em> 類中展示每一個從屬於 <em>ViewController</em> 類的tableview中被選擇的電影的細節。各自的場景都已經在Interface Builder中存在了，所以我們將做兩件事情：從 <em>ViewController</em> 傳遞合適的電影字典到這個類中，然後放置那些字典裡面的值到合適的UI控制器上，就是那些你能看到的所有的已經被宣告和連接的IBOutlet屬性。

為了與字典進行對話，讓我們接下來在 <em>MovieDetailsViewController</em> 類內面的最上方做一個宣告：

<pre lang="swift">var movieInfo: [String: String]!
</pre>

讓我們馬上回到 <em>ViewController</em> 檔中，去看看我們接下來當電影列被觸發時應該做什麼。當這件事情發生時，我們想要知道是哪一個列被按下。我們從 <em>moviesInfo</em> 陣列中挑選合適的字典傳遞到下一個view controller上，同時，一個叫做 <em>idSegueShowMovieDetails</em> 的segue將會被執行。取得該列的索引並不麻煩，我們只需要一個自定義的屬性來存放它就好了。因此我們在 <em>ViewController</em> 類里還需要宣告這個：

<pre lang="swift">var selectedMovieIndex: Int!
</pre>

然後，我們需要去處理tableview中的列被選擇時需要執行的方法：

<pre lang="swift">func tableView(tableView: UITableView, didSelectRowAtIndexPath indexPath: NSIndexPath) {
    selectedMovieIndex = indexPath.row
    performSegueWithIdentifier("idSegueShowMovieDetails", sender: self)
}
</pre>

我們一共做了兩件簡單的事情：我們先在我們自己的屬性中，儲存了被按下的列的索引，然後執行segue去讓我們的電影細節被展示出來。但這還不夠，我們還沒從 <em>moviesInfo</em> 陣列中挑選合適的電影字典，我們也沒有傳遞任何東西到 <em>MovieDetailsViewController</em> 中。我們應該怎麼做？答案是去覆寫 <em>prepareForSegue:sender:</em> 方法然後做我剛剛說的事情，我們看看是怎麼做到的：

<pre lang="swift">override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
    if let identifier = segue.identifier {
        if identifier == "idSegueShowMovieDetails" {
            let movieDetailsViewController = segue.destinationViewController as! MovieDetailsViewController
            movieDetailsViewController.movieInfo = moviesInfo[selectedMovieIndex] as! [String : String]
        }
    }
}
</pre>

這很簡單，我們通過segue的屬性  <em>destinationViewController</em> 取得到了 <em>MovieDetailsViewController</em> 的實例變量，然後我們將合適的電影字典分配到了剛剛在這部份開始宣告的屬性 <em>movieInfo</em> 上。

現在，再一次打開 <em>MovieDetailsViewController.swift</em> 檔，我們只要宣告一個自定的方法。在這方面里，我們將從 <em>movieInfo</em> 字典中分配合適的值到合適的UI控制器中，然後這一部份就基本告一段落了。這下面的代碼實現并不複雜，所以我不做其他解釋了：

<pre lang="swift">func populateMovieInfo() {
    lblTitle.text = movieInfo["Title"]!
    lblCategory.text = movieInfo["Category"]!
    lblDescription.text = movieInfo["Description"]!
    lblDirector.text = movieInfo["Director"]!
    lblStars.text = movieInfo["Stars"]!
    lblRating.text = movieInfo["Rating"]!
    imgMovieImage.image = UIImage(named: movieInfo["Image"]!)
}
</pre>

隨後，在 <em>viewWillAppear:</em> 方法中呼叫上述方法就好啦：

<pre lang="swift">override func viewWillAppear(animated: Bool) {
    ...

	if movieInfo != nil {
   		 populateMovieInfo()
	}
}
</pre>

這部份又告一段落啦，你可以再試一次這個App，然後去看看你從tableview中選擇的每一個電影細節吧。

<h2>對於Spotlight的索引數據</h2>

通過使用iOS 9中的Core Spotlight框架，任何一個應用的數據都可以通過Spotlight被搜索到。關鍵就是從Core Spotlight的API中特別詢問我們數據的 <em>索引</em> 是什麼，這樣數據就可以給使用者正確地尋找到了。但是我們的應用中，都還沒有讓我們的數據去使用CS API。我們需要去為我們的數據以特別的方式準備和提供那些API。

說的更清楚一點，那些我們想要通過Spotlight被搜尋到的數據都必須在 <em>CSSearchableItem</em> 物件中被描述出來，然後將它們以陣列的方式組合并用CS API做出索引。 單一 <em>CSSearchableItem</em> 物件包含了一些屬性讓iOS以更快更乾淨的方式來找到每一個能被搜尋的項目，比如說什麼數據應該在搜索的時候被展示出來（例如電影的名字，它的圖片和描述），還有是什麼關鍵字導致我們應用中的數據被Spotlight搜尋到之類。所有這些屬性都包含在一個叫做 <em>CSSearchableItemAttributeSet</em> 的物件中，它可以提供許多屬性讓我們去分配我們需要的值。如果你需要，我給你<a href="https://developer.apple.com/library/prerelease/ios/documentation/CoreSpotlight/Reference/CSSearchableItemAttributeSet_Class/index.html">官方文檔鏈接</a>，這樣你就可以看到所有被支持的屬性啦。

為Spotlight中的數據設置索引是最後才要做的事情。通常來說按照以下流程：

<ol>
<li>為每一段數據設置它們獨有的屬性，比如一個電影。（ <em>CSSearchableItemAttributeSet</em> ）</li>
<li>用之前的屬性為每一段數據初始化一個可被搜索的項目。（ <em>CSSearchableItem</em> ）</li>
<li>把這些可被搜索的項目都放入一個陣列中。</li>
<li>為陣列中的每一個項目設置一個索引。</li>
</ol>

我們將一步一步來。為了完成我們的目的，我們將在 <em>ViewController.swift</em> 檔中創立一個叫做 <em>setupSearchableContent()</em> 的自定義方法。在我們這部份最後的實例變量中你會發現讓你的數據變得可搜索化並不難。但是我們不會直接把整個方法一次實現。我會將代碼進行拆解，讓你可以足夠簡單地消化下去。這一點也不多。

在我們實現我們的方法之前，我們必須先import兩個框架：

<pre lang="swift">import CoreSpotlight
import MobileCoreServices
</pre>

我們可以開始給我們的新方法下定義了。在方法開始宣告一個用來收集可搜索化項目的集合：

<pre lang="swift">func setupSearchableContent() {
    var searchableItems = [CSSearchableItem]()
}
</pre>

在迴圈中我們開始分配每一個電影：

<pre lang="swift">func setupSearchableContent() {
    var searchableItems = [CSSearchableItem]()

    for i in 0...(moviesInfo.count - 1) {
        let movie = moviesInfo[i] as! [String: String]
    }
}
</pre>

對每個電影，我們都會創建一個 <em>CSSearchableItemAttributeSet</em> 物件，然後設置那些當在Spotlight被搜索時，我們想展示出來的搜索結果的屬性。在我們的範例中，我們會明確出電影的標題，描述和圖片來作為我們想給使用者看到的數據片段。

<pre lang="swift">func setupSearchableContent() {
    var searchableItems = [CSSearchableItem]()

    for i in 0...(moviesInfo.count - 1) {
        let movie = moviesInfo[i] as! [String: String]

        let searchableItemAttributeSet = CSSearchableItemAttributeSet(itemContentType: kUTTypeText as String)

        // Set the title.
        searchableItemAttributeSet.title = movie["Title"]!

        // Set the movie image.
        let imagePathParts = movie["Image"]!.componentsSeparatedByString(".")
        searchableItemAttributeSet.thumbnailURL = NSBundle.mainBundle().URLForResource(imagePathParts[0], withExtension: imagePathParts[1])

        // Set the description.
        searchableItemAttributeSet.contentDescription = movie["Description"]!
    }
}
</pre>

注意上面代碼片段中我們是怎麼將電影圖片設置為屬性的。其實有兩種方法來實現：無論是使用圖片的URL地址，或者為圖片提供一個 <em>NSData</em> 物件。對我們來說最簡單的方式就是為每一個電影圖片提供URL地址，因為我們知道這些圖片都存在在應用包(application bundle)中。然而，這個方式需要打破每個圖像文件的實際名和擴展名，所以我們將使用String類中的 <em>componentsSeparatedByString:</em>  方法來分開他們。剩下的都不難去理解。

現在我們該為那些想在Spotlight中被找到的應用數據設置關鍵字了。在想一些特殊的關鍵字之前，你必須明白你的決定是你的應用在Spotlight和用戶被發現的最終關鍵。在範例中我們將設置關鍵字包括電影的類型和它的明星。

<pre lang="swift">func setupSearchableContent() {
    var searchableItems = [CSSearchableItem]()

    for i in 0...(moviesInfo.count - 1) {
        ...

        var keywords = [String]()
        let movieCategories = movie["Category"]!.componentsSeparatedByString(", ")
        for movieCategory in movieCategories {
            keywords.append(movieCategory)
        }

        let stars = movie["Stars"]!.componentsSeparatedByString(", ")
        for star in stars {
            keywords.append(star)
        }

        searchableItemAttributeSet.keywords = keywords
    }
}
</pre>

要記得，電影的分類是被作為一個單個字串表述在 <em>MoviesData.plist</em> 檔中，因此它們會被空格所隔開。所以我們必須將分類的字串值打破然後讓它們分別儲存在 <em>movieCategories</em> 陣列中以方便訪問。然後將每一個值用一個inner迴圈加入 <em>keywords</em> 陣列中。我們對電影中的明星屬性做了一樣合適的步驟，并再一次地打破明星中的字串值，同樣加入keyword陣列。

最重要的一列就是最後一行，我們為每一個電影將我們的關鍵字加入相關屬性中。把這行忘記了意味著在Spotl中對我們的應用搜索不到任何結果。

現在我們已經為Spotlight設置了屬性和關鍵字，是時候去初始化一個可搜索化的項目并將其加入 <em>searchableItems</em> 陣列中了。

<pre lang="swift">func setupSearchableContent() {
    var searchableItems = [CSSearchableItem]()

    for i in 0...(moviesInfo.count - 1) {
        ...

        let searchableItem = CSSearchableItem(uniqueIdentifier: "com.appcoda.SpotIt.\(i)", domainIdentifier: "movies", attributeSet: searchableItemAttributeSet)

        searchableItems.append(searchableItem)
    }
}
</pre>

以上初始化接受了三個參數：

<ul>
<li><em>uniqueIdentifier</em>: 這個參數對在Spotlight中的可搜索化項目要求一個獨一無二的識別標誌。你可以用你喜歡的方式構成這個識別標誌，但注意一個小細節：在這個例子中我們對識別標誌加入當前電影的索引值，我們就會需要到它在隨後展示電影的細節中匹配的索引值。說的更明白點，這是一個好的主意在直向的數據中包含這個識別標誌，以顯示數據的細節。你再來會更明白這個有用的電影索引值。</li>
<li><em>domainIdentifier</em>: 用這個參數將可搜索化項目組成群組。</li>
<li><em>attributeSet</em>: 這個屬性用來設置那些我們剛剛分配值的物件。</li>
</ul>

最後，新的可搜索化項目會被加進 <em>searchableItems</em> 陣列中。

最後我們還有一步要做：運用Core Spotlight API去索引這些項目。這在 <em>for</em> 迴圈以外發生：

<pre lang="swift">func setupSearchableContent() {
    ...

    CSSearchableIndex.defaultSearchableIndex().indexSearchableItems(searchableItems) { (error) -> Void in
        if error != nil {
            print(error?.localizedDescription)
        }
    }
}
</pre>

上述方法已經全部完成，最後我們需要在 <em>viewDidLoad()</em> 中呼叫它：

<pre lang="swift">override func viewDidLoad() {
    ...

    setupSearchableContent()
}
</pre>


我們已經準備好第一次使用Spotlight來搜索電影了。運行這個App，離開它然後在Spotlight中輸入任何我們設置的關鍵字。你會發現搜尋結果會出現在你眼前，通過點擊任何一個結果，應用都會被自動啟動。

<img alt="Core Spotlight Demo 2" src="http://www.appcoda.com/wp-content/uploads/2015/12/t46_5_first_search-compressor.gif"/>

<h2>實現指向性登入</h2>

我們可以通過在Spotlight中搜索到我們的應用中的電影數據，實在是令人印象深刻的事情，但是，還有個地方做的不夠好。到目前為止，在Spotlight觸發項目，應用會進行啟動並且  <em>ViewController</em> 會展示到我們面前，但我們的目標是必須是讓被選擇的電影項目指向detail view controller中，並且看到更多的細節。

主要要做的事情是覆寫UIKit框架中的一個叫做 <em>restoreUserActivityState:</em> 的方法，然後在Spotlight中處理被選擇的項目。我們最終想達到的，是從可搜索化項目中用識別標誌取出<em>moviesInfo</em> 陣列中電影的索引（如果你還記得，我們在先前的部份中創建了動態性的識別標誌），然後我們使用它傳遞適合的電影字典并展示到 <em>MovieDetailsViewController</em> 視圖控制器中。

上述方法的參數是一個 <em>NSUserActivity</em> 物件。這個物件有一個字典屬性叫做 <em>userInfo</em> ，然後這個字典包含了在Spotlight中被選擇的可搜索化項目中的識別標誌。通過這個識別標誌，我們會從 <em>moviesInfo</em> 陣列中取出電影的索引值，然後展示整個細節視圖控制器。就這樣啦。

讓我們看看整個實現的過程：

<pre lang="swift">override func restoreUserActivityState(activity: NSUserActivity) {
    if activity.activityType == CSSearchableItemActionType {
        if let userInfo = activity.userInfo {
            let selectedMovie = userInfo[CSSearchableItemActivityIdentifier] as! String
            selectedMovieIndex = Int(selectedMovie.componentsSeparatedByString(".").last!)
            performSegueWithIdentifier("idSegueShowMovieDetails", sender: self)
        }
    }
}
</pre>

如你所見，我們必選先檢查 <em>activity type</em> 是否符合 <em>CSSearchableItemActionType</em> 的類型。不過其實這在這個範例App中并不重要，但是如果你在你的應用中處理多個 <em>NSUserActivity</em> 物件，這就是你不改忘記去做的東西（比如說， <em>Handoff</em> 特性首先就在iOS 8中使用 <em>NSUserActivity</em> 類）。識別標誌在 <em>userInfo</em> 字典中是一個字串值，一旦我們得到這個值，我們會打破其自身與點符號的組合，然後我們訪問最後一個物件，就是那個在電影集合中被選擇電影的索引值。剩下就簡單多了：我們分配出 <em>selectedMovieIndex</em> 屬性的索引值，然後執行segue。我們先前實現的東西會解決剩下的所有問題。

現在到 <em>AppDelegate.swift</em> 檔中。我們必須在其中去實現一個目前還沒存在的代理方法（delegate function）。當關聯到我們App的項目被觸發時，該方法每次都會被呼叫到，我們的責任就是實現這個被呼叫的方法，而這東西當然是通過user activity來實現的。讓我們看下去：

<pre lang="swift">func application(application: UIApplication, continueUserActivity userActivity: NSUserActivity, restorationHandler: ([AnyObject]?) -> Void) -> Bool {
    let viewController = (window?.rootViewController as! UINavigationController).viewControllers[0] as! ViewController
    viewController.restoreUserActivityState(userActivity)

    return true
}
</pre>

上述代碼片段中，第一件事情就是通過 <em>window</em> 屬性去訪問 <em>ViewController</em> 控制器以優先恢復用戶活動狀態。或者，你可以使用 <em>NSNotificationCenter</em> 然後反過來在 <em>ViewController</em> 類中處理并發出一個自定義通知來代替上述方法，但是上面那個方法更直接一點。

全部搞定！我們的範例App已經完成了，再運行一次然後看看當你觸發Spotlight的項目會發生什麼吧。

<img alt="Core Spotlight Final Demo" src="http://www.appcoda.com/wp-content/uploads/2015/12/t46_2_final_sample-compressor.gif"/>

<h2>總結</h2>

對開發者來說，在iOS 9中新的搜索API是非常有前景的，因為這允許應用能夠更容易被用戶發現和訪問。在這個教程中，我們通過這些方式去建立應用數據的索引，以便在Spotlight被發現，以及一個被選擇的項目是如何通過處理應用中的特殊數據以展示給使用者看。在你的應用中實現這些與衆不同的特性是會提高用戶體驗的，所以，這是你應該嚴肅考慮如何加進你當前和未來項目的重要API。我們再一次的迎來最後，我由衷希望你能在這個教程中找到你需要的幫助。玩的愉快！

如果你需要的話，你可以<a href="https://github.com/appcoda/CoreSpotlightDemo">從Github下載整個完整的Xcode項目</a>.

<h2>譯者介紹</h2>

謝岳庭，中文系出身，自學iOS開發，有著异於常人的求知慾。

你可以在Twitter或<a href="http://www.appcoda.com.tw/forum/">AppCoda討論區</a>中聯繫我并隨時歡迎你與我交流。

My Twitter: <a href="https://twitter.com/Lanaya_HSIEH">@Lanaya_HSIEH</a>