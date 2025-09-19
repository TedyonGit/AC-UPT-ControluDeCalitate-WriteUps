# sigdance

Dance to the rhythm of your hear...SIGKILLED

- On this challenge we find attached 3 files ``plugin.c, main.c and server.py``. Let's run the program to see what are we dealing with.

![first_run](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/sigdance/first_run.png)

- We see that every message we type we get ``nope``. In that case we need to find a right input to get the input. Let's open ``main.c`` to see what we need to type.

```c
#include <dlfcn.h>
#include <pthread.h>
#include <signal.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/time.h>
#include <time.h>
#include <unistd.h>

static volatile sig_atomic_t ac = 0, uc = 0;
static void h_alrm(int s) {
  (void)s;
  ac++;
}
static void h_usr1(int s) {
  (void)s;
  uc++;
}

static void *th(void *arg) {
  pid_t pid = *(pid_t *)arg;
  struct timespec ts = {0, 5000000};
  for (int i = 0; i < 13; i++) {
    nanosleep(&ts, NULL);
    kill(pid, SIGUSR1);
  }
  return NULL;
}

static void compute_counts(unsigned *A, unsigned *U) {
  ac = 0;
  uc = 0;
  sigset_t unb;
  sigemptyset(&unb);
  sigaddset(&unb, SIGALRM);
  sigaddset(&unb, SIGUSR1);
  pthread_sigmask(SIG_UNBLOCK, &unb, NULL);

  struct sigaction sa1;
  memset(&sa1, 0, sizeof(sa1));
  sigemptyset(&sa1.sa_mask);
  sa1.sa_flags = SA_RESTART;
  sa1.sa_handler = h_alrm;
  sigaction(SIGALRM, &sa1, NULL);
  struct sigaction sa2;
  memset(&sa2, 0, sizeof(sa2));
  sigemptyset(&sa2.sa_mask);
  sa2.sa_flags = SA_RESTART;
  sa2.sa_handler = h_usr1;
  sigaction(SIGUSR1, &sa2, NULL);

  struct itimerval it;
  it.it_value.tv_sec = 0;
  it.it_value.tv_usec = 7000;
  it.it_interval.tv_sec = 0;
  it.it_interval.tv_usec = 7000;
  setitimer(ITIMER_REAL, &it, NULL);

  pthread_t t;
  pid_t me = getpid();
  pthread_create(&t, NULL, th, &me);

  struct timespec s = {0, 777000000};
  nanosleep(&s, NULL);

  setitimer(ITIMER_REAL, &(struct itimerval){0}, NULL);
  pthread_join(t, NULL);

  *A = (unsigned)ac;
  *U = (unsigned)uc;
}

int main() {
  unsigned A, U;
  compute_counts(&A, &U);
  uint32_t PID = (uint32_t)getpid();
  srand((unsigned)time(NULL) ^ PID ^ A ^ U);
  printf("Hello from pid8 = %u\n", (unsigned)(PID & 255u));
  fflush(stdout);

  void *h = dlopen("./libcore.so", RTLD_NOW | RTLD_LOCAL);
  if (!h)
    return 2;
  int (*verify)(uint32_t, uint32_t, uint32_t, uint32_t) = dlsym(h, "verify");
  if (!verify)
    return 3;

  char buf[256];
  while (fgets(buf, sizeof(buf), stdin)) {
    char *e = buf;
    uint32_t prov = strtoul(buf, &e, 0);
    int ok = verify(prov, (uint32_t)A, (uint32_t)U, PID);
    if (ok) {
      const char *f = getenv("FLAG");
      if (!f)
        f = "FLAG{missing}";
      puts(f);
      dlclose(h);
      return 0;
    } else {
      puts("nope");
      fflush(stdout);
    }
  }
  dlclose(h);
  return 0;
}

```

- We can see that has a lot of functions, but i think some of them are just there, but not in use. 

```c
void *h = dlopen("./libcore.so", RTLD_NOW | RTLD_LOCAL);
  if (!h)
    return 2;
  int (*verify)(uint32_t, uint32_t, uint32_t, uint32_t) = dlsym(h, "verify");
  if (!verify)
    return 3;
```

