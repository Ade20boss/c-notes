
---

### The "Under the Hood" Guide to C Pointers & Structs

Pointers are essentially variables that store addresses of other values. As we know when we create a variable in C, something like:


```c
int x = 5;
```

This tells C to take a block of memory, put the value `5` into it, and slap the name `x` on it (that's what we'll be referring to `5` as). But this block of memory has an actual address on the RAM. Now, that's what pointers store—they store the addresses where a value is stored in memory. They themselves don't actually contain any objective data; they only contain the address to a particular value.

So the syntax for a pointer in C is this:


```c
int *y = &x;
```

This is essentially saying: store the address where the value of `x` is stored, give me another block of memory to store that address, and slap the name `y` on it. The `*` tells C that `y` is a pointer.

You might be wondering, "Why do we need to specify the type for a pointer?" The truth is that the type specified isn't actually for the pointer itself, but for the value stored at the address that the pointer points to. It's saying, "When you get to this address, treat the data there as an integer." (Ohhh, sweet Python, I just write `x = 5` and everything works...)

#### accessing the data (Dereferencing)

We can access the value at the address stored by a pointer using something called **dereferencing**. When we dereference a pointer, we're basically telling C to give us back what is inside that memory address.


```c
int x = 5;
int *y = &x;
int z = *y;
```

In the block of code above, we're telling the computer:

1. Give me 4 bytes of memory, store `0000...0101` (5) inside, and slap the name `x` on it.
    
2. Give me 8 bytes of memory (usually 8 bytes for an address on 64-bit architecture), store the memory address of `x` in there, and note that what's at that address is an `int`. Slap the name `y` on it.
    
3. Lastly, go to the address inside `y`, grab the value you find there, give me _another_ 4 bytes of memory, put that value into those 4 bytes, and slap the name `z` on it.
    

#### Why do we need this? (Did you see that??)

You might ask, "What's the point?" It boils down to two things: **Mutation** and **Speed**.

**1. Mutation (Changing stuff)**

When we create a function in C, the computer creates "stack memory" for that function (essentially the function's scratchpad). If we pass variables normally, C just makes copies.


```c
// The "Copy" Problem
void swap(int a, int b){
    int temp = a;
    a = b; 
    b = temp;
    // This only swaps the COPIES. The real x and y are untouched.
}
```

We wouldn't be able to modify data within a function this way. That's where pointers come in. We pass the _address_, so the function knows exactly where the real data lives and can change it directly.

```c
// The Pointer Solution
void swap(int *a, int *b){
    int temp = *a;
    *a = *b;     // Go to the address of 'a' and change it
    *b = temp;   // Go to the address of 'b' and change it
}
```

**2. Speed (The heavy lifting)**

Say we have a massive struct:

```c
typedef struct Employee {
    char name[50]; 
    short access_lvl; 
    unsigned int id; 
    // Imagine 100 more fields here...
} Employee;
```

If we pass this struct into a function normally, the computer has to copy _every single byte_ of that struct into the function's stack. That takes time. Instead, we just pass the address (8 bytes). The function gets a tiny map to the massive object rather than the object itself. **FASTER YEAH??**

---

### The Next Level: Structs & Relationships

Now, things get interesting when structs start interacting. We have two ways they can relate: **Hierarchy** vs. **Mutual Reference**.

#### 1. Hierarchy (The "Box inside a Box")

This is when one struct fully contains another. Think of a `Car` that comes with a `Wheel` installed. The `Wheel` is physically inside the `Car`'s memory.


```c
struct wheel { int size; };

struct Car {
    char *model;
    struct wheel myWheel; // Embedded directly!
};
```

Since it's physically there, we just drill down using the **Dot (.) Operator**:


```c
struct Car c;
c.myWheel.size = 20; // Open car, open wheel, set size.
```

#### 2. Mutual Structs (The "Infinite Loop" Trap)

But what if a `Person` owns a `Computer`, and that `Computer` is assigned to that `Person`?

If we try to put `Person` inside `Computer` and `Computer` inside `Person`, the compiler creates an infinite Russian nesting doll and crashes (Infinite size!).

**The Fix:** We use pointers again! We don't store the _actual_ Computer; we store the _address_ of the Computer.

We also need a **Forward Declaration** (telling C, "Hey, `struct Computer` exists, trust me, it's defined later").


```c
struct Computer; // "Trust me, this is coming."

struct Person {
    struct Computer *myComp; // Just a pointer (8 bytes). Safe.
};

struct Computer {
    struct Person *owner;    // Just a pointer. Safe.
};
```

#### The Golden Rule of Access

This is where people trip up. How do we access data?

- **Do you have the object itself?** Use the **Dot (`.`)**.
    
    - _Example:_ `myCar.model`
        
- **Do you have a pointer (an address)?** Use the **Arrow (`->`)**.
    
    - _Example:_ `myCarPtr->model`
        

The arrow is basically a shortcut for: "Go to this address `*`, then get the field `.`".

**Putting it all together (The "Bridge" Logic):**


```c
struct wheel;

struct Car{
	char *model;
	int tire;
	struct wheel *Wheel;
};

struct wheel{
	int diameter;
	struct Car *car;
};

int main(){
	struct wheel Mywheel;
	struct Car Mycar;

	// 1. Link them up (Slap the address of one into the pointer of the other)


	Mycar.model = "Toyota";
	Mywheel.diameter = 30;
	Mycar.tire = 4;
	Mywheel.car = &Mycar; 
	Mycar.Wheel = &Mywheel; 

	// 2. Cross the bridge
	// "Start at Mycar, use the 'Wheel' pointer to cross to the wheel struct,
	// then grab the diameter."
	int d = Mycar.Wheel->diameter; 

}
```

So effectively, by doing that, we're creating pointers to each struct _inside_ the other struct that we can use to access data remotely. It's like giving your neighbor a key to your house so they can come in and water your plants (modify your data) without physically living there.