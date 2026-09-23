---
title: 'Tests Now Or Tests Later?'
description: "We all know you're not going to test later"
pubDate: '2026-09-23'
tags: ["software", "testing", "unit testing", "definition of done"]
---

Everyone knows it's good practice to write unit tests, however nearly every project I've landed on, there's been very few, if any, unit tests at all. This has led to me building unit testing frameworks and implementing unit testing practices in a number of projects and there's been a couple of times where the following question has come up:

>Should we do the change now, get the PR through and then write the tests after?

I understand the goal here. There are people waiting downstream on a piece of work, be it other developers, manual testers etc. so getting the work to them ASAP would improve bandwidth and get work over the line faster, right?

Not necessarily.

I'll start from the manual testers perspective. It seems like it would be faster for unit testing and manual testing to take place in parellel since it's all testing afterall. What I don't like from this perspective is that you're essentially throwing an unknown quantity at your testers. I often see "you shouldn't just throw code over the fence for the testers", however this approach is just that, but with the promise that the developer is still looking at it.

Unit tests are much quicker to set up and execute compared to manual tests, and in some scenarios that does include writing them. You could discover an issue with the code faster than the manual testers, which would then warrant a new release. Since that code has changed, the manual testers probably should start their testing over again from the beginning, essentially wasting the time they'd already spent testing before the hotfix. Depending on how you approach this, it could happen multiple times per feature. That's a lot of time spent by testers looking at things that aren't ready for them yet.

From the perspective of other developers it could make even less sense. For developers reviewing your initial pull request, they lose the powerful validation tool which is the CI platform running the unit tests, giving them the assurance that "Yes, this new feature has passed these tests." Sure, they may get that assurance a bit later on, but working or not, the change is now merged into the main branch ready for others to use.

If a developer is waiting on this code change to progress their work then it's imperative that they get the working version of that feature. If during unit testing you discover an issue and have to make a change, that could then halt the progress of the other developer who's then got to merge that change into their branch. Depending on the change, this could result in that developer needing to rethink their approach to their implementation. If developer dependent on your feature is also making changes to code which you're currently writing and testing, they'll then have merge conflicts to consider. If the downstream developer has a small change, they could potentially be finished before your unit tests are ready and introduce a breaking change in your work which they can't easily identify because they don't yet have your unit tests to guide them.

Has time really been saved developing in this weird little [Hyrum's Law](https://www.hyrumslaw.com/) loop compared to waiting a bit longer for some unit tests?

Finally, are you _really_ going to write those tests later? I'm sure every project without unit tests I've started on, some good intentioned developer told themselves that they would write the unit tests later, let's just get the features out first. I feel like if you're saying that, you're lying to yourself. Features keep coming and the list of things to test is growing. Where do you even start at this point? It's overwhelming and everything looks to be working right now so best not to worry about it.

I've heard the argument in agile teams that because unit tests are part of the _Definition of Done_, the task stays in the _In Development_ status until they're written and merged. This serves as a reminder that tests need to be written but could easily fall under the radar. Since raising the PR for the main change is a milestone, it's easy to mentally give yourself a break from this task. If you're in an environment where you're switching context a lot, it could be days before you get around to picking up writing unit tests. By this point you're not as familiar with the scenario, so you'll need to rebuild the context before you start. Even then you may miss some tests since you're not as deep into the code as you were when you were initially writing it.

It's also easy to fall back into the pattern which caused us to not have unit tests in the first place. The change has been made, it's been deployed, the testers have looked at it, I can't remember all the details, there's other features in the _To Do_ column, I'll just move this task to completed without unit tests. Repeat this enough times and you're back to square one.

I've also had it suggested in an agile project that there should be separate tasks on the board for development and testing. This not only has the same issues as I've mentioned previously, but depending on the make up of the agile team, it potentially gives non-developers a say in the priority of unit tests, and since a customer isn't asking for unit tests, it's more likely to be de-prioritised versus a shiny new feature.

As with everything in software development, there's nuance to this. Maybe having a separate story for unit tests which get delivered as a separate PR work for you and your team, in which case don't change anything! However, weighing the pros and cons of this process, I agree that there may be unique scenarios where it makes sense to separate the work from the tests, but I don't think that the small amount of time potentially saved to parallelise a downstream piece of work is worth the risk of not shipping a change with its unit tests as standard. So I have always advocated for changes and their tests to be together.