---
title: "Java-Concurrency"
date: 2018-05-04 11:13
category: Java
tag: Java, Concurrency, Oracle
---

[TOC]

# Concurrency

## Processes and Threads

> In the Java programming language, concurrent programming is mostly concerned with **threads**.

### Processes

A process generally has a complete, private set of basic run-time resources; **each process has its own memory space**.

Processes are often seen as synonymous with programs or applications. However, what the user sees as a single application may in fact be a set of cooperating processes.

*Inter Process Communication (IPC)* resources, such as pipes and sockets, used not just for **communication between processes** on the same/different system.

### Threads

*lightweight processes*, both processes and threads provide an execution environment.

Threads exist within a process — every process has at least one. Threads share the process's resources, including memory and open files.

## Thread Objects

Each thread is associated with an instance of the class `Thread`.

There are two basic strategies for using Thread objects to create a concurrent application:
-   directly control thread creation and management, simply instantiate `Thread` each time the application needs to initiate an asynchronous task.
-   abstract thread management from the rest of your application, pass the application's tasks to an *executor*. (High level)

### Defining and Starting a Thread

An application that creates an instance of `Thread` must provide the code that will `run` in that thread, via two ways:
-   Object implements `Runnable`, passed to the `Thread` constructor. (**More General**)
-   Subclass `Thread`
and then invoke `Thread.start`

### Pausing Execution with Sleep

`Thread.sleep` causes the current thread to suspend execution for a specified period. This is an efficient means of **making processor time available** to the other threads of an application or other applications that might be running on a computer system.

### Interrupts

An *interrupt* is an **indication** to a thread that it should stop what it is doing and do something else. A thread sends an interrupt by invoking `interrupt` on the Thread object for **the thread to be interrupted**.

#### Supporting Interruption

Many methods that throw `InterruptedException`, such as `sleep`, are designed to **cancel their current operation and return immediately** when an interrupt is received.

```Java
for (int i = 0; i < info.length; ++i) {
    try {
        Thread.sleep(4000);
    } catch (InterruptedException e) {
        return;
    }
    System.out.print(info[i]);
}
```

If a thread goes a long time without invoking a method that throws `InterruptedException`, it must **periodically invoke `Thread.interrupted`**, which returns true if an interrupt has been received.

```java
if (Thread.interrupted()) {
    throw new InterruptedException();
}
```

#### The Interrupt Status Flag

The interrupt mechanism is implemented using an internal flag known as the *interrupt status*.
-   `Thread.interrupt` sets this flag.
-   static method `Thread.interrupted`, which thread checks for an interrupt by invoking, interrupt status is **cleared**.
-   non-static `isInterrupted` method, used by one thread to query the interrupt status of another, does not change the interrupt status flag.
-   **any method** that exits by throwing an `InterruptedException` clears interrupt status when it does so

### Joins

The `join` method allows one thread to **wait for the completion of another**.

Like `sleep`, `join` responds to an interrupt by exiting with an InterruptedException.

## Synchronization

### Thread Interference

Interference happens when two operations, running in different threads, but acting on the same data, *interleave*. This means that the two operations consist of multiple steps, and the sequences of steps overlap.

### Memory Consistency Errors

*Memory consistency errors* occur when different threads have inconsistent views of what should be the same data. The key to avoiding memory consistency errors is understanding the **happens-before** relationship. This relationship is simply a guarantee that memory writes by one specific statement are visible to another specific statement.

A list of actions that create happens-before relationships ([Summary page of the `java.util.concurrent` package](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/package-summary.html#MemoryVisibility))

> The results of a write by one thread are guaranteed to be visible to a read by another thread only if the write operation **happens-before** the read operation.

1.  The `synchronized` and `volatile` constructs, as well as the `Thread.start()` and `Thread.join()` methods, can form happens-before relationships.
2. Each action in a thread happens-before every action in that thread that comes later in the program's order.
3. An unlock (`synchronized` block or method exit) of a monitor happens-before every subsequent lock (`synchronized` block or method entry) of that same monitor. And because the happens-before relation is transitive, all actions of a thread prior to unlocking happen-before all actions subsequent to any thread locking that monitor.
4. A write to a `volatile` field happens-before every subsequent read of that same field. Writes and reads of `volatile` fields have **similar memory consistency effects** as entering and exiting monitors, but **do not entail mutual exclusion locking**.
5. A call to start on a thread happens-before any action in the started thread.
6. All actions in a thread happen-before any other thread successfully returns from a `join` on that thread.

(For High Level *Happen-before*, please refer to the link.)

### Synchronized Methods

Two basic synchronization idioms:
1. synchronized methods
2. synchronized statements (more complex)

When one thread is executing a synchronized method for an object, all other threads that invoke synchronized methods for the same object block (suspend execution) until the first thread is done with the object. When a synchronized method exits, it automatically establishes a happens-before relationship with any subsequent invocation of a synchronized method for the same object.

Constructors cannot be synchronized. `final` fields are important exception, which cannot be modified after the object is constructed, can be safely read through non-synchronized methods, once the object is constructed.

### Intrinsic Locks and Synchronization

Synchronization is built around an internal entity known as the `intrinsic lock` or `monitor lock`. When a thread invokes a synchronized method, it automatically acquires the intrinsic lock for that method's object and releases it when the method returns. The lock release occurs even if the return was caused by an uncaught exception. When a static synchronized method is invoked, the thread acquires the intrinsic lock for the `Class` object associated with the class. Thus access to class's static fields is controlled by a lock that's distinct from the lock for any instance of the class.

Synchronized statements must **specify the object that provides the intrinsic lock**.

```Java
public void addName(String name) {
    synchronized(this) {
        lastName = name;
        nameCount++;
    }
    nameList.add(name);
}
```

A thread cannot acquire a lock owned by another thread.

### Atomic Access

An *atomic* action is one that effectively happens all at once.

- Reads and writes are atomic for reference variables and for most primitive variables (all types except `long` and `double`).
- Reads and writes are atomic for all variables declared volatile (including long and double variables).

## Liveness

A concurrent application's ability to execute in a timely manner is known as its **liveness**.

### Deadlock

Deadlock describes a situation where two or more threads are blocked forever, **waiting for each other**.

### Starvation and Livelock

*Starvation* describes a situation where a thread is unable to gain regular access to shared resources and is unable to make progress. This happens when shared resources are made unavailable for long periods by "greedy" threads.

A thread often acts in response to the action of another thread. If the other thread's action is also a response to the action of another thread, then *livelock* may result. As with deadlock, *livelocked* threads are unable to make further progress. 
