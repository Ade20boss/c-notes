We know that computers are obsessed with numbers. They dream in binary. To a computer, "Red" doesn't mean anything. But 0? 1? They love that stuff. 

The problem is, humans(programmers in this case) are terrible at memorising what numbers stand for. If I write this in C:

```c
int state = 1;
```

what does 1 mean/ Is it "Active"? Is it "Paused"? Is it "Self-Destruct"? Who knows. We call these Magic numbers, and they're sort of the enemy of good code.

Enter the **enum**(Enumeration).

An Enum is C's way of letting us slap a human-readable sticker over a boring integer. It's a translator:
- You see: **RED**, **GREEN**, **BLUE**.
- The Computer sees: 0, 1, 2.

The syntax looks like this: 
```c
enum Level {
	LOW, //The computer secretly sees 0
	MEDIUM, //The computer secretly sees 1
	HIGH //The computer secretly sees 2
};
```


## UNDER THE HOOD

You might be thinking, "Oh, is this a special string type?" **Nope**
In C, Enums are just integers in a trench coat. They take up 4 bytes(usually) like a standard **int**.

**Proof**:
```c
enum Level myVar = MEDIUM;
printf("%d", myVar);
//Output: 1
```

We are just tricking the compiler into doing the memorisation for us.



### CUSTOMIZING THE NUMBERS(BEING BOSSY)
By default, C starts counting at 0 and goes up by 1. But we can boss it around if we want specific values.
```c
enum ErrorCode {
	NONE = 0,
	WARNING = 5,
	CRITICAL, //Wait for it
	FATAL = 10
}
```

In the block above:

1. `NONE` is `0`.
    
2. `WARNING` is explicitly set to `5`.
    
3. `CRITICAL`? Since we didn't give it a number, C just looks at the previous one and adds 1. So `CRITICAL` becomes `6`.
    
4. `FATAL` is explicitly `10`.

### WHY DO WE NEED THIS? (SANITY AND CONDITIONALS)
Two reasons: **Readability** and **Conditionals**
Imagine a traffic light system:

**The bad way (Magic numbers): **
```c
#include <stdio.h>
int main(){
	int state;
	if (state == 0){
		printf("Stop\n");
	}
	else if(state == 1){
		printf("Ready\n");
	}
	else{
		printf("Go\n");
	}
	
	return 0;
}

```

**The Enum way**
```c
typedef enum TrafficLight {
	RED,
	YELLOW,
	GREEN
} Switchstate;

int main() {
	Switchstate s = RED;
	if (light == RED){
		printf("Stop\n");
	}
	else if (light == YELLOW){
		printf("Ready\n");
	}
	else if (light == GREEN){
		printf("Go\n");
	}
	return 0;
}

```

#### Summary

- **Enums** are just lists of names that represent integers.
    
- **Under the hood**, the computer is still passing `0`, `1`, `2` around.
    
- **The Human Factor**: It saves us from memorizing what number stands for what state.
    
- **Usage**: Use them whenever you have a fixed set of options (Days of the week, Colors, States, Difficulty Levels).

One last thing, because C is...well, C, it doesn't actually enforce any grouping strictly like modern languages(Java or C#) do.
Since they are just ints deep down, C will happily let you do "Stupid" math that makes no logical sense but make mathematical CPU sense So, while they are **human labels**, the compiler still treats them as **raw numbers**
Also as we might have thought of, when we use the sizeof operator on an enum, it just multiply the size of int(usually 4 bytes but can vary depending on the computer architecture) by the total number of members of the enum, now if you think about it, one or more members or even all could have a size larger than the size of an int, that means C has to allocate more memory for that member that's larger than a regular int of 4bytes, so because of this, there could be some memory padding for cpu alignment
