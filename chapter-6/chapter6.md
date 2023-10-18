# 6 Synchronization
Page 257 in book

### 6.1 Background
Programs may execute on multiple threads, and a single processor may be rapidly switching between processes. This chapter deals with how parallel execution can create problems with shared data between processes.

#### The Problem
Given some variable count. Thread one is doing count++, and thread two is doing count--. Suppose the instructions get interweaved:
```
T0: producer execute register1 = count {register1 = 5}
T1: producer execute register1 = register1 +1 {register1 = 6}
T2: consumer execute register2 = count {register2 = 5}
T3: consumer execute register2 = register2 −1 {register2 = 4}
T4: producer execute count = register1 {count = 6}
T5: consumer execute count = register2 {count = 4}
```
The result should be 5++ = 6, 6-- = 5, but instead since we get 4. This is called a _race condition_, where two threads are attempting to manipulate the same data concurently. To prevent this, we need to make sure only one process at a time can modifiy any particular variable (here, count). We call this **process synchronization**.
### 6.2 Critical Section
Programs are split into sections:
1. **The Entry Section**, where the program request access to enter the critical section.
2. **The Critical Section** is the section that concenrs modifying or accessing data shared with at least one other process.
3. **The Exit Section** (_may be optional_), is principly concerned with critical section access, similar to the inverse of the entry section. 
4. **The Remainder Section**, which is simply the rest of the code.

The **critical section**  The difficult problem is making sure that _only one process_ at a time is allowed to execute in it's critical section.

The book goes on to note 3 criteria that a solution to this problem must maintain (some bits paraphrased, page 260-261):

> 1. Mutual exclusion. Only one process can execute in it's critical section at a time.
> 2. Progress. If no process is executing in its critical section and some processes
wish to enter their critical sections, then only those processes that
are not executing in their remainder sections can participate in deciding
which will enter its critical section next, and this selection cannot be
postponed indefinitely.
> 3. Bounded waiting. There exists a bound, or limit, on the number of times
that other processes are allowed to enter their critical sections after a
process has made a request to enter its critical section and before that
request is granted.

*Note*: The problem is easily avoidable on a single core enviorment by making sure that no interrupt can occur while in the critical section. Thus, a particular sequence of instructions could execute without problem. However, on multiple core systems this solution dissolves.

### 6.3 Peterson's Solution

Full walkthrough see [chapter 6.3](6.3.md).
**Summary**
Peterson's solution uses two variables, and is designed to work for *two* processes sharing the same memory. A boolean named turn, and a boolean array size 2, flag[2] are shared between the threads.

Whenever a process wants to write, it first assigns the turn value such that the other process is granted access to write. Then it checks, and if either of the following two conditions are true it will write:
1) The other process does not want to access memory
2) The turn variable has been changed to give the current process access.

This creates three possibilites:
1) Other process does not want to access memory, current process may proceed.
2) Other process wants to access memory, but has given it's turn to current process and it may proceed.
3) Other process wants to access memory, and has been given it's turn first. It will access memory, and current process will wait until it's done at which it will proceed.

### 6.4 Hardware Support

#### 6.4.1 Memory Locks
Processors fall into two memory models:


1. Strongly ordered, where a memory modification on one processor is
immediately visible to all other processors.
2. Weakly ordered, where modifications to memory on one processor may
not be immediately visible to other processors.
(Quote from page 265).

Low-level programmers use **memory barriers** or **memory fences**. A memory barrier simply stops the further execution of instructions until all register load/store operations have been completed.
E.g.,
```cpp
x = 100; // We know this WILL execute before the next assignment.
memory_barrier();
y = true;
```
#### 6.4.2 Hardware Instructions
> In concurrent programming, an operation (or set of operations) is **atomic** if it appears to the rest of the system to occur at once without being interrupted
[Source](https://www.it.uu.se/education/course/homepage/os/vt18/module-4/definitions/#:~:text=or%20a%20process.-,Atomic%20operations,%3A%20linearizable%2C%20indivisible%20or%20uninterruptible.)

Generally speaking hardware may support two types of instructions: 
##### 1. Test and Set
```cpp
bool test_and_set(bool *target)
{  /* test_and_set is implemented as a low-level call somewhere else; consider
   *  this as pseudo-code.
   *  Remember: this code will execute as a single uninterrupted block, and no other core and execute a test_and_set instruction
   *  While one is executing. */
   bool return_value = *target; // Copy the original value of the target before modifying.
   *target = true;

   return return_value;
}

// Program:
// lock is initialized to false somewhere prior and is shared between threads.
while(test_and_set(&lock)
  ; // wait

// Critical Section Here

lock = false;

```
When test_and_set is executed, it can be divided into one of two cases,
**Case 1:**
`lock == false`, the value of lock is false.
The code will return false, but set the value of the lock to true.
**Case 2:**
`lock == true`
The code will return true, and the lock value will remain true.

What this means is that if we wait on test_and_set() => `while(test_and_set(&lock)) ;`, either &lock is initially false, or we will wait until someone else sets `*lock = false;` This means the first thread will continue with execution, but set the lock to true so the second thread must wait until the first is done.

#### 2. Compare and Swap

Compare and Swap works nearly identically, 

```cpp
int compare_and_swap(int *value, int expected, int new_value)
{ // Operates Atomically
  int return_value = *value; // Save the current value as return value.

  if(*value == expected)
    *value = new_value;

  return temp;
}

// Program:
while(compare_and_swap(&lock, 0, 1) != 0)
  ; // Wait

// Critical section

lock = 0;

// Remainder section
```