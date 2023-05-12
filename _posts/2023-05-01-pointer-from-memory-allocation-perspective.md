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
int *Pointer; //variable Pointer stores the address of some variable with type of an integer
```

To initialize a pointer, use the "&" operator to assign it the address of a variable. For example, to refer a pointer to an integer variable called "Value", you might type :
```c
int Value = 12; 
int *Pointer = &Value; //variable Pointer stores the address of variable Value
```

The "`*`" operator can be used to obtain the value of a variable that a pointer points to. To print the value of the integer variable that "Pointer" points to, for example :
```c
printf("%d", *Pointer); //This will give the output of an integer value that is stored in the variable Value.
```

### Call by value and call by reference
If we use call by value then any change made in the called function will not reflect in the calling function. Below is an illustration of call by value.

<img src="/images/Pasted image 20230512152642.png">

If we send the argument via reference, all changes performed in the called method will be reflected in the calling function as well. Below is an illustration of call by reference.

<img src="/images/Pasted image 20230512113408.png">

We have two functions : funct1 and funct2. And we have a cup in funct1. In call by value, we only pass a copy of this cup to funct2. So any change made to this copy in funct2 will not reflect in cup in funct1. But in call by reference we are passing a reference and giving access to the original cup. So any change made to the cup will be reflected in funct1.

### Pointer Arithmetic
It is dependent on the data type. For example :
```c
int value = 100;
int *pointer = &value; //10000
pointer++; //10004
pointer--; //10000
```

Assume the value's address is 10000. Then, after incrementing, it points to address 10004 since the data type integer has a 4 bit size. If we decrease, it returns to address 10000.

<img src="/images/Pasted image 20230512100325.png">

<img src="/images/Pasted image 20230512100527.png">

### Void Pointer
A void pointer is a general-purpose pointer that can be used to store the address of any data type variable. It is not associated with a specific data type and can be used to point to variables of any type. An integer pointer cannot point to a variable of another data type, but a void pointer can point to any variable. 

```c
int numb = 100;
char charc = 'a';
void *pointer;
pointer = &numb;
printf("Value of numb is %d\n", *pointer); //Error
pointer = &charc;
printf("Value of charc is %c\n", *pointer); //Error
```

The main problem is that the compiler doesn't know how many bytes to read when referencing it. The void pointer must be typecast into the appropriate datatype.

```c
int numb = 100;
char charc = 'a';
void *pointer;
pointer = &numb;
printf("Value of numb is %d\n", *(int*)pointer); //appropriate datatype
pointer = &charc;
printf("Value of charc is %c\n", *(char*)pointer); //appropriate datatype
```

We can't execute void pointer arithmetic in standard C, such as Microsoft Visual C, since the size of the void is unknown and would produce an error. Because the size of the void is one, we may do void pointer arithmetic on GCC. However, void pointer arithmetic may be performed on standard C by typecasting it into the needed data type.

```c
int number = 100;
void *pointer = &number;
printf("pointer = %p\n", pointer);
printf("pointer+1 = %p\n", (int*)pointer+1);
```

#### Application in malloc() and calloc()
Instead of having a variety of data types for malloc and calloc, we may write a single generic malloc method that returns void*. It must be typecast to the required type. However, typecasting is done directly by the compiler in C. This is how we can utilize a pointer to build a generic function or program that can handle any data type. 

```c
int *ptrInt = malloc(sizeof(int));
char *ptrChar = malloc(sizeof(char));
float *ptrFloat = malloc(sizeof(float));
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
int *pointer; //pointer to integer
pointer = malloc(5 * sizeof(int)); //Allocate heap memory with size 5 * 4 bytes
//Assume the address stored in variable pointer is 10000
*(pointer+0) = 1; //*(10000) = 1
*(pointer+1) = 2; //*(10004) = 2
*(pointer+2) = 3; //*(10008) = 3
*(pointer+3) = 4; //*(10012) = 4
*(pointer+4) = 5; //*(10016) = 5
free(pointer); //If the heap are not freed, they cause memory leaks.
```

When you allocate memory on the heap with methods like malloc or calloc in C programming, the returned reference to the allocated memory is normally saved in a variable on the stack. Although memory is allocated on the heap, the pointer variable that carries the address of the allocated memory is saved on the stack. This lets you to indirectly access and control heap memory via the pointer variable. See the illustration below.

<img src="/images/Pasted image 20230511140620.png">

```
- The stack is used to store the pointer variable, while the pointer is used to manipulate the heap memory itself. 
- Pointers are necessary for managing heap memory and ensuring that allocated memory is correctly tracked, used, and deallocated when no longer required.
```

# Applications
## Data Structures
### Array
An array is a collection of variables of the same data type. In C programming language Array and pointers are more or less the same. For example :

```c
int array[5]={1, 2, 3, 4, 5};
```

Here is an illustration of how arrays are stored in memory. 

<img src="/images/Pasted image 20230512142938.png">

Variable array will point to the first element in the array 1. Hence variable array will have the address of `arr[0]` so `array` equal to `&arr[0]`. Printing array items with a pointer is the same as dereference with a pointer.

```c
int array[5]={1, 2, 3, 4, 5};
int i;
for(i=0; i<5; i++){
	printf("array[%d] = %d\n", i, *(array+i));
}
```

### Linked List
Linked lists are linear data structures with two sections for each node. The data section and the link to the following node. We can save the relevant information in the data section. It can be any data type, such as int, char, float, or double, and the reference component must be a pointer since it will retain the address of the next node. Below is a guide to building a linked list.

```c
struct node{
	int data;
	struct node *next;
}
struct node *head, *body, *tail;
//Create a list on the heap.
head = malloc(sizeof(struct node));
body = malloc(sizeof(struct node));
tail = malloc(sizeof(struct node));
//Give the list a value.
head->data = 1;
body->data = 2;
tail->data = 3;
//Create linked list.
head->next = body;
body->next = tail;
tail->next = NULL;
```

This is how it looks in memory :

<img src="/images/Pasted image 20230512120538.png">

This is how to print a linked list.

```c
struct node *temp = head;

