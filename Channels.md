# Channels 🤝🏼

## Understanding Concurrency

* **Concurrency** means the ability to make progress on **multiple tasks within the same time window**, even if they are not all executing at the exact same CPU moment.

* The opposite of concurrency is **sequential (synchronous) execution**, where code runs strictly line by line and one task must fully finish before the next can start.

* Concurrency is **not the same as parallelism**:
  * **Concurrency** is about *structure* (how tasks are organized and interleaved).
  * **Parallelism** is about *hardware execution* (tasks literally running at the same time on multiple CPU cores).

* On a **single-core CPU**, concurrency is achieved by **rapid task switching** (time slicing).

* On a **multi-core CPU**, concurrency may become **true parallelism**, where tasks run simultaneously.

* Importantly, **the same Go code** works in both cases; the runtime adapts to available hardware.

* Final mental model:
  * JavaScript concurrency = *event-loop-driven task switching*
      - Event-loop is a scheduler that lets JS **pause** functions at `await`, `fetch` (because they are **handed off** to the browser or runtime(the host environment that embeds the JS engine), not JS engine itself, like V8 in Chrome), run other queued work on the same thread and resume paused functions later
      - For example, when we do fetch(...) or set a timer, the JS engine does not implement TCP, DNS, TLS, or timer interrupts. Instead, it requests: “runtime, start this operation and tell me when it’s done.” The runtime performs the operation (often using OS mechanisms) and later signals completion back to the JS engine by placing a callback/continuation into a queue.
        * Scenario: user clicks a button while a network request is in flight.
  * Flow:

    * JS starts running:

      * `await fetch(...)` begins the network request.
      * The async request is handled by the browser network stack (not the JS engine).
      * The async function pauses at `await`.
    * The JS thread becomes free and the event loop can now run other queued tasks.
    * User clicks the page:

      * The click event handler is queued.
      * The event loop runs the click handler on the same JS thread.
    * Later, the fetch completes:

      * The runtime queues the continuation (“resume after await”).
      * The event loop eventually runs that continuation, and your async function continues on the next line.
  * Go concurrency = *runtime-scheduled goroutines*
  * Parallelism depends on hardware, but concurrency depends on program structure
## Threads
What *is* true:

* At any **instant**, a single **core** can execute **one instruction stream**.
* That instruction stream belongs to **one thread** at that moment.

* If you have:

  * 4 cores
  * 100 threads
* At most **4 threads** can run **at the same time**.
* The OS scheduler:

  * runs some threads
  * pauses them
  * resumes others
  * possibly on different cores

> A thread is an instruction stream; a CPU core executes one instruction stream at a time, and the OS dynamically schedules threads onto cores, so there is no permanent 1:1 mapping.

* **Single core**:

  * Executes **one** instruction stream at a time
  * Appears concurrent via **rapid switching**
* **Multiple cores**:

  * Each core executes its **own** instruction stream
  * True parallelism occurs

Example:

* 4 cores → up to 4 instruction streams truly parallel
* More threads than cores → time slicing

## Goroutines
```
Goroutines
   ↓ (Go runtime scheduler)
OS Threads (instruction streams)
   ↓ (OS scheduler)
CPU Cores
```
* Go was designed with concurrency as a **core language feature**, not a library add-on.

* Concurrency in Go is expressed using the `go` keyword:

  * `go f()` means: *start this function concurrently and do not wait for it here*.

* Using `go` spawns a **goroutine**, which is:
  * a lightweight, runtime-managed execution unit
  * much cheaper than an OS thread
  * scheduled by the Go runtime, not the operating system directly

* Goroutines are **not OS threads**:
  * They are "green threads" managed by the Go runtime (user-space scheduled, not 1:1 mapped to OS threads).
  * The runtime multiplexes many goroutines onto fewer OS threads (an M:N scheduler).
  * Why this matters: creating/switching goroutines is much cheaper than OS threads, and the runtime can pause/resume them around blocking I/O.

* Go controls parallelism with `GOMAXPROCS`:
  * If `GOMAXPROCS = 1`, all goroutines run on one OS thread (concurrency without parallelism).
  * If `GOMAXPROCS > 1` (default on multi-core machines), goroutines may run in true parallel on multiple cores.

* A common Go pattern is to **fire off slow work** (network calls, disk I/O, background processing) in goroutines so the main flow continues without blocking.

## Go vs JavaScript Concurrency

* Goroutines solve the same *class* of problems as async code in JavaScript: **non-blocking progress**.

* JavaScript uses a **single-threaded event loop** model:

  * Only **one thread** executes JavaScript code.
  * `async / await` does **not** create new threads.
  * `await` means: *pause this function and give control back to the event loop*.

* When a JavaScript function hits `await`:

  * The function’s execution is **paused**.
  * The rest of that function (the “next line”) is **not executed yet**.
  * The engine stores the continuation of the function.
  * The single JS thread is now free to run:

    * other event handlers
    * timers
    * promise callbacks
    * remaining top-level code

* Later, when the awaited operation completes, the event loop schedules the continuation of the paused function.

* This explains a key JS concept:

  * The thread is **busy**, but not **blocked**.
  * Other tasks run, but **not the next line of the same function**.

