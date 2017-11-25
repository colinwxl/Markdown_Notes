# python multithreading
***
<2017-1125> by Colin Wu (colinabc@qq.com)
**Focus on `multithreading`**
**Version: python3.6**
refer to [Python多线程和多进程编程总结](https://tracholar.github.io/wiki/python/python-multiprocessing-tutorial.htm)
refer to 《core python application programming》 chapter4

## Threads
Threads:
> Threads (sometimes called lightweight processes) are similar to processes except that they all execute within the same process, and thus all share the same context. They can be thought of as “mini-processes” running in parallel within a main process or “main thread.”
A thread has a beginning, an execution sequence, and a conclusion. It has an instruction pointer that keeps track of where within its context it is currently running. It can be preempted (interrupted) and temporarily put on hold (also known as sleeping) while other threads are running—this is called yielding(让步).

## [threading](https://docs.python.org/3.6/library/threading.html)
This module constructs higher-level threading interfaces on top of the lower level [`_thread`](https://docs.python.org/3.6/library/_thread.html#module-_thread) module. See also the [`queue`](https://docs.python.org/3.6/library/_thread.html#module-_thread) module.
The [`dummy_threading`](https://docs.python.org/3.6/library/dummy_threading.html#module-dummy_threading) module is provided for situations where threading cannot be used because _thread is missing.
This module defines the following functions(`threading.`):
1. active_count(): Return the number of `Thread` objects currently alive.
2. current_thread(): Return the current `Thread` object, corresponding to the caller’s thread of control.
3. get_ident(): Return the ‘thread identifier’ of the current thread.
4. enumerate(): Return a list of all `Thread` objects currently alive.
5. main_thread(): Return the main `Thread` object. In normal conditions, the main thread is the thread from which the Python interpreter was started.
6. settrace(func): Set a trace function for all threads started from the `threading` module. 
7. threading.setprofile(func): Set a profile function for all threads started from the `threading` module.
8. stack_size([size]): Return the thread stack size used when creating new threads.
9. TIMEOUT_MAX: The maximum value allowed for the timeout parameter of blocking functions (`Lock.acquire()`, `RLock.acquire()`, `Condition.wait()`, etc.). Specifying a timeout greater than this value will raise an `OverflowError`.

### Thread-local Data
Thread-local data is data whose values are thread specific. To manage thread-local data, just create an instance of `local` (or a subclass) and store attributes on it:
```
mydata = threading.local()
mydata.x = 1
```
*class* threading.**local**: A class that represents thread-local data. For more details and extensive examples, see the documentation string of the `_threading_local` module.

### Thread Objects
The `Thread` class represents an activity that is run in a separate thread of control. There are two ways to specify the activity: by passing a callable object to the constructor, or by overriding the [`run()`](https://docs.python.org/3.6/library/threading.html#threading.Thread.run) method in a subclass. No other methods (except for the constructor) should be overridden in a subclass. In other words, only override the `__init__()` and `run()` methods of this class.
Once a thread object is created, its activity must be started by calling the thread’s `start()` method. This invokes the `run()` method in a separate thread of control.
Once the thread’s activity is started, the thread is considered ‘alive’. It stops being alive when its `run()` method terminates – either normally, or by raising an unhandled exception. The `is_alive()` method tests whether the thread is alive.
Other threads can call a thread’s `join()` method. This blocks the calling thread until the thread whose `join()` method is called is terminated.
A thread has a name. The name can be passed to the constructor, and read or changed through the `name` attribute.
A thread can be flagged as a “daemon thread”. The significance of this flag is that the entire Python program exits when only daemon threads are left. The initial value is inherited from the creating thread. The flag can be set through the `daemon` property or the **daemon** constructor argument.
> **Note:** Daemon threads are abruptly stopped at shutdown. Their resources (such as open files, database transactions, etc.) may not be released properly. If you want your threads to stop gracefully, make them non-daemonic and use a suitable signalling mechanism such as an `Event`.

#### *class* threading.**Thread**(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)
> group should be `None`; reserved for future extension when a `ThreadGroup` class is implemented.
target is the callable object to be invoked by the `run()` method. Defaults to None, meaning nothing is called.
name is the thread name. By default, a unique name is constructed of the form “Thread-N” where N is a small decimal number.
args is the argument tuple for the target invocation. Defaults to `()`.
kwargs is a dictionary of keyword arguments for the target invocation. Defaults to `{}`.
If not `None`, daemon explicitly sets whether the thread is daemonic. If `None` (the default), the daemonic property is inherited from the current thread.
If the subclass overrides the constructor, it must make sure to invoke the base class constructor (`Thread.__init__()`) before doing anything else to the thread.

Methods:
`start()`; `run()`; `join(timeout=None)`; `name`; `ident`; `is_alive()`;
`daemon`
`join()`: 线程合并： 主线程结束后，子线程还在运行，`join`方法使得主线程等待子线程结束时才退出。

> CPython implementation detail: In CPython, due to the [Global Interpreter Lock](https://docs.python.org/3.6/glossary.html#term-global-interpreter-lock), only one thread can execute Python code at once (even though certain performance-oriented libraries might overcome this limitation). If you want your application to make better use of the computational resources of multi-core machines, you are advised to use `multiprocessing` or `concurrent.futures.ProcessPoolExecutor`. However, threading is still an appropriate model if you want to run multiple I/O-bound tasks simultaneously.

#### Lock Objects
A primitive lock is a synchronization primitive that is not owned by a particular thread when locked. In Python, it is currently the lowest level synchronization primitive available, implemented directly by the `_thread` extension module.
If an attempt is made to release an unlocked lock, a `RuntimeError` will be raised.
Locks also support the [context management protocol](https://docs.python.org/3.6/library/threading.html#with-locks).
##### *class* threading.**Lock**
1. acquire(blocking=True, timeout=-1)
2. release()
3. `with lock:`: 当程序执行到with语句时，acquire()方法将被调用，当程序执行完with语句时，release()方法会被调用.

#### RLock Objects
A reentrant lock is a synchronization primitive that may be acquired multiple times by the same thread. Internally, it uses the concepts of “owning thread” and “recursion level” in addition to the locked/unlocked state used by primitive locks. In the locked state, some thread owns the lock; in the unlocked state, no thread owns it.
Reentrant locks also support the [context management protocol](https://docs.python.org/3.6/library/threading.html#with-locks).
##### *class* threading.**RLock**
1. acquire(blocking=True, timeout=-1)
2. release()

#### Difference between Lock and RLock
refer to https://stackoverflow.com/questions/22885775/what-is-the-difference-between-lock-and-rlock
> The main difference is that a Lock can only be acquired once. It cannot be acquired again, until it is released. (After it's been released, it can be re-acaquired by any thread).
An RLock on the other hand, can be acquired multiple times, by the same thread. It needs to be released the same number of times in order to be "unlocked".
Another difference is that an acquired Lock can be released by any thread, while an acquired RLock can only be released by the thread which acquired it.

#### Condition Objects
A condition variable is always associated with some kind of lock; this can be passed in or one will be created by default. Passing one in is useful when several condition variables must share the same lock. The lock is part of the condition object: you don’t have to track it separately.
##### *class* threading.**Condition**(lock=None)
1. acquire(*args)
2. release()
3. wait(timeout=None)
4. wait_for(predicate, timeout=None)
5. notify(n=1): By default, wake up one thread waiting on this condition, if any. 
6. notify_all()
```
queue = []

con = threading.Condition()

class Producer(threading.Thread):
    def run(self):
        while True:
            if con.acquire():
                if len(queue) > 100:
                    con.wait()
                else:
                    elem = random.randrange(100)
                    queue.append(elem)
                    print "Producer a elem {}, Now size is {}".format(elem, len(queue))
                    time.sleep(random.random())
                    con.notify()
                con.release()

class Consumer(threading.Thread):
    def run(self):
        while True:
            if con.acquire():
                if len(queue) < 0:
                    con.wait()
                else:
                    elem = queue.pop()
                    print "Consumer a elem {}. Now size is {}".format(elem, len(queue))
                    time.sleep(random.random())
                    con.notify()
                con.release()

def main():
    for i in range(3):
        Producer().start()

    for i in range(2):
        Consumer().start()
```

#### Semaphore Objects
A semaphore manages an internal counter which is decremented by each `acquire()` call and incremented by each `release()` call. The counter can never go below zero; when `acquire()` finds that it is zero, it blocks, waiting until some other thread calls `release()`.
##### *class* threading.**Semaphore**(value=1)
The optional argument gives the initial value for the internal counter; it defaults to 1
1. acquire(blocking=True, timeout=None)
2. release()

##### *class* threading.**BoundedSemaphore**(value=1)
A bounded semaphore checks to make sure its current value doesn’t exceed its initial value. If it does, `ValueError` is raised. 

##### [Semaphore](https://docs.python.org/3.6/library/threading.html#threading.Semaphore) Example
Semaphores are often used to guard resources with limited capacity, for example, a database server. In any situation where the size of the resource is fixed, you should use a bounded semaphore. Before spawning any worker threads, your main thread would initialize the semaphore:
```
maxconnections = 5
# ...
pool_sema = BoundedSemaphore(value=maxconnections)
```
Once spawned, worker threads call the semaphore’s acquire and release methods when they need to connect to the server:
```
with pool_sema:
    conn = connectdb()
    try:
        # ... use connection ...
    finally:
        conn.close()
```
The use of a bounded semaphore reduces the chance that a programming error which causes the semaphore to be released more than it’s acquired will go undetected.

#### Event Objects
*class* threading.**Event**
An event manages a flag that can be set to true with the `set()` method and reset to false with the `clear()` method. The `wait()` method blocks until the flag is true. The flag is initially false.
`is_set()`; `set()`; `clear()`; `wait(timeout=None)`

#### Timer Objects
This class represents an action that should be run only after a certain amount of time has passed — a timer. [`Timer`](https://docs.python.org/3.6/library/threading.html#threading.Timer) is a subclass of `Thread` and as such also functions as an example of creating custom threads.
*class* threading.**Timer**(interval, function, args=None, kwargs=None)
Create a timer that will run function with arguments args and keyword arguments kwargs, after interval seconds have passed. If args is `None` (the default) then an empty list will be used. If kwargs is None (the default) then an empty dict will be used.
`cancel()`
```
def hello():
    print("hello, world")

t = Timer(30.0, hello)
t.start()  # after 30 seconds, "hello, world" will be printed
```

#### Barrier Objects
*class* threading.**Barrier**(parties, action=None, timeout=None)
Create a barrier object for parties number of threads. An action, when provided, is a callable to be called by one of the threads when they are released. timeout is the default timeout value if none is specified for the `wait()` method.
`wait(timeout=None)`; `reset()`; `abort()`; `parties()`; `n_waiting`; `broken`

#### Using locks, conditions, and semaphores in the [with](https://docs.python.org/3.6/reference/compound_stmts.html#with) statement
All of the objects provided by this module that have acquire() and release() methods can be used as context managers for a with statement. The acquire() method will be called when the block is entered, and release() will be called when the block is exited. Hence, the following snippet:
```
with some_lock:
    # do something...
```
is equivalent to:
```
some_lock.acquire()
try:
    # do something...
finally:
    some_lock.release()
```