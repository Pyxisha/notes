---
title: "2018-07-25 the vfork syscall"
date: 2018-07-25 22:44
---

## vfork vs fork 

like fork, vfork is a syscall to create a new process in Unix System. In the book *Advanced Programming in the Unix like Environment*, it says there two main difference between fork and vfork:

+ the child process created by vfork share the same address space with the parent process.
+ vfork will guarantees the child process run first, until the child call exec or exit.

we can use the follow program to test these two points:

```C
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int globval = 10;

int main(void){
    int var;
    pid_t pid;
    var = 89;
    
    printf("before vfork\n");
    if((pid = vfork()) < 0){
        printf("vfork error\n");
        exit(0);
    }else if(pid == 0){ // child
        globval++;
        var++;
        exit(0);
    }

    printf("global = %d, var = %d\n", globval, var);
    return 0;
}
```

the follow shows consequence in debian with 4.9 kernel:

![cons1](/images/vfork1.png)

we can see child ran first and child did share the same address space.

## the implemetion

the discussion is based on 3.10 kernel source code

both fork and vfork are define in kernel/fork.c.

![forks](/images/forks.png)

both fork and vfork will call do_fork,  the only difference between fork and vfork in kernel source code is that vfork has two more flags: CLONE_VFORK and CLONE_VM;

if CLONE_VM flag is given, the mm_struct of process's task_struct will not be dupped, only the user_num of mm_struct will increase 1. here is the piece of code of copy_mm function in kernel/fork.c.

![copy_mm](/images/copy_mm.png)

almost in the end of do_fork function, we can see if CLONE_VFORK flag is given, child process will be wake up first and parent will wait child to fork done.

![clone_vfork](/images/clone_vfork.png) 

## a trick when vfork are called inside a function

see the follow function:

```C
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

void test_fork(){
    int s = 1984;
    printf("before vfork\n");
    pid_t pid = vfork();
    if(pid < 0){
        printf("vfork error\n");
        exit(0);
    }else if(pid == 0){
        return 0;
    }else{
        printf("parent s = %d\n",s);
    }
}

int main(void){
    int var;
    var = 89;
    test_fork();
    
    printf("out test vfork\n");
    return 0;
}
```

the consequence are:

![vfork2](/images/vfork2.png) 

modify line 13 to `exit(0)`

the consequence are:

![vfork2](/images/vfork3.png)

parent process and child process share the same address space, the stack address is also shared, return from function may mess the stack, so we should be carefully when using vfork syscall. 





