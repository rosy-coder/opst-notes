### 1. Intro

* **Von Neumann model of computing **:

  Many millions of times every second,the processor `fetches` an instruction from memory,`decodes` it and `execute` it.  After it is done with this instruction, the processor moves on to the `next instruction`, and so on, and so on, until the program finally completes

  

**======》 **  a software to *make execute a computer program easily* as above -------- `Operating System(OS)`



* As a **standard library** provides APIs
* As a **resource manager**  manages `CPU`、`memory` 、`disk` and other resources

______



### 2. How does OS achieve this

- #### Virtualizing The CPU

​				*Turning a single CPU (or a small set of them) into a seemingly infinite number of CPUs and thus 				allowing many programs to seemingly run at once.*

- #### Virtualizing Memory

​				*Each process accesses its own private virtual address space.*

- #### Concurrency	

```c
1 #include <stdio.h>
2 #include <stdlib.h>
3 #include "common.h"
4 #include "common_threads.h"
5
6 volatile int counter = 0;
7 int loops;
8
9 void *worker(void *arg) {
10 int i;
11 for (i = 0; i < loops; i++) {
12 counter++;
13 }
14 return NULL;
15 }
16
17 int main(int argc, char *argv[]) {
18 if (argc != 2) {
19 fprintf(stderr, "usage: threads <value>\n");
20 exit(1);
21 }
22 loops = atoi(argv[1]);
23 pthread_t p1, p2;
24 printf("Initial value : %d\n", counter);
25
26 Pthread_create(&p1, NULL, worker, NULL);
27 Pthread_create(&p2, NULL, worker, NULL);
28 Pthread_join(p1, NULL);
29 Pthread_join(p2, NULL);
30 printf("Final value : %d\n", counter);
31 return 0;
32 }
```

```NObash
prompt> gcc -o thread thread.c -Wall -pthread
prompt> ./thread 1000
Initial value : 0
Final value : 2000


prompt> ./thread 100000
Initial value : 0
Final value : 143012 // huh??
prompt> ./thread 100000
Initial value : 0
Final value : 137298 // what the??

//Not Exactly twice when n is large because of the concurrency.
```

- #### Persistence

---

### 3. Design Goal

​			*high perfomance*

​			*minimize the overheads*

​			*reliablilty*

​			*energy-efficiency*

​			*security*

​			*mobility*

