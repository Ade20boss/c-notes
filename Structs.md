
---
# C - Structs (Structures)
**Tags:** #c #programming #memory #data_structures #pointers

## 1. The Concept
If an **Array** is a container for data of the *same* type (like a carton of eggs), a **Struct** is a container for data of *different* types (like a packed lunchbox with an apple, a sandwich, and juice). 

In low-level terms: **A struct allows you to treat a block of memory containing mixed data types as a single unit.**

## 2. The Blueprint (Declaration)
Defining a struct doesn't create a variable; it creates a **template** (a new custom data type).

```c
// The Blueprint
struct User {
    int id;             // 4 bytes
    char initial;       // 1 byte
    unsigned int pin;   // 4 bytes
}; // <--- Don't forget this semicolon!
````

## 3. Creating Variables (Instantiation)

Once the blueprint exists, you can create instances of it.

```C
// Creating the "Object"
struct User admin; 

// Initialization
admin.id = 1;
admin.initial = 'A';
admin.pin = 100345;
```

---

## 4. The Memory Layout & Padding

Struct members are stored **contiguously** in memory (next to each other). However, the compiler often inserts invisible **Padding Bytes** to align data for the CPU.

> [!WARNING] The Sizeof Trap
> 
> You might expect the User struct above to be 9 bytes (4 + 1 + 4).
> 
> Running sizeof(struct User) will likely return 12 bytes.
> 
> Why? The CPU hates reading unaligned memory (addresses not divisible by 4). It adds 3 bytes of "padding" after the char to make the next int line up perfectly.

Plaintext

```
[ ID (4) ] [ Initial (1) ] [ PAD (3) ] [ PIN (4) ]
```

---

## 5. Accessing Members: Dot (.) vs Arrow (->)

This is the most critical distinction in C Structs.

### The Dot Operator `.`

Use this when you have the **actual struct variable** (the object itself).



```c
struct User myUser;
myUser.pin = 1234;  // Direct access
```

### The Arrow Operator `->`

Use this when you have a **pointer** to the struct (the address).


```c
struct User *ptr = &myUser;
ptr->pin = 1234;    // "Go to address, THEN access pin"
```

The reason why we would use a pointer to a struct is to be able to modify it's actual content from another function, as we know, when a function runs, the CPU allocates stack memory for that function that initialises all it's dependencies including structs, if we try to modify that struct from another function, we would only be modifying a copy in that function's stack, so to actually modify the struct, we use the pointer to that struct.

> [!TIP] Mnemonic
> 
> If you are pointing to the data, use the Arrow (->).

---

## 6. Why Pass by Pointer?

We almost always pass structs to functions by pointer (`struct User *u`) rather than by value.

1. **Performance:** Structs can be huge. Passing by value copies _every single byte_ to a new stack frame. Passing by pointer only copies the address (8 bytes).
    
2. **Modifiability:** Passing by pointer allows the function to modify the original data.
    

## 7. Example Code


```c
#include <stdio.h>

struct User {
    int id;
    char name[50];
    unsigned int pin;
};

// Function accepts a POINTER (read-only via const)
void check_user(const struct User *u) {
    // Uses Arrow because 'u' is a pointer
    printf("Checking User: %s (ID: %d)\n", u->name, u->id);
}

int main() {
    struct User admin = {1, "Adeoluwa", 100345};
    
    // Pass the ADDRESS of admin
    check_user(&admin);
    
    return 0;
}

```

## 8. Structs vs. Classes (The Python/JS Analogy)
If you are coming from **Python** or **JavaScript**, you can think of a Struct as a **primitive Class** or a **Object Literal**, but stripped of all the "magic."

* **Python:** `class User:` or `dataclass`
* **JavaScript:** `{ name: "Ade", id: 1 }`
* **C Struct:** `struct User { ... }`

**The Key Difference:**
C Structs are **Data-Only**. They are "Plain Old Data" (POD).
* ❌ **No Methods:** You cannot define functions *inside* a struct (e.g., no `user.login()`). You must write a separate function and pass the struct to it (e.g., `login_user(&user)`).
* ❌ **No Inheritance:** A struct cannot "extend" another struct.
* ❌ **No Polymorphism:** A struct is exactly what it says it is.
* ✅ **Just Grouping:** They strictly group variables together in memory.

## 9. The "Grouping" Power (Why use them?)
Imagine coding a 3D game. You need to track a player's location.

**Without Structs (The "Drag"):**
You have to manage every coordinate as a loose variable. Functions become nightmares with huge argument lists.
```c
// messy and hard to manage
void move_player(int x, int y, int z, int speed, int direction) { ... }
```

**With Structs:** You group the related data into a logical unit.
```c
struct Vector3 {
    int x;
    int y;
    int z;
};

// Clean, semantic, and easy to pass around
void move_player(struct Vector3 *position) { ... }
```



## The `typedef` Keyword 🏷️

`typedef` (Type Definition) creates an **alias** (nickname) for an existing data type. In the context of Structs, it removes the need to type the keyword `struct` every time you declare a variable.

### Syntax


```C
// typedef [Existing Type] [New Nickname];
typedef unsigned int uint; // Example: Now you can type 'uint' instead of 'unsigned int'
```

### Best Practice with Structs

Combine the struct definition and the typedef into one step. This promotes the struct to look like a built-in type (like `int` or `float`).

**Code Comparison:**

|**Without typedef**|**With typedef**|
|---|---|
|**Definition:**<br><br>  <br><br>`struct Player { ... };`|**Definition:**<br><br>  <br><br>`typedef struct { ... } Player;`|
|**Declaration:**<br><br>  <br><br>`struct Player p1;`|**Declaration:**<br><br>  <br><br>`Player p1;`|

### Example


```C
// Define the struct and the type name 'Employee' at the same time
typedef struct {
    char name[50];
    int id;
} Employee; 

int main() {
    Employee boss; // Clean, readable, abstract.
}
```