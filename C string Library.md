The standard C library provides a comprehensive set of functions to manipulate strings in the <string.h> header file. Here are some of the most commonly used ones:

- **strcpy**: Copies a string to another;
```c
char src[] = "Hello";
char dest[6];
strcpy(dest, src);
dest[6] = '\0';
// dest now contains "Hello"
```

- **strncpy**: Copies a specified number of Characters from one string to another.
```c
char src[] = "Hello";
char dest[6];
strncpy(dest, src, 3);
dest[3] = '\0'
//dest now contains "Hel"
```

- **strcat**: Concatenates (appends) one string to another
```c
char dest[12] = "Hello";
char src[] = " World";
strcat(dest, src);
//dest now contains "Hello World"
```

- **strncat**:  Concatenates a specified number of characters from one string to another
```c
char dest[12] = "Hello";
char src[] = " World";
strncat(dest, src, 3);
// dest now contains "Hello Wo"
```

- **strlen**: Returns the length of a string(excluding the null terminator)
```c
char str[] = "Hello";
size_t len = strlen(str);
// len is 5
```

- **strcmp**: Compares two strings lexicographically
```c
char str1[] = "Hello";
char str2[] = "World";
int result = strcmp(str1, str2);
// result is negative since "Hello" < "World"
```

- **strchr**: Finds the first occurrence of a character in a string
```c
char str[] = "Hello";
char *pos = strchr(str, 'l');
// pos points to the first 'l' in "Hello"
```

- **strstr**: Finds the first occurrence of a sub-string in a string
```c
char str[] = "Hello World";
char *pos = strstr(str, "World");
// pos points to "World" in "Hello World"
```
