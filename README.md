Here is the **Universal Template** you can use for your lab exam. Memorize this structure, and you only need to change the logic inside the functions.

### 1\. The Universal C++ Code Template

Use this structure for every single program.

```cpp
// 1. HEADERS (Always write these)
#include <bits/stdc++.h>  // Covers arrays, vector, string, cout, cin
#include <thread>         // For Programs 1, 2, 3, 4, 5, 9, 10
#include <mutex>          // For Programs 3, 5, 6, 9
// #include <omp.h>       // UNCOMMENT ONLY FOR Program 7 & 8 (OpenMP)

using namespace std;

// 2. GLOBAL VARIABLES (Arrays, Mutex locks, Counters)
// Example: mutex mtx;
// Example: int result[10];

// 3. WORKER FUNCTION ( The logic goes here )
void task_function(int id) {
    // Your logic (Addition, Sorting, Searching, etc.)
}

// 4. MAIN FUNCTION
int main() {
    // Step A: Initialize Data (Fill arrays, set variables)
    
    // Step B: Create Threads
    thread t[4]; // Array of 4 threads
    for(int i=0; i<4; i++) {
        t[i] = thread(task_function, i);
    }

    // Step C: Join Threads (Wait for them to finish)
    for(int i=0; i<4; i++) {
        t[i].join();
    }

    // Step D: Print Final Output
    cout << "Final Result: " << endl;
    
    return 0;
}
```

-----

### 2\. The Universal Terminal Commands

This is the **exact sequence** you type in the Ubuntu terminal for every program.

#### **Step 1: Create/Edit the File**

Replace `X` with the program number (e.g., `1.cpp`, `2.cpp`).

```bash
gedit X.cpp
```

*(Paste your code, Save, and Close)*

#### **Step 2: Compile the Code**

You must choose the right command based on the program type.

**A. For Programs 1, 2, 3, 4, 5, 6, 9, 10 (Standard Threads):**
Use the `-pthread` flag.

```bash
g++ X.cpp -pthread -o run
```

**B. For Programs 7 & 8 (OpenMP):**
Use the `-fopenmp` flag.

```bash
g++ X.cpp -fopenmp -o run
```

#### **Step 3: Run the Code**

```bash
./run
```

-----

### 💡 Quick Cheat Sheet

| Program \# | Topic | Header Needed | Compile Flag |
| :--- | :--- | :--- | :--- |
| **1** | Matrix Add | `<thread>` | `-pthread` |
| **2** | Factorial | `<thread>` | `-pthread` |
| **3** | Search | `<thread>`, `<mutex>` | `-pthread` |
| **4** | Merge Sort | `<thread>` | `-pthread` |
| **5** | Mutex Bank | `<thread>`, `<mutex>` | `-pthread` |
| **6** | Producer/Consumer | `<thread>`, `<mutex>`, `<condition_variable>` | `-pthread` |
| **7** | **OpenMP Loop** | `<omp.h>` | **`-fopenmp`** |
| **8** | **OpenMP Task** | `<omp.h>` | **`-fopenmp`** |
| **9** | Deadlock | `<thread>`, `<mutex>` | `-pthread` |
| **10** | Work Stealing | `<thread>` | `-pthread` |

**Next Step:** Would you like me to generate that final **LaTeX PDF code** now so you can take your printout?
**💡 Lab Strategy:**

1.  **Variable Names:** I used short names (`t`, `i`, `mtx`) so you can type fast.
2.  **Headers:** I used `#include <bits/stdc++.h>` to cover everything (vectors, threads, locks, IO).
3.  **Compilation:**
      * **Progs 1–6, 9, 10:** Use `g++ file.cpp -pthread`
      * **Progs 7–8:** Use `g++ file.cpp -fopenmp`

-----

### Program 1: Matrix Addition (Multi-core)

**Logic:** 3 threads. Each thread adds one row.

```cpp
#include <bits/stdc++.h>
#include <thread>
using namespace std;
int A[3][3] = {{1,2,3},{4,5,6},{7,8,9}};
int B[3][3] = {{9,8,7},{6,5,4},{3,2,1}};
int C[3][3];

void add(int r) {
    for(int j=0; j<3; j++) C[r][j] = A[r][j] + B[r][j];
}

int main() {
    thread t[3];
    for(int i=0; i<3; i++) t[i] = thread(add, i);
    for(int i=0; i<3; i++) t[i].join();

    cout<<"Result:\n";
    for(int i=0; i<3; i++) {
        for(int j=0; j<3; j++) cout<<C[i][j]<<" ";
        cout<<endl;
    }
    return 0;
}
```

