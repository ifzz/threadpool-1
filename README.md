# threadpool
A fork / join framework threadpool that implements work-stealing and work helping.
View threadpool.c for my code. All other code and tests were written by the instructor. 

My thread pool structure contains a global queue of futures, a queue of worker threads,
a condition variable to signal when a task is ready to be executed, and one lock to control all 
the data in the pool. It also includes a shutdown flag and a barrier that syncs all the threads
at the start of execution. This is to ensure the threads gain the lock first and are doing most 
of the work rather then the main routine calling future_get. Each worker has their own queue of 
futures along with thread id. Each future stores the task and its data, a conditional variable 
to flag when it is done executing, its status, and reference to the pool it is contained in. 

A worker thread's flow goes like this: It first encounters a while loop that determine
if it should be sleeping or not. A worker thread should be sleeping if the global queue is empty,
its worker queue is empty, and all other worker queue's are empty. This is determined in the
static function sleeping(). If it should be sleeping it does so and awakens when the work_flag
is signaled. This flag is signaled when there is a thread pool submit call. Once a working thread
is awoken it must determine which queue to get its task from. First it checks its global queue, 
then its own. If both are empty then it steals a task from the bottom of the first worker queue 
that is not empty. All the queues that store futures only store futures that have not been started. 
Once a future is executing it is removed from whatever queue it was in. The worker queue releases
the lock as it no longer needs it and being executing the fork join task. When it is done executing 
it reacquires the lock, marks the future as completed, and signals the future's cond done flag to
let whatever is relying on this future know that it is done executing. The worker thread then 
repeats this process until the shutdown boolean is flagged.
                   
The only error valgrind is giving is when I run with the race checker, however I believe
it is a false positive. It is telling me that I am trying to destroy an unknown condition variable 
in future_get. However this condition variable is always initialized in future submit and cond_destroy 
is never called other than in future_free. 
