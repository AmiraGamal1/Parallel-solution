# Exercise Solution to [Lab(2): Data Parallism](https://khalid-elbadawi.github.io/C425/labs/lab02/)

I will present to you the data partition table for the improved data distribution along with a suggested implementation.

`Data size = 48`

`Number of processors = 4`

## Distribution table

|   P0  |   P1  |   P2  |   P3  |
| ----- | ----- | ----- | ----- |
|   0   |   1   |   2   |   3   |
|   4   |   5   |   6   |   7   |
|   8   |   9   |  10   |  11   |
|  12   |  13   |  14   |  15   |
|  16   |  17   |  18   |  19   |
|  20   |  21   |  22   |  23   |
|       |       |       |       |
|  27   |   26  |   25  |   24  |
|  31   |   30  |   29  |   28  |
|  35   |   34  |   33  |   32  |
|  39   |   38  |   37  |   36  |
|  43   |   42  |   41  |   40  |
|  47   |   46  |   45  |   44  |

Notice that the summation of the total Fibonacci number calculated by each processor is equal.

`To calculate fib(n), we need to calculate fibonacci numbers from 0 to n-2, n-1, so we calculate n fibonacci numbers.`


lets us explain how?

`P0 :`
`(0 + 47) + (4 + 43) + (8 + 39) + (12 + 35) + (16 + 31) + (20 + 27) = 282`

`P1 :`
`(1 + 46 ) + (5 + 42) + (9 + 38) + (13 + 34) + (17 + 30) + (21 + 26) = 282`

`P2 :`
`(2 + 45 ) + (6 + 41) + (10 + 37) + (14 + 33) + (18 + 29) + (22 + 25) = 282`

`P3 :`
`(3 + 44 ) + (7 + 40) + (11 + 36) + (15 + 32) + (19 + 28) + (23 + 24) = 282`

So the total Fibonacci numbers calculated by each processor are equal (`282`).

## CPP code implementaion

Here is the full code for fib, I modified the [fib code](https://www.onlinegdb.com/WbRkodALg).

```cpp
#include <iostream>
#include <pthread.h>
#include <chrono>

using namespace std;

const int N = 48;   // data size
const int NP = 4;   // number of processors

unsigned long long fib (unsigned int);
void computeFib (int);

int main()
{
    pthread_t t0, t1, t2, t3;
    using clock = std::chrono::steady_clock;
    clock::time_point start = clock::now();
 
    pthread_create(&t0, NULL, (void *(*)(void *))computeFib, (void *)0);
    pthread_create(&t1, NULL, (void *(*)(void *))computeFib, (void *)1);
    pthread_create(&t2, NULL, (void *(*)(void *))computeFib, (void *)2);
    pthread_create(&t3, NULL, (void *(*)(void *))computeFib, (void *)3);

    pthread_join(t0, NULL);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    pthread_join(t3, NULL);

    clock::time_point end = clock::now();

    clock::duration execution_time = end - start;
    std::cout << "Execution time is " 
              << std::chrono::duration_cast<std::chrono::milliseconds>(execution_time).count() 
              << "ms." << std::endl;

    return 0;
}

void computeFib(int id)
{

    cout << "Processor (" << id << ") started ..." << endl;

    int start = id;
    int end = N;
    int mid = (end + 1) / 2;
    // distribute the number from 0 to 23 among processor
    for (int n = start; n < mid; n += NP)
        cout << "fib(" << n << ") = " << fib(n) << endl;
    // distribute the number from 47 to 24 among processor
    for (int n = (end - id - 1); n >= mid; n -= NP)
        cout << "fib(" << n << ") = " << fib(n) << endl;

    cout << "Processor (" << id << ") finished." << endl;
}

unsigned long long fib(unsigned int n)
{
    if (n <= 1)
        return 1;
    else
        return fib(n-1) + fib(n-2);
}
```
The idea is to distribute numbers from `0 to 23` among processors.

`for (int n = start; n < mid; n += NP)`

Then distribute number from `47 to 24`.

`for (int n = (end - id - 1); n >= mid; n -= NP)`

{==
Notes:
You can achieve butter implementation by optimizing the fib function also [See this fib algrithms](https://www.geeksforgeeks.org/program-for-nth-fibonacci-number/).
==}