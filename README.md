# Swift Best Practices and Tips

Check out the [Toptal](https://www.toptal.com/swift/tips-and-practices) resource pages for additional information

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