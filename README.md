# Processes Homework

In this homework, you're going to play around with processes and interprocess communication. The big idea is to run an experiment in which we compute some function on a large input dataset that takes a long time to run on the computer. You will use a couple of different methods of computing the function and time each method to find out which is fastest.

The function we are going to use is the good old fashioned inner product (somtimes called dot product) between two vectors.


## Deliverables

1. Write a function `inner_product` that computes the inner product between two vectors, stored as arrays of `float`s.
2. Create two large arrays of `float`s and compute their inner product. Time the operation, and create a text document in your repository to report the results. On my computer, it takes about 2.5 seconds to populate arrays of length 100,000,000 and compute their inner product.
3. Use the `fork()` syscall to create two separate processes, each of which will compute the inner product of half the array. Time the parallelized version of your program, and report the results in the same text document you used in part (2).

## Inner Product

The inner product function between two vectors is the sum of their element-wise products. For example, if you have two vectors `v` and `w`:

    v = ( 1, 2, 3, 4 )
    w = ( 5, 6, 7, 8 )

Then their inner product is:

    <v, w> = 1*5 + 2*6 + 3*7 + 4*8 = 70



### Vectors in C

The C language provides the `float` and `double` datatypes to store real numbers. To store a vector of real numbers, just create an array of floats:


    float v[4] = {1, 2, 3, 4};
    float w[4] = {5, 6, 7, 8};

Unlike Java, arrays in C don't have an intrinsic `.length` property or method. You (the programmer) have to keep track of the length of arrays yourself.
Define a function to compute their inner product:

    /*
     * inner_product
     *
     * This function computes the inner product between two floating point
     * arrays a and b, both of length n.
     */
    float inner_product(float *a, float *b, unsigned int len) {
        // ... your code here ...
    }


Your `inner_product` function should return the inner product between vectors `a` and `b`. You can call it from main, passing the float arrays `v` and `w`:

    int main() {
        float v[4] = {1, 2, 3, 4};
        float w[4] = {5, 6, 7, 8};
        float inp = inner_product(v, w, 4);
        printf("inner product between v and w is %f\n", inp);
        return 0;
    }

### Random Numbers in C

Instead of hand-initializing vectors, it would be nice if the program could do the initializing for you. One way to do that is to generate random numbers for each array element. The `rand()` function in C generates an random integer that can be converted to a floating point:

    float num = (float)rand();


### Measuring Runtime

Measuring a program's runtime is an important part of understanding how your code is performing. There are generally two ways to measure runtime: (1) measure the runtime of the entire process and (2) measure the runtime of one particular segment of your C code.

#### Process-Level Runtime Measurement

You can use the shell's `time` to measure the runtime of an entire process. For example:

    neil@workstation ~ $ time ./program
    
    real	0m0.287s
    user	0m0.257s
    sys	0m0.023s

In the above example, the total runtime was 117ms (0.117 s). Of that, 1 ms (0.001 s) was spent running the actual program and 2 ms (0.002 s) was spent in operating system calls.

There is usually a bit of uncertainty when timing things. If you time the same program several times in a row, you're going to get slightly different results. To quantify the uncertainty, it's best to time your program several times in a row and compute the mean and standard deviation.

#### Getting the Time in your C Program

It seems like the process of initializing the `float` arrays is the thing that takes the most time in the inner product program.
I would prefer not to time the array initialization and only measure the time it takes to run the `inner_product` function.

Timing your code from inside your C program is more accurate because it allows you to measure only a subset of the code.

    #include <stdlib.h>
    #include <unistd.h>
    #include <stdint.h>
    #include <stdio.h>
    #include <time.h>
    // ...
    struct timespec before, after; // structs hold timestamps
    clock_gettime(CLOCK_MONOTONIC, &before);
    // Do some time-consuming operation...
    clock_gettime(CLOCK_MONOTONIC, &after);
    uint64_t total_time =  (int64_t)(after.tv_sec - before.tv_sec) * (int64_t)1000000000UL + (int64_t)(after.tv_nsec - before.tv_nsec);
    printf("time elapsed: %f seconds\n", (float)total_time / 1000000000.);

## `fork()`ing

    You can use the `fork()` syscall to create two separate processes, each with identical memory images. They both have the same code, same variables, same everything. The only difference between the two processes is that `fork()` returns `0` to the child process and some nonzero number to the parent. You can use the `fork()`'s return value to distinguish between the parent and child if you want.

    int pid = fork();
    if(pid == 0) {
        // This is the child process
        // Compute the inner product of the first half of your two vectors here
    } else {
        // This is the parent process
        // Compute the inner product of the second half of your two vectors here
    }


