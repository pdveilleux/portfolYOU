---
title: Creating a Table View with Various Actions
description: Using protocols, enums, and structs for reusable and concise table views, perfect for settings views.
tags: [iOS Development, Composed]
style: fill
color: primary
---

For version 1.0 of [Composed](/projects/1-composed) I rewrote the menu to be concise, reusable, and highly modifiable for any changes and additions I might make to it down the road. In this post I'm going to share with you how to create one like it.

What's important to know about this menu is that there are different types of actions that the user will be able to make such as composing an email and opening a URL. Using the method I've made I can easily insert new menu options, reorder them, and even extend the functionality for other actions not yet defined.

Below is a screenshot of the menu with the types of actions each row will perform.

{% include elements/figure.html image="/assets/posts/2020-07-28-creating-a-table-view-with-various-actions/table-view-with-various-actions-header.png" %}

## Creating the Data Source

#### Defining the Model

Let's get started by creating an enum and adding the different sections we want in our table.

```swift
enum MenuSection {
    case settings
    case contact
    case support
}
```

We can add a property for the title of each section...

```swift
enum MenuSection {
    ...

    var title: String {
        switch self {
        case .settings:     return "Settings"
        case .contact:      return "Reach out"
        case .support:      return "Help"
        }
    }
}
```

...and a property that defines the items for each section.

```swift
enum MenuSection {
    ...

    var items: [MenuOption] {
        switch self {
        case .settings:
            return []
        case .contact:
            return []
        case .support:
            return []
        }
    }
}
```

Here, we can see that `items` are defined as an array of type `MenuOption` which is a protocol that defines the essentials for each row in our table. Let's take a look at it now and come back to populating the arrays later!

```swift
protocol MenuOption {
    var title: String { get }
    var image: UIImage { get }
    var accessoryType: UITableViewCell.AccessoryType { get }
}
```

The title and image will be unique to each row but I want a default accessory type. We can use protocol extensions to give `MenuOption` some default behavior. This will make the definition and initialization of the structs we create in a bit easier because `accessoryType` won't have to be defined since it will have a default value.

```swift
extension MenuOption {
    var accessoryType: UITableViewCell.AccessoryType { return .none }
}
```

#### Specialized Actions using Protocol Inheritance

This gets us a good starting point to create each row, but different rows need to have different types of actions. Some of them will open a URL, one will compose an email, one will open a modal view controller, and one will toggle the dark mode app icon. Each of these different actions need a bit of unique data so each will get its own protocol to define what's required. Let's check them out.

```swift
protocol EmailMenuOption: MenuOption {
    var address: String { get }
}

protocol LinkableMenuOption: MenuOption {
    var url: URL { get }
}

protocol ModalDetailMenuOption: MenuOption {
    var controller: UIViewController { get }
}

protocol SwitchableMenuOption: MenuOption {
    var isOn: Bool { get set }
    var action: (Bool) -> () { get }
}
```

Now that we have the protocols defined we can create the structs to hold the data.

```swift
struct EmailMenuItem: EmailMenuOption {
    let title: String
    let image: UIImage
    let address: String
}

struct LinkableMenuItem: LinkableMenuOption {
    let title: String
    let image: UIImage
    let url: URL
}

struct ModalDetailMenuItem: ModalDetailMenuOption {
    let title: String
    let image: UIImage
    let controller: UIViewController
}

struct SwitchableMenuItem: SwitchableMenuOption {
    let title: String
    let image: UIImage
    var isOn: Bool {
        didSet {
            action(isOn)
        }
    }
    let action: (Bool) -> ()
}
```

Because each of the protocols these structs conform to inherit `MenuOption` each struct needs a title and image which will populate our rows. Each struct also gets its own unique data required for each action. Here we can note again that because we extended `MenuOption` with a default value for `accessoryType` we don't need to define that as a property in each of these structs even though it will still be accessible to us later.

#### Populating the Data Source

Let's now go back and populate the arrays in `MenuSection` using these structs which we can do because they conform to `MenuOption`.