while(temp != NULL){
	printf("%d\n", temp->data);
	temp = temp->next;
}
```

Start from the head and print the data and repeat till we have node's next equal to null.
1. First create a temporary node temp and assign the head node's address. Now we have a while loop while temp is not equal to null so this loop will continue till temp becomes null. Print the data which is present in temp node which is the head node now so it prints 1 and move temp to the next node.
2. Now temp points to 2024 then print the data at address 2024 which is 2. Then move temp to the next node.
3. Now temp equal to 3024 again check the condition. It is still true. Print the data at address 3024 which is 3. Move temp to the next node. 
4. Variable temp equal to null since temp next equal to null and it fails.

<img src="/images/Pasted image 20230512121405.png">

### Binary Tree

<img src="/images/Pasted image 20230512123608.png">

Each node will include the data component as well as two pointers to the left and right. Also, a tree is a non-linear data structure of type hierarchical data structure, and these nodes are related, and the links between these nodes are referred to as tree branches.

## Embedded System
Because of their ability to effectively change memory locations and access hardware registers, pointers are widely utilized in embedded system programming such as:
1. Pointers can be used to allocate and deallocate dynamic memory, manage memory pools, and build data structures such as linked lists, stacks, and queues.
2. Pointers are used to directly access and control peripheral devices and hardware registers by mapping them to specified memory addresses. This enables effective interaction with hardware components.
3. DMA (Direct Memory Access) operations use pointers to allow direct data transfers between peripherals and memory without the need for CPU intervention.
4. Pointers are used to store and alter interrupt service routine (ISR) addresses, allowing for efficient hardware interrupt management and event-driven programming.
5. Pointers make data manipulation more efficient, especially when big data sets or complicated data structures must be handled or altered.

## File Operations
Files are stored in the ROM of the read only memory. We can efficiently access the files using file pointers. The syntax to create a file pointer is

```c
FILE *file_pointer;
```

### Open File
```c
//Open a file
FILE *file_pointer = fopen("file_name.txt", "r");
```
It is important to note that the fopen function will not directly point the file's address. Because to read or write data directly to the secondary memory takes a lot of resources and time. For example if you want to read a single character from file_name.txt it will go through C library and then it will call the operating system. And then the operating system will initiate the driver to read the data.

So, when we open a file with the fopen function, we attach a buffer to it and it loads the entire file contents into the buffer. The buffer will be stored in random access memory and will collect all file structure information. This file structure comprises buffer size, buffer pointer, current buffer position, and other information, and it returns the file structure pointer to the file pointer.

<img src="/images/Pasted image 20230512131903.png">

Here are some examples of how pointers are commonly used in file operations:
1. Pointers are used in file handling to generate and modify file objects. A file object pointer enables activities like opening, shutting, reading, and writing to the file.
2. Pointers are used to store and retrieve the current location of a file. This permits data to be read or written at precise points inside the file.
3. To manage buffers for efficient reading and writing operations, pointers are utilized. Data can be temporarily held in memory via a reference to a buffer before being written to or retrieved from a file.
4. Pointers are used to read and write data from/to files. Data may be read from or written to specified memory regions using pointers, giving you more control and flexibility when dealing with file data.
5. File pointers are used to keep track of where you are in a file during sequential access. They are automatically incremented or decremented when data is read or written, allowing for quick file content traversal.
6. Pointers are used to manage and notify problems that arise during file operations. Error codes or error messages can be saved in a pointer variable to reflect the status of the file operation.

# Resource
1. [https://www.youtube.com/watch?v=_x1MmVhLOt4&list=PLhb7SOmGNUc4EBVjd7x5TiEyOKXt71whE&index=1&ab_channel=Log2Base2%C2%AE](https://www.youtube.com/watch?v=_x1MmVhLOt4&list=PLhb7SOmGNUc4EBVjd7x5TiEyOKXt71whE&index=1&ab_channel=Log2Base2%C2%AE)
2. [https://www.w3schools.com/c/c_pointers.php#:~:text=A%20pointer%20is%20a%20variable,another%20variable%20as%20its%20value](https://www.w3schools.com/c/c_pointers.php#:~:text=A%20pointer%20is%20a%20variable,another%20variable%20as%20its%20value)
3. [https://www.youtube.com/watch?v=h-HBipu_1P0&ab_channel=mycodeschool](https://www.youtube.com/watch?v=h-HBipu_1P0&ab_channel=mycodeschool)
4. [https://courses.engr.illinois.edu/cs225/fa2022/resources/stack-heap/](https://courses.engr.illinois.edu/cs225/fa2022/resources/stack-heap/)
5. [https://www.youtube.com/watch?v=X1DcpcgSUXw&ab_channel=mycodeschool](https://www.youtube.com/watch?v=X1DcpcgSUXw&ab_channel=mycodeschool)
6. [https://medium.com/@shoheiyokoyama/understanding-memory-layout-4ef452c2e709](https://medium.com/@shoheiyokoyama/understanding-memory-layout-4ef452c2e709)
7. [https://medium.com/fhinkel/confused-about-stack-and-heap-2cf3e6adb771](https://medium.com/fhinkel/confused-about-stack-and-heap-2cf3e6adb771)
8. [https://www.geeksforgeeks.org/void-pointer-c-cpp/](https://www.geeksforgeeks.org/void-pointer-c-cpp/)
9. [https://stackoverflow.com/questions/5672746/what-exactly-is-the-file-keyword-in-c](https://stackoverflow.com/questions/5672746/what-exactly-is-the-file-keyword-in-c)
