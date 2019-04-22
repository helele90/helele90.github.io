---
layout: post
title: "Swift Runtime"
date: 2019-04-22
categories: iOS
tags: [iOS, Swift]
---
# Swift Runtime
```c++
Class swift::swift_getInitializedObjCClass(Class c) {
  // Used when we have class metadata and we want to ensure a class has been
  // initialized by the Objective-C runtime. We need to do this because the
  // class "c" might be valid metadata, but it hasn't been initialized yet.
  return [c class];
}
```
## abc
