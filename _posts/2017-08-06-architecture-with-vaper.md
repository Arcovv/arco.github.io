---
layout: post
title:  "關於「Architecting iOS Apps with VIPER」自問自答與提煉"
date:   2017-08-06 20:00:00 +0800
categories: Architecture
---
[原文鏈接][1]。

非常好的文章，建議讀3次以上。每次讀之前都寫好你想問的問題並且回答出來。

---

## 自問自答

##### 什麼是 Interactor，它的功能和優點是什麼？
看這裡 [Interactor](#interactor)。

##### Presenter 的職責是什麼？它拆了 ViewController 的什麼東西？
這裡 [Presenter](#presenter)。

##### EventHandler 是一個協議對象，那麼誰應該實現這個協議？它又由誰持有？
我們以 _Add Module_ 為例，[這裡][5]看到， _VTDAddPresenter_ 實現了 _VTDAddModuleInterface_ 協議，而[這裡][6]表明，在 _VTDAddWireframe_ 中， _VTDAddViewController_ 的 _eventHandler_ 屬性被 _VTDAddWireframe_ 的 _addPresenter_ 所 set。
_EventHandler_ 被 _VTDAddWireframe_ 持有。
從這裡可以再複習一下，[Presenter 的概念](#presenter)，[Interactor 的概念](#interactor)。

##### Networking Layer 和 DataStore Layer 應該如何與 VIPER 協作？
在 _Interactor_ 中操作。中間可以再加上一層對應請求的 Manager/Request。

##### 是否能良好處理耦合的 UserManager 之類的問題？
這個問題我覺得我問的不太好。單例對象的問題可以參考一下[這篇文章][7]，基本上說明白單例對象確實是問題，但不是根本問題。根本問題還是在於開發者沒有理解和正確使用好這個概念，不然 Cocoa 中也不會出現大量使用這個概念的情況了。

那麼思考一下 _UserManager_ 的操作應該怎麼處理？把它當成一個 [Module/Feature](#Modules/Features-for-VIPER)，也就是屬於 _Business Logic_ 的部分，那就會包含 _Application Logic Layer_, _Interactor_, _Presenter_ 等；不太應該看成 _Application Logic Layer_ ，不然我覺得職責不清，會涉及到的其他 _layer_ 太多，不符合單一職責原則。

##### 和 MVVM 相比，它又什麼優勢，有什麼劣勢？
它多了 _Routing_ 一層，并將 _ViewModel_ 拆成 _Presenter_ 與 _Interactor_ 來處理。

劣勢就是有點繁瑣，一些概念不好理解，需要啃一點時間。

##### 如何與 RxSwift 之類的函數式框架共同使用？
_View_ 與 _Presenter_ 之間的可以放置，見[View 最後寫的一些東西](#View)， _Application Logic Layer_ 也可以使用。

##### Interactor 可以取得 entity 對象嗎？可以將其作為函數參數傳入其他部分嗎？
處於 _Entity_ 範圍內的各類對象，可以由 _Interactor_ 取得，取得過程需經過 _DataStore_ 或  _Networking Layer_ 等其他 _Application Logic Layer_ （因此， _Interactor_ 不可以直接操作 _NSManagedObject/Realm_ 之類的對象）。 _Interactor_ 不能將其傳入 _Presenter/ View_ 中。

##### JSON 對象由 Interactor 取得後，應該怎麼呈現？
_JSON_ 對象事實上由 _Networking Layer_ 取得後，傳遞給 _Interactor_ ， _Interactor_ 之後再傳入 _Presenter_ 或其他 _Logic Layer_ 中。

##### VIPER 的數據流程方式是什麼？
數據流動方向和文中的 [VIPER-Wireframe][4] 是一致的。

##### 如何處理 ViewController 之間傳遞數據的問題？
[見文章](#Rounting)。

---

## 文中的一些 Keywords
- Use Case
- Interactor
- Presenter
- Application logic layer: Data Store / Networking Layer
- Module / Feature

---

### 什麼叫 Use Case
_Apps are often implemented as a set of use cases. Use cases are also known as acceptance criteria, or behaviors, and describe what an app is meant to do. Maybe a list needs to be sortable by date, type, or name. That’s a use case. A use case is the layer of an application that is responsible for business logic. Use cases should be independent from the user interface implementation of them. They should also be small and well-defined. Deciding how to break down a complex app into smaller use cases is challenging and requires practice, but it’s a helpful way to limit the scope of each problem you are solving and each class that you are writing._

The use case also affects the user interface. Additionally, it’s important to consider how the use case fits together with other core components of an application, such as networking and data persistence. Components act like plugins to the use cases, and VIPER is a way of describing what the role of each of these components is and how they can interact with one another.

### Modules/Features for VIPER
Often when working with VIPER, you will find that a screen or set of screens tends to come together as a module. A module can be described in a few ways, but usually it’s best thought of as a feature. In a podcasting app, a module might be the audio player or the subscription browser. In our to-do list app, the list and add screens are each built as separate modules.

There are a few benefits to designing your app as a set of modules. One is that modules can have very clear and well-defined interfaces, as well as be independent of other modules. This makes it much easier to add/remove features, or to change the way your interface presents various modules to the user.

A module might include a common application logic layer of entities, interactors, and managers that can be used for multiple screens. This, of course, depends on the interaction between these screens and how similar they are.

Another benefit to building modules with VIPER is they become easier to extend to multiple form factors. Having the application logic for all of your use cases isolated at the Interactor layer allows you to focus on building the new user interface for tablet, phone, or Mac, while reusing your application layer.

Taking this a step further, the user interface for iPad apps may be able to reuse some of the views, view controllers, and presenters of the iPhone app. In this case, an iPad screen would be represented by 'super’ presenters and wireframes, which would compose the screen using existing presenters and wireframes that were written for the iPhone. Building and maintaining an app across multiple platforms can be quite challenging, but good architecture that promotes reuse across the model and application layer helps make this much easier.

### About VIPER
- View: displays what it is told to by the Presenter and relays user input back to the Presenter.
- Interactor: contains the business logic as specified by a use case.
- Presenter: contains view logic for preparing content for display (as received from the Interactor) and for reacting to user inputs (by requesting new data from the Interactor).
- Entity: contains basic model objects used by the Interactor.
- Routing: contains navigation logic for describing which screens are shown in which order.

This separation also conforms to the Single Responsibility Principle. The Interactor is responsible to the business analyst, the Presenter represents the interaction designer, and the View is responsible to the visual designer.

![VIPER-Wireframe](https://www.objc.io/images/issue-13/2014-06-07-viper-wireframe-76305b6d.png)

VIPER keeps CoreData/SQLite/Realm where it should be: at the data store layer.

#### VIPER for TDD
Building the Interactor first is a natural fit with TDD. If you develop the Interactor first, followed by the Presenter, you get to build out a suite of tests around those layers first and lay the foundation for implementing those use cases. You can iterate quickly on those classes, because you won’t have to interact with the UI in order to test them. Then, when you go to develop the View, you’ll have a working and tested logic and presentation layer to connect to it. By the time you finish developing the View, you might find that the first time you run the app everything just works, because all your passing tests tell you it will work.

---

### Interactor
An Interactor represents a single use case in the app. It contains the business logic to manipulate model objects (Entities) to carry out a specific task.

_The work done in an Interactor should be independent of any UI. The same Interactor could be used in an iOS app or an OS X app._

_If you are using Core Data, you will want your managed objects to remain behind your data layer. Interactors should not work with NSManagedObjects._

```objective-c
- (void)findUpcomingItems
{
    __weak typeof(self) welf = self;
    NSDate* today = [self.clock today];
    NSDate* endOfNextWeek = [[NSCalendar currentCalendar] dateForEndOfFollowingWeekWithDate:today];
    [self.dataManager todoItemsBetweenStartDate:today endDate:endOfNextWeek completionBlock:^(NSArray* todoItems) {
        [welf.output foundUpcomingItems:[welf upcomingItemsFromToDoItems:todoItems]];
    }];
}
```

#### Interactor for Networking Layer
Apps are usually much more compelling when they are connected to the network. But where should this networking take place and what should be responsible for initiating it? It’s typically up to the Interactor to initiate a network operation, but it won’t handle the networking code directly. It will ask a dependency, like a network manager or API client. The Interactor may have to aggregate data from multiple sources to provide the information needed to fulfill a use case. Then it’s up to the Presenter to take the data returned by the Interactor and format it for presentation.

#### Interactor for Data Store
As an Interactor applies its business logic, it will need to retrieve entities from the data store, manipulate the entities, and then put the updated entities back in the data store. The data store manages the persistence of the entities.

The Interactor should not know how to persist the entities either. Sometimes the Interactor may want to use a type of object called a data manager to facilitate its interaction with the data store. The data manager handles more of the store-specific types of operations, like creating fetch requests, building queries, etc. This allows the Interactor to focus more on application logic and not have to know anything about how entities are gathered or persisted.

#### Interactor for TDD
When using TDD to develop an Interactor, it is possible to switch out the production data store with a test double/mock. Not talking to a remote server (for a web service) or touching the disk (for a database) allows your tests to be faster and more repeatable.

One reason to keep the data store as a distinct layer with clear boundaries is that it allows you to delay choosing a specific persistence technology. If your data store is a single class, you can start your app with a basic persistence strategy, and then upgrade to SQLite or Core Data later if and when it makes sense to do so, all without changing anything else in your application’s code base.

The Interactor contains pure logic that is independent of any UI, which makes it easy to drive with tests.

By using TDD to test drive the API for the Interactor, you will have a better understanding of the relationship between the UI and the use case.

---

### Entity
Entities are the model objects manipulated by an Interactor. 

_Entities are only manipulated by the Interactor. The Interactor never passes entities to the presentation layer (i.e. Presenter)._

```objective-c
@interface VTDTodoItem : NSObject

@property (nonatomic, strong)   NSDate*     dueDate;
@property (nonatomic, copy)     NSString*   name;

+ (instancetype)todoItemWithDueDate:(NSDate*)dueDate name:(NSString*)name;

@end
```

#### Entity for Data Store
Entities do not know about the data store, so entities do not know how to persist themselves.

---

### Presenter
The Presenter is a PONSO that mainly consists of logic to drive the UI. It knows when to present the user interface. It gathers input from user interactions so it can update the UI and send requests to an Interactor.

_When the user taps the + button to add a new to-do item, addNewEntry gets called. For this action, the Presenter asks the wireframe to present the UI for adding a new item:_

```objective-c
- (void)addNewEntry
{
    [self.listWireframe presentAddInterface];
}
```

The Presenter also receives results from an Interactor and converts the results into a form that is efficient to display in a View.

```objective-c
- (void)foundUpcomingItems:(NSArray*)upcomingItems
{
    if ([upcomingItems count] == 0)
    {
        [self.userInterface showNoContentMessage];
    }
    else
    {
        [self updateUserInterfaceWithUpcomingItems:upcomingItems];
    }
}
```

_Entities are never passed from the Interactor to the Presenter. Instead, simple data structures that have no behavior are passed from the Interactor to the Presenter. This prevents any ‘real work’ from being done in the Presenter. The Presenter can only prepare the data for display in the View._

#### Presenter for TDD
The Presenter contains logic to prepare data for display and is independent of any UIKit widgets.

---

### View
The View is passive. It waits for the Presenter to give it content to display; it never asks the Presenter for data. Methods defined for a View (e.g. LoginView for a login screen) should allow a Presenter to communicate at a higher level of abstraction, expressed in terms of its content, and not how that content is to be displayed. 

_The Presenter does not know about the existence of UILabel, UIButton, etc. The Presenter only knows about the content it maintains and when it should be displayed. It is up to the View to determine how the content is displayed._

The View is an abstract interface, defined in Objective-C with a protocol. A UIViewController or one of its subclasses will implement the View protocol. For example, the 'add’ screen from our example has the following interface:

```objective-c
@protocol VTDAddViewInterface <NSObject>

- (void)setEntryName:(NSString *)name;
- (void)setEntryDueDate:(NSDate *)date;

@end
```

To keep our view controllers lean, we need to give them a way to inform interested parties when a user takes certain actions. The view controller shouldn’t be making decisions based on these actions, but it should pass these events along to something that can.
In our example, Add View Controller has an event handler property that conforms to the following interface:

```objective-c
@protocol VTDAddModuleInterface <NSObject>

- (void)cancelAddAction;
- (void)saveAddActionWithName:(NSString *)name dueDate:(NSDate *)dueDate

@end
```

When the user taps on the cancel button, the view controller tells this event handler that the user has indicated that it should cancel the add action. That way, the event handler can take care of dismissing the add view controller and telling the list view to update.

_The boundary between the View and the Presenter is also a great place for ReactiveCocoa. In this example, the view controller could also provide methods to return signals that represent button actions. This would allow the Presenter to easily respond to those signals without breaking separation of responsibilities._

#### View for Storyboard
The compromise we tend to make is to choose not to use segues. There may be some cases where using the segue makes sense, but the danger with segues is they make it very difficult to keep the separation between screens – as well as between UI and application logic – intact. As a rule of thumb, we try not to use segues if implementing the prepareForSegue method appears necessary.

Otherwise, storyboards are a great way to implement the layout for your user interface, especially while using Auto Layout.

---

### Rounting
In VIPER, the responsibility for Routing is shared between two objects: the Presenter, and the wireframe. A wireframe object owns the UIWindow, UINavigationController, UIViewController, etc. It is responsible for creating a View/ViewController and installing it in the window.

Since the Presenter contains the logic to react to user inputs, it is the Presenter that knows when to navigate to another screen, and which screen to navigate to. Meanwhile, the wireframe knows how to navigate. So, the Presenter will use the wireframe to perform the navigation. Together, they describe a route from one screen to the next.

The wireframe is also an obvious place to handle navigation transition animations.

Since the wireframe is responsible for performing the transition, it becomes the transitioning delegate for the add view controller and can return the appropriate transition animations.

```objective-c
@implementation VTDAddWireframe

- (void)presentAddInterfaceFromViewController:(UIViewController *)viewController 
{
    VTDAddViewController *addViewController = [self addViewController];
    addViewController.eventHandler = self.addPresenter;
    addViewController.modalPresentationStyle = UIModalPresentationCustom;
    addViewController.transitioningDelegate = self;

    [viewController presentViewController:addViewController animated:YES completion:nil];

    self.presentedViewController = viewController;
}

#pragma mark - UIViewControllerTransitioningDelegate Methods

- (id<UIViewControllerAnimatedTransitioning>)animationControllerForDismissedController:(UIViewController *)dismissed 
{
    return [[VTDAddDismissalTransition alloc] init];
}

- (id<UIViewControllerAnimatedTransitioning>)animationControllerForPresentedController:(UIViewController *)presented presentingController:(UIViewController *)presenting sourceController:(UIViewController *) source 
{
    return [[VTDAddPresentationTransition alloc] init];
}

@end
```

---

## 後話
VIPER / MVVM / MVP / MVC 等各類 Architecture 所描述的是範圍，屬於抽象的思維。

舉 _MVCS_ 來說，雖然 _Service_ 屬於 Networking Layer 的部分，但對於 Networking Layer 來說，并非一定要求只有一個 singleton 對象：既可以是使用 Factory Design Pattern 的方式來構造，還可以使用類似 [YTKNetwork][2] 的思路來構造出許多 Networking Request 對象，也可以只是 Singleton/Shared。

同樣，文中提到的關於 Interactor 與 Entity 互交的時候，不只是經過 DataStore，對於每一個 DataStore 的操作都被封裝在不同的 DataManager 中，如 [VTDListDataManager][3]，形成了 YTKNetwork 封裝 Request 的思路。

[1]: https://www.objc.io/issues/13-architecture/viper/
[2]: https://github.com/yuantiku/YTKNetwork
[3]: https://github.com/objcio/issue-13-viper/blob/master/VIPER%20TODO/Classes/Modules/List/Application%20Logic/Manager/VTDListDataManager.m
[4]: https://www.objc.io/images/issue-13/2014-06-07-viper-wireframe-76305b6d.png
[5]: https://github.com/objcio/issue-13-viper/blob/master/VIPER%20TODO/Classes/Modules/Add/User%20Interface/Presenter/VTDAddPresenter.h
[6]: https://github.com/objcio/issue-13-viper/blob/master/VIPER%20TODO/Classes/Modules/Add/User%20Interface/Wireframe/VTDAddWireframe.m
[7]: https://cocoacasts.com/are-singletons-bad/?utm_content=buffer9bd49&utm_medium=social&utm_source=twitter.com&utm_campaign=buffer