---
title: Understanding Concurrency in Golang
date: 2025-05-05 23:45:40
tags:
  - Golang
categories:
  - Study
description:
  In this post, I’ll explore what goroutines and channels are, how they work under the hood, and how to use them effectively — all explained in a clear and beginner-friendly way.
---

One of Go’s standout features is its built-in support for **concurrency**, made possible through **goroutines** and **channels**. These two primitives make it easy to write concurrent programs that are readable, efficient, and safe.

## What is a Goroutine?

A **goroutine** is a lightweight thread managed by the Go runtime.

Starting a goroutine is as simple as using the go keyword:

```go
go doSomething()
```

That’s it — now doSomething() runs **concurrently** with the rest of your program.

Unlike traditional threads, goroutines:

- Are extremely lightweight (a few KB of stack to start)
- Are multiplexed onto system threads by the Go scheduler
- Scale efficiently — you can easily run thousands of them

**Example:**

```go
func sayHello() {
    fmt.Println("Hello from goroutine!")
}

func main() {
    go sayHello()
    fmt.Println("Hello from main!")
    time.Sleep(1 * time.Second)
}
```

Without the time.Sleep, the program might exit before the goroutine finishes — because the main function exits immediately. This highlights the need for **synchronization**, which brings us to…

## What is a Channel?

A **channel** is a typed conduit through which goroutines can communicate.

Think of channels as **pipes** that connect goroutines. One goroutine sends data, and another receives it.

```go
ch := make(chan int) // create a channel of int
```

**Basic Usage**

```go
func worker(ch chan string) {
    msg := <-ch // receive from channel
    fmt.Println("Received:", msg)
}

func main() {
    ch := make(chan string)
    go worker(ch)
    ch <- "Hello, channel!" // send into channel
}
```

When you send or receive on a channel, the operation blocks until the other side is ready. This is known as **synchronous communication**.

## Channel Direction

You can restrict the direction of channel usage to make APIs clearer:

```go
func send(ch chan<- int) {
    ch <- 42
}

func receive(ch <-chan int) {
    fmt.Println(<-ch)
}
```

This helps prevent bugs by making the contract explicit — a great feature for building reusable components.

## Buffered Channels

By default, channels are **unbuffered**: they block until both sender and receiver are ready.

You can also create **buffered** channels:

```go
ch := make(chan int, 2)
ch <- 1
ch <- 2
// ch <- 3 would block because buffer is full
```

Buffered channels are useful when you want to decouple sender and receiver speed.

## Select: Waiting on Multiple Channels

Go provides the select statement to wait on multiple channel operations:

```go
select {
case msg1 := <-ch1:
    fmt.Println("Received", msg1)
case msg2 := <-ch2:
    fmt.Println("Received", msg2)
default:
    fmt.Println("No message received")
}
```

This is especially powerful for building responsive systems, implementing timeouts, or handling multiple sources of data.

## Common Pitfalls and Best Practices

### Deadlocks

A deadlock happens when all goroutines are waiting and none can proceed. Example:

```go
func main() {
    ch := make(chan int)
    ch <- 1 // blocks forever (no receiver)
}
```

Always ensure sends and receives match up, or use buffering carefully.

### Leaky Goroutines

Forgetting to cancel or return from goroutines can lead to **goroutine leaks**, which consume memory and resources.

Use context cancellation or done channels:

```go
func worker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            return
        default:
            // work
        }
    }
}
```

### Avoid Sharing Memories by Default

Go’s philosophy is:

> “**Do not communicate by sharing memory; share memory by communicating.**”

Instead of locking shared state with mutexes, try to use channels to synchronize access.

## Real-World Example: Fan-Out Pattern

Let’s say you want to parallelize some work:

```go
func worker(id int, jobs <-chan int, results chan<- int) {
    for j := range jobs {
        fmt.Println("Worker", id, "processing job", j)
        time.Sleep(time.Second)
        results <- j * 2
    }
}

func main() {
    jobs := make(chan int, 5)
    results := make(chan int, 5)

    for i := 1; i <= 3; i++ {
        go worker(i, jobs, results)
    }

    for j := 1; j <= 5; j++ {
        jobs <- j
    }
    close(jobs)

    for i := 1; i <= 5; i++ {
        fmt.Println("Result:", <-results)
    }
}
```

This is a classic **fan-out/fan-in** pattern — multiple workers process jobs in parallel and send results back.

## Conclusion

Goroutines and channels are the foundation of Go’s concurrency model. With just a few keywords (go, chan, select), you can build complex, concurrent systems that are still readable and safe.

If you’re new to concurrency, Go is a fantastic place to start. Just remember:

- Use channels to coordinate work
- Avoid shared memory unless absolutely necessary
- Watch out for goroutine leaks and deadlocks
- Leverage context for cancellation

**Further Reading**

- [Go Tour: Concurrency](https://go.dev/tour/concurrency/1)
- [Effective Go - Concurrency](https://go.dev/doc/effective_go#concurrency)
- [Go Blog: Concurrency Patterns](https://go.dev/blog/pipelines)
