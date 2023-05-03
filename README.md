Download Link: https://assignmentchef.com/product/solved-cs2200-project4
<br>
<h1>1        Overview</h1>

In this project, you will implement a multiprocessor operating system simulator using a popular userspace threading library for linux called pthreads. The framework for the multithreaded OS simulator is nearly complete, but missing one critical component: the CPU scheduler! Your task is to implement the CPU scheduler, using three different scheduling algorithms.

<strong>Note: Make sure that multiple CPU cores are enabled in your virtual machine, otherwise you will receive incorrect results. See the TAs if you need help.</strong>

If you are using the CS 2200 Vagrant box, the number of cores should default to 2. You can run nproc –all to see how many cores are available to your VM.

We have provided you with source files that constitute the framework for your simulator. You will only need to modify answers.txt and student.c. However, just because you are only modifying two files doesn’t mean that you should ignore the other ones – there is helpful information in the other files. We have provided you these files:

<ol>

 <li>Makefile – Working one provided for you; do not modify.</li>

 <li>os-sim.c – Code for the operating system simulator which calls your CPU scheduler.</li>

 <li>os-sim.h – Header file for the simulator.</li>

 <li>c – Descriptions of the simulated processes.</li>

 <li>h – Header file for the process data.</li>

 <li>c – This file contains stub functions for your CPU scheduler.</li>

 <li>h – Header file for your code to interface with the OS simulator. Also contains ready queue struct definition.</li>

</ol>

<strong>Reminder: The only files that you need to edit are </strong>student.c <strong>and </strong>answers.txt<strong>. If you edit any other files, your code may fail the autograder!</strong>

<h2>1.1       Scheduling Algorithms</h2>

For your simulator, you will implement the following three CPU scheduling algorithms:

<ol>

 <li><strong>First Come, First Serve (FCFS) </strong>– Runnable processes are kept in a ready queue. FCFS is nonpreemptive; once a process begins running on a CPU, it will continue running until it either completes or blocks for I/O.</li>

 <li><strong>Round-Robin </strong>– Similar to FCFS, except preemptive. Each process is assigned a timeslice when it is scheduled. At the end of the timeslice, if the process is still running, the process is preempted, and moved to the tail of the ready queue.</li>

 <li><strong>Round-Robin with Priority </strong>– Similar to Round-Robin, but includes priority. Processes with higher priority get to run first and processes with lower priority get preempted for a process with higher priority.</li>

</ol>

<h2>1.2       Process States</h2>

In our OS simulation, there are five possible states for a process, which are listed in the process state t enum in os-sim.h:

<ol>

 <li>NEW – The process is being created, and has not yet begun executing.</li>

 <li>READY – The process is ready to execute, and is waiting to be scheduled on a CPU.</li>

 <li>RUNNING – The process is currently executing on a CPU.</li>

 <li>WAITING – The process has temporarily stopped executing, and is waiting on an I/O request to complete.</li>

 <li>TERMINATED – The process has completed.</li>

</ol>

There is a field named state in the PCB, which must be updated with the current state of the process. The simulator will use this field to collect statistics.

Figure 1: Process States

<h2>1.3       The Ready Queue</h2>

On most systems, there are a large number of processes, but only one or two CPUs on which to execute them. When there are more processes ready to execute than CPUs, processes must wait in the READY state until a CPU becomes available. To keep track of the processes waiting to execute, we keep a ready queue of the processes in the READY state

Since the ready queue is accessed by multiple processors, which may add and remove processes from the ready queue, the ready queue must be protected by some form of synchronization–for this project, you will use a mutex lock that we have provided called ready mutex.

<h2>1.4       Scheduling Processes</h2>

schedule() is the core function of the CPU scheduler. It is invoked whenever a CPU becomes available for running a process. schedule() must search the ready queue, select a runnable process, and call the context switch() function to switch the process onto the CPU.

Note that in a multiprocessor environment, we cannot mandate that the currently running process be at the head of the ready q. There is an array (one entry for each cpu) that will hold the pointer to the PCB currently running on that cpu.

There is a special process, the idle process, which is scheduled whenever there are no processes in the READY state. This process simply waits for something new to be added to the ready queue and then calls schedule().

<h2>1.5       CPU Scheduler Invocation</h2>

There are five events which will cause the simulator to invoke schedule():

<ol>

 <li>yield() – A process completes its CPU operations and yields the processor to perform an I/O request.</li>

 <li>wake up() – A process that previously yielded completes its I/O request, and is ready to perform CPU operations. wake up() is also called when a process in the NEW state becomes runnable.</li>

 <li>preempt() – When using a Round-Robin or Round-Robin with Priority scheduling algorithm, a CPUbound process may be preempted before it completes its CPU operations.</li>

 <li>terminate() – A process exits or is killed.</li>

 <li>idle() – Waits for a new process to be added to the ready queue (details below).</li>

