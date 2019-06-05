---
layout: post
title: Threads and Locks
date: 2016-07-12
tag: Leetcode
---

### Difference between Threads and Processes

```
Processes and threads are related to each other but are fundamentally different.

A process can be thought of as an instance of a program in execution. A process is an independent entity to which system resources (e.g., CPU time and memory) are allocated. Each process is executed in a separate address space, and one process cannot access the variables and data structures of another process. If a process wishes to access another process' resources, inter-process communications have to be used.These include pipes, files, sockets, and other forms.

A thread exists within a process and shares the process' resources (including its heap space). Multiple threads within the same process will share the same heap space.This is very different from processes, which cannot directly access the memory of another process. Each thread still has its own registers and its own stack, but other threads can read and write the heap memory.
A thread is a particular execution path of a process. When one thread modifies a process resource, the change is immediately visible to sibling threads.
```

### Synchronized Blocks in Java

Only one thread in a multi-thread process can run the block at a time:

```
public class MyClass extends Thread {
  ...
  public void run() {
    myObj.foo(name);
  }
}
public class MyObject {
  public void foo(String name){
    synchronized(this){
      ...
    }
  }
}
```

### Locks

A thread gets access to a shared resource by first acquiring the lock associated with the resource. At any given time, at most one thread can hold the lock and, therefore, only one thread can access the shared resource. 只有取到了lock的thread才能继续运行,其它必须等待.

```
public class ATM {
  private Lock lock;
  private int balance = 100;

  public ATM(){
    lock = new ReentrantLock();
  }

  public withDraw(int value) {
      lock.lock();            <---- lock
      int temp = balance;
      try {
        Thread.sleep(100);
        temp = temp - value;
        Thread.sleep(100);
        balance = temp;
      } catch (InterruptedException e){ }
      lock.unlock();         <---- unlock
      return temp;
  }

  ....
}
```

### DeadLock

最著名的死锁的例子是 Dinning Philosophers 问题.

还有一个例子是AB两人同时转账的问题.