- Loads the function ``verify`` from a library, which i suppose is ``plugin.c``.
```c
int ok = verify(prov, (uint32_t)A, (uint32_t)U, PID);
    if (ok) {
      const char *f = getenv("FLAG");
      if (!f)
        f = "FLAG{missing}";
      puts(f);
      dlclose(h);
      return 0;
    } else {
      puts("nope");
      fflush(stdout);
    }
```
- Yes! So we need to check ``verify`` to solve the challenge. 

```c
int verify(uint32_t provided, uint32_t ac, uint32_t uc, uint32_t pid) {
  uint32_t token = ((ac << 16) ^ (uc << 8) ^ (pid & 255u));
  return provided == token;
}
```

- Ok, we see some shifting bytes, and we guess ``provided`` is our input, but what is ``ac`` and ``uc``?
- Let's go back and see what are those parameters.

```c
unsigned A, U;
compute_counts(&A, &U);

...

int ok = verify(prov, (uint32_t)A, (uint32_t)U, PID);
```

- At the beginning of the function ``main`` those 2 values are initated with the function ``compute_counts``, let's check that function as well.

```c
static void compute_counts(unsigned *A, unsigned *U) {
  ac = 0;
  uc = 0;
  sigset_t unb;
  sigemptyset(&unb);
  sigaddset(&unb, SIGALRM);
  sigaddset(&unb, SIGUSR1);
  pthread_sigmask(SIG_UNBLOCK, &unb, NULL);

  struct sigaction sa1;
  memset(&sa1, 0, sizeof(sa1));
  sigemptyset(&sa1.sa_mask);
  sa1.sa_flags = SA_RESTART;
  sa1.sa_handler = h_alrm;
  sigaction(SIGALRM, &sa1, NULL);
  struct sigaction sa2;
  memset(&sa2, 0, sizeof(sa2));
  sigemptyset(&sa2.sa_mask);
  sa2.sa_flags = SA_RESTART;
  sa2.sa_handler = h_usr1;
  sigaction(SIGUSR1, &sa2, NULL);

  struct itimerval it;
  it.it_value.tv_sec = 0;
  it.it_value.tv_usec = 7000;
  it.it_interval.tv_sec = 0;
  it.it_interval.tv_usec = 7000;
  setitimer(ITIMER_REAL, &it, NULL);

  pthread_t t;
  pid_t me = getpid();
  pthread_create(&t, NULL, th, &me);

  struct timespec s = {0, 777000000};
  nanosleep(&s, NULL);

  setitimer(ITIMER_REAL, &(struct itimerval){0}, NULL);
  pthread_join(t, NULL);

  *A = (unsigned)ac;
  *U = (unsigned)uc;
}
```

- Hmm.... dosen't seems like are getting modified or something, let's add a ``printf`` and check if are getting modified or not.

```c
printf("%d %d\n", ac, uc)
```

- First execution of printf:

![first_printf](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/sigdance/first_printf.png)

- Second:

![second_printf](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/sigdance/second_printf.png)

- Bingo! So we see ``A`` and ``U`` are static values ``0``, ``13`` and we also get PID from the print when we are connecting to the server. All we have to do is to create a script in python to connect to the host and read the message and compute the input by ``verify`` function rules.

```python
import socket
import sys

host = "0.0.0.0"
port = 12345

ac = 0
au = 13

try:
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect((host, port))
	try :
		StringPID = s.recv(1024).strip().decode('utf-8');
		PID = int(StringPID.split(" = ")[1])
		Solve = ((ac << 16) ^ (au << 8) ^ (PID & 255))
		s.send(str(Solve).encode())
		s.send(str("\n").encode())
		Flag = s.recv(1024).strip().decode('utf-8');
		print(Flag)
	except KeyboardInterrupt:
		s.close();
	except EOFError:
		s.close();

except socket.error:
	print("Unable to connect")
```

- Let's see if it works or not.

![bingo](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/sigdance/bingo.png)

- Bingo! Now let's try it on the ctf's host.

![solve](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/sigdance/solve.png)

- GG! We found our flag: CTF{cbc4e1be639219dad8912bb764b566200023e15152635eef87be047c41bd995a}

# Notes

- This is just an alternative solve, one more solve could be by just printing the result of the ``verify`` on local server and just match the pid of the host to have the same input.
