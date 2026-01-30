Remember how we said that the computer's memory was just a large array of bytes, well it turns out that was just an oversimplification and it has more structure than that(ehhn,not really though but allow), it's divided into two main regions; **The stack and The heap.** We'll talk about the heap later. **The stack: It's a specified portion of the computer's main memory that's used by functions as some sort of scratchpad.**
The stack is incredibly fast because it is self managing, meaning you don't have to manually handle memory and also because it follows the **Last In, First Out (LIFO) protocol** much like a physical stack of dinner plates. You only every interact with the plate at the very top.

---

## 1. The Anatomy of a Stack Frame

When you call a function in your program, the computer carves out a specific block of stack memory just for that call. This block is called a **Stack Frame(or an Activation Record)** 
A stack frame typically contains three things:
- **Return Address:** A "bookmark" so the CPU knows exactly which line of code to jump back to once the function finishes.
- **Parameters/Arguments:** The inputs you passed in (like `int sugar =2`). FIRST THING
- **Local variables:** The variables you declare inside the function. SECOND THING

So if we have this:

```c
void create_typist(int uses_nvim){
	int wpm = 150;
	// ...
}
```

When we call `create_typist(1)`, the computer moves the "stack pointer" to reserve space, stores `1`(the argument), stores the return address, and then allocates space for `wpm`. It essentially says: "Give me 4 bytes for `wpm`, store `150` in there, and for now, that address is what we call `wpm`

## 2. THE STACK POINTER (The "Finger" on the Page)
You might be wondering, "How does the computer know where the top of the stack is and how does it track different stack frames for different functions?"

It uses a special register called the **Stack Pointer (RSP/ESP).** This is just a variable inside the CPU that holds the address of the very top of the stack. 
- **Allocation(Pushing):** To create space, the CPU just moves this pointer.
- **Deallocation(Popping):** When the function hits that closing `}`, the CPU doesn't go through and scrub the memory to zeros(that's too slow). It just moves the Stack Pointer back to where it was before the function started.
The data is essentially "abandoned". It's still there physically, but it's in "No man's Land" now. The next function call will just ruthlessly overwrite it.

## 3. The "Upside Down" World
Shit gets weird here. On most modern systems, the Stack grows **downwards.**
- **The Heap(where big data lives):** Starts at low addresses (`0x0000`) and grows **UP.**
- **The Stack:** Starts at high addresses (`0xFFFF`) and grows **DOWN**
Why? It's a "Clashing Heads" design. They share the empty space in the middle. We only run out of memory (Stack Overflow) when the stack grows down so far that it crashes into the heap growing up. This was done in order to maximize memory, if the stack and the heap both started from the bottom and grew upwards, the stack could run out of space when the heap still has free space.

### 4. THE REBEL VARIABLE: STATIC
Now, thing gets interesting. We established that variables inside a function live on the stack and die when the function returns. But .... what if we do this?

```c
void my_function() {
	static int x = 10;
	x++;
}
```

By adding `static`, we are essentially breaking the rules. 
1. **Lifetime:** `x` is **NOT** stored on the Stack. It is stored in the **Data Segment** (a permanent storage area). It is born when the program starts and only dies when the program ends.
2. **Scope:** Even though it lives forever, only `my_function` is allowed to see/use it.

So if you call `my_function()` three times, `x` will go 11, 12, 13. It remembers! It's effectively a global variable that is hiding inside a function so no one else can mess with it.

### C vs. PYTHON
You might ask, "Does Python this do too?"

Well...In Python? Python is basically your mommy. It doesn't tell you how things work, it just does everything for you, effectively turning you into a big mommy's boy(I was a mommy's boy once, until i saw the light). Back to the question...The answer is yes and no.

In C, `int x = 10` inside a function puts the raw number `10` right there on the Stack. Fast. Efficient.

In Python, `x = 10` does this:
1. Goes to the **Heap,** creates a massive "Integer Object" (which is like a C struct).
2. Goes to the **Stack,** and puts a **Reference(a pointer)** to that Heap object.
So while C uses the Stack for the actual data (speed!), Python mostly uses the Stack just to hold pointers to objects living on the Heap(essentially an illusion that we're using the stack when we're actually using the heap).

#### Why do we care? (The "Hacker" Angle)

It boils down to **Control**.

Because the Stack contains the **Return Address** (the "where do I go next?" instruction) right next to your local variables, if you are careless and write too much data into a variable (Buffer Overflow), you can accidentally overwrite the Return Address.

If you overwrite it with a specific address, you can trick the CPU into jumping to _your_ code instead of back to the main program. That's how we "bend the system without breaking it."


## RECURSION AND TAIL CALL OPTIMIZATION
As we know every function call pushes a stack frame unto the stack and if we recurse 10,000 times, you need 10,000 stack frames and a **Stack overflow**; where the stack and heap clash into each other.
**Tail call Optimization** prevents this stack overflow. It allows a function to be called(in this case recursively) 10,000,00 times if it wants to without using extra stack memory.

Let's check it out

---
**---

### 1. The Problem: "Standard" Recursion**
Let's check a standard factorial recursive function
```c
int factorial(int n){
	if (n == 0) return 1;
	return n * factorial(n-1);
}
```
The last line. The function says: "Call `factorial(n-1)`", wait for it to return , and then multiply that result by `n`.
Because it has to do some math after the helper return, it must keep the current stack frame alive to remember what n is.
- `factorial(5)` waits for `factorial(4)`
- `factorial(4)` waits for `factorial(3)`
- .... and so on.
Now there's a bunch of stack frames on the stack

---
### The Solution: "Tail" Recursion
Now, let's check this version. We use an accumulator(`acc`) to pass the result forward, so that we don't have to do any math - in other words, the previous function call, doesn't have to wait for the next, so it's frame is popped and we can just reuse it's frame for the new function call, effectively preventing a stack overflow.

```c
int factorial(int n, int acc){
	if (n == 0) return 1;
	
	return factorial(n-1, n*acc)
}
```
So the function doesn't have to wait for it's next call like in standard recursion, it just passes n -1 and the updated accumulator to the next function call. So since the current function(current stack) has nothing to do, the compiler decides that there's no need for a new stack frame, when we can just pop this one and reuse it, effectively running on **O(1)** stack space unlike standard recursion that uses **O(N)** linear space.
So when we say **tail call optimization** comes in when you turn on optimization flags (like `-02` or `03`), the compiler would almost always detect tail recursion and optimize it, it's basically free speed.
Now **Python** to my knowledge doesn't support **TCO** (imagine, python lovers..damnnn) unlike **C and C++.**

 