-----

### Program 2: Factorial

**Logic:** 4 threads. Each calculates factorial for a different number (`5 + id`).

```cpp
#include <bits/stdc++.h>
#include <thread>
using namespace std;
long long res[4];

void fact(int id) {
    long long f = 1;
    for(int i=1; i<=5+id; i++) f *= i;
    res[id] = f;
}

int main() {
    thread t[4];
    for(int i=0; i<4; i++) t[i] = thread(fact, i);
    for(int i=0; i<4; i++) {
        t[i].join();
        cout<<"Thread "<<i<<": "<<res[i]<<endl;
    }
    return 0;
}
```

-----

### Program 3: Parallel Search

**Logic:** Split array into 4 parts. If found, lock the mutex and save the index.

```cpp
#include <bits/stdc++.h>
#include <thread>
#include <mutex>
using namespace std;

#define SIZE 100

int data_array[SIZE];
int found_index = -1;
mutex mtx; // Mutex to protect the shared variable 'found_index'

void search_element(int start_index) {
    int end_index = start_index + 25;
    
    for (int i = start_index; i < end_index; i++) {
        if (data_array[i] == 50) {
            mtx.lock();      // Lock before updating shared variable
            found_index = i;
            mtx.unlock();    // Unlock after updating
            return; 
        }
    }
}

int main() {
    // Initialize array with values 1 to 100
    for (int i = 0; i < SIZE; i++) {
        data_array[i] = i + 1;
    }

    thread threads[4];

    // Create 4 threads, each searching a quarter of the array
    for (int i = 0; i < 4; i++) {
        int start = i * 25;
        threads[i] = thread(search_element, start);
    }

    // Join all threads
    for (int i = 0; i < 4; i++) {
        threads[i].join();
    }

    // Print result
    if (found_index != -1) {
        cout << "Number 50 found at index: " << found_index << endl;
    } else {
        cout << "Number 50 not found." << endl;
    }

    return 0;
}
```

-----

### Program 4: Merge Sort (Parallel)

**Logic:** 4 threads sort 4 quarters. The main thread merges them at the end.
*Note: You must memorize the standard `merge` logic.*

```cpp
#include <bits/stdc++.h>
#include <thread>

using namespace std;
int arr[100], temp[100];

void merge(int l, int m, int r){
    int i=l;
    int j=m+1;
    int k=l;
  while (i <= m && j <= r) {
        if(arr[i]< arr[j]) temp[k++]=arr[i++];
    else temp[k++]=arr[j++];
    }
    while((i<=m) ) temp[k++]=arr[i++];
     while( j<=r) temp[k++]=arr[j++];

    for(int i=l; i<= r; i++) arr[i]=temp[i];
    
}
void merge_sort(int l, int r){
    if(l<r){
        int mid=( l+r )/2;
        merge_sort(l,mid);
        merge_sort(mid+1,r);
        merge(l,mid,r);
        
    }
    
}

void worker(int part) {
    int l = part * 25;
    merge_sort(l, l+24);
}
int main() {
	// your code goes here
	for(int i=0; i<100; i++) arr[i] = rand()%1000;
		
		
	thread t[4];
	for(int i=0; i<4; i++) t[i]=thread(worker,i);
	for(int  i=0; i<4; i++) t[i].join();
	
	merge(0,24,49);
	merge(50,74,99);
	merge(0,49,99); 
	
	for(int i=0; i<100; i++) std::cout << arr[i] << " ";
	cout<< std::endl;

}

```

-----

### Program 5: Mutex (Bank)

**Logic:** Lock before changing balance. Unlock after.

```cpp
#include <bits/stdc++.h>
#include <thread>
#include <mutex>
using namespace std;
int bal = 1000;
mutex mtx;

void trans(int amt) {
    mtx.lock();
    if(bal + amt >= 0) {
        bal += amt;
        cout<<"Success. Bal: "<<bal<<endl;
    } else cout<<"Failed."<<endl;
    mtx.unlock();
}

int main() {
    thread t[5];
    int vals[] = {-200, 100, -300, 150, -400};
    for(int i=0; i<5; i++) t[i] = thread(trans, vals[i]);
    for(int i=0; i<5; i++) t[i].join();
    cout<<"Final: "<<bal<<endl;
    return 0;
}
```

