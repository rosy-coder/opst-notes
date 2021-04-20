## Interlude: Memory API

### Types of Memory

* stack/automatic
* heap

---

### The malloc() Call

> The malloc() call is quite simple: you pass it a size asking for some room on the heap, and it either succeeds and gives you back a pointer to the newly-allocated space, or fails and returns NULL .

For **sizeof()** ,it is a compile-time operator not the run-time function.

---

### The free() Call

```c
int *x = malloc(10 * sizeof(int));
//...
free(x);
```

----

###  Common Errors

* Forgetting To Allocate Memory

  ```c
  char *src = "hello";
  char *dst; // oops! unallocated
  strcpy(dst, src); // segfault and die
  ```

* No Allocating Enough Memory

  ```c
  char *src = "hello";
  char *dst = (char *) malloc(strlen(src)); // too small!
  strcpy(dst, src); // work properly
  ```

* Forgetting to Initialize Allocated Memory

* Forgetting To Free Memory

  > In long-running applications or systems (such as the OS itself), this is a huge problem, as slowly leaking memory eventually leads one to run out of memory, at which point a restart is required.

* Freeing Memory Before You Are Done With It
* Freeing Memory Repeatedly
* Calling free() Incorrectly

---

### Underlying OS Support

* brk/sbrk

* mmap()

  > By passing in the correct arguments, mmap() can create an anonymous memory region within your program — a region which is not associated with any particular file but rather with swap space, something we’ll discuss in detail later on in virtual memory.

---

### Other Calls 

* calloc()

  malloc and initial.

* realloc()

  > when you’ve allocated space for something (say, an array), and then need to add something to it: realloc() makes a new larger region of memory, copies the old region into it, and returns the pointer to the new region.