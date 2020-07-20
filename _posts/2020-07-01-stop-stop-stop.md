---
title: Stop, Stop, Stop...
description: A letter to my future self about not reinventing the wheel.
tags: [iOS Development, Composed]
style: fill
color: danger
---

{% include elements/figure.html image="/assets/posts/2020-07-01-stop-stop-stop/stop-stop-stop.png" %}

**What are you doing, future me!**

You know better than to be doing what you’re doing. You don’t need to roll your own versions of everything. Others have come before us to make it easier, faster, more reliable, and more predictable for our users. You just feel like you need to make your own version because you don’t know what exists out in the world. Do your research!

“But I made the editable data text field for Composed,” I hear you rebutting. Yes, granted. But that’s one example since 2013 against countless others where we both know you’ve wasted your time building multitudes of standards that already exist.

Let’s take a look at the mess I’ve made for you now as a reminder.

### Persistence

Apple didn’t introduce a ‘Swifty’ version of Core Data at WWDC 2020 so you better brush the dust off that memory bank and reacclimate yourself with `NSManagedObject`. This JSON archiving of user data to the documents directory isn’t working for Composed’s persistence goals.

1. Minimize the write processes when updating properties in a task or project.
2. Immediately save all property changes.
3. Allow a single task or project to be referenced by multiple other tasks or projects as a sub-task or sub-project.
4. Enable data retrieval to be quickly filtered and sorted by various properties.
5. Enable cross-platform data syncing for future versions of the app on watchOS and macOS.

### Menu and Settings

I know it’s easy to get drawn towards what other apps do. That can be good because of inter-app consistency for the user, BUT lean into what is native and easy to implement also. The swipe/pan menu takes cues from Twitter, but why not just have it be an easy modal view controller? It would look and act more native.

### Custom Fonts

It’s unnecessary. Can it be cool? Yes. Is it making my life harder to support Dynamic Type? Yes. Remember, you’re not a designer but a developer so follow the [Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/) and put your painting pants on later. Use the standard fonts.

### TL;DR

* Don’t reinvent the wheel. Use what’s native.
* Read the documentation.
* Follow the HIG.
* Reference the WWDC videos.
* And always learn, learn, learn!
