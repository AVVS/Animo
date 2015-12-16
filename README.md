# Animo
[![Version](https://img.shields.io/cocoapods/v/Animo.svg?style=flat)](http://cocoadocs.org/docsets/Animo)
[![Platform](https://img.shields.io/cocoapods/p/Animo.svg?style=flat)](http://cocoadocs.org/docsets/Animo)
[![License](https://img.shields.io/cocoapods/l/Animo.svg?style=flat)](https://raw.githubusercontent.com/eure/Animo/master/LICENSE)
[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)


Bring life to CALayers with SpriteKit-like animation builders.

---

## Introduction

GCDKit implements the following constructs. To see how they work, jump right to the **Common Usage Patterns** section below.

- `GCDQueue`: An abstraction of the `dispatch_queue_*` API. `GCDQueue`s are expressed as `enum`s, because we typically only need to deal with these queue types:

```swift
public enum GCDQueue {
    case Main
    case UserInteractive
    case UserInitiated
    case Default
    case Utility
    case Background
    case Custom(dispatch_queue_t) // serial or concurrent
}
```

- `GCDBlock`: A struct that abstracts iOS 8's new `dispatch_block_*` family of APIs.
- `GCDGroup`: An abstraction of the `dispatch_group_*` API.
- `GCDSemaphore`: An abstraction of the `dispatch_semaphore_*` API.

## Common Usage Patterns

Here are some common use cases and how you can write them using GCDKit.

### Creating and initializing queues

With the iOS SDK:

```swift
let mainQueue = dispatch_get_main_queue()
let defaultQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)
let customSerialQueue = dispatch_queue_create("mySerialQueue", DISPATCH_QUEUE_SERIAL)
let customConcurrentQueue = dispatch_queue_create("myConcurrentQueue", DISPATCH_QUEUE_CONCURRENT)
```

With GCDKit:

```swift
let mainQueue: GCDQueue = .Main
let defaultQueue: GCDQueue = .Default
let customSerialQueue: GCDQueue = .createSerial("mySerialQueue")
let customConcurrentQueue: GCDQueue = .createConcurrent("myConcurrentQueue")
```

In addition, custom queues created with `dispatch_queue_create(...)` typically target a default-priority global queue unless you use the `dispatch_set_target_queue(...)` API. For example, to raise the priority level of a created queue with the current SDK:

```swift
let customConcurrentQueue = dispatch_queue_create("myConcurrentQueue", DISPATCH_QUEUE_CONCURRENT)
dispatch_set_target_queue(customConcurrentQueue, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0))
```

With GCDKit, you can immediately specify the target queue on creation:
```swift
let customConcurrentQueue: GCDQueue = .createConcurrent("myConcurrentQueue", targetQueue: .UserInteractive)
```

### Submitting tasks to queues

With the iOS SDK:

```swift
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)) {
    // some heavy work...
    dispatch_async(dispatch_get_main_queue()) {
        // send results to UI
    }
}
```

With GCDKit you can either start a task chain from `GCDBlock.async(...)`:

```swift
GCDBlock.async(.Default) {
    // some heavy work...
}.notify(.Main) {
    // send results to UI
}
```

or directly from a specific `GCDQueue` `enum` value:

```swift
GCDQueue.Default.async {
    // some heavy work...
}.notify(.Main) {
    // send results to UI
}
```

You can chain `.notify(...)` calls as much as needed.

### Launching and reporting completion of a group of tasks

With the iOS SDK:

```swift
let dispatchGroup = dispatch_group_create()

dispatch_group_enter(dispatchGroup)
doSomeLongTaskWithCompletion {
    dispatch_group_leave(dispatchGroup)
}

dispatch_group_async(dispatchGroup, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)) {
    // do another long task
}
dispatch_group_async(dispatchGroup, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)) {
    // do yet another long task
}

dispatch_group_notify(dispatchGroup, dispatch_get_main_queue()) {
    // report completion
}
```

With GCDKit you can still use the usual `enter()`-`leave()` pairs and you can chain async calls to the dispatch group:

```swift
let dispatchGroup = GCDGroup()

dispatchGroup.enter()
doSomeLongTaskWithCompletion {
    dispatchGroup.leave()
}

dispatchGroup.async(.Default) {
    // do another long task
}.async(.Default) {
    // do yet another long task
}.notify(.Main) {
    // report completion
}
```

### Semaphores

With the iOS SDK:

```swift
let numberOfIterations: UInt = 10
let semaphore = dispatch_semaphore_create(Int(numberOfIterations))

dispatch_apply(numberOfIterations, dispatch_queue_create("myConcurrentQueue", DISPATCH_QUEUE_CONCURRENT)) {
    (iteration: UInt) -> Void in

    // do work for iteration
    dispatch_semaphore_signal(semaphore)
}

dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER)
```

With GCDKit:

```swift
let numberOfIterations: UInt = 10
let semaphore = GCDSemaphore(numberOfIterations)

GCDQueue.createConcurrent("myConcurrentQueue").apply(numberOfIterations) {
    (iteration: UInt) -> Void in

    // do work for iteration
    semaphore.signal()
}

semaphore.wait()
```


# Installation
- Requires:
- iOS 8 SDK and above
- Swift 2.0 (XCode 7 beta 6)
- Dependencies:
- [GCDKit](https://github.com/JohnEstropia/GCDKit)



### Install with Cocoapods
```
pod 'Animo'
```
This installs CoreStore as a framework. Declare `import CoreStore` in your swift file to use the library.

### Install with Carthage
```
github "JohnEstropia/CoreStore" >= 1.3.0
```

### Install as Git Submodule
```
git submodule add https://github.com/JohnEstropia/CoreStore.git <destination directory>
```
Drag and drop **CoreStore.xcodeproj** to your project.

#### To install as a framework:
Drag and drop **CoreStore.xcodeproj** to your project.

#### To include directly in your app module:
Add all *.swift* files to your project.


## License

Animo is released under an MIT license. See the LICENSE file for more information.