-----

### Program 6: Producer-Consumer

**Logic:** Baker makes bread, rings bell (`notify`). Customer waits for bell (`wait`).

```cpp
#include <bits/stdc++.h>
#include <thread>
#include <mutex>
#include <condition_variable>
using namespace std;

int bread = 0;
mutex m;
condition_variable cv;

void baker() {
    while (true) {
        unique_lock<mutex> lock(m);

        cv.wait(lock, [] { return bread < 5; });   // wait until space is available

        ++bread;
        cout << "Made bread. Total: " << bread << endl;

        cv.notify_one();
        lock.unlock();

        this_thread::sleep_for(chrono::seconds(1));
    }
}

void eater() {
    while (true) {
        unique_lock<mutex> lock(m);

        cv.wait(lock, [] { return bread > 0; });   // wait until bread exists

        --bread;
        cout << "Ate bread. Total: " << bread << endl;

        cv.notify_one();
        lock.unlock();

        this_thread::sleep_for(chrono::seconds(2));
    }
}

int main() {
    thread t1(baker), t2(eater);
    t1.join();
    t2.join();
}

```

-----

### Program 7: OpenMP Loop

**Logic:** `#pragma omp parallel for reduction(+:sum)` does all the magic.
**Compile:** `g++ 7.cpp -fopenmp`

```cpp
#include <bits/stdc++.h>
#include <omp.h>
using namespace std;

int main() {
    int arr[1000];
    for(int i=0; i<1000; i++) arr[i] = 1;
    
    long long sum = 0;
    
    #pragma omp parallel for reduction(+:sum)
    for(int i=0; i<1000; i++) {
        sum += arr[i];
    }
    
    cout<<"Sum: "<<sum<<endl;
    return 0;
}
```

-----

### Program 8: OpenMP Tasks

**Logic:** One `single` manager creates `task` notes.
**Compile:** `g++ 8.cpp -fopenmp`

```cpp
#include <bits/stdc++.h>
#include <omp.h>
using namespace std;

void task(string msg) {
    cout<<msg<<endl;
}

int main() {
    #pragma omp parallel
    {
        #pragma omp single
        {
            #pragma omp task
            task("Blurring");
            #pragma omp task
            task("Sharpening");
            #pragma omp task
            task("Resizing");
        }
    }
    return 0;
}
```

-----

### Program 9: Deadlock (Try Lock)

**Logic:** Try to get lock 2. If failed, drop lock 1 and retry.

```cpp
#include <bits/stdc++.h>
#include <thread>
#include <mutex>
#include <chrono>

using namespace std;

mutex m1, m2;

void thread_function(int id) {
    mutex &first  = (id == 0 ? m1 : m2);
    mutex &second = (id == 0 ? m2 : m1);

    first.lock();
    this_thread::sleep_for(chrono::seconds(1));

    cout << "Thread " << id << ": Waiting for second lock..." << endl;

    if (second.try_lock()) {
        cout << "Thread " << id << ": Acquired both locks, doing work..." << endl;
        this_thread::sleep_for(chrono::seconds(1));
        second.unlock();
    } 
    else {
        cout << "Thread " << id << ": Failed to acquire second lock, backing off." << endl;
    }

    first.unlock();
}

int main() {
    thread t1(thread_function, 0);
    thread t2(thread_function, 1);

    t1.join();
    t2.join();
}

```

-----

### Program 10: Work Stealing (Stride)

**Logic:** Thread `id` picks tasks `id`, `id+4`, `id+8`... (Stride = Number of Threads).

```cpp
#include <bits/stdc++.h>
#include <thread>
using namespace std;

void work(int id) {
    for(int i=id; i<10; i+=4) {
        cout<<"Thread "<<id<<" doing task "<<i+1<<endl;
        this_thread::sleep_for(chrono::seconds(1));
    }
}

int main() {
    thread t[4];
    for(int i=0; i<4; i++) t[i] = thread(work, i);
    for(int i=0; i<4; i++) t[i].join();
    return 0;
}
```