```swift
enum MenuSection {
    ...

    var items: [MenuOption] {
        switch self {
        case .settings:
            return [
                SwitchableMenuItem(title: "Dark App Icon",
                                   image: UIImage(systemName: "square.fill")!,
                                   isOn: UIApplication.shared.alternateIconName == "Logo_Dark",
                                   action: { isOn in
                                        if isOn {
                                            UIApplication.shared.setAlternateIconName("Logo_Dark")
                                        } else {
                                            UIApplication.shared.setAlternateIconName(nil)
                                        }
                })
            ]
        case .contact:
            return [
                EmailMenuItem(title: "Email",
                              image: UIImage(systemName: "envelope.fill")!,
                              address: "composedsoftware@gmail.com"),
                LinkableMenuItem(title: "Twitter",
                                 image: UIImage(systemName: "text.bubble.fill")!,
                                 url: URL(string: "https://www.twitter.com/composedpm")!),
                LinkableMenuItem(title: "Website",
                                 image: UIImage(systemName: "globe")!,
                                 url: URL(string: "https://www.composedpm.com")!)
            ]
        case .support:
            return [
                ModalDetailMenuItem(title: "Help",
                                    image: UIImage(systemName: "questionmark.diamond.fill")!,
                                    controller: GuideViewController()),
                LinkableMenuItem(title: "Privacy Policy",
                                 image: UIImage(systemName: "hand.raised.fill")!,
                                 url: URL(string: "https://www.composedpm.com/privacy-policy")!)
            ]
        }
    }
}
```

This gives us all the data we need to create our rows. Let's make one small adjustment to `MenuSection` for convenience.

```swift
enum MenuSection: CaseIterable {
    ...
}
```

Making `MenuSection` conform to `CaseIterable` will give us a property `allCases` which is a collection of all values for the type. This will make it easy to specify how many sections and rows there will be in our table.

## Applying the Data Source

Let's jump over to our `UITableViewDataSource` implementation and see how it works.

```swift
extension MenuViewController: UITableViewDataSource {
    func numberOfSections(in tableView: UITableView) -> Int {
        return MenuSection.allCases.count
    }

    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return MenuSection.allCases[section].items.count
    }

    ...
}
```

Here, we've defined the number of sections to be equal to the number of cases, one each for `settings`, `contact`, and `support`, and the number of rows per section to be defined by the number of items in `MenuSection.items`.

```swift
extension MenuViewController: UITableViewDataSource {
    ...

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let item = MenuSection.allCases[indexPath.section].items[indexPath.row]
        if var item = item as? SwitchableMenuOption,
            let cell = tableView.dequeueReusableCell(withIdentifier: kTOGGLE_CELL_ID,
                                                     for: indexPath) as? ToggleTableViewCell {
            cell.textLabel?.text = item.title
            cell.imageView?.image = item.image
            cell.imageView?.tintColor = .label
            cell.accessoryType = item.accessoryType
            cell.toggle.isOn = item.isOn
            cell.toggleSwitchClosure = { isOn in
                item.isOn = isOn
            }
            cell.selectionStyle = .none
            return cell
        } else {
            let cell = tableView.dequeueReusableCell(withIdentifier: kSTANDARD_CELL_ID,
                                                     for: indexPath) as UITableViewCell
            cell.textLabel?.text = item.title
            cell.imageView?.image = item.image
            cell.imageView?.tintColor = .label
            cell.accessoryType = item.accessoryType
            cell.selectionStyle = .default
            return cell
        }
    }

    ...
}
```

There's a little more going on here, but nothing too complicated. There are two types of cells that we need to create; one is a standard cell and one has a `UISwitch`. Because the cell with the switch is only used when the `MenuSection` item conforms to `SwitchableMenuOption` we can wrap that in an `if let` statement (in this case `if var`, more on this in a moment) and let the `else` case handle all other cells.

Let's look at the standard cell in the `else` case first. We set the text, image, and accessory type to the corresponding values in `item`. I've set `tintColor` on `imageView` to be the same color as the text label using `UIColor.label`, but you could use another color. Finally, we set `selectionStyle` to `.default` so the user gets a visual indication of their selection.

For the cell with the switch we set all the same properties we just went over with the one difference being `selectionStyle`. Because all the interaction in that cell happens with the `UISwitch` and nothing will happen on a selection I want to visually indicate that by setting `selectionStyle` to `.none`. The additional configuration of `cell.toggle.isOn` is used to set the initial state of the `UISwitch` and the closure is simply updating the state. This is important so that if this cell were to be reloaded after the switch was toggled the state of the data source would still be correct and it would display the cell correctly.

> Note: It is this state updating of the data source that makes us change `if let` to `if var` so that we can modify `item`.

