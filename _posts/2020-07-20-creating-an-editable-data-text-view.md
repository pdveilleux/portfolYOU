---
title: Creating an Editable Data Text View
description: A UITextView subclass that allows editing and detection of links, phone numbers, email addresses, and more.
tags: [iOS Development, Composed]
style: border
color: primary
---

I often want `UITextView` to allow both editing and detection of different types of data within its text property. Both of these features are supported individually so you can have an editable text view that the user can type in and modify and you can have a text view that detects data like phone numbers and links. But having both so that a user can edit, add, and delete text and then have whatever actionable information they've added be selectable is not supported natively.

`EditableDataTextView` supports this!

### Creating an Instance

Instantiate one the same as you would with `UITextView`.

```swift
let textView = EditableDataTextView()
textView.text = "Hello, World!"
```

Almost everything else is self contained in the class. The only thing you have to do is update `isEditable` when the user is done editing so that data detection and the tap recognizer are enabled again.

```swift
textView.isEditable = false
```

And that's it! You can check out my gist below or my repo on [GitHub](https://github.com/pdveilleux/Editable-Data-Text-View).

{%- gist f2f72328fca3c0bef5277e1cda96697a %}