</ol>

The CPU scheduler also contains one other important function: idle(). idle() contains the code that gets by the idle process. In the real world, the idle process puts the processor in a low-power mode and waits. For our OS simulation, you will use a pthread condition variable to block the thread until a process enters the ready queue.

<h2>1.6       The Simulator</h2>

We will use pthreads to simulate an operating system on a multiprocessor computer. We will use one thread per CPU and one thread as a ’supervisor’ for our simulation. The supervisor thread will spawn new processes (as if a user started a process). The CPU threads will simulate the currently-running processes on each CPU, and the supervisor thread will print output.

Since the code you write will be called from multiple threads, the CPU scheduler you write must be threadsafe! This means that all data structures you use, including your ready queue, must be protected using mutexes.

The number of CPUs is specified as a command-line parameter to the simulator. For this project, you will be performing experiments with 1, 2, and 4 CPU simulations.

Also, for demonstration purposes, the simulator executes much slower than a real system would. In the real world, a CPU burst might range from one to a few hundred milliseconds, whereas in this simulator, they range from 0.2 to 2.0 seconds.

Figure 2: Simulator Function Calls

The above diagram should give you a good overview of how the system works in terms of the functions being called and PCBs moving around. Below is a second diagram that shows the entire system overview and the code that you need to write is inside of the green cloud at the bottom. All of the items outside of the green cloud are part of the simulator and will not need to be modified by you. If you would like to zoom in to get a better look, this image is included in the project files as system-diagram.jpg.

Figure 3: System overview

Compile and run the simulator with ./os-sim 2. After a few seconds, hit Control-C to exit. You will see the output below:

Figure 4: Sample Output

The simulator generates a Gantt Chart, showing the current state of the OS at every 100ms interval. The leftmost column shows the current time, in seconds. The next three columns show the number of Running, Ready, and Waiting processes, respectively. The next two columns show the process currently running on each CPU. The rightmost column shows the processes which are currently in the I/O queue, with the head of the queue on the left and the tail of the queue on the right.

As you can see, nothing is executing. This is because we have no CPU scheduler to select processes to execute! Once you complete Problem 1 and implement a basic FIFO scheduler, you will see the processes executing on the CPUs.

<h1>2        Problem 0: The Ready Queue</h1>

Implement the helper functions enqueue(), dequeue(), and is empty() in student.c for the queue struct provided. The struct will serve as your ready queue, and you should be using these helper functions to add and remove processes from the ready queue in the problems to follow. You can find the declarations of queue t, enqueue(), dequeue(), and is empty() in student.h.

<h2>2.1       Hints</h2>

<ul>

 <li>Your queue should be backed by a linked list with each PCB acting as a node. There is a field in the PCB, next, which you may use to build linked lists of PCBs.</li>

 <li>You should not be editing any of the fields of the PCBs inside the queue aside from next.</li>

 <li>There is an edge case for dequeue() which you must handle. Take a look at the documentation for the function for more details.</li>

 <li>When using the ready queue helper functions in the following problems, make sure to call them in a thread-safe manner. Read up on how to use mutex locks and lock/unlock your queue struct if and when you call these functions.</li>

</ul>

<h1>3        Problem 1: FCFS Scheduler</h1>

<strong>NOTE: Part B of this and each following problem requires you to put your answer down in answers.txt</strong>

<strong>Part A. </strong>Implement the CPU scheduler using the FCFS scheduling algorithm. You may do this however you like, however, we suggest the following:

<ul>

 <li>Implement the yield(), wake up(), and terminate() preempt() is not necessary for this stage of the project. See the overview and the comments in the code for the proper behavior of these events.</li>

 <li>Implement idle(). idle() must wait on a condition variable that is signalled whenever a process is added to the ready queue.</li>

 <li>Implement schedule(). schedule() should extract the first process in the ready queue, then call context switch() to select the process to execute. If there are no runnable processes, schedule() should call context switch() with a NULL pointer as the PCB to execute the idle process.</li>

</ul>

<h2>3.1       Hints</h2>

