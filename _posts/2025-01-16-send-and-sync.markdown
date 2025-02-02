---
layout: post
title: "Send and Sync"
date: 2025-01-16 09:00:00 -0800
updated: 2025-02-01 09:00:00 -0800
categories: rust
---

### Send

A type is send if and only if it's safe to send across threads. Most types are send - the only types that break this inference are types that make assunptions on how they're used.

#### Mutex

At this point, there's an important distinction be drawn between a Mutex and Mutex Guard. A Mutex is the type that encaplsulates thread safety primitives and provides methods on getting a lock safely. The only way to get the Gaurd is to call the lock function (this is an instance of a super useful pattern called typestate! It's another topic I'll be covering in a future post).

The actual thread safety trait bounds of a mutex are dictated entirely by if the inner type T is Send. The reason for the send bound is fairly obvious from the above passage but why is the Mutex not requiring T to be Sync? Afterall, it sounds counterintuitive considering that we can create any number of immutable pointers but still mutate the underlying item! This again ties back to the fact that access to the innter type first requires a lock to be taken. This means that a mutex is inherintly single threaded! Using this assumption we can safely drop the Sync requirement for a Mutex.

```rust
impl<T: Send> Send for Mutex<T>
impl<T: Send> Sync for Mutex<T>
```

#### Mutex Gaurd

MutexGuard is not Send because it explicitly requires the thread that took the lock to drop it on certain OS. However, the guard itself can be shared via sync since an immutable pointer cannot be used to drop or mutate it's interior structure.

#### RC

RC is a particular type that makes strong assumptions on being single threaded. Being

### Sync

#### Interior Mutability Types
