# Threading Guildlines for C#

##     1. What is a mutex?
               A mutex object is a synchronization object whose state is set to signaled when it is not owned by any thread, 
               and nonsignaled when it is owned. Only one thread at a time can own a mutex object, whose name comes from the 
               fact that it is useful in coordinating mutually exclusive access to a shared resource. For example, to prevent 
               two threads from writing to shared memory at the same time, each thread waits for ownership of a mutex object 
               before executing the code that accesses the memory. After writing to the shared memory, the thread releases the 
               mutex object.
               
               msdn for mutex example:
               https://msdn.microsoft.com/en-us/library/system.threading.mutex(v=vs.110).aspx
               
               Simply put the mutex object is a type of object created that can resemble a gaurd post - it is created and then
               at any point a thread can activate it in a method to make sure all other threads stop at that post. once the
               mutex is released the next method can enter and so on.
               Of course mutex also has additional properties such as timeout where all methods trying to access it can be diverted
               elswhere in case enouph time has passed an so on.
               
## 2. What is a semaphore?
               A semaphore for all purposes is almost the same thing as a mutex just that it restricts access using a counter
               determining the amount of threads that can pass it at a time (access its restricted content). 
               In other words, if a semaphore is started at zero and is given a max of three, at most three threads will be able
               to enter its restricted area at any given time (each thread that exits the semaphore will release its hold and
               allow a new thread to enter).
               
### 2.. What is the difference between mutex and semaphore?
               In this case the difference is simple, a semaphore can provide and dissallow access based on more than one thread - 
               as in, it has a counter where mutex can work with one thread owning it at a time.
               BUT - another question that can be asked is what is the difference between a mutex and a binary semaphore?
               A good answer is at: http://stackoverflow.com/questions/62814/difference-between-binary-semaphore-and-mutex
               The gist of it is that the mutex has an owner, each time the mutex is taken control of it can be relinquished only
               by its owner (kind of like a talking stick where only the owner can say when they finished talking and pass it on
               to the manager so they can give it to the next person who wants to talk).
               A semaphore on the other hand can be accessed to be given or taken as needed between processess. 
               (TODO: understand a bit better when a semaphore can be transfered and why its a good idea).
               
               In windows a binary semaphore is very similare to an event.
                  
##  3. What is a critical section?
               A critical section is an area of code that's access must be limited in a multithreaded enviroment since concurrent
               access to the area might cause issues. In other words - the critical section is a delicate area of code that
               requires mutual exclusion or the codes outcome might do things you dont want it to do.
               
               A critical section may consist of multiple discontiguous parts of the program's code. For example, one part of 
               a program might read from a file that another part wishes to modify. These parts together form a single critical 
               section, since simultaneous readings and modifications may interfere with each other.
          
               A critical section will usually terminate in finite time,[1] and a thread, task, or process will have to wait for 
               a fixed time to enter it (aka bounded waiting). Some synchronization mechanism is required at the entry and exit 
               of the critical section to ensure exclusive use, for example a semaphore.
               
               The function lock(referencedObject){critical section} is an example of a critical section.
               The lock keyword ensures that one thread does not enter a critical section of code while another thread is 
               in the critical section. If another thread tries to enter a locked code, it will wait, block, until the object 
               is released.
               
               In general, avoid locking on a public type, or instances beyond your code's control. 
               The common constructs lock (this), lock (typeof (MyType)), and lock ("myLock") violate this guideline:
         
               lock (this) is a problem if the instance can be accessed publicly.
               lock (typeof (MyType)) is a problem if MyType is publicly accessible.
               lock("myLock") is a problem because any other code in the process using the same string, will share the same lock.
               Best practice is to define a private object to lock on, or a private static object variable to protect data common 
               to all instances.
          
### 3..What is the difference between a critical section and a mutex?
               For Windows, critical sections are lighter-weight than mutexes.
               Mutexes can be shared between processes, but always result in a system call to the kernel which has some overhead.
               Critical sections can only be used within one process, but have the advantage that they only switch to kernel mode
               in the case of contention - Uncontended acquires, which should be the common case, are incredibly fast. In the case
               of contention, they enter the kernel to wait on some synchronization primitive (like an event or semaphore).
               