<ul>

 <li>Be sure to update the state field of the PCB. The library will read this field to generate the Running, Ready, and Waiting columns, and to generate the statistics at the end of the simulation.</li>

 <li>Four of the five entry points into the scheduler (idle(), yield(), terminate(), and preempt()) should cause a new process to be scheduled on the CPU. In your handlers, be sure to call schedule(), which will select a runnable process, and then call context switch(). When these four functions return, the library will simulate the execution of the process selected by context switch().</li>

 <li>context switch() takes a timeslice parameter, which is used for preemptive scheduling algorithms. Since FCFS is non-preemptive, use -1 for this parameter to give the process an infinite timeslice.</li>

 <li>Make sure to use the helper functions in a thread-safe manner when adding and removing processes from the ready queue!</li>

 <li>The current[] array should be used to keep track of the process currently executing on each CPU. Since this array is accessed by multiple CPU threads, it must be protected by a mutex. current mutex has been provided for you.</li>

</ul>

<strong>Part B. </strong>Run your OS simulation with 1, 2, and 4 CPUs. Compare the total execution time of each. Is there a linear relationship between the number of CPUs and total execution time? Why or why not? Keep in mind that the execution time refers to the simulated execution time.

<h1>4        Problem 2: Round-Robin Scheduler</h1>

<strong>Part A. </strong>Add Round-Robin scheduling functionality to your code. You should modify main() to add a command line option, -r, which selects the Round-Robin scheduling algorithm, and accepts a parameter, the length of the timeslice. For this project, timeslices are measured in tenths of seconds. E.g.:

./os-sim <em>&lt;</em># CPUs<em>&gt; </em>-r 5

should run a Round-Robin scheduler with timeslices of 500 ms. While:

./os-sim <em>&lt;</em># of CPUs<em>&gt;</em>

should continue to run a FCFS scheduler. You should also make sure preempt is implemented in this section of the project.

To specify a timeslice when scheduling a process, use the timeslice parameter of context switch(). The simulator will simulate a timer interrupt to preempt the process and call your preempt() handler if the process executes on the CPU for the length of the timeslice without terminating or yielding for I/O.

<strong>Part B. </strong>Run your Round-Robin scheduler with timeslices of 800ms, 600ms, 400ms, and 200ms. Use only one CPU for your tests. Compare the statistics at the end of the simulation. Is there a relationship between the total waiting time and timeslice length? If so, what is it? In contrast, in a real OS, the shortest timeslice possible is usually not the best choice. Why not?

<h1>5        Problem 3: Round-Robin with Priority</h1>

<strong>Part A. </strong>Add Round-Robin with Priority scheduling to your code. Modify main() to accept the -rp parameter to select the Round-Robin with Priority algorithm. The -r and default FCFS scheduler should continue to work as normal and Round Robin with Priority should also take the timeslice parameter as Round Robin. The scheduler should use the priority field of the PCB to prioritize processes that have a higher priority in their CPU burst. Your ready queue should become a priority queue, that is the queue should be arranged in descending order of priority.

You need to <strong>increment </strong>the priority of a process if does I/O execution (or “yields”) and <strong>decrement </strong>the priority everytime it gets scheduled on the CPU. For Round-Robin with Priority scheduling, you will need to make use of the force preempt() function which preempts a running process before its timeslice expires. Your wake up() handler should make use of this function to preempt a process when a process with lower time remaining needs a CPU.

<strong>Part B. </strong>Consider the Round Robin scheduler with Priority that you just added. Noting that you run 8 processes in each simulation, calculate the total throughput for the cases of 1, 2, and 4 CPUs using a 600ms timeslice. Does the throughput improve in proportion to the number of CPUs? If not, what do you think is the cause?

Run each of the scheduling algorithms using one CPU and compare the total waiting times. Which one had the lowest? Why?

<h1>6        Problem 4: The Priority Inversion Problem (Short Answer)</h1>

Assume we have three processes X, Y, and Z; X has high priority, Y has medium priority, and Z has low priority. Assume X and Z work with/access a shared resource S, <em>and that Y does not need or use S</em>. There exists a locking mechanism so that for any process P which is currently using S, no other process can use S unless P gives up the lock.

Assume that, currently, Z has control of S. If X tries to get access to S, X must wait until Z (a lower priority process) gives up S (and releases the locking mechanism).

If Y becomes runnable in this time interval, during the time that X is waiting on Z to give up S, Y will preempt Z, as Y has higher priority than Z and doesn’t require the usage/access to S. As a result, Z is unable to give up access to S (because the preemption done by Y didn’t affect Z’s holding of the lock). We face an interesting problem: we observe starvation of a high priority process X by lower priority processes Y and Z. This is known as a <em>priority inversion problem</em>.

Assume we have a scheduler that does have preemption (and we cannot use non-preemptive schedulers). Assume also that we can decrease or increase the priority of a process <em>during its runtime on the CPU</em>. Given these constraints, how might we ensure that X is able to execute before Y?


