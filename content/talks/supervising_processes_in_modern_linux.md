---
title: 'Supervising processes in modern Linux'
date: "2021-10-15T13:30:00"
categories: ["workshop"]
tags: ["supervision", "c"]
---
class: center, middle
name: start

# Supervising processes </br> in modern Linux ![:i](fab fa-linux)

.footer[15th October 2021</br>_Friday Talks_ @ ![:i](fab fa-suse)]
---
class: left, middle

# Danilo Spinella

### _Software Engineer in Packaging_

### **SUSE** ![:i](fab fa-suse)

---
class: middle, left

## ![:i](fas fa-list) Contents

What is a supervisor

Initial setup

Supervising the child using `pidfd`

Listening on signals using `signalfd`

Logging child `stdout` and `stderr` to a file

---
class: left, middle

# ![:i](fas fa-chevron-right) What is a supervisor

---
class: left, middle

A supervisor is a software which only purpose is monitoring and controlling other processes

---
class: middle, left

## Given a supervised process it can

Check its status

Restart it when it dies_*_

Log its output

.footer[_*_ for long lived processes only]
---
class: middle, left

## ![:i](fas fa-tools) It can be found in...

Init (service manager)

Containers

Supervisors

---
class: left, middle

# ![:i](fas fa-chevron-right) Initial setup

---
class: left, middle

## ![:i](fas fa-bullseye) Goals

Supervising a long lived process

Restart it when it dies/exits

Kill it when the supervisor exits

Log its output

---
class: left, middle

##Long lived process vs daemon

Daemons double fork and become child of PID 1

---
class: left, middle

## Process to supervise

Write the current time to stdout every 15 seconds

---
class: left, middle

## ![:i](fas fa-code) Writing `sleepy.c`

---
class: left, middle

```c
#include <stdio.h>
#include <time.h>
#include <unistd.h>

int main() {
  while (1) {
    time_t rawtime;
    struct tm *timeinfo;

    time(&rawtime);
    timeinfo = localtime(&rawtime);

    printf("[%d %d %d %d:%02d:%02d]\n", timeinfo->tm_mday, timeinfo->tm_mon + 1,
           timeinfo->tm_year + 1900, timeinfo->tm_hour, timeinfo->tm_min,
           timeinfo->tm_sec);

    sleep(15);
  }
}
```

---
class: left, middle

## ![:i](fas fa-code) Writing `supervise.c`

---
class: left, middle

## `fork(3p)` / `exec(3p)`

---

class: left, middle

# Supervising the child using `pidfd`

---
class: left, middle

## `pidfd_open(2)` 1/2

```c
int pidfd_open(pid_t pid, unsigned int flags)
```

Create a file descriptor that refers to the process whose PID is specified in _pid_.

---
class: left, middle

## `pidfd_open(2)` 2/2

```c
static int
pidfd_open(pid_t pid, unsigned int flags)
{
  return syscall(__NR_pidfd_open, pid, flags);
}
```

```c
pid_t pid = fork();
if (pid != 0) {
  int pidfd = pidfd_open(pid, 0);
} else {
  ...
```

---
class: left, middle

## `poll(3p)`

Input/output multiplexing

```c
int poll(struct pollfd fds[], nfds_t nfds, int timeout);
```

```c
struct pollfd pollfds;
pollfds.fd = myfd;
pollfds.events = POLLIN;

poll(&pollfd, 1, -1);
```

---
class: left, middle

## `waitid(2)`

```c
int waitid(idtype_t idtype, id_t id, siginfo_t *infop, int options);
```

```c
static int
waitid_pidfd(int pidfd) {
  siginfo_t siginfo;

  return syscall(SYS_waitid, P_PIDFD, pidfd, &siginfo, WEXITED);
}
```

```c
waitid_pidfd(pidfd);
```

---
class: middle, left

Opening a `/proc/[pid]` directory gives a `pidfd`...

...but it can't be waited using `waitid`

---
class: left, middle

# Listening on signals using `signalfd`

---
class: left, middle

The supervisor receive `SIGINT` ![:i](fas fa-bolt)

---

class: left, middle

# Listening on signals

Signal handler

Waiting for signals in a thread

Using signalfd

---
class: left, middle

## `signalfd(2)`

```c
int signalfd(int fd, const sigset_t *mask, int flags);
```

***Note***: _The set of signals must be blocked in the current thread/process_.

---
class: left, middle

## Kill the supervised process

---
class: left, middle

## `pidfd_send_signal(2)` 1/2

```c
int pidfd_send_signal(int pidfd, int sig, siginfo_t *info, unsigned int flags);
```

```c
static int
pidfd_send_signal(int pidfd, int sig) {
  return syscall(SYS_pidfd_send_signal, pidfd, sig, NULL, 0);
}
```

---
class: middle, left

## ![:i](fas fa-check) Success

The supervisor is working

---
class: middle, left

## ![:i](fas fa-thumbs-up) signalfd

Easier interface

Can be handled in user-code

Don't require a separate thread

---
class: middle, left

## ![:i](fas fa-thumbs-down) signalfd

Share the same problems as the other methods

---
class: middle, left

## ![:i](fas fa-thumbs-up) pidfd

No race condition

Safely send a signal to a process

---
class: middle, left

## ![:i](fas fa-thumbs-down) pidfd

`send_signal` works only with processes in the same PID namespace or in one of its descendants

`waidid` works only with children

---
class: middle, left

## `SUBREAPER` 1/2

A subreaper fulfills the role of init(1) for its descendant processes

---
class: middle, left

## `SUBREAPER` 2/2

The subreaper will receive a `SIGCHLD` signal and is able to `wait(2)` on the process

---
class: middle, left

# Logging child </br>`stdout` and `stderr`</br>to a file

---
class: middle, left

`stdin` and `stdout` are `fd`

---
background-image: url(/img/supervising_processes_in_modern_linux/pipe.svg)

---
background-image: url(/img/supervising_processes_in_modern_linux/pipe2.svg)

---

class: middle, left

Separate thread to `poll` both `stdout` and `stdin`

---
class: left, middle

## ![:i](fas fa-check-double) Success

---
class: center, middle

# Questions ![:i](fas fa-question)

---
class: center, middle

#![:i](fab fa-creative-commons) ![:i](fab fa-creative-commons-by) ![:i](fab fa-creative-commons-sa)

## This work is licensed under

## _Attribution-ShareAlike 4.0 International</br> (CC BY-SA 4.0)_

.footer[Icons from _www.onlinewebfonts.com_, which are licensed under CC BY 3.0]

---
template: start