##    4. TPL - Task Parallel Library
###   4.a.What are tasks?
                   Tasks are .NET managed threads. 
                   The Task class represents a single operation that does not return a value and that usually executes 
                   asynchronously.
                   A task is started using the run function. In addition it has many other functions that work with it such
                   as wait that provides the capability to wait for another task to complete, waiting for an amount of time, 
                   synchronization and so forth, many of the threads methods are translated in practice to the task form.
                   
###   4.b.What is the difference between tasks and threads?
                   Tasks are more efficient and more scalable use of system resources.
                   Behind the scenes, tasks are queued to the ThreadPool, which has been enhanced with algorithms that 
                   determine and adjust to the number of threads and that provide load balancing to maximize throughput. 
                   This makes tasks relatively lightweight, and you can create many of them to enable fine-grained parallelism.
                   
                   Tasks provide more programmatic control than is possible with a thread or work item.
                   Tasks and the framework built around them provide a rich set of APIs that support waiting, cancellation, 
                   continuations, robust exception handling, detailed status, custom scheduling, and more.
                   
                   Like the ThreadPool, a task does not create its own OS thread. Instead, tasks are executed by a TaskScheduler; 
                   the default scheduler simply runs on the ThreadPool.
                   Unlike the ThreadPool, Task also allows you to find out when it finishes, and (via the generic Task<T>) to 
                   return a result. You can call ContinueWith() on an existing Task to make it run more code once the task finishes.
                   
                   Since tasks still run on the ThreadPool, they should not be used for long-running operations, since they can 
                   still fill up the thread pool and block new work. Instead, Task provides a LongRunning option, which will 
                   tell the TaskScheduler to spin up a new thread rather than running on the ThreadPool.
                   
                   The bottom line is that Task is almost always the best option; it provides a much more powerful API and 
                   avoids wasting OS threads.
                   The only reasons to explicitly create your own Threads in modern code are setting per-thread options, or 
                   maintaining a persistent thread that needs to maintain its own identity.
         
###     4.c.Tasks implement IDisposable, do you need to dispose of tasks?
                   The short answer is -No.
                   The longer answer is not unless you are having performace standards that you need to meet and are makeing
                   1000000% sure that the tasks are no longer in use when you are disposing of them.
                   The main reason Tasks implement IDisposable is because the contain IDisposable recourses and according to the
                   .net framework guildlines this meens its better to also implement this in the parent.
                   
##      5. Threads
###   5.a.Whats are threads?
                   
                   C# supports parallel execution of code through multithreading. A thread is an independent execution path, able
                   to run simultaneously with other threads.
                   
                   Each thread has its own stack & kernal recources.
                   
                   Multiple threads can be created via a threadpool which is managed by the clr.
                   The problem with Thread is that OS threads are costly. Each thread you have consumes a non-trivial amount of 
                   memory for its stack, and adds additional CPU overhead as the processor context-switch between threads. 
                   Instead, it is better to have a small pool of threads execute your code as work becomes available.
                   
###    5.b.What are issues that may occure with threads?     
                   
                   Improved performance and concurrency
                       For certain applications, performance and concurrency can be improved by using multithreading and 
                       multicontexting together. In other applications, performance can be unaffected or even degraded by 
                       using multithreading and multicontexting together. How performance is affected depends on your application.
                       
                   Threads are very costly when it comes to memory and control. 
                       Each thread has its own stack and recources and switching between threads can be expencive. The same can 
                       be said for threadpools, even though they do provide a way to manage multiple threads more easily the main 
                       drawback is the inability to control what happens when in what order and when its done + what it returns 
                       (doesnt get the return values).
                       
                   Difficulty of writing code
                      Multithreaded and multicontexted applications are not easy to write. Only experienced programmers should 
                      undertake coding for these types of applications.
               
                   Difficulty of debugging
                       It is much harder to replicate an error in a multithreaded or multicontexted application than it is to do 
                       so in a single-threaded, single-contexted application. As a result, it is more difficult, in the former 
                       case, to identify and verify root causes when errors occur.
                   
                   Difficulty of managing concurrency
                       The task of managing concurrency among threads is difficult and has the potential to introduce new problems 
                       into an application.
                   
                   Difficulty of testing
                       Testing a multithreaded application is more difficult than testing a single-threaded application because 
                       defects are often timing-related and more difficult to reproduce.
                   
                   Difficulty of porting existing code
                       Existing code often requires significant re-architecting to take advantage of multithreading and 
                       multicontexting. Programmers need to:
                           Remove static variables
                           Replace any function calls that are not thread-safe
                           Replace any other code that is not thread-safe
                       Because the completed port must be tested and re-tested, the work required to port a multithreaded and/or 
                       multicontexted application is substantial.                   
                   
