### Semiphors

#### Definitions

> A sempahore (S) is an integer variable that, apart from initialization, is accessed only through two standard atomic operations: `wait()` and `signal`

Definition of `wait()`:
```cpp
wait(S)
{
	while(S <= 0)
		; // busy wait
	S--
}
```
Definition of `signal()`:
```cpp
signal(S)
{
	s++;
}
```
#### Explaination

The testing, `while(s <= 0)`, and modification of S (`S++, S--`) must be preformed atomically. However, in order for a semphore to work, S may be modified _after_ it is checked in the while loop.

The value `S` should reflect the number of resources available; often this is a binary
semaphore where only 1 resource is available. 

A binary semiphor is essentially a mutex lock, where-as a counting sempahore is used to control access to a resource with multiple instances.

When a process wants to access the critical region it preforms a wait() operation.

If the value of S, or resources available is 0, the program will wait until another process increments it with the `signal()` function. 


#### Unit 8 Synchronization Part 2.pptx slide 4 class example

_Note: example slightly modified for clarity._

*Initialization*
```
Semaphore a = 0;
Semaphore b = 0;
```
*Parent*
```cpp
for(int i=1;i<=3;i++)
{
	cout << "P:" << i << " ";
	signal(a); // Adds 1 to semaphore A 
	// allowing execution of next wait
	wait(b); // Waits on sempahore B to become 1
}
```
*Child*
```cpp
for(int i=1;i<=3;i++)
{
	wait(a); // Waits until signal(a); in parent
	cout << "C:" << i << " ";
	signal(b); // Allows wait(b) in parent to continue exeuction.
}
```

Result:
```
P:1 C:1 P:2 C:2 P:3 C:3
```

Comment:

First, we say "parent and child" but formally speaking this is incorrect as these are _threads_ not forks.
Here we see that "P: " will execute 1 time, initially couting `P:1 `. Next it signals to `a` and waits on `b`, allowing the child to execute, and waiting for it's go ahead.
Next, the child after being allowed to execute couts `C:1 `, signals that parent may run, iterates through 1 and proceeds to wait on parent, effectively reseting to the starting start.

