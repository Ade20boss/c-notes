
---
# The Do...While Loop 🔄

The do...while loop is a control flow statement known as an Exit Controlled Loop.

Unlike standard while or for loops (which check the condition before entering), the do...while loop executes the code block first and checks the condition afterwards.

**Key Rule:** The code inside the loop is guaranteed to run **at least once**, even if the condition is false from the start.

### 1. The Syntax

Notice the semicolon `;` at the very end. This is mandatory and often forgotten.

```C
do {
    // Body of the loop
    // This code runs immediately
} while (condition); // <--- Note the semicolon!
```

### 2. The Logic Flow

1. **Execute:** The program enters the `do` block and runs the code.
    
2. **Check:** It reaches the `while(condition)`.
    
3. **Decision:**
    
    - If **True**: It jumps back up to the `do` and repeats.
        
    - If **False**: It exits the loop and continues down the program.
        

### 3. Comparison: `while` vs `do...while`

|**Feature**|**while Loop**|**do...while Loop**|
|---|---|---|
|**Type**|Entry Controlled|Exit Controlled|
|**Execution**|0 or more times|1 or more times|
|**Logic**|"Check ID, then enter party"|"Enter party, check ID at exit"|
|**Use Case**|Reading files, counters|User Input, Menus, "Try Again" logic|

### 4. Practical Use Case: Input Validation 🛡️

The most common use for `do...while` is forcing a user to enter valid data. Since you must ask the user _at least once_, this loop is perfect.

Example: The "Stubborn Gatekeeper"

This program refuses to proceed until the user enters a specific number (e.g., a PIN or menu option).

```C

#include <stdio.h>

void my_scanf(int *ptr){
	int total = 0;
	int c = getchar();
	
	while (c == ' ' || c == '\n' || c == '\t'){
		c = getchar();
	}
	
	while(c >= '0' && c<= '9'){
		c = c - '0';
		total = (total * 10) + c
		c = getchar();
	}
	*ptr = total;
	
	void input_validator(const int PIN){
		int entry;
		do{
			printf("Enter pin: );
			my_scanf(&entry);
			if(entry == PIN){
				printf("PIN MATCH, ACCESS GRANTED\n");
			}
			else{
				printf("INCORRECT PIN, ACCESS DENIED\n")
			}
		} while(entry != PIN);
	}
	
}

int main(){
	input_validator(100345);
}

```

### 5. Summary

- **Guaranteed Execution:** Use it when the action _must_ happen at least once (like rendering a game menu or asking for a password).
    
- **The Semicolon:** Always remember the `;` after the `while(condition)`.
    
- **Logic Reversal:** The condition defines when to **continue**, not when to stop. (e.g., `while(number <= 0)` means "continue looping if the number is bad").