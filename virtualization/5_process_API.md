## Process API

### Intro

*This chapter shows system calls about controlling process*

---

### fork

```c
//p1.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int
main(int argc, char *argv[])
{
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) {
        // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {
        // child (new process)
        printf("hello, I am child (pid:%d)\n", (int) getpid());
    } else {
        // parent goes down this path (original process)
        printf("hello, I am parent of %d (pid:%d)\n",
	       rc, (int) getpid());
    }
    return 0;
}
```

```bash
prompt> ./p1
hello world (pid:29146)
hello, I am parent of 29147 (pid:29146)
hello, I am child (pid:29147)
prompt> ./p1
hello world (pid:29146)
hello, I am child (pid:29147)
hello, I am parent of 29147 (pid:29146)
prompt>

```

* *the value it returns to the caller of fork() is different. Specifically, while the parent receives* `the PID of the newly-created child`, *the child receives a return code of* `zero`.
* *the output (of p1.c) is not deterministic. When the child process is created, there are now two active processes in the system that we care about: the parent and the child. Assuming we are running on a system with a single CPU (for simplicity), then either the child or the parent might run at that point.*

---

### wait

*it is quite useful for a parent to wait for a child process to finish what it has been doing.*

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int
main(int argc, char *argv[])
{
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) {
        // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {
        // child (new process)
        printf("hello, I am child (pid:%d)\n", (int) getpid());
	sleep(1);
    } else {
        // parent goes down this path (original process)
        int wc = wait(NULL);
        printf("hello, I am parent of %d (wc:%d) (pid:%d)\n",
	       rc, wc, (int) getpid());
    }
    return 0;
}
```

```bash
prompt> ./p1
hello world (pid:29146)
hello, I am child (pid:29147)
hello, I am parent of 29147 (pid:29146)
prompt>
```

---

### exec

* 查看exec函数族，可以使用`man exec`

  *On Linux, there are six variants of exec(): execl, execlp(), execle(), execv(), execvp(), and execvpe(). Read the man pages to learn more.*

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

int
main(int argc, char *argv[])
{
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) {
        // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {
        // child (new process)
        printf("hello, I am child (pid:%d)\n", (int) getpid());
        char *myargs[3];
        myargs[0] = strdup("wc");   // program: "wc" (word count)
        myargs[1] = strdup("p3.c"); // argument: file to count
        myargs[2] = NULL;           // marks end of array
        execvp(myargs[0], myargs);  // runs word count
        printf("this shouldn't print out");
    } else {
        // parent goes down this path (original process)
        int wc = wait(NULL);
        printf("hello, I am parent of %d (wc:%d) (pid:%d)\n",
	       rc, wc, (int) getpid());
    }
    return 0;
}
```

* *it loads code (and static data) from thatexecutable and overwrites its current code segment (and current static data) with it; the heap and stack and other parts of the memory space of the program are re-initialized*
* *a successful call to exec() never returns.*

---

### Why? Motivating the API

*The separation of fork() and exec() allows the shell to do a whole bunch of useful things rather easily.*

* **redirected**

  ```bash
  prompt> wc p3.c > newfile.txt
  ```

  this redirect the `wc routine`'s result to newfile.txt

  see just like this 

  ```c
  #include <stdio.h>
  #include <stdlib.h>
  #include <unistd.h>
  #include <string.h>
  #include <fcntl.h>
  #include <assert.h>
  #include <sys/wait.h>
  
  int
  main(int argc, char *argv[])
  {
      int rc = fork();
      if (rc < 0) {
          // fork failed; exit
          fprintf(stderr, "fork failed\n");
          exit(1);
      } else if (rc == 0) {
  	// child: redirect standard output to a file
  	close(STDOUT_FILENO); 
  	open("./p4.output", O_CREAT|O_WRONLY|O_TRUNC, S_IRWXU);
  
  	// now exec "wc"...
          char *myargs[3];
          myargs[0] = strdup("wc");   // program: "wc" (word count)
          myargs[1] = strdup("p4.c"); // argument: file to count
          myargs[2] = NULL;           // marks end of array
          execvp(myargs[0], myargs);  // runs word count
      } else {
          // parent goes down this path (original process)
          int wc = wait(NULL);
  	assert(wc >= 0);
      }
      return 0;
  }
  ```

* **pipe**

  ```bash
  prompt> grep -o foo file | wc -l
  ```

---

### Process Control And  Users

* ***kill*** *is used to send signals to a process*

* *Generally, the systems we use can have multiple people using them at the same time; if one of these people can arbitrarily send signals such as SIGINT (to interrupt a process, likely terminating it), the usability and security of the system will be compromised. As a result, modern systems include a strong conception of the notion of a user*

---

### Useful Tools

* **ps**
* **top**
* **kill/killall**

---

### Summary

* *Each process has a name; in most systems, that name is a number known as a **process ID (PID)**.*
* *The **fork()** system call is used in UNIX systems to create a new process. The creator is called the **parent**; the newly created process is called the **child**. As sometimes occurs in real life [J16], the child process is a nearly identical copy of the parent.*
* *The wait() system call allows a parent to wait for its child to complete execution.*
* *The exec() family of system calls allows a child to break free from its similarity to its parent and execute an entirely new program.*
* *A UNIX shell commonly uses fork(), wait(), and exec() to launch user commands; the separation of fork and exec enables features like **input/output redirection, pipes**, and other cool features, all without changing anything about the programs being run.*
* *Process control is available in the form of **signals**, which can cause jobs to stop, continue, or even terminate.*
* *Which processes can be controlled by a particular person is encapsulated in the notion of a **user**; the operating system allows multiple users onto the system, and ensures users can only control their own processes.*
* *A **superuser** can control all processes (and indeed do many other things); this role should be assumed infrequently and with caution for security reasons.*





