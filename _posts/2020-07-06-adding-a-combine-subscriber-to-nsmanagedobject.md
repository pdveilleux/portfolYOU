---
title: Adding a Combine Subscriber to NSManagedObject
description: Working around a crash after deleting the entity.
tags: [iOS Development, Core Data]
style: border
color: primary
---

While migrating the data persistence layer of Composed to Core Data I wanted to have a computed property in an `NSManagedObject` subclass. This can be useful for values that don't need to be stored permanently and that you also don't want to compute at the time of retrieval. Here's how to set one up and how to avoid a potential crash.

There are two methods of `NSManagedObject` that you'll want to pay attention to.

```swift
func awakeFromFetch()
func awakeFromInsert()
```

These two methods allow you insert code into the lifecycle of the managed object when it gets fetched from the database and when a new instance is inserted, respectively.

> Learn more about these methods and NSManagedObject from the [docs](https://developer.apple.com/documentation/coredata/nsmanagedobject).

```swift
public class TaskProject: NSManagedObject {

    // Since this is only being used to update another property `kind` we don't
    // need to make this accessible outside of the class so we mark it `private`.
    private var kindSubscription: AnyCancellable!

    public override func awakeFromFetch() {
        super.awakeFromFetch() // Need to call `super` before adding our own functionality.
        setupKindSubscription()
    }

    public override func awakeFromInsert() {
        super.awakeFromInsert() // Need to call `super` before adding our own functionality.
        setupKindSubscription()
    }

    // Create a single method that we can call from both `awakeFromFetch` and `awakeFromInsert`.
    // We mark this `private` also since, like `kindSubscription`, this doesn't need to be
    // accessible outside of the class.
    private func setupKindSubscription() {
        kindSubscription = self.publisher(for: \.children)
            // We use [weak self] here to prevent retain cycles. For more information about
            // retain cycles you should read this SwiftLee article:
            // https://www.avanderlee.com/swift/weak-self/.
            .sink { [weak self] (children) in
                // I'm using this to update `kind` which takes an enum value I made to
                // differentiate a task versus a project based upon the number of `children`.
                // Replace this with your own functionality.
                self?.kind = children.count == 0 ? .task : .project
        }
    }
}
```

This will setup the subscription whenever the `NSManagedObject` subclass is either fetched from, or inserted into, the `NSManagedObjectContext`.

> **NOTE:** You may experience a crash when saving the `NSManagedObjectContext` after deleting an object with a subscription property. I believe this to be a bug and will be reporting it to Apple. In the meantime, see below for an easy way to make it work.

If you experience a crash described by `Fatal error: UnsafeRawBufferPointer with negative count at line 872` try adding the following to your `NSManagedObject` subclass.

```swift
public override func prepareForDeletion() {
    super.prepareForDeletion()

    // Remove any subscriptions otherwise saving will crash
    kindSubscription = nil
}
```
