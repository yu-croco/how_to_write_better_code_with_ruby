# How to write better code with Ruby
- This repository is about how to write better (cleaner/ more receivable) code with Ruby.
- This is inspired by [Sandi Metz' Rules For Developers - Thoughtbot](https://thoughtbot.com/blog/sandi-metz-rules-for-developers) and [Practical Object-Oriented Design in Ruby](https://www.amazon.co.jp/Practical-Object-Oriented-Design-Ruby-Addison-Wesley/dp/0321721330/ref=sr_1_6?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&crid=2EGHAGFK7S380&keywords=object+oriented+programming&qid=1563603741&s=gateway&sprefix=object+ori%2Caps%2C295&sr=8-6). I recommend you to read them. :)

## Table of Contents
1. The most important things when you write code
2. Concrete simple example to write better code with Ruby
3. Other tips to write cleaner codes (coming soon...)

## 1. The most important things
Although there are many theories, DRY / SOLID / design patterns or so on, in the world, one of the most important things is `Separation by interest` called Single Responsibility Principle in my opinion.

There are a few reasons.
- We need to implement feature, considering specification changes.
  - If we implement a feature at one method and specification changes happen...
- Simple code helps you to understand easily what it really does.
  - easy to read, understand and modify.

In the real world, we always encounter specification changes, like or not.
So we need to have to care how to implement freatures with `flexible code`.
In order to implement features with flexible code, `Separation by interest` is the key.

