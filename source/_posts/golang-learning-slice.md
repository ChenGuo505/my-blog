---
title: Understanding Slice in Golang
date: 2025-05-04 23:35:08
tags:
  - Golang
categories:
  - Study
description:
  In this post, we’ll explore the internal structure of slices, how they behave with append, the cost of copying, and tips to use them safely and efficiently.
---

Go’s slice is one of the most commonly used yet often misunderstood types. It looks like a simple dynamic array on the surface, but under the hood, slices have some subtle behaviors that every Go developer should understand to avoid bugs and performance issues.

## What Is a Slice, Really?

A **slice** in Go is a lightweight data structure that provides a view into an underlying array. It is defined by three components:

```go
type slice struct {
    ptr    *T     // pointer to the underlying array
    len    int    // number of elements in the slice
    cap    int    // capacity: max number of elements from ptr
}
```

In other words, a slice does **not** own its data. It’s just a window over an array. This makes slicing operations fast — but also introduces pitfalls.

## **Slice Creation: Literal,** make, and nil

There are multiple ways to create a slice:

```go
// Slice literal (backed by array of length 3)
a := []int{1, 2, 3}

// Make slice with length 5, capacity 10
b := make([]int, 5, 10)

// Nil slice: length and capacity are both 0, not allocated
var c []int
```

Understanding the difference between a nil slice and an empty but allocated slice is important when checking for initialization.

```go
var s []int         // nil slice
t := make([]int, 0) // non-nil empty slice

fmt.Println(s == nil) // true
fmt.Println(t == nil) // false
```

## Slice Growth and Append

One of the most powerful features of slices is append:

```go
s := []int{1, 2}
s = append(s, 3) // creates new slice if capacity is exceeded
```

When the capacity is not enough, Go allocates a **new backing array**, copies the old elements, and appends the new ones.

**But beware**: this means if you share a slice between variables and append on one, the changes might or might not be visible to the others — depending on whether a new array was allocated.

```go
a := []int{1, 2, 3}
b := a[:2]
b = append(b, 99)
fmt.Println("a:", a) // Might print [1 2 99], depending on capacity
```

To be safe, assume slices can **share memory** and mutate each other unless explicitly copied.

## Copying Slice

If you want a true copy of a slice, use copy:

```go
original := []int{1, 2, 3}
clone := make([]int, len(original))
copy(clone, original)
```

This ensures the two slices are backed by different arrays.

## Slicing and Memory Leaks

Since slices point to the underlying array, if you take a small slice from a large array, the entire array remains in memory. This can cause **unexpected memory leaks**.

```go
big := make([]byte, 1<<20)  // 1MB array
small := big[:10]           // small slice
big = nil                   // big is gone, but the array isn’t
```

Here, small still holds a reference to the entire 1MB array. To avoid this, explicitly copy the data:

```go
smallCopy := append([]byte(nil), small...)
```

## Reslicing and Capacity

You can reslice a slice as long as you stay within capacity:

```go
a := []int{1, 2, 3, 4, 5}
b := a[:3]     // [1 2 3]
c := b[:cap(b)] // [1 2 3 4 5]
```

This is useful for buffer reuse, but can also be dangerous if you’re not careful — reslicing too far will panic.

## Pre-allocating for Performance

When you know the size of a slice in advance, pre-allocate it with make to avoid multiple reallocations:

```go
data := make([]int, 0, 1000) // zero-length slice with room for 1000 elements
```

Using append on this slice is efficient because it avoids frequent copying during growth.

## Gotchas and Tips

1. **Slice assignment shares the same backing array**:

   ```go
   a := []int{1, 2}
   b := a
   b[0] = 9 // a[0] is also 9
   ```

2. **Modifying subslices affects the original**:

   ```go
   full := []int{1, 2, 3}
   part := full[1:]
   part[0] = 99 // full[1] == 99
   ```

3. **Be cautious with loop variables and slices**:

   ```go
   var res []*int
   for _, v := range []int{1, 2, 3} {
       res = append(res, &v) // All point to same `v`
   }
   ```

   Instead, capture v in a new variable inside the loop.

## Final Thoughts

Slices are powerful but subtly tricky. Understanding their backing array, growth behavior, and memory sharing is crucial for writing performant and bug-free Go code. Whether you’re building a high-throughput server or just manipulating data collections, a solid grasp of slices will take you far.

**Further Reading:**

- [Go Blog: Go Slices: usage and internals](https://go.dev/blog/slices-intro)
- [Go source code (slice implementation)](https://github.com/golang/go)
