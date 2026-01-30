If a **Struct** is a house where every member gets their own private room, a **Union** is a single room that everyone has to share...one at a time.

Syntactically, they look identical to structs. **Functionally, they are complete opposites.**

## THE CORE CONCEPT
Imagine you have a `Motorcycle` and a `Truck`.
- **Struct:** You buy a parking lot with two spaces. Both vehicles can park there at the same time.
- **Union:** You buy a single parking spot big enough for the Truck. You can park the Truck **or** the Motorcycle.

**The Catch:** If you park the Truck while the Motorcycle is still there, the Motorcycle gets crushed.

## UNDER THE HOOD
C likes efficiency cause with C, you're dealing directly with memory and we want to optimize memory. Sometimes, you have a variable that needs to be an `int` sometimes, and a  `float`other times, but never **but never both at once.** Why waste RAM allocating space for both?

1. **The Struct Way (Rich & Wasteful)**
```c
struct MyStruct {
    int a;    // 4 bytes
    double b; // 8 bytes
};
// Total Size: 12 bytes. 
// Both 'a' and 'b' live happily side-by-side.
```


**2. The Union Way (Efficient & Dangerous)**
```c
union MyUnion {
    int a;    // 4 bytes
    double b; // 8 bytes
};
// Total Size: 8 bytes (Size of the largest member only!)
// 'a' and 'b' SHARE the same physical memory address.
```

The Union looks at its members, finds the "fattest" one(the `double` in this case), and allocates just enough memory for that. Everyone else overlaps in that same space.

**3. The "Overwrite" Trap (The Chaos)**
This is where shit gets spicy. Because these member share the same physical RAM address, changing one corrupts the other, essentially, once we define a member of the union, we have to use it immediately before defining the second one because if we define the second member, we have effectively overwritten the first member, meaning we can't use both(or all or some depending on the number of members) members at the same time at any one moment.

```c
union Data {
	int i;
	float f;
};

int main(){
	union Data data;
	data.i = 10;
	//Memory: [0000...1010] (Looks like 10)
	
	data.f = 220.5;
	//Memory: [0100...1101] (Looks like 220.5 in float format)
	//The 10 is GONE. Vaporized.
	
	//If we try to read 'i' now...
	printf("%d", data.i);
	//Output: 1129532621 (Absolute Garbage)
	
	return 0;
}
```

This happens because doesn't know who is currently using the memory. It just sees raw bits. If you write floats and try to read them as int bits, you get rubbish.

**The Solution: The "Tagged Union"**
So, how do we remember which member is currently alive? We have to create a manual memory system. We combine the **Union** with and **Enum** inside a **Struct**
This is called a **Tagged Union**

```c
typedef enum {
	IS_INT,
	IS_FLOAT;
} Tag;

typedef struct {
	Tag type;
	union {
		int i;
		float f;
	} value;
} SmartPacket;

void print_packet(SmartPacket *p){
	if (p -> type == IS_INT){
		printf("%d\n", p -> value.i);
	}
	else if (p -> type == IS_FLOAT){
		printf("%f\n", p -> value.f);
	}
}
```

#### Summary

1. **Unions** save memory by stacking variables on top of each other.
    
2. **The Risk:** You can only use **one member at a time**. Writing to one destroys the others.
    
3. **The Fix:** Always wrap your Union in a Struct with an Enum ("Tag") to keep track of what is actually stored inside. Without the tag, you are just guessing at bits.

**NOTE THAT THE SIZE OF THE UNION IS ALWAYS EQUAL TO THE MEMBER WITH THE LARGEST BYTES ALLOCATED TO IT.**