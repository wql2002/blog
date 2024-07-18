---
date:
  created: 2024-02-01
readtime: 10
---

My notes on miscellaneous topics.
<!-- more -->

## non-blocking read

> [c - Non-blocking call for reading descriptor - Stack Overflow](https://stackoverflow.com/questions/5616092/non-blocking-call-for-reading-descriptor)

- set fd as non-blocking mode:

```C
// directly begin to open
fd = open("/path/to/file", O_RDWR | O_NONBLOCK);

// reset an existing fd with `fcntl`
int flags = fcntl(fd, F_GETFL, 0); // get origin flags
fcntl(fd, F_SETFL, flags | O_NONBLOCK); // add the new non-blocking flag

// reset with `ioctl`
const int one = 1;
ioctl(fd, FIOBIO, &one);
```

- use `fd_set` to set the fds we want to control

```c
#include <sys/types.h> // include some macros

fd_set myfds;
int fd1, fd2;
// set all bits of fd_set to zero
FD_ZERO(&myfds);
// pull up the bits we want
FD_SET(fd1, &myfds);
FD_SET(fd2, &myfds);
```

- use `select()` to listen

```c
// compute fd_max as an argument of select()
int fd_max = 1 + fd1<fd2?fd2:fd1;
// use select to listen
if(select(fd_max, &myfds, (fd_set *)0, (fd_set *)0, (struct timeval *)0) < 0) {
    if(errno != EINTR) {
        // Here you had some failures
        // EINTR means that some interrupt happened...
        // in this case we usually start over
    }
  	else {
        continue;
    }
}
```

- use `FD_ISSET` to check availability

```c
if(FD_ISSET(fd1, &myfds)) {
    // something available from fd1, read it and process it.
}
if(FD_ISSET(fd2, &myfds)) {
    // similarly
}
```



## Kill zombies

a _zombie process_ is the child process that was forked by its parent, and **terminated before the parent calls `wait()` to cleanup** its resources because OS will reserve some of the child's information that might be helpful to the parent. 

A good strategy is to prevent them happening:

```c
#include <sys/types.h>	// include this before other headers
#include <sys/wait.h> 	// header for waitpid() and macros
#include <signal.h>		// header for signal functions
#include <stdio.h>		// header for fprintf()
#include <unistd.h>		// header for fork()

void sig_chld(int);		// prototype for our SIGCHLD hander

int main() {
    struct sigaction act;
    pid_t pid;
    
    // assign sig_chld as our SIGCHLD handler
    act.sa_handler = sig_chld;
    
    // we don't want to block any other signals in this example
    sigemptyset(&act.sa_mask);
    
    // we're only interested in children that have terminated,
    // not ones which have been stopped (eg user pressing Ctrl-Z)
    act.sa_flags = SA_NOCLDSTOP;
    
    // make these values effective. If we were writing a real
    // application, we could probably save the old value instead of
    // passing NULL
    if (sigaction(SIGCHLD, &act, NULL) < 0) {
        fprintf(stderr, "sigaction failed\n");
        return 1;
    }
    
    // Fork
    if ((pid = fork()) < 0) {
        fprintf(stderr, "fork failed\n");
        return 1;
    } else if (pid == 0) {
        return 7; 		// exit status = 7
    } else {
        sleep(10);		// give child time to finish
        return 0;
    }
}

// The signal handler function - only gets called when a SIGCHLD
// is received, ie when a child terminates
void sig_chld(int signo) {
    int status, child_val;
    
    // Wait for any child without blocking
    if (watipid(-1, &status, WNOHANG) < 0) {
        // calling standard I/O functions like fprintf() in a 
        // signal handler is not recommanded, but probably OK
        // in toy programs like this one.
        fprintf(stderr, "waitpid failed\n");
    }
    
    // We now have the info in 'status' and can manipulate it using
    // the macros in wait.h
    if (WIFEXITED(status))	// did child exit normally?
        child_val = WEXITSTATUS(status); // get child's exit status
    
    printf("child's exited normally with status %d\n", child_val);
}
```



## Daemon mode

- `fork()`: so that parent can exit, guarantee the new process is not a process group leader
