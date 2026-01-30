
---

### Level 1: The Architect (Drawing the Map)

**Scenario:** You are debugging a crash in your matching engine. You need to visualize exactly what the memory looks like at a specific microsecond.

**The Code:**


```c
void execute_trade(int quantity, int price) {
    char side = 'B';     // 'B' for Buy
    int trade_id = 99;   // Local ID
    // CRASH HAPPENS HERE
}

int main() {
    execute_trade(10, 500);
    return 0;
}
```

**The Task:**

Draw the **Stack Frame** for `execute_trade` at the moment the crash happens.

1. Where is the **Return Address**?
    
2. Where are `quantity` and `price`?
    
3. Where are `side` and `trade_id`?
    
4. If the stack grows **DOWN** (from high address to low address), which variable has the **lowest** memory address?


---

### Level 2: The Security Audit (The "Dangling" Trap)

**Scenario:** You are reviewing code from a junior developer who is trying to optimize the engine to avoid copying data. They wrote this function.

**The Code:**

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

**The Task:**

Explain to the junior developer **exactly why** this code fails using the terms: **Stack Frame**, **Lifetime**, and **Stack Pointer**.

- _Why is `p` pointing to a ghost?_
    

---

### Level 3: The Python "Bureaucracy" Check

**Scenario:** Your AI engineer writes this Python script for the trading bot. He thinks it’s efficient because it’s "just one line."

**The Code:**

Python

```
prices = [10, 20, 30]
# He appends a new price
prices.append(40) 
```

**The Task:**

Translate that single `.append(40)` line into the **actual work** the C-Program (Python VM) has to do.

- List at least **3 distinct steps** dealing with the **Heap** and **Pointers** that happen behind the scenes.
    
- _Bonus:_ If `prices` was full (capacity reached), what massive operation just triggered?
    







**ANSWERS**

for level 1, The return address, quantity, price, side and trade_id, all live in the stack frame of the execute_trade function. The lowest address would be that of trade_id

for level 2, The reason why p is pointing to a ghost is because the get_price_pointer returns an address to a variable that was created in the function; the variable created in the function is stored in the stack, and as we know the variable created in the stack dies immediately the function dies - meaning the lifetime of the variable depends on the lifetime of the function, so the function actually return an address but it's now garbage, it returns gabbage

for level 3, when we create a list of integers in python, first things first, python has to check each of the items in the list to identify their data types, since they are integers, python then goes on to create three integer objects on the heap and then the list just stores the addresses(pointers) to the integer objects, Now when we try to append another item to the list, it still has to check the type, now the interesting thing is when we created the list, python doesn't just allocate the exact amount needed by the list, it adds extra space, just incase you want to add another item, so first it checks if the list has reached its limit, if it hasn't them it's an easy process, it just creates another integer object on the heap and passes the address to the list but the problem arises if the list has reached its limit, python has to create another list entirely and copy the data to the new list and adds the new item you wanted to append and then garbage collects the old one(crazy shit)
