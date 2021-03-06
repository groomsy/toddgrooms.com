---
layout: post
title: Introducing CoreDataMate
comments: true
tags: development
---

A few years ago, I started development on a project that required long term persistence. Up until that point, most all of my persistence was handled on the server. This project required that I persist data locally on the user's device, in conjunction with persisting data remotely on a server. As this was an iOS project and Apple is "all-in" with Core Data, I decided I had better get "all-in" with Core Data as well.

I read through every Core Data tutorial I could find on the internet. Regardless of how many times I tried, I just could not grasp what was happening in Core Data. After complaining about this to some of my coworkers, I had a few of them recommend [MagicalRecord](https://github.com/magicalpanda/MagicalRecord). I am not going to question the quality of this library; I have no doubt that it is well written and works as advertised. The issue is that I don't understand what the library is doing behind the curtain. I use libraries all the time, but when I use them, I want to understand what they are doing (more or less) without blindly trusting them. For whatever reason, I simply cannot wrap my head around what MagicalRecord is doing. It seems as though you can arbitrarily create objects on any context and save them. This is one of the library's goals. However, one thing I have come to understand is that you need to ensure that every thread has it's very own context so that you don't "cross the streams" so to speak. Try as I might, I simply could not ensure this using MagicalRecord. This is when I decided to go old school and purchase a book: [Core Data (2nd Edition): Data Storage and Management for iOS, OS X, and iCloud](http://pragprog.com/book/mzcd2/core-data).

![Don't cross the streams](/images/dont_cross_the_streams.jpg)

After reading through _Core Data (2nd Edition)_, I started writing what I called a "data manager". The data manager is a simple class that acts as a gateway to your persistent store. The data manager houses a private context which is considered to be the master copy. You do not have direct access to this context from any public facing methods in the data manager. The context that you work with predominantly will be a child context of the private context. This child context is a main queue context that I have appropriately named `mainContext`. The main queue context will be the context that you predominantly work with. This is the context with which your UI will directly interact.

This is all great, except for the very real use case when you will need to write content to your persistent store. What then? Well, the data manager's current pattern dictates that you create a new temporary context for your thread (this can even be the main thread if you like, the point is that you just want a temporary context with which to work) whose parent will be the data manager's `mainContext`; you will then make any changes that are necessary on this temporary context. After you are finished, you'll persist your temporary context which will write those changes upstream to the data manager's `mainContext`. If that persist is successful, simply call:

    [[CDMDataManager sharedManager] persist:YES];

which will write the `mainContext`'s changes upstream to the data manager's private context (the BOOL value indicates that it should be a synchronous save; a NO here would result in an asynchronous save). At that point, your new content is in your persistent store and you have the added benefit of your UI already being updated with this content as your changes already exist on the `mainContext`.

This is a rudimentary example, but I wanted to show how easy Core Data can be to work with, with a bit of caution and understanding. Ready to check out the source code or give it a spin yourself? Head on over to [CoreDataMate on Github](https://github.com/groomsy/coredatamate) to check out the source. If you're impatient and want to play with the code (and you use [Cocoapods](http://cocoapods.org)) just add this to your Podfile: `pod "CoreDataMate"`.