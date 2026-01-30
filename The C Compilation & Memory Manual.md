
When you run `gcc main.c`, we aren't running one program as we think(to just compile), we're actually running some sort of chain reaction of four seperate tools. This pipeline is often abstracted away and called **"BUILD PROCESS"**.

## PHASE 1: THE PREPROCESSOR

- **Input:** The takes in the source code(The .c file)
    
- **Output:** Modified source code(often .i)
    
- **Tool:** `cpp`(C Preprocessor)
    

The preprocessor doesn't know anything about C syntax, it's practically dumb, all it does is look for lines starting with `#` and obeys them:

1. **REMOVAL:** It strips out all of your comments, The computer doesn't need them
    
2. **EXPANSION(#include):** When it sees `#include <stdio.h>`, it goes to your hard drive, finds the file `stdio.h`, and copy-pastes its entire contents into your file.
    
3. **REPLACEMENT(#define):** It finds every instant of your constants and replaces them with the number assigned.
    

The result is a massive, pure C file with no comments and no `#` symbols ready for translation.

## PHASE 2: THE COMPILER

- **Input:** Modified Source Code(.i)
    
- **Output:** Assembly Code(.s)
    
- **Tool:** `ccl`(The actual C compiler)
    

This takes the C code gotten from the preprocessor and translates it into **ASSEMBLY LANUGUAGE**. This does two major things; **SYNTAX CHECKING** and **OPTIMIZATION**(cases of smart compilers(most modern compilers)) - It might rewrite loops to make them faster or delete variables that are never reused.

The result is a `.s` file that contains low-level assembly instructions specific to the machine's processor.

## PHASE 3: THE ASSEMBLER

- **Input:** Assembly Code(.s)
    
- **Output:** Object Code(.o)
    
- **Tool:** `as`(The assembler)
    

This is the phase that actually translates the assembly code into machine readable binary. It transforms things like `"add %eax, $10"` into something like `01001011`. It creates an **OBJECT FILE**(`main.o`) but the thing is this isn't executable yet, there still need to be a final step(Linking), if we try to run this object file, we encounter an error because the assembler doesn't know what "printf" is, it just leaves a blank space(a placeholder) saying: 'fill this in".

## PHASE 4: THE LINKER

- **Input:** Object Files(.o) + Libraries
    
- **Output:** Executable File
    
- **Tools:** `ld`(Linker)
    

What this does, like the name implies is to link:

- **MERGE:** If the program splits across multiple file, the linker stitches their object files together into one big block.
    
- **RESOLVE SYMBOLS:** It sees the placeholder for something like `printf` from PHASE 3, it looks in the Standard C library, finds the actual binary code for `printf`, and connects your function call to that memory address(It's basically saying, when you encounter this, go to this memory address).
    

---

# Constants in C

This can be done using two methods: using the const keyword and using #define.

These two ways create constants but they live in seperate worlds and these worlds different because of the processing level, #define uses the preprocessor while const uses the compiler.

### THE MECHANISM

#define - This happens before the code is even compiled(It's more of a copy-paste tool)

When you write #define MAX 100; the Preprocessor scans your file(the source code). It literally deletes every instance of the word MAX and types 100 in its place. The compiler never sees MAX. It only sees 100, it's basically text REPLACEMENT.

const - This happens during compilation

When you write const int limit = 100;, the compiler creates a real variable in memory. It assigns it the value 100 and then puts some sort of read only flag on that memory address. If you try to change it, the compiler throws an error.

Also constants created using #define are global, they don't care about scope while const variables respect scope.

Another thing to note is that #define constants are type blind, so if you try to perform type-specific operations that the type contained in the constant doesn't support, he compiler throws an error while const variables are type-strict.

---

# THE C STREAM MODEL

> "Text input or output, regardless of where it originates or where it goes to, is dealt with as streams of characters."

This is some sort of abstraction in C, since data comes from different sources, it would be very inefficient to create specific data pipelines for data from each source, so what C does is that it treats data from all sources(files, networ) as a stream of text - This abstraction is what's known as **THE STREAM MODEL**.

C puts a pipe(the stream) between your C program and the messy data sources, the C standard library and the OS forces the messy data into a stream of characters and gives us functions like getchar(), printf() to access this data.

The C program sits at the end of the pipe, it sees say "H", "e", "l", "l", "o", the program doesn't really care if they came out of a file, keyboard or network.

### GETCHAR() and PUTCHAR()

So we're standing at the end of this pipe, `getchar()` is more like a valve, it lets just one drop of water(one character) out of the pipe. You catch it in a variable, say `c = getchar();`, if the pipe is empty(you haven't typed anything or the file isn't loaded), the program waits until a drop arrives.

PUTCHAR():

The getchar works with the input stream while the putchar works with the output stream. You have a drop of water,you drop it into the output pipe, that pipe could lead to the screen or a file.

### THE EOF(END OF FILE) CHARACTER

It's not exactly a character, it's more of a status signal like the one we tell our main function to return, when we use `getchar()` the pipe runs dry eventually, the OS needs a way to say: "Stop asking, there's no more characters". That message is the EOF. The standard C library has `EOF` to be `-1`.

And that's why we can't use the `char` type for the buffer, since `char` type only allocates 8bit of memory which is usually unsigned(that's 0 to 255), because if we use the `char` type, the poor variable gets confused yeah? and basically stores 255 as -1, and if we use something like: `while((c = getchar()) != EOF)`, the loop never stope because 255 does not equal -1. Also since EOF isn't an actual character, we need to send some sort of special signals, usually `ctrl + D` on linux(can't remember the command for windows) - which is the EOF that closes the stream.

---

# STRINGS IN C

### 1. The Core Philosophy: The "Lie" of C Strings

In Python, a string is a smart Object. In C, there is no such thing as a "String" data type.

- **Reality:** A C string is just a contiguous Array of Characters (bytes) in memory.
    
- **The Terminator:** Because arrays in C do not know their own length, every string must end with a specific "Stop Sign" byte: the Null Terminator (`\0`).
    
    - **ASCII Value:** 0
        
    - Without it, functions like `printf` will keep reading past the end of the array, causing garbage output or crashes.
        

### 2. The Two Ways to Declare Strings

This is the most critical distinction in Systems Programming. The syntax looks similar, but the memory behavior is completely different.

**A. The Array (The "Box")**

C

```
char str[] = "Hello";
```

- **Memory Location:** Stack (The "Desk").
    
- **Mechanism:** The OS reads "Hello" from the binary and copies the letters into a new array on the stack.
    
- **Permissions:** Read/Write. You own this copy.
    
- **Mutability:** `str[0] = 'J';` is Valid. (Changes "Hello" to "Jello").
    
- **Pointer Constancy:** You cannot change where `str` points (it is bound to that stack location).
    

**B. The Pointer (The "Address Slip")**

C

```
char *ptr = "Hello";
```

- **Memory Location:** Text Segment / Read-Only Data (The "Museum").
    
- **Mechanism:** The string "Hello" is stored once in protected memory. `ptr` just holds the address of that protected memory.
    
- **Permissions:** Read-Only.
    
- **Mutability:** `ptr[0] = 'J';` causes a Segmentation Fault (Crash). You are trying to write to protected memory.
    
- **Pointer Mutability:** `ptr = "World";` is Valid. You are just changing the address written on the slip of paper.
    

### 3. Memory Layout Visualization

|**Feature**|**char str[]**|**char *ptr**|
|---|---|---|
|**Analogy**|A Photocopy (Your own copy)|A Library Book (Shared, view-only)|
|**Size (sizeof)**|Size of string + 1 (e.g., 6 bytes)|Size of address (8 bytes on 64-bit)|
|**Editing Data**|Safe (`str[0] = 'X'`)|Crash (`ptr[0] = 'X'`)|
|**Speed**|Slower (Requires copy time)|Faster (Instant pointer assignment)|
|**Use Case**|Inputs, Buffers, String Building|Constants, Error Messages, Labels|

### 4. Iteration: Indexing vs. Pointer Arithmetic

We can access data in the string in two ways.

Method 1: Indexing (The "Python" Way)

Using an integer counter to calculate the offset from the start.

```C
int i = 0;
while (str[i] != '\0') {
    printf("%c", str[i]);
    i++;
}
```

Method 2: Pointer Arithmetic (The "Systems" Way)

Using the pointer itself as a moving cursor. This is often faster and preferred in low-level code.

```C
char *s = str; // Point to the start
while (*s != '\0') {
    printf("%c", *s); // 1. Read the value at the current address
    s++;              // 2. Increment address by 1 byte (move right)
}
```

### 5. Key Takeaways for Systems Engineering

- **Efficiency:** Using `char *` avoids copying data unnecessarily, which is why C is fast.
    
- **Safety:** You must always ensure strings are null-terminated (`\0`), or you risk reading restricted memory.
    
- **Memory Awareness:** You must always know if your pointer points to the Stack (writable) or Text Segment (read-only) before trying to modify it.
    

### SOMETHINGS TO NOTE ABOUT CHAR * AND CHAR[]:

When we create a string using char *, the computer allocates read-only memory for the string literal, so the string literal sits in memory and then the variable is effectively a pointer(it contains the memory address where the string literal stays) and since it's read only it cannot be Modified.

But the thing is we can update the pointer to contain another memory address; this is what i referred to as pointer mutability above, so the thing is when we do this, the old string literal still stays in read-only memory but we can't access it anymore since we effectively erased it's address.

When we create a string variable using `char[]`, what happens is the computer allocates read-only memory for the string literal, but when the function runs the computer allocates stack memory to the function and then copies the string literal into stack memory, because it's in stack memory, we can modify it, but note that we're only modifying stack memory, the string literal still stays as it is in the memory(read-only data segment), so whenever the function runs, the string is still initialized as what it was originally and not what it was modified to.

---

# FUNCTION RETURN TYPE

When the C program runs, it launches a process by the OS and when it finishes, the OS requires some sort of report.

int main (Standard):

Your program runs and finishes, it returns an integer(usually 0) to the OS to indicate whether the operation was successful.

void main (Non standard):

The program runs and finishes, it returns nothing, the OS doesn't know if it worked or failed, in some systems, this could be interpreted as a crash or error, causing programs to fail.

### FUNCTION ARGUMENTS

- `int main(void)`: This tells C that the function takes strictly zero ARGUMENTS
    
- `int main()`: This function takes an unspecified number of arguments, it technic1ally allows something like: `main(10, "hello", 5.5)` which is dangerous.
    

So verdict is, when we want a function(apart from main) that doesn't return an argument:

void(main){some code}

We can't use this for the main function, since the OS directly interacts with it, so the verdict is:


```C
int main(void){
    return 0;
}
```

---

# THE SIZEOF OPERATOR

The sizeof operator is a COMPILE-TIME UNARY OPERATOR, this means that it is not a function even though we use parantheses like sizeof(int), it does not run code.

It runs at compile time, meaning that when the compiler converts the c code to assembly, the compiler replaces sizeof(...) with a hardcoded number(what the current computer says the size is) and converts it to assembly, so even before the code gets executed by the cpu, the sizeof(...) has already been evaluated.

It's called a unary operator because it only works on one thing. It returns a size_t(an unsigned long integer) representing the number of bytes.

### WHAT IT DOES

The `sizeof` operator measures the capacity of the container, never the volume the contents, when you statically declare a variable like you do in c, the computer allocates a specific block of RAM, it is that block that the `sizeof` operator measures and returns.

### BEHAVIOURS

There's three main behaviours to how `sizeof` works;

PRIMITIVE TYPES:

It returns the standard size defined by your cpu architecture


```C
sizeof(char); //8bits
sizeof(int);  //32bits(usually)
sizeof(short); //16bits
sizeof(long);  //64bits(usually)
sizeof(float); //32bits(usually)
sizeof(double); //64bits(usually)
```

ARRAYS:

If you apply it to an array, it returns the total space allocated to the array


```C
char name[50] = "Bob";
sizeof(name); //50bytes
```

It does not count the letters "Bob", it does not stop at the null terminator, it counts the 50 slots reserved.

POINTERS(This is the dangerous one):

When you store values in an array and put the memory address of the first value in a pointer, something like:

```C
char *name = "Bob";
```

What the pointer stores as we know is the memory address, so when we use the `sizeof` operator on name, we get 8bits because the `sizeof` operator reports the size of the variable which is a `char`(8bits) and not the amount of space allocated to the array.

---

# 📚 Technical Note: `sizeof` and Struct Padding

### 1. The Expectation vs. The Reality

When you use sizeof on a int, it returns 4. When you use it on a char, it returns 1.

Logic suggests that if you combine them in a struct, the size should be the sum.

The Naive Math:

$$\text{Size} = \text{Size}(\text{Member}_1) + \text{Size}(\text{Member}_2) + \dots$$

The Actual Math:

$$\text{Size} = \sum(\text{Members}) + \text{PADDING}$$

### 2. Why does Padding exist? (Memory Alignment)

The CPU does not read memory byte-by-byte. It reads memory in **"Words"** (chunks of 4 or 8 bytes).

- **The Rule:** A variable of size $N$ bytes must start at a memory address divisible by $N$.
    
    - `int` (4 bytes) -> Must start at address 0, 4, 8, 12...
        
    - `double` (8 bytes) -> Must start at address 0, 8, 16...
        
- **The Penalty:** If data is "misaligned" (e.g., an `int` starting at address 5), the CPU has to perform two reads to fetch one number, slowing down the program significantly.
    
- **The Solution:** The compiler inserts invisible "junk bytes" (Padding) to force alignment.
    

---

### 3. Case Study: The `BankAccount` Struct

Let's trace how the compiler packs this specific struct into memory.

C

```
struct BankAccount {
    char name[50]; // 50 bytes
    int id;        // 4 bytes
    float balance; // 4 bytes
};
```

**Step-by-Step Memory Layout:**

1. **`char name[50]`**
    
    - **Starts:** Address 0.
        
    - **Ends:** Address 49.
        
    - **Current Address:** 50.
        
2. **`int id` (4 bytes)**
    
    - **Requirement:** Must start on a multiple of 4.
        
    - **Check:** Is Address 50 divisible by 4? **No** ($50 / 4 = 12.5$).
        
    - **Action:** Compiler skips Address 50 and 51. These are **Padding Bytes**.
        
    - **New Start:** Address 52 ($52 / 4 = 13$). **Yes.**
        
    - **Ends:** Address 55.
        
3. **`float balance` (4 bytes)**
    
    - **Requirement:** Must start on a multiple of 4.
        
    - **Check:** Is Address 56 divisible by 4? **Yes.**
        
    - **Starts:** Address 56.
        
    - **Ends:** Address 59.
        

**Final Calculation:**

- **Used Data:** $50 + 4 + 4 = 58$ bytes.
    
- **Wasted Space:** 2 bytes.
    
- **Total `sizeof`:** **60 bytes**.
    

---

### 4. Structure Packing "Tetris" (Pro Tip)

The order in which you declare variables matters! You can save memory just by rearranging the lines.

**Bad Ordering (Wastes Space):**

C

```
struct Bad {
    char c;      // 1 byte
    // 3 bytes PADDING
    int i;       // 4 bytes
    char d;      // 1 byte
    // 3 bytes PADDING
}; 
// Total: 12 bytes
```

**Good Ordering (Saves Space):**

C

```
struct Good {
    int i;       // 4 bytes
    char c;      // 1 byte
    char d;      // 1 byte
    // 2 bytes PADDING (to round up total size)
};
// Total: 8 bytes
```

### 5. Summary

- **`sizeof` measures the container**, including the empty "air gaps" (padding).
    
- **Padding is not a bug**; it is a feature for CPU speed.
    
- **Reordering variables** from largest to smallest is a common trick to minimize padding.