###         5.c.What are threads good for? What are they used for?
                   
                   Improved performance and concurrency
                       For certain applications, performance and concurrency can be improved by using multithreading and 
                       multicontexting together. In other applications, performance can be unaffected or even degraded by 
                       using multithreading and multicontexting together. How performance is affected depends on your application.
                       
                    Simplified coding of remote procedure calls and conversations
                       In some applications it is easier to code different remote procedure calls and conversations in separate 
                       threads than to manage them from the same thread.
                       
                    Simultaneous access to multiple applications
                       Your BEA Tuxedo clients can be connected to more than one application at a time.
         
                    Reduced number of required servers Because one server can dispatch multiple service threads the number 
                    of servers to start for your application is reduced. 
                       This capability for multiple dispatched threads is especially useful for conversational servers, which 
                       otherwise must be dedicated to one client for the entire duration of a conversation.
                   
###          5.d.What is a threadpool?
                   
                   ThreadPool is a wrapper around a pool of threads maintained by the CLR. ThreadPool gives you no control at all;
                   you can submit work to execute at some point, and you can control the size of the pool, but you can’t set 
                   anything else. You can’t even tell when the pool will start running the work you submit to it.
                   
                   Using ThreadPool avoids the overhead of creating too many threads. However, if you submit too many long-running 
                   tasks to the threadpool, it can get full, and later work that you submit can end up waiting for the earlier 
                   long-running items to finish. In addition, the ThreadPool offers no way to find out when a work item has been 
                   completed (unlike Thread.Join()), nor a way to get the result. Therefore, ThreadPool is best used for short 
                   operations where the caller does not need the result.
                   
## 6. What are some good to know issues you might encounter when using threads\tasks?
###              6..Deadlock:
                           Deadlocks are when two threads\ tasks lock a section\s while remaining dependant in a way that neither
                           can unlock and neither can continue their function such that they both remain in a constant state where
                           they freeze waiting for the other. Deadlock in short freez the system (in the threads scope and whatever
                           is dependant on it)
                           
###                6..Starvation:
                           Starvation is a situation where the time threads get on tasks it not managed properly and the result is
                           that a thread (or a group that does not include all the threads) hogs up all the cpu and "starves" a
                           thread, a group of threads or all threads except one. In this case certain actions will not go through
                           since they never get cpu time.
                       
###                    6..the for issue
                           for(int i = 0; i < 10; i++){
                               doSomething(i);
                           }
                           In the for loop its easy to see that the once inside two threads can cause one another synchronization
                           issues but where does this stem from?
                           So the i incrementation is also something thats easy to see but what is even more important is the more
                           general i++ rule:
                           i++ behind the scenes is translated to:
                           j = i + 1
                           i = j
                           therefore the threads can cause issues inside of the i++ itself and stop anywhere in the sub implementation.
                           
###                    6..The tasks result async issue.
                           When working with tasks there are two methods that might cause issue if invoked with dependancy:
                           task t = async task.run;
                           v = t.result;
                           
                           since the async forces the result to return and continue on the same thread it was shipped from
                           and the resuly locks the entire thread until it is given its result. A situation can happen where
                           the async leaves, the result is requested and locks the thread, when the async returns it tries to enter
                           the thread but its locked since the result is waiting for an answer... which is the async, so in fact
                           the async will never be able to enter and the result will never open the lock resulting in a deadlock.
                           /
