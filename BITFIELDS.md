Imagine you want to store a boolean variable: `is_on`. A standard `int` is usually **32 bits(4 bytes).** A simple `1` or `0` only needs **1 bit.**

Writing `int is_on = 1;` is like renting a massive moving van to transport a single apple. It works, but it's 97% wasted space.

#### The Solution: The Bitfield

C allows you to tell the struct **exactly** how many bits to use for a variable.
**Synatx:** You add a colon `:` and a number after the variable definition.
```c
struct Status {
	unsigned int is_wifi_on : 1; //Use only 1 bit (0 or 1)
	unsigned int is_bluetooth_on : 1; //Use only 1 bit
	unsigned int volume_level : 4;
}
```

#### Under the Hood (The Packing)
Without bitfields, that struct would likely take up to **12 bytes**(4 bytes per int). **With bitfields**, the compiler packs all those variables into a **single 4-byte integer**(Or even less, depending on the machine).

- `is_wifi_on` takes bit 0.
- `is_bluetooth_on` takes bit 1.
- `volume_level` takes bits 2 -5.

It creates a "mini-struct" inside a single integer.

**The "Catch" (There's always a catch with C)**
Since these variables don't own a full "byte" of memory (they live inside a byte), **you cannot take their address**

```c
struct Status s;
int *ptr = &s.is_wifi_on; // ERROR
//You cannot point a pointer to a specific BIT, only a BYTE
```

