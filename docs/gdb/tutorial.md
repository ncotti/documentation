# GDB Tutorial

In this tutorial we will compile, execute and debug a simple C code.

Let's first take a look at the code we will be debugging. This code prints the sum of the first prime numbers.

```C {.yaml .copy}
#include <stdlib.h>
#include <stdio.h>
#include <math.h>

int add_array(int *arr, int length) {
    int sum = 0;
    for(int i=0; i<length; i++) {
        sum += arr[i];
    }
    return sum;
}

int is_prime(int x) {
    if(x < 2) {
        return 0;
    } else if(x == 2) {
        return 1;
    } else if(x % 2 == 0) {
        return 0;
    }
    for(int i=3; i<=sqrt(x); i+=2) {
        if(x % i == 0) {
            return 0;
        }
    }
    return 1;
}

int *get_first_primes(int number_of_primes) {
    int* result = (int*) malloc(sizeof(int) *number_of_primes);
    int i=1, x=3;

    result[0] = 2;
    while(i < number_of_primes) {
        if(is_prime(x)) {
            result[i] = x;
            i++;
        }
        x += 2;
    }
    return result;
}

int main(int argc, char **argv) {
    int prime_number_qtty = 3;
    int sum;
    int* primes;

    if(argc == 2) {
        prime_number_qtty = atoi(argv[1]);
    }
    primes = get_first_primes(prime_number_qtty);

    sum = add_array(primes, prime_number_qtty);
    printf(">>> The sum of the first %d prime numbers is %d.\n", 
        prime_number_qtty, sum);
    free(primes);
    return 0;
}
```

We will be using GCC to compile it and we need to add the `-g` flag in order to include the debug information in the binary file. Therefore, create a file called `primes.c` with the contents of the previous code snippet and then compile with:

```bash {.yaml .copy}
gcc -Wall -g primes.c -lm
```

Let's see that our program works as intended by executing it:

```bash
./a.out
>>> The sum of the first 3 prime numbers is 10.
./a.out 5
>>> The sum of the first 5 prime numbers is 28.
```

Before executing GDB, we need to first setup a few configurations to avoid some warnings showing up:

```bash
mkdir -p ~/.config/gdb
echo "set auto-load safe-path /" >> ~/.config/gdb/gdbinit
echo "set debuginfod enabled off > .gdbinit
```

Let's execute GDB now. you should see a bunch of messages, and the last ones are:

```bash
gdb a.out
...
Reading symbols from a.out...
(gdb) 
```

!!! note
    At any time, you can quit by typing `(gdb) quit` or `(gdb) q`.

This is the interactive GDB terminal, where we can run our program and see how it executes in real time.

The first thing that should be done after starting any GBD terminal is to activate the **eXtra Awesome mode**, with the combination of keys `Ctrl + x; a` (first Ctrl + x, then a). You should now see the code and the GDB terminal at once.

Lets start by running our program with:

```gdb
(gdb) run
...
>>> The sum of the first 3 prime numbers is 10.
...
(gdb) set args 5
(gdb) run
...
>>> The sum of the first 3 prime numbers is 28.
...
```

As you can see, our program ran and printed the same message as if it was run from the console. Now, let's try running the code again, but this time step by step. While running these commands, you will first set a breakpoint at the start of the function `main`, in the file `primes.c`. Later, you will run the program, and execute the `next` line of code on that file, skipping the internal execution of the functions. When reaching the call to the function `add_array`, you will `step` into the function, and execute some of its lines before `finish` its execution, and returning to the caller. Finally, we `continue` execution until the next breakpoint. Since there are none, the program ends.

```gdb
(gdb) break primes.c:main
(gdb) run
(gdb) next
(gdb) n
(gdb) n
(gdb) n
(gdb) step
(gdb) n
(gdb) 
(gdb) 
(gdb) 
(gdb) finish
(gdb) n
(gdb) n
(gdb) continue
```

!!! note
    An empty terminal line like `(gdb)` means to press enter. It is a shortcut for repeating the last command. The same applies for using a singe letter like `n` instead of `next`.

We traversed all the aspects of our code, but we still didn't see any of the variables' values, only which lines were executed. We know that our code adds the first prime numbers, but we would like to check that the numbers added are actually prime numbers.

To do that, we will first, set a breakpoint in the "add_array" function, and `watch` (add a watchpoint) to the variable sum. This will pause the execution of the program each time that variable changes, and print its value. Besides, we will `print` the value of the prime number array `arr[i]` that we are adding each time and finally, we will print the whole primes array.

```gdb
(gdb) break primes.c:add_array
(gdb) set args 10
(gdb) run
(gdb) watch sum
(gdb) c
(gdb) c
(gdb) c
(gdb) print arr[i]
(gdb) c
(gdb) delete 2
(gdb) finish
(gdb) print *primes@prime_number_qtty
(gdb) c
(gdb) quit
```

In this tutorial we covered the basics of using the GNU Debugger. We executed our code line by line and saw the values of the variables as they were changing in real time.
