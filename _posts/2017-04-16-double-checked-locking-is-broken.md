---
layout: post
title: "Double Checked Locking is Broken"
description: ""
category: "design pattern"
tags: [singleton]
---
{% include JB/setup %}

> [The "Double-Checked Locking is Broken" Declaration](http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html)

**Double-Checked Locking** 被广泛引用，并且在多线程环境中被作为一种延迟初始化的有效方式。

不幸的是，它在与平台无关的 `Java` 语言实现中，在没有额外的同步条件下，并不能保证其可信性。比如在 `C++` 语言中的实现，需要依赖于处理器的内存模型，编译器的重排执行和编译器与同步类库之间的交互。由于以上条件在 `C++` 语言中没有明确规定，因此很难确定地说哪种情形下这种锁机制才管用。我们确实可以通过显示地内存屏障(memory barriers)让其在 `C++` 中正常工作，但这并不适用于 `Java`。

为了先解释我们所期望的行为，考虑下面的代码：

``` java
// Single threaded version
class Foo {
	private Helper helper = null;
	public Helper getHelper() {
		if (helper == null)
			helper = new Helper();
		return helper;
	}
	// other functions and members...
}
```

如果这段代码在多线程环境中执行，会出现许多问题。最明显的是，不止一个 `Helper` 对象被构造。(稍后我们会提到更多的问题)。解决这个问题只需要简单的添加 `synchronize` 到 `getHelper()` 方法即可。

``` java
// Correct multithreaded version
class Foo {
	private Helper helper = null;
	public synchronized Helper getHelper() {
		if (helper == null)
			helper = new Helper();
		return helper;
	}
	// other functions and members...
}
```

上面的代码会同步执行 `getHelper()`。**double-checked locking** 原语试图在 `helper` 被构造一次之后避免同步。

``` java
// Broken multithreaded version
// "Double-Checked Locking" idiom
class Foo {
	private Helper helper = null;
	public Helper getHelper() {
		if (helper == null) {
			synchronized(this) {
				if (helper == null)
					helper = new Helper();
			}
			return helper;
		}
		// other functions and members...
	}
}
```

不幸地是，这段代码在编译器优化，或者多处理器共享内存的情形下并不管用。

### 不起作用

有很多理由证明 **double-checked locking** 不起作用。我们先来介绍几个比较明显的理由，理解了之后，你可能试图想设计一种方式来解决这个问题。但是你并不会成功，因为有更多的微小的理由会出现，当你再一次理解了这些理由之后，并自信地提出了一种新的解决方案，它依然无法正常工作，因为又出现了更多微小的理由。

很多非常聪明的人花费了许多时间来解决这个问题，但是除了让每个线程同步地访问 `helper` 对象，没有其它的方式能解决这个问题。

#### 不起作用理由1

最明显的理由是构造 `Helper` 对象与赋值给 `helper` 字段会出现次序颠倒。因此，一个线程调用 `getHelper()` 查看 `helper` 对象字段的默认值时，可能会看到一个非空(non-null)的对 `helper` 对象的引用，而不是通过构造函数设置的值。

如果编译器内联调用到构造函数，那么在编译器在保证不抛出任何异常或者同步执行的条件下，初始化对象和赋值给 `helper` 字段可以被自由地重新排序。

即使编译器不会重排这些写指令，在一个多处理器环境，当一个线程运行在另一个处理器上时，处理器或内存系统可能会重排这些写操作。

