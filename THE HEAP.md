The heap as opposed to the **"stack"**, is a pool of long-lived memory shared across the entire program. Stack memory is automatically allocated and deallocated when a function is called and dies, but heap memory is allocated and deallocated as needed, independent of function calls, it allows you to **dynamic memory allocation**.
**Dynamic** means I don't know how much memory i need until the program is actually running, the **Lifetime** of things stored in the heap is manual, variables you create here live until you explicitly kill them(or the garbage collector does, which is impossible in C) but the problem with the heap is that it's slow, allocating stack memory is fairly easy, i mean its just moving a pointer(1 instruction).. Allocating heap memory is more of a quest

- **The request:** You ask the heap memory for X bytes
- **The search:** The allocator looks through the heap to find a chunk of memory that is large enough for your need
- **The split:** If it finds a big chunk, it cuts of the slice you need and records the rest as 'still empty'
- **The handover:** It gives you a **Pointer** to that address.
Now heap is slow in the sense that this search is **non-deterministic.** It might take 10 nanoseconds, it might take 200 if memory if fragmented.

## ENEMY: FRAGMENTATION
Since you can allocate and free memory in any order, this turns the heap into a chaos.

Imagine this sequence:
1. Allocate **Block A (10 MB)**
2. Allocate **Block B (10 MB)**
3. Allocate **Block C (10 MB)**
4. **Free Block B**

Now you have a 10 MB hole in the middle. If you try to allocate **20 MB**, you cannot use that space. Even if you have 100 MB of free space total, if it's all tiny 1 MB chunks, you can't fit a 20 MB object. This is **External fragmentation.** You might then ask, if there's external fragmentation, there should be internal fragmentation. **Internal fragmentation** happens when the memory allocator gives you **more memory than you asked for.**
Imagine you order a **usb stick** from amazon, they send it to you in a **shoebox** , you use 5% of the box, 95% is useless
**In systems Terms:**
- **You ask:** `malloc(10 bytes)`
- **The system says:** "I only hand out memory in chunks of 16 bytes for alignment reasons."
- **The waste:** 6 bytes are trapped inside that allocation. They are technically "used" (according to the OS), but they're empty.

As we know, data that was created by a function(in a function) lives as long as the function and we might need to store data that outlives the function or store data that can be used across functions or say a function needs to return data that was created inside the function, This is where the heap comes in (heap - dynamic memory) - allocated and deallocated as needed. 

Let's take a look at this;

```c
int* get_price_pointer() {
    int current_price = 1500; // The price of the stock
    return &current_price;    // Return the address so we can use it later
}

void aggressive_trader() {
    int *p = get_price_pointer();
    printf("The price is: %d\n", *p); // This prints garbage or crashes.
}
```

in the code above, we create a variable `current_price` in the function `get_price_pointer()` and then try to return the address of the `current_price` variable as a pointer, the pointer still holds a valid address(obviously) but remember that the lifetime of the `current_price` variable depends on the lifetime of the `get_price_pointer()` function, immediately the function returns, the variable becomes non-existent(well not exactly, it's still in existence but the computer regards its address as free space, so new function use it's space, so whatever is in that address is overwritten so it becomes garbage for our use case). 

Now say we want a function that creates an array of integers of dynamic size and returns it to the rest of our program, so let's use **MALLLLLLLLLLLLLLLLLLLLLLLLLLOOOOOOOOOCCCCCCCCC!!!!!!!!**

```c
int *new_int_array(int size){
	int *new_arr = (int*)malloc(size * sizeof(int));
	if (new_arr == NULL){
		fprintf(stderr, "Memory alocation failed\n");
		exit(1)
	}
	return new_arr;
}

int main(){
	int *arr_of_6 = new_int_array(6);
	for (int i = 0; i < 6; i++){
		arr_of_6[i] = i;
	}
	for (int i = 0; i < 6; i++){
		printf("%d", arr_of_6[i]);
	}
	free(arr_of_6);
}
```

The **new_int_array()** is function that creates an array of the specified size(which is passed as the argument **size** to the function), it uses **malloc** to dynamically allocate memory needed for the array; Lets explain line by line:

```c
int *new_int_array(int size)
```

I believe this is basic and anybody should understand this, but allow, i'll still explain, so we create a function that returns a pointer, the address held by the pointer stores an integer that's the point of the `int`(we're creating an array - and in c, an array is just the pointer to the first element, another reason we need to return a pointer is because we're allocating memory on the heap, the memory allocator doesn't actually give us the data we stored, it only gives us the address to that data, so either ways, we still need to return a pointer).

```c
int *new_arr = (int*)malloc(size * sizeof(int));
```

This is the most important line in the program, what we're doing here is invoking the memory allocator(malloc), so we create a pointer that points to the heap (specifically the address we want to store in the pointer holds an integer)`int *new_arr`,  then we tell malloc, go to the heap and get a space that is `size * sizeof(int)` large, malloc finds it and gives us the address to the first 4 bytes block, we cast that 4 bytes block as an integer because malloc returns a void pointer (`void*`) that doesn't know what it's pointing to. Yeah and that **literally it, dufus(that's myself guys)**

```c
	if (new_arr == NULL){
		fprintf(stderr, "Memory allocation failed\n");
		exit(1)
	}
```

The only line we might not understand is the `fprintf(stderr, "Memory allocation failed\n)` and it's actually pretty simple, `f` stands for **file** unlike normal printf(which assumes you want to talk to the main screen), `fprintf` asks which file or stream do you want to write to?
`stderr`(Standard Error): This is the destination, it is a special output channel reserved for errors and warning, You might if `stdout(standard output)` and `stderr(standard error)` both go to the screen, why do we need two?

The answer is **Redirection**
If you try to redirect the output of program to a file printf usages gets redirected to the file, something like `printf("Error")`, you wouldn't get any errors on the screen and you'd think everything is fine and deploy it and everything crashes, **you dufus**, you would think that the but say you use `fprintf(stderr, "Error")`, This is forces the error message to the screen, it stays on the screen so you know something is wrong

**There's no need to explain the rest of the code, it's basic C**

