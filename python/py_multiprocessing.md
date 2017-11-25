# python multiprocessing
***
<2017-1124> by Colin Wu (colinabc@qq.com)

**Focus** on `multiprocessing`

**Version: python3.6**

refer to [Python多线程和多进程编程总结](https://tracholar.github.io/wiki/python/python-multiprocessing-tutorial.htm)

## GIL(Global Interpreter Lock)
> In CPython, the global interpreter lock, or GIL, is a mutex(互斥元) that prevents multiple native threads from executing Python bytecodes at once. This lock is necessary mainly because CPython’s memory management is not thread-safe. (However, since the GIL exists, other features have grown to depend on the guarantees that it enforces.)
> The mechanism used by the CPython interpreter to assure that only one thread executes Python bytecode at a time. This simplifies the CPython implementation by making the object model (including critical built-in types such as dict) implicitly safe against concurrent access. Locking the entire interpreter makes it easier for the interpreter to be multi-threaded, at the expense of much of the parallelism afforded by multi-processor machines.

## processes and threads
Processes:
> A process (sometimes called a heavyweight process) is a program in execution. Each process has its own address space, memory, a data stack, and other auxiliary data to keep track of execution. The operating system manages the execution of all processes on the system, dividing the time fairly between all processes. Processes can also *fork* or *spawn* new processes to perform other tasks, but each new process has its own memory, data stack, etc., and cannot generally share information unless *interprocess communication* (IPC) is employed.

## [multiprocessing](https://docs.python.org/3.6/library/multiprocessing.html#using-a-pool-of-workers)
### Instroduction
> multiprocessing is a package that supports spawning processes using an API similar to the threading module. The multiprocessing package offers both local and remote concurrency, effectively side-stepping the Global Interpreter Lock by using subprocesses instead of threads. Due to this, the multiprocessing module allows the programmer to fully leverage multiple processors on a given machine. It runs on both Unix and Windows.

#### The `Process` class
In `multiprocessing`, processes are spawned by creating a `Process` object and then calling its `start()` method.
```
from multiprocessing import Process
import os

def info(title):
    print(title)
    print('module name:', __name__)
    print('parent process:', os.getppid())
    print('process id:', os.getpid())

def f(name):
    info('function f')
    print('hello', name)

if __name__ == '__main__':
    info('main line')
    p = Process(target=f, args=('bob',))
    p.start()
    p.join()
```
#### Contexts and start methods
1. spawn
> The parent process starts a fresh python interpreter process. The child process will only inherit those resources necessary to run the process objects `run()` method. In particular, unnecessary file descriptors and handles from the parent process will not be inherited. Starting a process using this method is rather slow compared to using fork or forkserver.
Available on Unix and Windows. The default on Windows.
2. fork
> The parent process uses `os.fork()` to fork the Python interpreter. The child process, when it begins, is effectively identical to the parent process. All resources of the parent are inherited by the child process. Note that safely forking a multithreaded process is problematic.
Available on Unix only. The default on Unix.
3. forkserver
> When the program starts and selects the forkserver start method, a server process is started. From then on, whenever a new process is needed, the parent process connects to the server and requests that it fork a new process. The fork server process is single threaded so it is safe for it to use `os.fork()`. No unnecessary resources are inherited.
Available on Unix platforms which support passing file descriptors over Unix pipes.

To select a start method you use the [`set_start_method()`](https://docs.python.org/3.6/library/multiprocessing.html#multiprocessing.set_start_method) in the `if __name__ == '__main__'` clause of the main module. `set_start_method()` should not be used more than once in the program. For example:
```
import multiprocessing as mp

def foo(q):
    q.put('hello')

if __name__ == '__main__':
    mp.set_start_method('spawn')
    q = mp.Queue()
    p = mp.Process(target=foo, args=(q,))
    p.start()
    print(q.get())
    p.join()
```
Alternatively, you can use [`get_context()`](https://docs.python.org/3.6/library/multiprocessing.html#multiprocessing.get_context) to obtain a context object. Context objects have the same API as the multiprocessing module, and allow one to use multiple start methods in the same program.
```
import multiprocessing as mp

def foo(q):
    q.put('hello')

if __name__ == '__main__':
    ctx = mp.get_context('spawn')
    q = ctx.Queue()
    p = ctx.Process(target=foo, args=(q,))
    p.start()
    print(q.get())
    p.join()
```

#### Exchanging objectS between processes
multiprocessing supports two types of communication channel between processes:

1. **Queues**
The `Queue` class is a near clone of queue.Queue. Queues are thread and process safe. For example:
```
from multiprocessing import Process, Queue
def f(q):
    q.put([42, None, 'hello'])
if __name__ == '__main__':
    q = Queue()
    p = Process(target=f, args=(q,))
    p.start()
    print(q.get())    # prints "[42, None, 'hello']"
    p.join()
```

2. **Pipes**
The `Pipe()` function returns a pair of connection objects connected by a pipe which by default is duplex (two-way). For example:
```
from multiprocessing import Process, Pipe
def f(conn):
    conn.send([42, None, 'hello'])
    conn.close()
if __name__ == '__main__':
    parent_conn, child_conn = Pipe()
    p = Process(target=f, args=(child_conn,))
    p.start()
    print(parent_conn.recv())   # prints "[42, None, 'hello']"
    p.join()
```
The two connection objects returned by `Pipe()` represent the two ends of the pipe. Each connection object has `send()` and `recv()` methods (among others). Note that data in a pipe may become corrupted if two processes (or threads) try to read from or write to the same end of the pipe at the same time. Of course there is no risk of corruption from processes using different ends of the pipe at the same time.

#### Synchronization between processes
multiprocessing contains equivalents of all the synchronization primitives from threading. For instance one can use a **loc**k to ensure that only one process prints to standard output at a time:
```
from multiprocessing import Process, Lock

def f(l, i):
    l.acquire()
    try:
        print('hello world', i)
    finally:
        l.release()

if __name__ == '__main__':
    lock = Lock()

    for num in range(10):
        Process(target=f, args=(lock, num)).start()
```
#### Sharing state between processes
It is usually best to avoid using shared state as far as possible. 
1. Shared memory: `Value` or `Array`
2. Sever process: A manager returned by `Manager()` will support types `list`, `dict`, `Namespace`, `Lock`, `RLock`, `Semaphore`, `BoundedSemaphore`, `Condition`, `Event`, `Barrier`, `Queue`, `Value` and `Array`. 

#### Using a pool of worker
The [`Pool`](The Pool class represents a pool of worker processes. It has methods which allows tasks to be offloaded to the worker processes in a few different ways.) class represents a pool of worker processes. It has methods which allows tasks to be offloaded to the worker processes in a few different ways.
For example:
```
from multiprocessing import Pool, TimeoutError
import time
import os

def f(x):
    return x*x

if __name__ == '__main__':
    # start 4 worker processes
    with Pool(processes=4) as pool:

        # print "[0, 1, 4,..., 81]"
        print(pool.map(f, range(10)))

        # print same numbers in arbitrary order
        for i in pool.imap_unordered(f, range(10)):
            print(i)

        # evaluate "f(20)" asynchronously
        res = pool.apply_async(f, (20,))      # runs in *only* one process
        print(res.get(timeout=1))             # prints "400"

        # evaluate "os.getpid()" asynchronously
        res = pool.apply_async(os.getpid, ()) # runs in *only* one process
        print(res.get(timeout=1))             # prints the PID of that process

        # launching multiple evaluations asynchronously *may* use more processes
        multiple_results = [pool.apply_async(os.getpid, ()) for i in range(4)]
        print([res.get(timeout=1) for res in multiple_results])

        # make a single worker sleep for 10 secs
        res = pool.apply_async(time.sleep, (10,))
        try:
            print(res.get(timeout=1))
        except TimeoutError:
            print("We lacked patience and got a multiprocessing.TimeoutError")

        print("For the moment, the pool remains available for more work")

    # exiting the 'with'-block has stopped the pool
    print("Now the pool is closed and no longer available")
```

### Reference
#### Process and exception
*class* `multiprocessing.`**Process**(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)

> The constructor should always be called with keyword arguments. group should always be `None`; it exists solely for compatibility with `threading.Thread`. target is the callable object to be invoked by the `run()` method. It defaults to `None`, meaning nothing is called. name is the process name (see [name](https://docs.python.org/3.6/library/multiprocessing.html#multiprocessing.Process.name) for more details). args is the argument tuple for the target invocation. kwargs is a dictionary of keyword arguments for the target invocation. If provided, the keyword-only daemon argument sets the process [daemon](https://docs.python.org/3.6/library/multiprocessing.html#multiprocessing.Process.daemon) flag to `True` or `False`. If None (the default), this flag will be inherited from the creating process.

> By default, no arguments are passed to target.

> If a subclass overrides the constructor, it must make sure it invokes the base class constructor (`Process.__init__()`) before doing anything else to the process.

Methods: `run()`; `start()`; `join([*timeout*])`; `name`; `is_alive()`; `daemon` 'When a process exits, it attempts to terminate all of its daemonic child processes.'; `pid`; `exitcode`; `authkey`; `sentinel`; `termianate()`.
> Note that the start(), join(), is_alive(), terminate() and exitcode methods should only be called by the process that created the process object.

Exceptions: `ProcessError`; `BufferTooShort` (when the supplied buffer object is too small for the message read.); `AuthenticationError`; `TimeoutError` (Raised by methods with a timeout when the timeout expires.).

Usage: Subclass Process and Create Subclass Instance
```
import multiprocessing
import time

class ClockProcess(multiprocessing.Process):
    def __init__(self, interval):
        multiprocessing.Process.__init__(self)
        self.interval = interval

    def run(self):
        n = 5
        while n > 0:
            print("the time is {0}".format(time.ctime()))
            time.sleep(self.interval)
            n -= 1

if __name__ == '__main__':
    p = ClockProcess(3)
    p.start()
```
> [Attention] A daemon child process will be terminated when the script(the main process) reaches its end.

#### Pipes and Queues
For passing messages one can use `Pipe()` (for a connection between two processes) or a queue (which allows multiple producers and consumers).
> **Note:** `multiprocessing` uses the usual `queue.Empty` and `queue.Full` exceptions to signal a timeout. They are not available in the `multiprocessing` namespace so you need to import them from `queue`.

1. multiprocessing.**Pipe**([*duplex*]): Returns a pair (`conn1`, `conn2`) of [Connection](https://docs.python.org/3.6/library/multiprocessing.html#multiprocessing.Connection) objects representing the ends of a pipe.
If duplex is True (the default) then the pipe is bidirectional. If duplex is False then the pipe is unidirectional: conn1 can only be used for receiving messages and conn2 can only be used for sending messages.
`send()`; `recv()`
send和recv方法分别是发送和接受消息的方法。例如，在全双工模式下，可以调用conn1.send发送消息，conn1.recv接收消息。如果没有消息可接收，recv方法会一直阻塞。如果管道已经被关闭，那么recv方法会抛出EOFError。

2. *class* multiprocessing.**Queue**([*maxsize*]): Returns a process shared queue implemented using a pipe and a few locks/semaphores. When a process first puts an item on the queue a feeder thread is started which transfers objects from a buffer into the pipe.
Queue implements all the methods of queue.Queue except for task_done() and join().
Methods: `qsize()`; `empty()`; `full()`; `put_nowait(obj)` 'Equivalent to put(obj, False)'; `get_nowait()` 'Equivalent to get(False)'; `close()`; `join_thread()`; `cancel_join_thread()`
1> `put(obj[, block[, timeout]])`
Put obj into the queue. If the optional argument block is True (the default) and timeout is None (the default), block if necessary until a free slot is available. If timeout is a positive number, it blocks at most timeout seconds and raises the queue.Full exception if no free slot was available within that time. Otherwise (block is False), put an item on the queue if a free slot is immediately available, else raise the queue.Full exception (timeout is ignored in that case).
2> `get([block[, timeout]])`
Remove and return an item from the queue. If optional args block is True (the default) and timeout is None (the default), block if necessary until an item is available. If timeout is a positive number, it blocks at most timeout seconds and raises the queue.Empty exception if no item was available within that time. Otherwise (block is False), return an item if one is immediately available, else raise the queue.Empty exception (timeout is ignored in that case).

3. *class* multiprocessing.**SimpleQueue**
`empty`; `get()`; `put(item)`

4. *class* multiprocessing.**JoinableQueue**([*maxsize*])
`task_done()`; `join()`

Examples:
```
import multiprocessing

def writer_proc(q):      
    try:         
        q.put(1, block = False) 
    except:         
        pass   

def reader_proc(q):      
    try:         
        print q.get(block = False) 
    except:         
        pass

if __name__ == "__main__":
    q = multiprocessing.Queue()
    writer = multiprocessing.Process(target=writer_proc, args=(q,))  
    writer.start()   

    reader = multiprocessing.Process(target=reader_proc, args=(q,))  
    reader.start()  

    reader.join()  
    writer.join()
```
```
import multiprocessing
import time

def proc1(pipe):
    while True:
        for i in xrange(10000):
            print "send: %s" %(i)
            pipe.send(i)
            time.sleep(1)

def proc2(pipe):
    while True:
        print "proc2 rev:", pipe.recv()
        time.sleep(1)

def proc3(pipe):
    while True:
        print "PROC3 rev:", pipe.recv()
        time.sleep(1)

if __name__ == "__main__":
    pipe = multiprocessing.Pipe()
    p1 = multiprocessing.Process(target=proc1, args=(pipe[0],))
    p2 = multiprocessing.Process(target=proc2, args=(pipe[1],))
    #p3 = multiprocessing.Process(target=proc3, args=(pipe[1],))

    p1.start()
    p2.start()
    #p3.start()

    p1.join()
    p2.join()
    #p3.join()
```

#### Miscellaneous
1. multiprocessing.**active_children**(): Return list of all live children of the current process.
2. multiprocessing.**cpu_count**(): Return the number of CPUs in the system.
3. multiprocessing.**current_process**(): Return the Process object corresponding to the current process.
4. multiprocessing.**freeze_support**(): Add support for when a program which uses multiprocessing has been frozen to produce a Windows executable. 
5. multiprocessing.**get_all_start_methods**(): Returns a list of the supported start methods, the first of which is the default. 
6. multiprocessing.**get_context**(method=None): Return a context object which has the same attributes as the multiprocessing module.
7. multiprocessing.**get_start_method**(allow_none=False): Return the name of start method used for starting processes.
8. multiprocessing.**set_executable**(): Sets the path of the Python interpreter to use when starting a child process.
9. multiprocessing.**set_start_method**(method): Set the method which should be used to start child processes.

#### Connection Objects
*class* multiprocessing.**Connection**
`send()`; `recv`; `fileno()`; `close()`; `poll([timeout])`; `send_bytes(buffer[, offset[, size]])`; `recv_bytes([maxlength])`; `recv_bytes_into(buffer[, offset])`

#### Synchronization primitives(同步原语)
1. *class* multiprocessing.**Barrier**(parties[, action[, timeout]])
2. *class* multiprocessing.**BoundedSemaphore**([value])
3. *class* multiprocessing.**Condition**([lock])
4. *class* multiprocessing.**Event**
```
import multiprocessing
import time
def wait_for_event(e):
    print("wait_for_event: starting")
    e.wait()
    print("wairt_for_event: e.is_set()->" + str(e.is_set()))
def wait_for_event_timeout(e, t):
    print("wait_for_event_timeout:starting")
    e.wait(t)
    print("wait_for_event_timeout:e.is_set->" + str(e.is_set()))
if __name__ == "__main__":
    e = multiprocessing.Event()
    w1 = multiprocessing.Process(name = "block",
            target = wait_for_event,
            args = (e,))
    w2 = multiprocessing.Process(name = "non-block",
            target = wait_for_event_timeout,
            args = (e, 2))
    w1.start()
    w2.start()
    time.sleep(3)
    e.set()
    print("main: event is set")
```
Results:
```
wait_for_event: starting
wait_for_event_timeout:starting
wait_for_event_timeout:e.is_set->False
main: event is set
wairt_for_event: e.is_set()->True
```
5. *class* multiprocessing.**Lock**
`acquire(block=True, timeout=None)`; `release()`
6. *class* multiprocessing.**RLock**: A recursive lock object.
`acquire(block=True, timeout=None)`; `release()`
7. *class* multiprocessing.**Semaphore**([value]): 信号量 (or **BoundedSemaphore**())
Semaphores are some of the oldest synchronization primitives out there. They’re basically counters that decrement when a resource is being consumed (and increment again when the resource is released).
`acquire()`; `release()`
#### Shared [ctypes](https://docs.python.org/3.6/library/ctypes.html#module-ctypes) Objects
1. multiprocessing.**Value**(typecode_or_type, *args, lock=True)
2. multiprocessing.**Array**(typecode_or_type, size_or_initializer, *, lock=True)
##### The [multiprocessing.sharedctypes](https://docs.python.org/3.6/library/multiprocessing.html#module-multiprocessing.sharedctypes) module
 1. multiprocessing.sharedctypes.**RawArray**(typecode_or_type, size_or_initializer)
 2. multiprocessing.sharedctypes.**RawValue**(typecode_or_type, *args)
 3. multiprocessing.sharedctypes.**Array**(typecode_or_type, size_or_initializer, *, lock=True)
 4. multiprocessing.sharedctypes.**Value**(typecode_or_type, *args, lock=True)
 5. multiprocessing.sharedctypes.**copy**(obj)
 6. multiprocessing.sharedctypes.**synchronized**(obj[, lock])

#### Managers
1. multiprocessing.**Manager**(): Returns a started [SyncManager](https://docs.python.org/3.6/library/multiprocessing.html#multiprocessing.managers.SyncManager) object which can be used for sharing objects between processes. 
2. class multiprocessing.managers.**BaseManager**([address[, authkey]])
`start([initializer[, initargs]])`; `get_server()`; `connect()`; `shutdown()`; `register(typeid[, callable[, proxytype[, exposed[, method_to_typeid[, create_method]]]]])`; `address`
3. class multiprocessing.managers.**SyncManager**: A subclass of BaseManager which can be used for the synchronization of processes.
`Barrier(parties[, action[, timeout]])`; `BoundedSemaphore([value])`; `Condition([lock])`; `Event()`; `Lock()`; `Namespace()`; `Queue([maxsize])`; `RLock()`; `Semaphore([value])`; `Array(typecode, sequence)`; `Value(typecode, value)`; `dict()`; `dict(mapping)`; `dict(sequence)`; `lsit()`; `list(sequece)`
4. *class* multiprocessing.managers.**Namespace**: A type that can register with SyncManager.

##### Customized managers
o create one’s own manager, one creates a subclass of [`BaseManager`](https://docs.python.org/3.6/library/multiprocessing.html#multiprocessing.managers.BaseManager) and uses the [`register()`](https://docs.python.org/3.6/library/multiprocessing.html#multiprocessing.managers.BaseManager.register) classmethod to register new types or callables with the manager class. For example:
```
from multiprocessing.managers import BaseManager

class MathsClass:
    def add(self, x, y):
        return x + y
    def mul(self, x, y):
        return x * y

class MyManager(BaseManager):
    pass

MyManager.register('Maths', MathsClass)

if __name__ == '__main__':
    with MyManager() as manager:
        maths = manager.Maths()
        print(maths.add(4, 3))         # prints 7
        print(maths.mul(7, 8))         # prints 56
```
##### Using A remote manager
It is possible to run a manager server on one machine and have clients use it from other machines (assuming that the firewalls involved allow it).
Running the following commands creates a server for a single shared queue which remote clients can access:
```
>>> from multiprocessing.managers import BaseManager
>>> import queue
>>> queue = queue.Queue()
>>> class QueueManager(BaseManager): pass
>>> QueueManager.register('get_queue', callable=lambda:queue)
>>> m = QueueManager(address=('', 50000), authkey=b'abracadabra')
>>> s = m.get_server()
>>> s.serve_forever()
```
One client can access the server as follows:
```
>>> from multiprocessing.managers import BaseManager
>>> class QueueManager(BaseManager): pass
>>> QueueManager.register('get_queue')
>>> m = QueueManager(address=('foo.bar.org', 50000), authkey=b'abracadabra')
>>> m.connect()
>>> queue = m.get_queue()
>>> queue.put('hello')
```
Another client can also use it:
```
>>> from multiprocessing.managers import BaseManager
>>> class QueueManager(BaseManager): pass
>>> QueueManager.register('get_queue')
>>> m = QueueManager(address=('foo.bar.org', 50000), authkey=b'abracadabra')
>>> m.connect()
>>> queue = m.get_queue()
>>> queue.get()
'hello'
```
Local processes can also access that queue, using the code from above on the client to access it remotely:
```
>>> from multiprocessing import Process, Queue
>>> from multiprocessing.managers import BaseManager
>>> class Worker(Process):
...     def __init__(self, q):
...         self.q = q
...         super(Worker, self).__init__()
...     def run(self):
...         self.q.put('local hello')
...
>>> queue = Queue()
>>> w = Worker(queue)
>>> w.start()
>>> class QueueManager(BaseManager): pass
...
>>> QueueManager.register('get_queue', callable=lambda: queue)
>>> m = QueueManager(address=('', 50000), authkey=b'abracadabra')
>>> s = m.get_server()
>>> s.serve_forever()
```

#### Proxy Objects
A proxy is an object which refers to a shared object which lives (presumably) in a different process. The shared object is said to be the referent of the proxy. Multiple proxy objects may have the same referent.
A proxy object has methods which invoke corresponding methods of its referent (although not every method of the referent will necessarily be available through the proxy). In this way, a proxy can be used just like its referent can:
```
>>> from multiprocessing import Manager
>>> manager = Manager()
>>> l = manager.list([i*i for i in range(10)])
>>> print(l)
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
>>> print(repr(l))
<ListProxy object, typeid 'list' at 0x...>
>>> l[4]
16
>>> l[2:5]
[4, 9, 16]
```
Notice that applying `str()` to a proxy will return the representation of the referent, whereas applying [`repr()`](Notice that applying str() to a proxy will return the representation of the referent, whereas applying repr() will return the representation of the proxy.) will return the representation of the proxy.

*class* multiprocessing.managers.**BaseProxy**
  `_callmethod(methodname[, args[, kwds]])`
  `_getvalue()`
  `__repr__()`
  `__str__()`
A proxy object uses a weakref callback so that when it gets garbage collected it deregisters itself from the manager which owns its referent.
A shared object gets deleted from the manager process when there are no longer any proxies referring to it.

#### Process Pools
在利用Python进行系统管理的时候，特别是同时操作多个文件目录，或者远程控制多台主机，并行操作可以节约大量的时间。当被操作对象数目不大时，可以直接利用multiprocessing中的Process动态成生多个进程，十几个还好，但如果是上百个，上千个目标，手动的去限制进程数量却又太过繁琐，此时可以发挥进程池的功效。
Pool可以提供指定数量的进程，供用户调用，当有新的请求提交到pool中时，如果池还没有满，那么就会创建一个新的进程用来执行该请求；但如果池中的进程数已经达到规定最大值，那么该请求就会等待，直到池中有进程结束，才会创建新的进程来它。

##### *class* multiprocessing.**Pool**([processes[, initializer[, initargs[, maxtasksperchild[, context]]]]])
A process pool object which controls a pool of worker processes to which jobs can be submitted. It supports asynchronous results with timeouts and callbacks and has a parallel map implementation.
processes is the number of worker processes to use. If processes is `None` then the number returned by `os.cpu_count()` is used.
If initializer is not `None` then each worker process will call `initializer(*initargs)` when it starts.
maxtasksperchild is the number of tasks a worker process can complete before it will exit and be replaced with a fresh worker process, to enable unused resources to be freed. The default maxtasksperchild is `None`, which means worker processes will live as long as the pool.
context can be used to specify the context used for starting the worker processes. Usually a pool is created using the function `multiprocessing.Pool()` or the [`Pool()`](https://docs.python.org/3.6/library/multiprocessing.html#multiprocessing.pool.Pool) method of a context object. In both cases context is set appropriately.
1. `apply(func[, args[, kwds]])` 阻塞
Call func with arguments args and keyword arguments kwds. It blocks until the result is ready. Given this blocks, apply_async() is better suited for performing work in parallel. Additionally, func is only executed in one of the workers of the pool.
2. `apply_async(func[, args[, kwds[, callback[, error_callback]]]])` 非阻塞，主函数会继续执行，不等待子进程的结果。
3. `map(func, iterable[, chunksize])`
This method chops the iterable into a number of chunks which it submits to the process pool as separate tasks. The (approximate) size of these chunks can be specified by setting chunksize to a positive integer.
4. `map_async(func, iterable[, chunksize[, callback[, error_callback]]])`
5. `imap(func, iterable[, chunksize])`
6. `imap_unordered(func, iterable[, chunksize])`
7. `starmap(func, iterable[, chunksize])`
8. `starmap_async(func, iterable[, chunksize[, callback[, error_callback]]])`
9. `close()`: Prevents any more tasks from being submitted to the pool. Once all the tasks have been completed the worker processes will exit.
10. `terminate()`
11. `join()`: Wait for the worker processes to exit. One must call `close()` or `terminate()` before using `join()`.

##### *class* multiprocessing.pool.**AsyncResult**
The class of the result returned by `Pool.apply_async()` and `Pool.map_async()`.
1. `get([timeout])`
2. `wait([timeout])`
3. `ready()`
4. `successful()`

```
from multiprocessing import Pool
import time

def f(x):
    return x*x

if __name__ == '__main__':
    with Pool(processes=4) as pool:         # start 4 worker processes
        result = pool.apply_async(f, (10,)) # evaluate "f(10)" asynchronously in a single process
        print(result.get(timeout=1))        # prints "100" unless your computer is *very* slow

        print(pool.map(f, range(10)))       # prints "[0, 1, 4,..., 81]"

        it = pool.imap(f, range(10))
        print(next(it))                     # prints "0"
        print(next(it))                     # prints "1"
        print(it.next(timeout=1))           # prints "4" unless your computer is *very* slow

        result = pool.apply_async(time.sleep, (10,))
        print(result.get(timeout=1))        # raises multiprocessing.TimeoutError
```

#### Listeners and Clients
1. multiprocessing.connection.**deliver_challenge**(connection, authkey)
2. multiprocessing.connection.**answer_challenge**(connection, authkey)
3. multiprocessing.connection.**Client**(address[, family[, authkey]])
4. class multiprocessing.connection.**Listener**([address[, family[, backlog[, authkey]]]])
5. multiprocessing.connection.**wait**(object_list, timeout=None)

#### Authentication keys
An authentication key is a byte string which can be thought of as a password: once a connection is established both ends will demand proof that the other knows the authentication key. (Demonstrating that both ends are using the same key does not involve sending the key over the connection.)

#### Logging
1. multiprocessing.**get_logger**()
2. multiprocessing.**log_to_stderr**()

#### The [multiprocessing.dummy](https://docs.python.org/3.6/library/multiprocessing.html#module-multiprocessing.dummy)

### Programming guidelines

#### All start methods
1. Avoid shared state
2. Picklability
3. Thread safety of proxies
4. Joining zombie processes
5. Better to inherit than pickle/unpickle
6. Avoid terminating processes
7. Joining processes that use queues
8. Explicitly pass resources to child processes
9. Beware of replacing `sys.stdin` with a “file like object”

#### The spawn and forkserver start methods
1. More picklability
2. Global variables
3. Safe importing of main module

### Examples
#### Demonstration of how to create and use customized managers and proxies:

```
from multiprocessing import freeze_support
from multiprocessing.managers import BaseManager, BaseProxy
import operator

class Foo:
    def f(self):
        print('you called Foo.f()')
    def g(self):
        print('you called Foo.g()')
    def _h(self):
        print('you called Foo._h()')

# A simple generator function
def baz():
    for i in range(10):
        yield i*i

# Proxy type for generator objects
class GeneratorProxy(BaseProxy):
    _exposed_ = ['__next__']
    def __iter__(self):
        return self
    def __next__(self):
        return self._callmethod('__next__')

# Function to return the operator module
def get_operator_module():
    return operator

class MyManager(BaseManager):
    pass

# register the Foo class; make `f()` and `g()` accessible via proxy
MyManager.register('Foo1', Foo)
# register the Foo class; make `g()` and `_h()` accessible via proxy
MyManager.register('Foo2', Foo, exposed=('g', '_h'))
# register the generator function baz; use `GeneratorProxy` to make proxies
MyManager.register('baz', baz, proxytype=GeneratorProxy)
# register get_operator_module(); make public functions accessible via proxy

MyManager.register('operator', get_operator_module)

def test():
    manager = MyManager()
    manager.start()
    print('-' * 20)
    f1 = manager.Foo1()
    f1.f()
    f1.g()
    assert not hasattr(f1, '_h')
    assert sorted(f1._exposed_) == sorted(['f', 'g'])
    print('-' * 20)
    f2 = manager.Foo2()
    f2.g()
    f2._h()
    assert not hasattr(f2, 'f')
    assert sorted(f2._exposed_) == sorted(['g', '_h'])
    print('-' * 20)
    it = manager.baz()
    for i in it:
        print('<%d>' % i, end=' ')
    print()
    print('-' * 20)
    op = manager.operator()
    print('op.add(23, 45) =', op.add(23, 45))
    print('op.pow(2, 94) =', op.pow(2, 94))
    print('op._exposed_ =', op._exposed_)

if __name__ == '__main__':
    freeze_support()
    test()
```

#### Using Pool:

```
import multiprocessing
import time
import random
import sys

## Functions used by test code

def calculate(func, args):
    result = func(*args)
    return '%s says that %s%s = %s' % (
        multiprocessing.current_process().name,
        func.__name__, args, result
        )

def calculatestar(args):
    return calculate(*args)

def mul(a, b):
    time.sleep(0.5 * random.random())
    return a * b

def plus(a, b):
    time.sleep(0.5 * random.random())
    return a + b

def f(x):
    return 1.0 / (x - 5.0)

def pow3(x):
    return x ** 3

def noop(x):
    pass

## Test code

def test():
    PROCESSES = 4
    print('Creating pool with %d processes\n' % PROCESSES)

    with multiprocessing.Pool(PROCESSES) as pool:

        ## Tests

        TASKS = [(mul, (i, 7)) for i in range(10)] + \
                [(plus, (i, 8)) for i in range(10)]

        results = [pool.apply_async(calculate, t) for t in TASKS]
        imap_it = pool.imap(calculatestar, TASKS)
        imap_unordered_it = pool.imap_unordered(calculatestar, TASKS)

        print('Ordered results using pool.apply_async():')
        for r in results:
            print('\t', r.get())
        print()

        print('Ordered results using pool.imap():')
        for x in imap_it:
            print('\t', x)
        print()

        print('Unordered results using pool.imap_unordered():')
        for x in imap_unordered_it:
            print('\t', x)
        print()

        print('Ordered results using pool.map() --- will block till complete:')
        for x in pool.map(calculatestar, TASKS):
            print('\t', x)
        print()

        ## Test error handling

        print('Testing error handling:')

        try:
            print(pool.apply(f, (5,)))
        except ZeroDivisionError:
            print('\tGot ZeroDivisionError as expected from pool.apply()')
        else:
            raise AssertionError('expected ZeroDivisionError')

        try:
            print(pool.map(f, list(range(10))))
        except ZeroDivisionError:
            print('\tGot ZeroDivisionError as expected from pool.map()')
        else:
            raise AssertionError('expected ZeroDivisionError')

        try:
            print(list(pool.imap(f, list(range(10)))))
        except ZeroDivisionError:
            print('\tGot ZeroDivisionError as expected from list(pool.imap())')
        else:
            raise AssertionError('expected ZeroDivisionError')

        it = pool.imap(f, list(range(10)))
        for i in range(10):
            try:
                x = next(it)
            except ZeroDivisionError:
                if i == 5:
                    pass
            except StopIteration:
                break
            else:
                if i == 5:
                    raise AssertionError('expected ZeroDivisionError')

        assert i == 9
        print('\tGot ZeroDivisionError as expected from IMapIterator.next()')
        print()

        ## Testing timeouts

        print('Testing ApplyResult.get() with timeout:', end=' ')
        res = pool.apply_async(calculate, TASKS[0])
        while 1:
            sys.stdout.flush()
            try:
                sys.stdout.write('\n\t%s' % res.get(0.02))
                break
            except multiprocessing.TimeoutError:
                sys.stdout.write('.')
        print()
        print()

        print('Testing IMapIterator.next() with timeout:', end=' ')
        it = pool.imap(calculatestar, TASKS)
        while 1:
            sys.stdout.flush()
            try:
                sys.stdout.write('\n\t%s' % it.next(0.02))
            except StopIteration:
                break
            except multiprocessing.TimeoutError:
                sys.stdout.write('.')
        print()
        print()

if __name__ == '__main__':
    multiprocessing.freeze_support()
    test()
```

#### An example showing how to use queues to feed tasks to a collection of worker processes and collect the results:

```
import time
import random

from multiprocessing import Process, Queue, current_process, freeze_support

## Function run by worker processes

def worker(input, output):
    for func, args in iter(input.get, 'STOP'):
        result = calculate(func, args)
        output.put(result)

## Function used to calculate result

def calculate(func, args):
    result = func(*args)
    return '%s says that %s%s = %s' % \
        (current_process().name, func.__name__, args, result)

## Functions referenced by tasks

def mul(a, b):
    time.sleep(0.5*random.random())
    return a * b

def plus(a, b):
    time.sleep(0.5*random.random())
    return a + b

def test():
    NUMBER_OF_PROCESSES = 4
    TASKS1 = [(mul, (i, 7)) for i in range(20)]
    TASKS2 = [(plus, (i, 8)) for i in range(10)]

    # Create queues
    task_queue = Queue()
    done_queue = Queue()

    # Submit tasks
    for task in TASKS1:
        task_queue.put(task)

    # Start worker processes
    for i in range(NUMBER_OF_PROCESSES):
        Process(target=worker, args=(task_queue, done_queue)).start()

    # Get and print results
    print('Unordered results:')
    for i in range(len(TASKS1)):
        print('\t', done_queue.get())

    # Add more tasks using `put()`
    for task in TASKS2:
        task_queue.put(task)

    # Get and print some more results
    for i in range(len(TASKS2)):
        print('\t', done_queue.get())

    # Tell child processes to stop
    for i in range(NUMBER_OF_PROCESSES):
        task_queue.put('STOP')


if __name__ == '__main__':
    freeze_support()
    test()
```