Doug Lea 写了一篇 [more detailed description of compiler-based reorderings](http://gee.cs.oswego.edu/dl/cpj/jmm.html)。

##### 一个测试例子展示不起作用

Paul Jakubik 发现一个使用 `double-checked locking` 不能正确工作的例子 [A slightly cleaned up version of that code is available here](http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckTest.java)。

当运行在一个使用 Symantec JIT 上的系统时，它不起作用。尤其是 Symantec JIT 编译 `singletons[i].reference = new Singleton();` 为以下代码(注意 Symantec JIT 使用 handle-based 对象分配系统)。

```asm
0206106A   mov         eax,0F97E78h
0206106F   call        01F6B210                  ; allocate space for
                                                 ; Singleton, return result in eax
02061074   mov         dword ptr [ebp],eax       ; EBP is &singletons[i].reference 
                                                ; store the unconstructed object here.
02061077   mov         ecx,dword ptr [eax]       ; dereference the handle to
                                                 ; get the raw pointer
02061079   mov         dword ptr [ecx],100h      ; Next 4 lines are
0206107F   mov         dword ptr [ecx+4],200h    ; Singleton's inlined constructor
02061086   mov         dword ptr [ecx+8],400h
0206108D   mov         dword ptr [ecx+0Ch],0F84030h
```

正如你所看到的，赋值给 `sinletons[i].reference` 在调用构造 `Singleton` 之前。在现有的 `Java` 内存模型中是完全合法的，并且在 `C` 和 `C++` 中也是合法的(因为它们没有内存模型)。

##### 一个不起作用的修复

考虑到上面的解释后，一些人建议这样做：

``` java
// (Still) Broken multithreaded version
// "Double-Checked Locking" idiom
class Foo {
	private Helper helper = null;
	public Helper getHelper() {
		if (helper == null) {
			Helper h;
			synchronized(this) {
				h = helper;
				if (h == null)
					synchronized(this) {
						h = new Helper();
					}  // release inner synchronization lock
				helper = h;
			}
		}
		return helper;
	}
	// other functions and members...
}
```
这段代码将构造 `Helper` 对象放到了内部的同步块。直观的思想是同步释放处应该有一个内存屏障，用于阻止 `Helper` 对象的初始化和字段的赋值重排。

不幸的是，这种意图完全错了。同步的规则不会按这种方式执行。Monitorexit(如释放同步)的规则是在 `monitorexit` 之前的操作，在 `monitor` 被释放前一定要被执行。然后没有规则说 monitorexit 之后的操作可能不被完成在 `monitor` 释放之前。编译器移动 `helper = h;` 语句到同步块中是完全有理由且合法的，这又回到了我们之前的情景。许多处理器提供执行这种单方向内存屏障的指令。改变要求释放一个锁的语义为一个完全的内存屏障导致性能受损。

##### 更多不起作用的修复

你可以通过某种方式强制写入器执行一个完全的双向的内存屏障。但这种方式是粗略且低效的，且当 `Java` 内存模型发生更改时无法保证能继续正常工作。不要这样使用，这里可以了解更多，[I've put a description of this technique on a separate page](http://www.cs.umd.edu/~pugh/java/memoryModel/BidirectionalMemoryBarrier.html)。再次强调，不要使用。

然后，即使线程初始化 `helper` 对象通过一个完全的内存屏障，它仍然无法正常工作。

问题是在一些系统上，线程看到一个非空的 `helper` 字段的值仍需要执行内存屏障。

为什么？因为处理器有它们自己对内存的本地缓存副本。在一些处理器上，除非处理器执行一个高速缓存一致性(cache coherence)指令(如一个内存屏障)，否则即使其它处理器使用内存屏障强制写入全局内存，仍然会读到陈旧的本地缓存副本。(注：翻译有待校验)

我创建了[a separate web page](http://www.cs.umd.edu/~pugh/java/memoryModel/AlphaReordering.html)来讨论在 Alpha 处理器上是怎么发生的。

#### 值得这么繁琐吗？

对于大多数程序来说，简单地使 `getHelper()` 方法同步花费并不高。但如果你了解到它导致一个程序产生巨大的开销(substantial overhead)，这时你应该考虑这种细节的优化了。

经常更聪明的是，比如使用内建的合并排序而不是交换排序(查看 SPECJVM DB 基准测试程序)会产生更大的影响。

### 让静态单例起作用

如果你创建的单例是静态的(比如只有唯一的一个 Helper 被创建)，与另一个对象的属性相反(比如一个 Foo 对象持有一个 Helper 对象)。

仅仅只是定义单例作为在独立类中得静态字段。`Java` 语义保证这个字段直到字段被引用时才会被初始化，并且任何线程访问这个字段都可以看到字段初始化的所有写入结果。

``` java
class HelperSingleton {
	static Helper singleton = new Helper();
}
```

### 对32位原始值(primitive values)起作用

尽管 `double-checked locking` 原语不能被用于引用对象，却可以用于32位额原始值(比如int和float)。注意对long和double不起作用，因为64位非同步的读/写基原(primitives)不保证是原子的。

```java
// Correct Double-Checked Locking for 32-bit primitives
class Foo {
	private int cachedHashCode = 0;
	public int hashCode() {
		int h = cachedHashCode;
		if (h == 0)
			synchronized(this) {
				if (cachedHashCode != 0) return cachedHashCode;
				h = computeHashCode();
				cachedHashCode = h;
			}
			return h;
	}
	// other functions and members...
}
```

事实上，假设 `computeHashCode` 函数总是返回相同的结果且没有副作用(i.e., idempotent)，你甚至可以忽略所有的同步。

```java
// Lazy initialization 32-bit primitives
// Thread-safe if computeHashCode is idempotent
class Foo {
	private int cachedHashCode = 0;
	public int hashCode() {
		int h = cachedHashCode;
		if (h == 0) {
			h = computeHashCode();
			cachedHashCode = h;
		}
		return h;
	}
	// other functions and members...
}
```

### 使用内存屏障

如果显示地使用内存屏障指令可能会让 `double checked locking` 起作用。例如，如果你使用 `C++` 编程，你可以使用 Doug Shcmidt et al's 书中的代码：

``` c++
// C++ implementation with explicit memory barriers
// Should work on any platform, including DEC Alphas
// From "Patterns for Concurrent and Distributed Objects",
// by Doug Schmidt
template <class TYPE, class LOCK> TYPE *
Singleton<TYPE, LOCK>::instance (void) {
	// First check
	TYPE* tmp = instance_;
	// Insert the CPU-specific memory barrier instruction
	// to synchronize the cache lines on multi-processor.
	asm("memoryBarrier");
	if (tmp == 0) {
		// Ensure serialization (guard
		// constructor acquires lock_).
		Guard<LOCK> guard (lock_);
		// Double check.
		tmp = instance_;
		if (tmp == 0) {
			tmp = new TYPE;
			// Insert the CPU-specific memory barrier instruction
			// to synchronize the cache lines on multi-processor.
			asm("memoryBarrier");
			instance_ = tmp;
		}
	}
	return tmp;
}
```

### 使用线程本地存储(Thread Local Storage)

Alexander Terekhov (TEREKHOV@de.ibm.com)提出一个聪明的建议，用线程本地存储实现 `double checked locking`。每个线程保持一个决定线程是否完成同步请求的线程本地标识(flag)。

```java
class Foo {
	/** If perThreadInstance.get() returns a non-null value, this thread
	has done synchronization needed to see initialization
	of helper */
	private final ThreadLocal perThreadInstance = new ThreadLocal();
	private Helper helper = null;
	public Helper getHelper() {
		if (perThreadInstance.get() == null) createHelper();
		return helper;
	}
	private final void createHelper() {
		synchronized(this) {
			if (helper == null)
				helper = new Helper();
		}
		// Any non-null value would do as the argument here
		perThreadInstance.set(perThreadInstance);
	}
}
```

这种技术方式的性能有点依赖于你当前的 JDK 实现。在 Sun's 1.2 实现中，线程本地非常慢。在 1.3 中有显著提升。[Doug Lea analyzed the performance of some techniques for implementing lazy initialization](http://www.cs.umd.edu/~pugh/java/memoryModel/DCL-performance.html)

### 新的 Java 内存模型

在 JDK5 中，存在 [a new Java Memory Model and Thread specification](http://www.cs.umd.edu/~pugh/java/memoryModel)

#### 使用 Volatile

JDK5 和之后的版本扩展了 `volatile` 的语义，该语义使系统不被允许重排 volatile 的写操作与其之前的任何读和写操作，并且不被允许重排 volatile 的读操作与其之后的任何读和写操作。查看[this entry in Jeremy Manson's blog](http://jeremymanson.blogspot.com/2008/05/double-checked-locking.html)获取更多细节。

针对这一改变，通过声明 `helper` 字段为 `volatile` 即可实现 `Double-Checked Locking`。但在 JDK4 和早期版本该实现无效。

```java
// Works with acquire/release semantics for volatile
// Broken under current semantics for volatile
class Foo {
	private volatile Helper helper = null;
	public Helper getHelper() {
		if (helper == null) {
			synchronized(this) {
				if (helper == null)
					helper = new Helper();
			}
		}
		return helper;
	}
}
```

#### 不可变对象(Immutable Objects)

如果 `Helper` 是一个不可变对象，所有 `Helper` 的字段都被标记为 `final`，那么不需要使用 `volatile` 也能使 `double-checked locking` 正常工作。原因是引用不可变对象(如 String 或 Integer)的行为应该表现的和 `int` 或 `float` 一致，读和写引用不可变对象是原子操作。

## double-check 原语的描述

- [Reality Check](http://www.cs.wustl.edu/~schmidt/editorial-3.html), Douglas C. Schmidt, C++ Report, SIGS, Vol. 8, No. 3, March 1996.
- [Double-Checked Locking: An Optimization Pattern for Efficiently Initializing and Accessing Thread-safe Objects](http://www.cs.wustl.edu/~schmidt/DC-Locking.ps.gz), Douglas Schmidt and Tim Harrison. 3rd annual Pattern Languages of Program Design conference, 1996
- [Lazy instantiation](http://www.javaworld.com/article/2077568/learn-java/java-tip-67--lazy-instantiation.html), Philip Bishop and Nigel Warren, JavaWorld Magazine
- [Programming Java threads in the real world, Part 7](http://www.javaworld.com/), Allen Holub, Javaworld Magazine, April 1999.
- [Java 2 Performance and Idiom Guide](http://www.phptr.com/ptrbooks/ptr_0130142603.html), Craig Larman and Rhett Guthrie, p100.
- [Java in Practice: Design Styles and Idioms for Effective Java](http://www.google.com/search?q=Java+Design+Styles+Nigel+Bishop), Nigel Warren and Philip Bishop, p142.
- Rule 99, [The Elements of Java Style](http://www.google.com/search?q=elements+java+style+Ambler), Allan Vermeulen, Scott Ambler, Greg Bumgardner, Eldon Metz, Trvor Misfeldt, Jim Shur, Patrick Thompson, SIGS Reference library
- [Global Variables in Java with the Singleton Pattern](http://gamelan.earthweb.com/journal/techfocus/022300_singleton.html), Wiebe de Jong, Gamelan