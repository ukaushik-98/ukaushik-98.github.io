---
layout: post
title: "Send and Sync"
date: 2025-01-16 09:00:00 -0800
updated: 2025-02-01 09:00:00 -0800
categories: rust
---

### Send

A type is send if and only if it's safe to send across threads. Most types are send - the only types that break this inference are types that make assunptions on how they're used.

### Sync

A type T is sync if and only if &T is Send. Everything but the interior mutatbility types and RC are Sync.

### The special case of RC

RC is a particular type that makes strong assumptions on being single threaded. It leverages an innerstate and a counter to mantain how many references exist. Once the last reference is dropped, the entire type is dropped.

```rust
impl<T> !Send for RC<T>
impl<T> !Sync for RC<T>
```

### Arc

Arc is the primary way to share immutable pointers across threads, and implementation wise, it's almost entirely the same as RC. However, since we're in a multithreaded context, we must have the reference counter be atomic, thus the name Atomic Reference Counter.

Arc is interesting since we need it to be both Sync and Send. Sync is for the obvious reasons of being the primary method of sharing references across threads. Send, however, is necessary because we need drop permissions on the inner object.

```rust
impl<T: Send + Sync> Send for Arc<T>
impl<T: Send + Sync> Sync for Arc<T>
```

### Interior Mutability Types

These types allow mutable access to occur with only an immutable reference. Yet again, what we're seeing here is a single threaded assumption on the usage for these types. And it makes complete sense! Afterall, we want to avoid data races entirely. In order to achieve this though, the following strict assumptions are made:

1. The T in the Interior Mutability type is never used outside once it's placed in the Cell itself. Indeed, we see the enforcement of this by the Interior Mutability type taking ownership of the passed in T:

```rust
pub const fn new(value: T) -> Cell<T>
```

2. Interior Mutability types are not sync. Afterall, all it takes is one immutable pointer to mutate the inner type and we can't have that happening.

```rust
impl<T: Send> Send for Cell<T>
impl<T> !Sync for Cell<T>
```

#### Mutex

At this point, there's an important distinction be drawn between a Mutex and Mutex Guard. A Mutex is the type that encaplsulates thread safety primitives and provides methods on getting a lock safely. The only way to get the Guard is to call the lock function (this is an instance of a super useful pattern called typestate! It's another topic I'll be covering in a future post).

The actual thread safety trait bounds of a mutex are dictated entirely by if the inner type T is Send. The reason for the send bound is fairly obvious from the above passage but why is the Mutex not requiring T to be Sync? Afterall, it sounds counterintuitive considering that we can create any number of immutable pointers but still mutate the underlying item! This again ties back to the fact that access to the innter type first requires a lock to be taken. This means that a mutex is inherintly single threaded! Using this assumption we can safely drop the Sync requirement for a Mutex.

```rust
impl<T: Send> Send for Mutex<T>
impl<T: Send> Sync for Mutex<T>
```

#### Mutex Guard

MutexGuard is not Send because it explicitly requires the thread that took the lock to drop it on certain OS. However, the guard itself can be shared via sync since an immutable pointer cannot be used to drop or mutate it's interior structure.

```rust
impl<T> !Send for MutexGuard<T>
impl<T: Sync> Sync for MutexGuard<T>
```
