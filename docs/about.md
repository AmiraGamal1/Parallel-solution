# Exercise Solution to [Lab(3): Preserving Dependency](https://khalid-elbadawi.github.io/C425/labs/lab03/)

## PI estimation use  [Monte Carlo](https://www.101computing.net/estimating-pi-using-the-monte-carlo-method/#:~:text=One%20method%20to%20estimate%20the%20value%20of%20%CF%80,count%20how%20many%20fall%20in%20the%20enclosed%20circle.) method

Monte Carlo methods are subset of computational algorithms that use the process of **repeated random sampling** to make numerical estimations of unknown parameters.

### How to estimate PI use Mote Carlo method?

The idea is to simulate random (x, y) points (the points are below to the [uniform distribution](https://www.bing.com/ck/a?!&&p=19c5cd0843e4904aJmltdHM9MTcyOTkwMDgwMCZpZ3VpZD0yY2VmNzBlYy01YWNmLTZjNmMtM2I4My02MjUyNWJhNDZkNTQmaW5zaWQ9NTIyMA&ptn=3&ver=2&hsh=3&fclid=2cef70ec-5acf-6c6c-3b83-62525ba46d54&psq=Uniformly+distribution&u=a1aHR0cHM6Ly9ieWp1cy5jb20vbWF0aHMvdW5pZm9ybS1kaXN0cmlidXRpb24v&ntb=1)) in a 2-D plane with the domain as a square of side 2r units centered on (0.0) and draw a circle with radious r inside the square. Then we calculate the ratio of number points that lied inside the circle and the total number of generated points.

$$
\text{Area of the circle} = \pi r^2
$$

$$
\text{Area of the square} =  2r * 2r = 4r^2
$$

$$
\frac{\text{Area of the circle}}{\text{Area of the square}} = \frac{\pi}{4}
$$

$$
\pi = 4 * \frac{\text{Area of the circle}}{\text{Area of the square}}
$$

## CPP code to estimate PI

```cpp
#define _USE_MATH_DEFINES

#include <pthread.h>
#include <chrono>
#include <math.h>
#include <iomanip>
#include <iostream>
#include <random>

using namespace::std;

constexpr unsigned long long number_of_tosses = 1'000'000'000;
constexpr int NP = 4; // 8, 16, 32, ...
unsigned long long block_length = number_of_tosses / NP;
unsigned long long sum[NP]; /* Array to store the sum for each thread */
/* Thread-local random number generator */
thread_local std::mt19937 generator(std::random_device{}());

/* Function to generate a random number in the range [-1.0, 1.0] */
double random_num_generator() {
  static thread_local std::uniform_real_distribution<> dis(-1.0, 1.0);
  return dis(generator);
}

/* Function to calculate the square of the distance from the origin */
double distance_square() {
  double x, y;

  x = random_num_generator();
  y = random_num_generator();

  return x * x + y * y;
}

/* Thread function to compute the number of points inside the circle */
void *computePI(void *tid) {
  int id = (int)(uintptr_t)tid;
  unsigned long long toss;
  unsigned long long my_sum = 0;
  double square_root;

  for (toss = 0; toss < block_length; toss++) {
    square_root = distance_square();
    if (square_root <= 1) {
      my_sum++;
    }
  }

  sum[id] = my_sum;
  return nullptr;
}

int main(int argc, char **argv) {
  pthread_t tid[NP];
  unsigned long long number_in_circle = 0;
  double pi_estimate;
  using clock = std::chrono::steady_clock;
  clock::time_point start = clock::now();

  /* Create threads */
  for (int i = 0; i < NP; i++) {
    pthread_create(&tid[i], nullptr, computePI, (void *)(uintptr_t)i);
  }

  /* Wait for all threads to finish */
  for (int i = 0; i < NP; i++) {
    pthread_join(tid[i], nullptr);
  }

  /* Sum up the results from all threads */
  for (int i = 0; i < NP; i++) {
    number_in_circle += sum[i];
  }

  pi_estimate = 4.0 * number_in_circle / static_cast<double>(number_of_tosses);
  cout << "PI estimate: " << std::setprecision(15) << pi_estimate << endl;

  clock::time_point end = clock::now();

  clock::duration execution_time = end - start;
  cout << "Execution time is "
       << std::chrono::duration_cast<std::chrono::milliseconds>(execution_time)
              .count()
       << "ms." << endl;
  cout << "value of PI is = " << M_PI << endl;

  return 0;
}
```

**Let us break the code !!**

1. **Generate random number**

    I include the `<random>` header to use [std::mt19937](https://www.geeksforgeeks.org/stdmt19937-class-in-cpp/) Mersenne Twister random number engine to generate high-quality pseudo-random numbers.

    Create a mt19937 engine to generate a random number locally for each thread.

    `thread_local std::mt19937 generator(std::random_device{}());`

    Create objects `dis` that are uniformly distributed in the `range(-1.0, 1.0)` and are once initiated per thread (`static thread_local`). Then use the generator with `dis` to generate random numbers.

    ```cpp
    double random_num_generator() {
        static thread_local std::uniform_real_distribution<> dis(-1.0, 1.0);
        return dis(generator);
    }
    ```

2. **Compute distance square** to check if the point falls inside the circle or not: `distance_square()`.

3. **`computePI`** is thread function to compute the number of points inside the circle. Here I divide the random number generation among threads and let each thread generate $\frac{N}{NP}$ points and sum the total number of points inside the circle, then store it in the array with the id of the process.

4. **In the main function** I create the thread and let them wait to all finish after that let the master thread sum the total numbers of point inside the circle and estimate `PI`.

{==
Notes:
Here I use Array to store the number of points inside the circle generated by each process, but you can also use Vector.
Or you can directly update the number_in_circle using mutex (see your lecture notes).  
==}