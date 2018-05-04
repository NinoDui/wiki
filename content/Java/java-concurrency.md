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