**HOWEVER,** there's still a problem here! We're using structs which use value semantics which means that `if let item = item...` creates a copy so when the closure is called we're modifying the copy of the data source item instead of the data source item itself. Let's change `SwitchableMenuItem` to a class so that we can use reference semantics instead. This means that `if let item = item...` will create a reference to the data source item and so the closure will update the data source itself, truly keeping the state correct.

```swift
class SwitchableMenuItem: SwitchableMenuOption {
    var title: String
    var image: UIImage
    var isOn: Bool {
        didSet {
            action(isOn)
        }
    }
    var action: (Bool) -> ()

    init(title: String, image: UIImage, isOn: Bool, action: @escaping (Bool) -> ()) {
        self.title = title
        self.image = image
        self.isOn = isOn
        self.action = action
    }
}
```

Since classes don't have memberwise initializers we have to specify our own initializer too.

#### Custom UITableViewCell with a UISwitch

Let's look into the class definition of `ToggleTableViewCell` to better understand how it works.

```swift
class ToggleTableViewCell: UITableViewCell {

    var toggleSwitchClosure: ((Bool) -> ())?
    var toggle: UISwitch!

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)

        toggle = UISwitch()
        toggle.addTarget(self, action: #selector(didSwitch(_:)), for: .valueChanged)
        accessoryView = toggle
    }

    required init?(coder: NSCoder) {
        super.init(coder: coder)
    }

    @objc func didSwitch(_ sender: UISwitch) {
        toggleSwitchClosure?(sender.isOn)
    }
}
```

We can see that `ToggleTableViewCell` is quite similar to its superclass `UITableViewCell`. When initialized we create a `UISwitch`, tell it to call `didSwitch(_:)` whenever its value changes, and assign it to the `accessoryView` of the cell. `didSwitch(_:)` calls the closure and passes it the state of the switch's `isOn` property.

#### Reusable Cells

Back in the `UITableViewDataSource` implementation, notice that we need to register any class for reuse by the table view when using `dequeueReusableCell` so let's add the following to `viewDidLoad()`.

```swift
tableView.register(ToggleTableViewCell.self, forCellReuseIdentifier: kTOGGLE_CELL_ID)
tableView.register(UITableViewCell.self, forCellReuseIdentifier: kSTANDARD_CELL_ID)
```

And we'll also set the reuse identifiers as constant properties on our view controller.

```swift
let kTOGGLE_CELL_ID = "ToggleCellID"
let kSTANDARD_CELL_ID = "StandardCellID"
```

#### Section Headers

Let's wrap up the `UITableViewDataSource` implementation by setting the title for each section.

```swift
extension MenuViewController: UITableViewDataSource {
    ...

    func tableView(_ tableView: UITableView, titleForHeaderInSection section: Int) -> String? {
        return MenuSection.allCases[section].title
    }
}
```

To get the same type of table view layout I have, set the table view style to `.grouped` when initializing it.

```swift
let tableViewController = UITableViewController(style: .grouped)
// or
let tableView = UITableView(frame: aFrame, style: .grouped)
```

**We're almost there!**

## Handling Cell Selection

We've created the data source, applied it to the table, and now we're going to make it usable by intercepting the user actions! Below, we define `tableView(_:didSelectRowAt:)` and give the different menu items that we created different actions.

```swift
extension MenuViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        let item = MenuSection.allCases[indexPath.section].items[indexPath.row]
        if let _ = item as? SwitchableMenuOption {
            return
        } else if let item = item as? EmailMenuOption {
            composeMail(to: item.address)
        } else if let item = item as? LinkableMenuOption {
            openURL(item.url)
        } else if let item = item as? ModalDetailMenuOption {
            present(item.controller, animated: true)
        }
        tableView.deselectRow(at: indexPath, animated: true)
    }
}
```

We access the menu item and then use `if let` statements to check if it conforms to the different protocols. This let's us safely access the unique data for the different types of actions without forcibly unwrapping.

Let's go through each case.
- For `SwitchableMenuOption` we don't actually want a tap on the cell to do anything because we're handing the action when the user interacts with the `UISwitch`.
- For `EmailMenuOption` we call a custom method, `composeMail(to:)`, and pass it the address for the recipient.
- For `LinkableMenuOption` we call a custom method, `openURL()`, and pass it the URL to open.
- And for `ModalDetailMenuOption` we call the `UIViewController` method `present(_:animated:completion:)`.

## Recap

So what have we done?
- Create a data source for our table using an enum.
- Create specialized structs (and a class) that conform to a generic protocol used to define the cells of the table.
- Applied the data source to the table and conditionally handled the user interactions.
