---
name: Composed
tools: [Swift, Core Data]
image: /assets/projects/composed.png
description: The project manager app that helps you stay calm, collected, and composed.
#external_url: https://www.composedpm.com
---

# Composed

Composed gives you a unique way to track the progress of your tasks and projects and know if you’re on schedule. It can be overwhelming to think about what there is to do but with Composed you can stay on top of it all and feel relaxed too. Progress bars and an intuitive color scheme let you quickly understand how things are going.

{% include elements/figure.html image="/assets/projects/composed-header.png" %}

## Core Concept - TaskProject

One of the most important foundational decisions in developing Composed was to decide how to handle the differences and similarities between tasks and projects. When using the original version I made back in 2013 one of the changes I desired was the ability to create sub-projects which would support any number of tiered lists and therefore more advanced and complex lists of things that I needed to do.

Because I had this goal for this rebuilt version of Composed I knew tasks and projects would have to be treated quite similarly. And with the decision that each task could also receive its own due date, notes, and notifications, the two are almost identical in structure.

So here's where TaskProject comes into play.

Tasks and projects are both the same type: `TaskProject`. When needed, I can differentiate between the two types by using a computed property `kind` which checks to see how many children (which are also of type `TaskProject`) it has and returns either `.task` or `.project`, which are cases in an enum, such that if there are no children it's a task and if there are any it's a project.

What this allows me to do is easily convert a task to a project and vice versa just by adding or removing children.

#### Learn More

- For more reading on the development of Composed check out [my blog articles](/blog/tags) and look under "Composed."
- Download [Composed on the App Store](https://apps.apple.com/us/app/composed-tasks-and-projects/id1511037239?mt=8)
- Check out [Composed on Twitter](https://www.twitter.com/composedpm)
- Visit [Composed's homepage](https://www.composedpm.com)