* `await` therefore enforces **function-level blocking**, not **thread-level blocking**.

* Conceptual mapping between Go and JavaScript:

  * `go f()` in Go ≈ “schedule work and don’t wait”
  * `await f()` in JS ≈ “pause here until done”
  * calling `f()` directly in Go ≈ synchronous execution

* Crucial distinction:
  * JavaScript is **fundamentally single-threaded** for user code.
  * Go can scale from single-threaded concurrency to multi-core parallelism **without changing code**.

## What Are Channels?

* Channels are a typed communication + synchronization primitive that allows different goroutines to communicate.
  * Buffered channels can behave like a small queue.
  * Unbuffered channels behave like a **rendezvous** (a handoff).

* **Key concept: Variables store values. Channels coordinate execution.**

* The problem channels solve is: **coordination across goroutines**.

### Why `return` Works in Synchronous Code

In *normal* (synchronous) code:

```go
func A() {
  B()
  fmt.Println("after B")
}
```

What does `return` from `B()` mean?

* `A` **cannot continue** until `B` finishes.
* The call stack enforces waiting.
* Execution is linear.

So yes — **return is already a perfect signal** in synchronous code.

### What Breaks When Concurrency Enters?

Now change one thing:

```go
func A() {
  go B()
  fmt.Println("after B")
}
```

This single keyword changes the meaning of `return` as a signal:

* `B` is no longer part of `A`’s call stack.
* `A` does not wait for `B`.
* `B` may finish before, after, or never relative to `A`.

So now ask:

> When `B` returns… who is waiting?

Answer: **nobody**.

### Why `return` Cannot Signal Across Goroutines

A return:

* signals only the **caller**
* on the **same call stack**
* in the **same goroutine**

But goroutines don’t share a call stack. They run independently.

So:

> **Return is a control-flow signal, not a synchronization signal.**

### What Do We Need Instead?

We need something that:

* works *across goroutines*
* allows waiting
* allows ordering
* allows safe memory visibility

That thing is a **channel**.

### Why a “Handle” Is Necessary

Once you start work in the background:

```go
go doWork()
```

You have cut the caller off from:

* when the work finishes
* whether it finished
* whether it even started

Unless you give the caller a **handle**.

A handle is:

* not the data
* not the call stack
* but a synchronization point

That’s what the channel is.

### Why Return Cannot Replace the Channel

```go
func downloadData() {
  go func() {
    // work
  }()
}
```

This function returns immediately. The caller has no place to block.

So you must return (or pass in) **something else** that represents completion.

### Why Channels Are Visible in Multiple Scopes (and Why That’s OK)

It’s normal (and intentional) for a channel to appear in multiple scopes:

* one function **creates** the channel
* a goroutine **signals** on it
* another goroutine **waits** on it

This is **shared coordination**, not shared state.

Key distinction:

* shared variables = dangerous
* shared channels = intentional synchronization

### One Sentence That Usually Makes It Click

> In concurrent code, “return” answers “what is the result?” Channels answer “when is it safe to continue?”

Forget the syntax for a moment:

A **channel** is simply a safe meeting point where one goroutine hands a value to another.

Two guarantees:

* the value exists
* the timing is correct

Imagine two people:

* **Sender**: “I will give you a value.”
* **Receiver**: “I will wait until you give it to me.”

Neither proceeds until both are present.

That’s a channel.

### Rendezvous (Unbuffered Channels)

* An **unbuffered** channel (`make(chan T)`) has no capacity.
* A send and a receive must meet at the same time: this is a **rendezvous**.
  * Send blocks until a receiver is ready.
  * Receive blocks until a sender is ready.
  * When using channels as completion signals, the receiver must consume exactly one signal per completed task, otherwise the sender will block forever.

### Buffered Channels (Queue-like Behavior)

* A **buffered** channel (`make(chan T, n)`) has capacity `n`.
  * Send blocks only when the buffer is **full**.
  * Receive blocks only when the buffer is **empty**.
* This is why “channel = queue” is sometimes true, but only really matches buffered channels.

### Happens-Before Relationship (Why Channels Matter)

* Channels don’t just move values — they also create a **happens-before** relationship.

* Intuition: a happens-before edge means “all writes done before this point in goroutine A are guaranteed to be visible after this point in goroutine B”.

* In practice (the key guarantee you lean on):
  * If goroutine A does some work, then **sends** on a channel, and goroutine B **receives** that value, then A’s work **happens-before** B continues past the receive.
  * So channels give you both:
    * the right **timing** (blocking/coordination)
    * the right **memory visibility** (no stale reads if you synchronize through the channel)

- The `<-` operator is called the **channel operator**.
  - Send: `ch <- 69`
  - Receive: `v := <-ch`
  - (Mnemonic: send points **into** the channel, receive points **out of** the channel.)
- Blocking rule of thumb:
  - Unbuffered channel: send/receive block until they rendezvous.
  - Buffered channel: send blocks when full, receive blocks when empty.
- A **deadlock** happens when *everyone is waiting, and nobody can move*:
    - Sending to a channel blocks until someone receives.
    - Receiving from a channel blocks until someone sends.
