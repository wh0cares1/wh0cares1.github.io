---
layout: single
title: "Pointer from Memory Allocation Perspective"
classes: wide
toc: true
---

# Memory
## Basic Memory
- Computer memory is a linear sequence of bytes.
- Each byte in memory has a unique address.
- The address can be represented in decimal or hexadecimal values.
- Basic data types like character, integer, float, and double use different amounts of [memory](https://learn.microsoft.com/en-us/cpp/cpp/data-type-ranges?view=msvc-170).

<img src="/images/Pasted image 20230511191759.png">

## Pointer
Pointer refers to memory addresses, using that reference we access the value stored in that memory.

<img src="/images/Pasted image 20230510233656.png">

Use the "`*`" symbol before the variable name in C to declare a pointer. To define a pointer to an integer variable, for example :
```c
int *Pointer; // variable Pointer stores the address of some variable with type of an integer
```

To initialize a pointer, use the "&" operator to assign it the address of a variable. For example, to refer a pointer to an integer variable called "Value", you might type :
```c
int Value = 12; 
int *Pointer = &Value; // variable Pointer stores the address of variable Value
```

The "`*`" operator can be used to obtain the value of a variable that a pointer points to. To print the value of the integer variable that "Pointer" points to, for example :
```c
printf("%d", *Pointer); // This give the output of an integer value that is stored in the variable Value.
```

Generally, reading `*` as `the thing pointed to by` is a good way to view `*` in C.
```c
int numb; // numb is an int
int func(); // func() is an int
int *numb; // *numb is an int
int *func(); // *func() is an int
```

### Call by value and call by reference
If we use call by value then any change made in the called function will not reflect in the calling function. Below is an illustration of call by value.

<img src="/images/Pasted image 20230512152642.png">

If we send the argument via reference, all changes performed in the called method will be reflected in the calling function as well. Below is an illustration of call by reference.

<img src="/images/Pasted image 20230512113408.png">

We have two functions : `funct1` and `funct2`. And we have a cup in `funct1`. In call by value, we only pass a copy of this cup to `funct2`. So any change made to this copy in `funct2` will not reflect in cup in `funct1`. But in call by reference we are passing a reference and giving access to the original cup. So any change made to the cup will be reflected in `funct1`.

It's vital to understand that "pass by reference" in C++ is not the same as "pass by sharing" or "pass by value" in many other languages. A copy of the value is created and provided to the function in "pass by sharing," but "pass by reference" allows direct access to the original variable. A lot of beginners were confused by this. We use C in this blog, thus pointers, like everything else in C, are passed by value.

### Pointer Arithmetic
It is dependent on the data type. For example :
```c
int value = 100;
int *pointer = &value; // 10000
pointer++; // 10004
pointer--; // 10000
```

Assume the `value`'s address is 10000. Then, after incrementing, it points to address 10004 since the data type integer has a 4 bit size. If we decrease, it returns to address 10000.

<img src="/images/Pasted image 20230512100325.png">

<img src="/images/Pasted image 20230512100527.png">

Pointer arithmetic is well-defined in C and C++ for both `pointer + integer` and `integer + pointer` expressions. As long as the types are suitable, the result is the same regardless of the order of the operands.
```c
int numb = 10;
int *pointer = &numb;
int *offset1 = pointer + 3; // Advances the pointer by 3 * sizeof(int) bytes
int *offset2 = 3 + pointer; // Same as above, order of operands doesn't matter
```
The type of operands important in pointer arithmetic because it defines the scaling factor used during the arithmetic operation. It guarantees that the pointer is accurately changed by the required amount of bytes dependent on the size of the underlying type.

### Void Pointer
A void pointer is a general-purpose pointer that can be used to store the address of any data type variable. It is not associated with a specific data type and can be used to point to variables of any type. An integer pointer cannot point to a variable of another data type, but a void pointer can point to any variable. 

```c
int numb = 100;
char charc = 'a';
void *pointer;
pointer = &numb;
printf("Value of numb is %d\n", *pointer); // Error
pointer = &charc;
printf("Value of charc is %c\n", *pointer); // Error
```

The main problem is that the compiler doesn't know how many bytes to read when referencing it. The void pointer must be typecast into the appropriate datatype.

```c
int numb = 100;
char charc = 'a';
void *pointer;
pointer = &numb;
printf("Value of numb is %d\n", *(int*)pointer); // Appropriate datatype
pointer = &charc;
printf("Value of charc is %c\n", *(char*)pointer); // Appropriate datatype
```

We can't execute void pointer arithmetic in standard C, such as Microsoft Visual C, since the size of the void is unknown and would produce an error. Because the size of the void is one, we may do void pointer arithmetic on GCC. However, void pointer arithmetic may be performed on standard C by typecasting it into the needed data type.

```c
int number = 100;
void *pointer = &number;
printf("pointer = %p\n", pointer);
printf("pointer+1 = %p\n", (int*)pointer+1);
```

#### Application in malloc() and calloc()
Instead of having a variety of data types for `malloc` and `calloc`, we may write a single generic malloc method that returns `void*`. It must be typecast to the required type. However, typecasting is done directly by the compiler in C. This is how we can utilize a pointer to build a generic function or program that can handle any data type. 

```c
int *ptrInt = malloc(sizeof(int));
char *ptrChar = malloc(sizeof(char));
float *ptrFloat = malloc(sizeof(float));
```

### Double Pointer
In C, when a pointer is supplied as an argument to a function, the function parameter becomes an alias for the original pointer. Modifying the parameter within the method has an impact on the original pointer outside the function, independent of where it is (local variable, array index, object property, etc.).
```c
void func(char **pointer) {
  *pointer = NULL; // Modifying *pointer will also modify the original pointer passed in
}
char *a = "a";
func(&a); // After the function call, a will be NULL
```

Utilizing a pointer to a pointer allows us to directly edit a pointer variable rather than returning a modified value or utilizing a global variable. It allows you to modify the value of a pointer in the calling function.

### Pointer to Structure
Structures (also known as structs) are a means to gather together multiple related variables in one location. Each variable in the structure is referred to as a structure member. A structure, unlike an array, may hold a wide range of data types (int, float, char, and so on).

```c
struct people{
	char name[5];
	int age;
};

struct people person1 = {"John", 24};
struct people *pointer = &person1;
printf("Name : %s\n", (*pointer).name);
printf("Age : %d\n", (*pointer).age);
```

This is how structs looks in memory.

<img src="/images/struct.png">

We create a structure pointer with variable `pointer` and assign the address of `person1` which is 10024 to it. Then access structure members using pointers. Here, `pointer` is pointing to structure variable `person1`. So dereferencing `pointer` variable like `*pointer` is functionally equivalent to structure variable `person1`. To access the member we have to use the member access operator dot(.) so we can use `*pointer.structure_member`. But since dot operator has higher precedence over the indirection operator `*`, we have to use parenthesis around `*pointer`. That is  `*pointer` in parentheses dot structure member.

The previous method was confusing, so here comes the arrow pointer :

```c
printf("Name : %s\n", pointer->name);
printf("Age : %d\n", pointer->age);
```

### Pointer to Function
Like every variable, function also has a unique memory address. Here is the syntax for pointer to function.
```c
return_type (*pointer_name) (parameters);
```

This is an example of using a pointer to a function.
```c
void hello(){
	printf("HELLO!\n");
}
int main(){
	void (*pvoid)(); //Create pointer to function
	pvoid = hello; //Assign function to pointer
	(*pvoid)(); //Dereference
	return 0;
}
```
This is pointer to function looks in memory.

<img src="/images/pointer-to-function.png">

A better name convention by using typedef. Here is the syntax :
```c
typedef return_type (*pointer_name) (parameters);
```

For example :
```c
typedef int (*FP)(int,int);
int add(int a, int b){
	return a+b;
}
int main(){
	int answer;
	pfunc = add; //Assign the address of function add to pointer pfunc
	answer = (*pfunc)(23,32); //Assign the answer to variable answer
	printf("23+32=%d\n", answer);
	return 0;
}
```

## Memory Allocation
### Static (Stack)
The data structure for the stack is LIFO (last-in-first-out). A stack is an abstract data type in computer science that acts as a collection of objects and has two main operations:

Push adds an element to the collection; pop takes away the most recent element that hasn't already been removed from the collection.

<img src="/images/Pasted image 20230511134023.png">

Local variables, details of function calls, and other information pertaining to function execution are kept in memory in a location known as the stack. It grows or shrinks when functions are called and returned. The stack is usually of restricted size and has a fixed memory allocation pattern.

### Dynamic (Heap)
The heap provides runtime memory allocation and deallocation using operations like as malloc(), calloc(), realloc() and free() in [stdlib.h](https://www.ibm.com/docs/en/zos/2.3.0?topic=files-stdlibh-standard-library-functions). It does not follow a preset allocation pattern and is often bigger than the stack because the memory size is dynamically allocated at run-time.

<img src="/images/Pasted image 20230511135418.png">

### Corelation of the Stack and Heap
```c
int *pointer; // pointer to integer
pointer = malloc(5 * sizeof(int)); // Allocate heap memory with size 5 * 4 bytes
// Assuming the address stored in variable pointer is 10000
*(pointer+0) = 1; // *(10000) = 1
*(pointer+1) = 2; // *(10004) = 2
*(pointer+2) = 3; // *(10008) = 3
*(pointer+3) = 4; // *(10012) = 4
*(pointer+4) = 5; // *(10016) = 5
free(pointer); // If the heap are not freed, they cause memory leaks.
```

When you allocate memory on the heap with methods like malloc or calloc in C programming, although memory is allocated on the heap, the pointer variable that carries the address of the allocated memory is saved on the stack. This lets you to indirectly access and control heap memory via the pointer variable. See the illustration below.

<img src="/images/Pasted image 20230511140620.png">

# Applications
## Data Structures
### Array
An array is a collection of variables of the same data type. In C programming language array and pointers aren't the same. Arrays are a contiguous block of memory that stores multiple elements of the same type, while pointers are variables that store memory addresses. For example :

```c
int array[5]={1, 2, 3, 4, 5};
```

Here is an illustration of how arrays are stored in memory. 

<img src="/images/array.png">

Variable array will point to the first element in the array, 1. Hence variable array will have the address of `array[0]` so `array` equal to `&array[0]`. Printing array items with a pointer is the same as dereference with a pointer.

```c
int array[5]={1, 2, 3, 4, 5};
int i;
for(i=0; i<5; i++){
	printf("array[%d] = %d\n", i, *(array+i));
}
```

For arrays and pointers, the `sizeof` operator acts differently. `sizeof(array)` returns the array's entire size in bytes, whereas `sizeof(pointer)` returns the size of the pointer itself (usually the size of a memory location).
```c
int numb[5];
int *pointer = numb;
printf("%zu\n", sizeof(numb)); // 20 (assuming sizeof(int) = 4 bit)
printf("%zu\n", sizeof(pointer)); // 8 (assuming 64-bit system)
```

When an array is used in an expression, it is automatically converted (or "decays") to a pointer to its first element. This pointer represents the memory address where the array starts. This behavior can lead to some confusion and the misconception that "arrays are pointers". So, `sizeof` used within the function would give you the size of the pointer, not the original array.

The square brackets `[]` are not strictly "array operators" in C and C++, although they are used for array subscripting. `*(a + b)`, where `a` is a pointer and `b` is an index, is equivalent to `a[b]`. This similarity is due to pointer arithmetic characteristics, which state that adding an integer value to a pointer moves it by that many elements. Because addition is commutative, both `a[b]` and `b[a]` are valid and have the same meaning. This gives rise to the amusing example of `0["x"]` which is identical to `*(0 + "x")` or `"x"[0]`.

### Linked List
Linked lists are linear data structures with two sections for each node. The data section and the link to the following node. We can save the relevant information in the data section. It can be any data type, such as int, char, float, or double, and the reference component must be a pointer since it will retain the address of the next node. Below is a guide to building a linked list.

```c
struct node{
	int data;
	struct node *next;
}
struct node *head, *body, *tail;
// Create a list on the heap
head = malloc(sizeof(struct node));
body = malloc(sizeof(struct node));
tail = malloc(sizeof(struct node));
// Give the list a value
head->data = 1;
body->data = 2;
tail->data = 3;
// Create a linked list
head->next = body;
body->next = tail;
tail->next = NULL;
```

This is how linked list looks in memory :

<img src="/images/Pasted image 20230512120538.png">

This is how to print a linked list.

```c
struct node *temp = head; // Create temporary variable pointer

while(temp != NULL){
	printf("%d\n", temp->data);
	temp = temp->next;
}
```

This is how print a linked list looks in memory :

<img src="/images/print-linked-list.png">

1. First create a temporary node `temp` and assign the head node's address. While `temp` is not equal to `NULL` so this loop will print the data which is present in `temp` node which is the head node. So, it prints 1 and move `temp` to the next node.
2. Now `temp` points to 2024 then print the data at address 2024 which is 2. Then move `temp` to the next node.
3. Now `temp` equal to 3024 again check the condition. It is still true. Print the data at address 3024 which is 3. Move `temp` to the next node. 
4. Variable `temp` equal to `NULL` and it fails.

### Binary Tree

<img src="/images/Pasted image 20230512123608.png">

Each node will include the data component as well as two pointers to the left and right. Also, a tree is a non-linear data structure of type hierarchical data structure, and these nodes are related, and the links between these nodes are referred to as tree branches.

Below is a guide on how you can create a node.

```c
struct node{
	int data;
	struct node *left;
	struct node *right;
};
struct node *root;
root = malloc(sizeof(struct node));
root->left = NULL;
root->data = 100;
root->right = NULL;
```
This is how a node looks in memory.

<img src="/images/create-new-node.png">

## File Operations
Files are stored in the ROM of the read only memory. We can efficiently access the files using file pointers. The syntax to create a file pointer is

```c
FILE *file_pointer;
```

Here are some examples of how pointers are commonly used in file operations:
1. Pointers are used in file handling to generate and modify file objects. A file object pointer enables activities like opening, shutting, reading, and writing to the file.
2. Pointers are used to store and retrieve the current location of a file. This permits data to be read or written at precise points inside the file.
3. To manage buffers for efficient reading and writing operations, pointers are utilized. Data can be temporarily held in memory via a reference to a buffer before being written to or retrieved from a file.
4. Pointers are used to read and write data from/to files. Data may be read from or written to specified memory regions using pointers, giving you more control and flexibility when dealing with file data.
5. File pointers are used to keep track of where you are in a file during sequential access. They are automatically incremented or decremented when data is read or written, allowing for quick file content traversal.
6. Pointers are used to manage and notify problems that arise during file operations. Error codes or error messages can be saved in a pointer variable to reflect the status of the file operation.

### Open File
```c
// Open a file
FILE *file_pointer = fopen("file_name.txt", "r");
```
It is important to note that the fopen function will not directly point the file's address. Because to read or write data directly to the secondary memory takes a lot of resources and time. For example if you want to read a single character from file_name.txt it will go through C library and then it will call the operating system. And then the operating system will initiate the driver to read the data.

So, when we open a file with the fopen function, we attach a buffer to it and it loads the entire file contents into the buffer. The buffer will be stored in random access memory and will collect all file structure information. This file structure comprises buffer size, buffer pointer, current buffer position, and other information, and it returns the file structure pointer to the file pointer.

<img src="/images/Pasted image 20230512131903.png">

### Read File
In this example, we use fgetc() for reading char by char. The other function you can use is fgets() for reading string by string. But be careful when reading data from a file because it may cause a vulnerability.
```c
fgetc(FILE *pointer);
fgets(char *strings, int size, FILE *file_pointer);
```

This is an example of how to implement reading data from a file.
```c
FILE *file_pointer = fopen("file_name.txt", "r"); //Mode read only
if(file_pointer != NULL){ //Make sure there is file_name.txt on disk
	char charc = fgetc(file_pointer); //Read current symbol from file_name.txt
	while(charc != EOF){ //Check EOF read/returned by last fgetc() call
		printf("%c", charc); //Output lasts read symbol
		charc = fgets(file_pointer); //Read next symbols from filename.txt
	}
}
fclose(file_pointer); //Remember to close the file after finish
```

This is how read from file_name.txt looks in memory.

<img src="/images/file-operation-read.png">

Function fgetc(file_pointer) will return a single character from the buffer and automatically move the position of the pointer to the next character until EOF. EOF stands for end-of-file, a marker used to indicate the end of a file of data. If we close the file fclose(file_pointer), the buffer associated with the file is removed from the memory.

### Write File
In this example, we use fputc() for writing char by char. The other function you can use is fputs() for writing string by string.
```c
fputc(char charc, FILE *file_pointer);
fputs(char *strings, FILE *file_pointer);
```

This is an example of how to implement writing data to a file.
```c
FILE *file_pointer = fopen("file_name.txt", "w"); //Mode write only
if(file_pointer != NULL){ //Check availability file_name.txt on disk
	fputc('H', file_pointer); //Write 'H' into buffer then move pointer to the next buffer
	fputc('E', file_pointer); //Write 'E' into buffer then move pointer to the next buffer
	fputc('L', file_pointer); //Write 'L' into buffer then move pointer to the next buffer
	fputc('L', file_pointer); //Write 'L' into buffer then move pointer to the next buffer
	fputc('O', file_pointer); //Write 'O' into buffer then move pointer to the next buffer
	fclose(file_pointer); //Automatically assign EOF
}
fclose(file_pointer); //Remember to close the file after finish
```

If the specified file exists, we can write the data to it. But if the specified file name does not exist, it will create a new file. The data will not be written directly to the file, but it will be written into the attached buffer. When we close the file, the whole content will be returned to the file.

## Embedded System
Because of their ability to effectively change memory locations and access hardware registers, pointers are widely utilized in embedded system programming such as:
1. Pointers can be used to allocate and deallocate dynamic memory, manage memory pools, and build data structures such as linked lists, stacks, and queues.
2. Pointers are used to directly access and control peripheral devices and hardware registers by mapping them to specified memory addresses. This enables effective interaction with hardware components.
3. DMA (Direct Memory Access) operations use pointers to allow direct data transfers between peripherals and memory without the need for CPU intervention.
4. Pointers are used to store and alter interrupt service routine (ISR) addresses, allowing for efficient hardware interrupt management and event-driven programming.
5. Pointers make data manipulation more efficient, especially when big data sets or complicated data structures must be handled or altered.

# Key Takeaways
- The stack is used to store the pointer variable, while the pointer is used to manipulate the heap memory itself. 
- Pointers are necessary for managing heap memory and ensuring that allocated memory is correctly tracked, used, and deallocated when no longer required.
- Pointers aren't just numbers, but they are vectors that related to the ideas of affine spaces, translations, and distances.
- Pointers, like everything else in C, are passed by value.

# What's next?
Pointers are complicated. In this article, I only managed to give a small example of a pointer, so beginners wouldn't get confused. If you want to understand pointers better, here are some good articles you may find interesting to read.
- [Pointers Are Complicated, or: What's in a Byte?](https://www.ralfj.de/blog/2018/07/24/pointers-and-bytes.html)
- [Pointers Are Complicated II, or: We need better language specs](https://www.ralfj.de/blog/2020/12/14/provenance.html)
- [Pointers Are Complicated III, or: Pointer-integer casts exposed](https://www.ralfj.de/blog/2022/04/11/provenance-exposed.html)
- [Reconciling High-level Optimizations and Low-level Code in LLVM](https://sf.snu.ac.kr/llvmtwin/files/presentation.pdf)
- [What’s the difference between an integer and a pointer?](https://blog.regehr.org/archives/1621)

# Sources
1. [https://www.youtube.com/watch?list=PLhb7SOmGNUc4EBVjd7x5TiEyOKXt71whE](https://www.youtube.com/watch?v=_x1MmVhLOt4&list=PLhb7SOmGNUc4EBVjd7x5TiEyOKXt71whE&index=1&ab_channel=Log2Base2%C2%AE)
2. [https://www.w3schools.com/c/c_pointers.php](https://www.w3schools.com/c/c_pointers.php#:~:text=A%20pointer%20is%20a%20variable,another%20variable%20as%20its%20value)
3. [https://www.youtube.com/watch?v=h-HBipu_1P0](https://www.youtube.com/watch?v=h-HBipu_1P0&ab_channel=mycodeschool)
4. [https://courses.engr.illinois.edu/cs225/fa2022/resources/stack-heap/](https://courses.engr.illinois.edu/cs225/fa2022/resources/stack-heap/)
5. [https://www.youtube.com/watch?v=X1DcpcgSUXw](https://www.youtube.com/watch?v=X1DcpcgSUXw&ab_channel=mycodeschool)
6. [https://medium.com/@shoheiyokoyama/understanding-memory-layout-4ef452c2e709](https://medium.com/@shoheiyokoyama/understanding-memory-layout-4ef452c2e709)
7. [https://medium.com/fhinkel/confused-about-stack-and-heap-2cf3e6adb771](https://medium.com/fhinkel/confused-about-stack-and-heap-2cf3e6adb771)
8. [https://www.geeksforgeeks.org/void-pointer-c-cpp/](https://www.geeksforgeeks.org/void-pointer-c-cpp/)
9. [https://stackoverflow.com/questions/5672746/what-exactly-is-the-file-keyword-in-c](https://stackoverflow.com/questions/5672746/what-exactly-is-the-file-keyword-in-c)
10. [https://twitter.com/thingskatedid/status/1535708903345750016](https://twitter.com/thingskatedid/status/1535708903345750016)
11. [https://raphlinus.github.io/programming/rust/2018/08/17/undefined-behavior.html](https://raphlinus.github.io/programming/rust/2018/08/17/undefined-behavior.html)
12. [https://stackoverflow.com/questions/17649629/understanding-fgetc-program](https://stackoverflow.com/questions/17649629/understanding-fgetc-program)
13. [https://www.w3schools.com/c/c_structs.php](https://www.w3schools.com/c/c_structs.php)
14. [https://medium.com/@hazemu/tricky-pointer-basics-explained-ba87656c9a9a](https://medium.com/@hazemu/tricky-pointer-basics-explained-ba87656c9a9a)
