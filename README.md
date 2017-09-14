# Swift Best Practices and Tips

Check out the [Toptal](https://www.toptal.com/swift/tips-and-practices) resource pages for additional information

## Table content:
1. [How to hide long UIViewController loading syntax from the UIStoryboard](https://github.com/ngominhtrint/swiftbestpractice#how-to-hide-long-uiviewcontroller-loading-syntax-from-the-uistoryboard)
2. [How to hide long UIView loading syntax from the UINib](https://github.com/ngominhtrint/swiftbestpractice#how-to-hide-long-uiview-loading-syntax-from-the-uinib)
3. [Use extensions to better organize your code](https://github.com/ngominhtrint/swiftbestpractice#use-extensions-to-better-organize-your-code)
4. [Consider using the GUARD statement to check for required conditions](https://github.com/ngominhtrint/swiftbestpractice#consider-using-the-guard-statement-to-check-for-required-conditions)
5. [How to properly use NSDateFormatter?](https://github.com/ngominhtrint/swiftbestpractice#how-to-properly-use-nsdateformatter)
6. [How to avoid scope in Closure?](https://github.com/ngominhtrint/swiftbestpractice#how-to-avoid-scope-in-closure)
7. [How to use protocol extensions to bind model classes with interfaces?](https://github.com/ngominhtrint/swiftbestpractice#how-to-use-protocol-extensions-to-bind-model-classes-with-interfaces)
8. [How to force highlight TODO and FIXME in Swift?](https://github.com/ngominhtrint/swiftbestpractice#how-to-force-highlight-todo-and-fixme-in-swift)
9. [How to create protocols with optional methods or properties?](https://github.com/ngominhtrint/swiftbestpractice#how-to-create-protocols-with-optional-methods-or-properties)
10. [How to avoid a retain cycle?](https://github.com/ngominhtrint/swiftbestpractice#how-to-avoid-a-retain-cycle)
11. [How to use font awesome in Swift?](https://github.com/ngominhtrint/swiftbestpractice#how-to-use-font-awesome-in-swift)
12. [Why access control matters?](https://github.com/ngominhtrint/swiftbestpractice#why-access-control-matters)
13. [How to leverage Swift to match enums?](https://github.com/ngominhtrint/swiftbestpractice#how-to-leverage-swift-to-match-enums)


## How to hide long UIViewController loading syntax from the UIStoryboard

If you are using storyboards for building your screens, and want to instantiate `UIViewController` from within your code, you have probably used syntax similar to this:

```
let userProfileViewController = UIStoryboard(name: "Main", bundle: nil).instantiateViewController(withIdentifier: "UserProfileViewController") as! UserProfileViewController

let alsoUserProfileViewController = userProfileViewController.storyboard?.instantiateViewController(withIdentifier: "UserProfileViewController") as! UserProfileViewController

let otherViewController: UserProfileViewController = UIStoryboard(name: "Main", bundle: nil).instantiateViewController(withIdentifier: "UserProfileViewController") as! UserProfileViewController

let longSyntaxViewController: UserProfileViewController = otherViewController.storyboard?.instantiateViewController(withIdentifier: "UserProfileViewController") as! UserProfileViewController
```

Let’s be honest - this is ugly. And long.

Let’s try to do something about this. One approach we can take is to create a UIStoryboard extension and move it into a separate file. You could name it UIStoryboard+Loader.swift, for example.

```
import UIKit

fileprivate enum Storyboard : String {
    case main = "Main"
    // add enum case for each storyboard in your project
}

fileprivate extension UIStoryboard {
    
    static func loadFromMain(_ identifier: String) -> UIViewController {
        return load(from: .main, identifier: identifier)
    }
    
    // optionally add convenience methods for other storyboards here ...
    
    // ... or use the main loading method directly when
    // instantiating view controller from a specific storyboard
    static func load(from storyboard: Storyboard, identifier: String) -> UIViewController {
        let uiStoryboard = UIStoryboard(name: storyboard.rawValue, bundle: nil)
        return uiStoryboard.instantiateViewController(withIdentifier: identifier)
    }
}

// MARK: App View Controllers

extension UIStoryboard {
    class func loadUserProfileViewController() -> UserProfileViewController {
        return loadFromMain("UserProfileViewController") as! UserProfileViewController
        
        // or a bit longer syntax if you did not add convenience method in the fileprivate extension
        // return load(from: .main, identifier: "UserProfileViewController") as! UserProfileViewController
    }
    
    // add other app view controller load methods here ...
}
```

Now, UIStoryboard loading and instantiation of UIViewController instances can be shortened to this:

```
let userProfileViewController = UIStoryboard.loadUserProfileViewController()
```

If your codebase is large and you have lots of storyboards in there, then a solution like this can help you better organize your code.

[Back to table content](https://github.com/ngominhtrint/swiftbestpractice#table-content)

## How to hide long UIView loading syntax from the UINib

If you create a view in the .xib file, then that view needs to be loaded somehow from the .xib file. Loading syntax looks something like this:

```
let avatarViewFromNib = UINib(nibName: "AvatarView", bundle: nil).instantiate(withOwner: self, options: nil)[0] as! UIView
```

We can take a similar approach to what we discussed above with the `UIStoryboard` extension case. This time, we create a `UINib` extension, which we can name `UINib+Loader.swift`, for example:

```
import UIKit

fileprivate extension UINib {
    
    static func nib(named nibName: String) -> UINib {
        return UINib(nibName: nibName, bundle: nil)
    }
        
    static func loadSingleView(_ nibName: String, owner: Any?) -> UIView {
        return nib(named: nibName).instantiate(withOwner: owner, options: nil)[0] as! UIView
    }
}

// MARK: App Views

extension UINib {
    
    class func loadAvatarView(withOwner owner: AnyObject) -> UIView {
        return loadSingleView("AvatarView", owner: owner)
    }
    
    // add other app view load methods here ...
    
}
```

Now creating of that view looks like the following code:

```
let avatarViewFromNib = UINib.loadAvatarView(withOwner: self)
```

[Back to table content](https://github.com/ngominhtrint/swiftbestpractice#table-content)

## Use extensions to better organize your code

We can use an extension to separate code which is part of the `UITextFieldDelegate` protocol implementation:

```
class MyViewController: UIViewController, UITextFieldDelegate {
    
    override func viewDidLoad() {
        ..
    }
    
    override func viewWillAppear(_ animated: Bool) {
        ..
    }
    
    func textFieldDidBeginEditing(textField: UITextField) {
        ...
    }
    
    func setupView() {
        ...
    }

    func textFieldShouldReturn(textField: UITextField) -> Bool {
        ...
    }
    
    func bindViewModel() {
        ...
    }
}
```

As shown in the example, without an extension, we can have a hard time finding all methods that are part of the protocol. With an extension, we have a clear separation of responsibility, as shown below.

```
class MyViewController: UIViewController {
	 override func viewDidLoad() {
        ..
    }
    
    override func viewWillAppear(_ animated: Bool) {
        ..
    }
    
    func setupView() {
        ...
    }
    
    func bindViewModel() {
        ...
    }
}

extension MyViewController: UITextFieldDelegate {
	func textFieldDidBeginEditing(textField: UITextField) {
        ...
    }
    
    func textFieldShouldReturn(textField: UITextField) -> Bool {
        ...
    }
}
```

Another valuable use of extensions is to group certain methods together that are responsible for some common task or functionality (e.g., animations or validation).

Before:

```
class MyViewController: UIViewController {
    
    override func viewDidLoad() {
        ...
    }
    
    func animateSaveButton() {
        ...
    }
    
    func validatePersonalInfo() -> Bool {
        ...
    }
    
    func validatePaymentInfo() -> Bool {
        ...
    }
    
    func animateCustomActivityIndicator() {
        ...
    }
}
```

After:

```
class MyViewController: UIViewController {
	...
}

// MARK: - Animation Methods
extension MyViewController {
	func animateSaveButton() {
        ...
    }
    
    func animateCustomActivityIndicator() {
        ...
    }
}

// MARK: - Validation Methods
extension MyViewController {
	func validatePersonalInfo() -> Bool {
		...
	}
	
	func validatePaymentInfo() -> Bool {
		...
	}
}
```

**Final hint:** If an extension contains a lot of code, consider separating that extension into a separate .swift file.

[Back to table content](https://github.com/ngominhtrint/swiftbestpractice#table-content)

## Consider using the GUARD statement to check for required conditions

Why should you use the guard statement when you can use the if-else statement? Let’s consider the following two examples:

```
// IF-ELSE 
func checkPassword(password: String?) -> Bool {
    if password == nil {
        return false
    }
    if let password = password, password.characters.count < 6 ||
        !containsRequiredCharacters(password: password) {

        showPasswordError()
        return false
    }
	
	var lenght: Int
	if let password = password {
		lenght = password.characters.count
	}
	
    return true
}

```

```
// GUARD
func checkPassword(password: String?) -> Bool {
    guard let password = password, password.characters.count >= 6 &&
        containsRequiredCharacters(password: password) else {
                
        showPasswordError()
        return false
    }
    let lenght = password.characters.count
    return true
}
```

First, with the guard statement, we are checking for what constitutes a valid password (such as minimum length and required characters), whereas, with the if-else statement, we are only checking for what makes a password invalid.

This is a relatively simple example, but if we need to check for the more complex requirements, it would be much easier and more natural to specify what we do want rather than what we don’t want.

Let’s take a look at the next two examples:

```
if value < 0 || (value > 100 && value < 200) || value > 300 {
    // wrong value
    return
}
```

```
guard (value >= 0 && value == 200 && value <= 300) else {
    // wrong value
    return
}
```

You can see how the `guard` statement is much easier and more natural for people to read than its `if-else` statement counterpart.

An additional reason to use the `guard` statement is that an optional variable will be unwrapped if it passes the guard statement. When using the `if-else` statement, we need to unwrap it by ourselves.

[Back to table content](https://github.com/ngominhtrint/swiftbestpractice#table-content)

## How to properly use NSDateFormatter?

Creating an `NSDateFormatter` is an expensive task. For this reason, it is a good idea to have a system to reuse formatters throughout the lifecycle of our apps. Let’s see how we can do this.

First, we will define an `NSDateFormatter` extension where we declare all the formatters we want to use in our app as static constants. This way, any time we use them, they will be reused instead of being created again and again:

```
extension NSDateFormatter {
    
    @nonobjc static let shortDateAndTime: NSDateFormatter = {
        let formatter = NSDateFormatter()
        formatter.dateStyle = .ShortStyle
        formatter.timeStyle = .ShortStyle
        return formatter
    }()
    
    @nonobjc static let dayMonthAndYear: NSDateFormatter = {
        let formatter = NSDateFormatter()
        formatter.dateFormat = "MM/dd/yyyy"
        return formatter
    }()
    
    @nonobjc static let monthAndYear: NSDateFormatter = {
        let formatter = NSDateFormatter()
        formatter.setLocalizedDateFormatFromTemplate("MMMyyyy")
        return formatter
    }()
}
```

Maybe you are asking yourself about the @nonobjc attribute? Try to remove it, and you’ll see that the compiler complains:

>A declaration cannot be both ‘final’ and ‘dynamic'.

This issue arises because Swift is trying to generate a dynamic accessor for the static property, for ,Objective-C compatibility, since the class inherits from `NSObject`. Similarly, notice that adding the `@objc` will make your extension incompatible with Objective-C.

Next, we are going to create an `NSDate` extension to make a simple API to convert to strings and back:

```
extension NSDate {
    
    /// Prints a string representation for the date with the given formatter
    func string(with format: NSDateFormatter) -> String {
        return format.stringFromDate(self)
    }
    
    /// Creates an `NSDate` from the given string and formatter. Nil if the string couldn't be parsed
    convenience init?(string: String, formatter: NSDateFormatter) {
        guard let date = formatter.dateFromString(string) else { return nil }
        self.init(timeIntervalSince1970: date.timeIntervalSince1970)
    }
}
```

Now, we just have to call the `NSDate` extension methods.

Let’s see some examples.

To create a `String` from a `NSDate`:

```
let date = NSDate()
let string = date.string(with: .shortDateAndTime)
// let string = date.string(with: .dayMonthAndYear)
// let string = date.string(with: .monthAndYear)
```

To create a `NSDate` from a `String`:

```
let string = "06/17/2016"
let date = NSDate(string: string, formatter: .dayMonthAndYear)
```

[Back to table content](https://github.com/ngominhtrint/swiftbestpractice#table-content)

## How to avoid scope in Closure?

There are many situations in Swift development where we use closures. For example, when we are making calls to the server, a closure may be called as a callback, either with a success or failure block. In most cases, delegates are substituted with closures as well. It is also worth mentioning that Apple’s new API methods prefer closures as well.

```
let alert = UIAlertController()
alert.addAction(UIAlertAction(title: "Submit", style: .Default) { _ in
    // a block of code executed after tapping on submit action
})
```

However, there are cases where we just don’t need to use the scope for `self` in the closure. When we don’t have any declaration for the closure, the compiler will by default store a retained reference to `self` as a strong type. Of course, it is possible to declare a closure to store with a type of `weak` or `unowned`.

So, if you want to avoid a retain reference in a closure and don’t want to worry about the memory leaks, the solution is to use `@noescape` before closure declaration.

```
class Requester {
    
    func getItems(@noescape completion: () -> Void) {
        completion()
    }
    
    deinit {
        print("deinit Requester")
    }
}

class Model {
    
    let requester = Requester()
    
    func updateData() {
        requester.getItems {
            build()
        }
    }
    
    func build() {
        // here you can build some data
        print("build")
    }
    
    deinit {
        print("deinit Model")
    }
}

_ = {
    let model = Model()
    model.updateData()
}()
// after calling updateData(), a console log will print out
// build
// deinit Model
// deinit Requester
```

Here, `_ = { … }()` is just an anonymous function which will be executed. Otherwise, `deinit` methods won’t be fired because a global scope will never be finished.

If you copy and paste the example above into the Swift playground, an output console log will confirm that all objects had been deallocated, demonstrating that there is no memory leak in the example above.

To repeat, the attribute `@noescape` does two main things:

1. It avoids us needing to write an explicit `self` in the closure.
2. If we try to assign a completion block to a property, the compiler won’t allow us to do that because `@noclosure` says the block can’t be assigned. Otherwise, we would face a potentially serious memory leak.

[Back to table content](https://github.com/ngominhtrint/swiftbestpractice#table-content)

## How to use protocol extensions to bind model classes with interfaces?

Sometimes you have to bind the same kind of objects with multiple views, like UIViewController, UITableViewCell, or UICollectionViewCell. Very often, the binding code is quite similar in all the views, so it would be a great idea if you can reuse it. We will use Swift protocol and protocol extensions to do it.

Let’s suppose we have a `User` class:

```
class User {
    var name = ""
    var email = ""
    var bio = ""
    var image: UIImage? = nil
    
    init(name: String, email: String, bio: String) {
        self.name = name
        self.email = email
        self.bio = bio
    }
}
```

We will create a protocol which will adopt all the interfaces that we want to bind with our `User`instances. We will call it `UserBindable`.

```
protocol UserBindable: AnyObject {
    var user: User? { get set }
    
    var nameLabel: UILabel! { get }
    var emailLabel: UILabel! { get }
    var bioLabel: UILabel! { get }
    var imageView: UIImageView! { get }
}
```

The first var `user` is the user to bind, and all the others are `UIView` subclasses that we can use to bind the user.

Then, we create a protocol extension:

```
extension UserBindable {
    
    // Make the views optionals
        
    var nameLabel: UILabel! {
        return nil
    }
    
    var emailLabel: UILabel! {
        return nil
    }
    
    var bioLabel: UILabel! {
        return nil
    }
    
    var imageView: UIImageView! {
        return nil
    }
    
    // Bind
    
    func bind(user: User) {
        self.user = user
        bind()
    }
    
    func bind() {
        
        guard let user = self.user else {
            return
        }
    
        if let nameLabel = self.nameLabel {
            nameLabel.text = user.name
        }
        
        if let bioLabel = self.bioLabel {
            bioLabel.text = user.bio
        }
        
        if let emailLabel = self.emailLabel {
            emailLabel.text = user.email
        }
        
        if let imageView = self.imageView {
            imageView.image = user.image
        }
    }
}
```

Here, we split the extension into two sections:

The first one is used to provide a default value (nil) for all views, so the objects that will implement the protocol won’t need to have all of them. For instance, some of our views could exclude the `image`, but some others can include it.

We take the values of the user properties and set them to views.

Now, most of the job is done. If we have to present a user list, we will want to create a `UITableViewCell`. Let`s imagine that this cell just needs to show the name and the email:

```
class UserTableViewCell: UITableViewCell, UserBindable {

    var user: User?
    
    // we can set the labels in interface builder or with by code. 
    @IBOutlet weak var nameLabel: UILabel!
    @IBOutlet weak var emailLabel: UILabel!
}
```

And that’s all. Our cell implements `UserBindable`, so it knows how to bind its interface to a user object. In our `UITableViewDataSource` we can do:

```
    func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCellWithIdentifier("Cell") as! UserTableViewCell
        let user = // find the user from your array or whatever
        cell.bind(user)
        return cell
    }
```

If we want to show the detail of this user after touching it, we can have a view controller like this:

```
class UserDetailViewController: UIViewController, UserBindable {
    
    var user: User?
    
    @IBOutlet weak var nameLabel: UILabel!
    @IBOutlet weak var emailLabel: UILabel!
    @IBOutlet weak var bioLabel: UILabel!
    @IBOutlet weak var imageView: UIImageView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
	 // here we suppose that we have set the user value before the viewDidLoad
        bind()
    }
}
```

[Back to table content](https://github.com/ngominhtrint/swiftbestpractice#table-content)

## How to force highlight TODO and FIXME in Swift?

In the old days of **Objective-C**, it was very easy to force warnings that let us know when one piece of code needs a fix or some additional work. We just had to write:

```
#warning TODO: Finish implementation
- (void) foo {
}
```

In **Swift**, we have lost the ability to force warnings to remind us to do these kind of things. However, we have another features that allows us to mark something as **TODO** or **FIXME**:

```
// TODO: Please, finish implementation
func foo() {
}

// FIXME: Please, fix it always crashes!
func bar() {
    let array: [String] = []
    array[1]
}
```

The downside of these tags is that they are not highlighted by the compiler. We can see them on the method dropdown box in **Xcode**, but they are forgotten easily.

A workaround to make the compiler complain about these marks is to just add the following script as a new _Build Phase_:

```
# Set this var a false if you prefer to include your cocoa pods warnings
EXCLUDEPODS=true

# Add the tags you want to mark as warnings  and or errors
WARNINGTAGS="TODO:|FIXME:|WARNING:"
ERRORTAG="ERROR:"

if $EXCLUDEPODS; then PODSTRING="-path ./Pods -prune"; else PODSTRING=""; fi

find "${SRCROOT}" $PODSTRING \( -name "*.h" -or -name "*.m" -or -name "*.swift" \) -print0 | xargs -0 egrep --with-filename --line-number --only-matching "($WARNINGTAGS).*\$|($ERRORTAG).*\$" | perl -p -e "s/($WARNINGTAGS)/ warning: 🛠 \$1/" | perl -p -e "s/($ERRORTAG)/ error: 😱 \$1/"
```

After adding this, the build phase tags **TODO** and **FIXME** are treated by the compiler as warnings. Moreover, we can have two more added tags to force a warning or even an error:

```
// WARNING: Something is wrong here
// ERROR: Something is really wrong here
```

It is important to notice that the **ERROR** tag won’t make the build fail, but it will be marked in read as tagged as an error.

[Back to table content](https://github.com/ngominhtrint/swiftbestpractice#table-content)

## How to create protocols with optional methods or properties?

Populating tables is one of the most common activities in iOS. TableView’s DataSource contains many methods you can implement; however, did you notice that DataSource has only two required methods?

UIKit uses an old way to achieve optional methods, using Objective-C style. However, there is certainly a better approach using an elegant code. Compounding multiple features together like extensions and computed properties will help us to avoid implementing all protocol’s methods and properties

Implementing required protocols is straightforward:

```
CommentDelegate {
    func didSubmitComment()
    func didRemoveComment()
    var isSingleComment: Bool { get }
}

class A: RequiredCommentDelegate {
    func didSubmitComment() {
        // implementation
    }
    func didRemoveComment() {
        // implementation
    }
    var isSingleComment = true
}
```

Class A has to implement two methods and a boolean property. They are all mandatory, otherwise a compiler will warn you. If we are not interested in `didRemoveComment()` method, we can omit a body of the method, but class A must contain at least a declaration. It is not necessarily a huge problem, but imagine a complex project with many other methods and dependencies. Having a transparent and structured code is beneficial for testing and can help us to avoid further errors.

Let’s have a look on the more _swifty_ way how to accomplish optional methods and properties in pure Swift.

Our new protocol will look like the previous one:

```
protocol OptionalCommentDelegate {
    func didSubmitComment()
    func didRemoveComment()
    var isSingleComment: Bool { get }
}
```

Now, the power of extensions comes to the action. Creating extensions over the protocols will deliver the desired effect. For methods it’s quite straight:

```
extension OptionalCommentDelegate {
    func didSubmitComment() { }
    func didRemoveComment() { }
}
```

Implementing methods and variables with an empty body will make them optional. So simple, right? However, properties and extensions behave differently. Our first assumption would be to implement property like this:

```
extension OptionalCommentDelegate {
    var isSingleComment = true
}
```

After writing these lines of codes, the compiler will tell us: “Extensions may not contain stores properties.” In other words, we can’t create properties and assign them values in extensions.

Going deeper and experimenting with the properties, we find out that there is a simple trick how to avoid the mentioned error - using Computed properties.

```
extension OptionalCommentDelegate {
    var isSingleComment: Bool {
        return false
    }
}
```

Every time we call `isSingleComment`, it will execute the body of the property, which is in this case `return false`. For those who are not familiar with computed properties and their shortcuts, we can rewrite the code to:

```
extension OptionalCommentDelegate {
    var isSingleComment: Bool {
        get {
            return false
        }
    }
}
```

Now `isSingleComment` is an optional property in a protocol.

[Back to table content](https://github.com/ngominhtrint/swiftbestpractice#table-content)

## How to avoid a retain cycle?

Retaining cycles are one of the most dangerous mistakes programmers can make, as these cycles can lead to unexpected crashes and huge memory consumption. There are multiple ways to tackle well-known retain cycle issues.

Imagine the following code:

```
class Object {
    func doStuff(completion: (() -> Void)?) {
        completion?()
    }
}

class Controller: UIViewController {
    let object = Object()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        object.doStuff {
            self.update()
        }
    }
    
    func update() {
        // implementation
    }
}
```

In our case, method `update()` causes retaining because the closure is capturing self as a strong reference to it. Our first assumption is to make a weak copy of `self` in the closure.

```
 override func viewDidLoad() {
        super.viewDidLoad()
        
        object.doStuff { [weak self] in
            self?.update()
        }
    }
```

Why is `weak self` inside the square brackets? As it implies, it is an array of objects you can put inside brackets to specify multiple capture values in a closure. The solution above could be sufficient, but let’s dive into an even more _swifty_ solution. If it’s possible to write code better, you should always do it.

Swift 2.0 introduced a new statement, `guard`, to simplify a code structure and to finally get rid of scary pyramids of IF-statements. Embracing the power of guards inside closures can rapidly enhance our lives and set a smart convention in programming.

```
override func viewDidLoad() {
    super.viewDidLoad()
    
    object.doStuff { [weak self] in
        guard let aSelf = self else { return }
        aSelf.update()
    }
}
```

The guard statement was used to protect `self`, not to be `nil`, and if so a code won’t continue in the closure. Using a local variable `aSelf` will ensure that there will be no retain cycle.

A syntax sugar in the end - wouldn’t it be great to use just `self` instead of `aSelf`? Yes, it would. Using back quotes together with a guard statement can lead to the code below:

```
guard let `self` = self else { return }
```

To summarize:

```
object.doStuff { [weak self] in
    guard let `self` = self else { return }
    self.update()
}
```

I use this in almost every closure to avoid potential retain cycle. There are some exceptions when you don’t have to use `[weak self]`, like in the following example:

```
UIView.animateWithDuration(1) { 
    self.update()
}
```

Why isn’t it necessary to take care of retain cycles in the example above? The closure will be destroyed automatically after execution, and the most important object where `animateWithDuration` is called does not retain the block.

[Back to table content](https://github.com/ngominhtrint/swiftbestpractice#table-content)

## How to use font awesome in Swift?

Today, icons are used on every website you can imagine. What if you want to use Font Awesome icons in your Swift project? Importing new font requires adding a property into `info.plist`, but the most annoying part is to match the correct Unicode code point with a symbol in the font file. We would have to know that `symbol` is encoded to `f042` hexadecimal number. It is quite challenging and cumbersome to know all hexadecimal numbers. Not to mention, the result would look like this:

```
label.text = "\u{f037}"
```

There is certainly a better way how to add icons to labels or buttons. Wouldn’t it be better to use simply the words that are already predefined and which describe icons we want to add as a text, like FAArrows or FACog?

[Font Awesome Swift](https://github.com/Vaberer/Font-Awesome-Swift) library is designed exactly for these purposes. A very elegant solution comes with this lightweight library:

```
label.FAIcon = .FATwitter
```

Compounding icons with additional text can be straightforward as well:

```
label.setFAText(prefixText: "prefix ", icon: .FATwitter, postfixText: " prefix", size: 30, iconSize: 20)
```

Have you noticed the adjusting size of text and icon in one string? `NSAttributedStrings` is the core that runs Font Awesome Swift library behind the scenes.

[Back to table content](https://github.com/ngominhtrint/swiftbestpractice#table-content)

## Why access control matters?

Swift gives us the option to write multiple classes in one file without taking care of headers like in Objective-C. In other words, every property in `.h` file means it is visible from outside the class, or to put it simply - it’s public. In Swift, all properties and methods are internal. They are accessible from their defining module, but not in any source file outside of that module.

```
class A {
    var myVariable1 = true
    internal var myVariable2 = true
}

```

Both properties have the same access control, and we can omit this attribute. They are accessible from outside, but other frameworks can’t access the properties, like libraries added via CocoaPods. Even if the variables are public, they still are not visible because the `class A` is marked as `internal` by default. We would have to write explicitly `public class A { }` to reveal it publicly. By contrast, a private access control can be accessed only from inside the class, so debugging can be much easier because there is no outside factor to change the behavior of the class.

My convention is to mark everything private, and then expose only those parts of which you are aware that they have to be changed from outside. The more private properties and methods, the better for you. On the contrary, creating pods requires a caution of where to add public attributes and where to not. Without marking them public, your pod won’t be usable.

```
class A {    
    public var myVariable = true
}
```

The code above doesn’t make sense. It’s great that we revealed `myVariable` property, but other modules can’t access the class at all. Don’t worry, a compiler will warn us: “Declaring a public var for an internal class”. Adding public in the front of `class A` will solve the problem. However, what about unit testing and access control?

In general, all classes which are not marked as public, can’t be seen by other modules, and testing targets belong to different modules as well. Just imagine the disaster when everything has to be marked as public for unit tests. Keep in mind, this was a reality before Swift 2.0. Now, Swift provides an attribute `@testable` to avoid this behavior.

Attribute `@testable import MyProject` will unveil internal properties and methods in unit test classes. Just remember those private properties won’t be visible from outside to other modules. If you need to have access to private properties, consider to decouple a code or think about the architecture of code. It’s a sign that your code isn’t designed for testing. The test driven development would point out this problem much sooner.

[Back to table content](https://github.com/ngominhtrint/swiftbestpractice#table-content)

## How to leverage Swift to match enums?

An enumeration defines a common type for a group of related values which are logically connected:

```
enum Goods {
    case Bread
    case Roll
}
let myGoods: Goods = .Bread
```

We encapsulate `Bread` and `Roll` into one entity, `Goods`. Matching simple enums are evident because it’s a standard comparison.

```
if myGoods == .Bread {
    // matched
}
```

However, it is sometimes useful to be able to store extra information about other types together with these case values and allow this information to be altered at any time later. Swift calls this feature Associated Values.

For example, let’s suppose to have all stuff stored in one property with extra information about how many pieces an inventory has. In Swift, the enumeration to define stuff can look like this:

```
enum Stuff {
    case Chair(amount: Int)
    case Table(amount: Int)
}
let myStuff: Stuff = .Chair(amount: 10)
```

Extracting associated values can be done via the switch or special IF-statement. Regular IF-condition doesn’t work with associated enums.

```
if myStuff == .Chair(amount: _) {
    // implementation
}
```

Instead, we use a new enhanced IF-statement and extra information, the amount that is now extracted.

```
if case let .Chair(amount: amount) = myStuff {
    print(amount)
}
```

There are cases when we are not interested in an associated value. A shorter version would be:

```
if case .Chair(amount: _) = myStuff {
    // implementation
}
```

[Back to table content](https://github.com/ngominhtrint/swiftbestpractice#table-content)

