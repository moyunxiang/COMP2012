# Object-Oriented Programming and Data Structures  
## COMP2012: Static Data Members and Member Functions

Cecia Chan  
Brian Mak  
Pedro Sander  

Department of Computer Science & Engineering  
The Hong Kong University of Science and Technology  
Hong Kong SAR, China  

---

# Static Variables with File / Function Scope

Static variables:

- are **global variables created only once**
- reside in the **static data region** of the program
- exist during the **entire execution of the program**
- are still controlled by **scope**:
  - file
  - function
  - class

Initialization behavior:

- basic types → **zero-initialized**
- objects → **default initialized**

Program memory layout:

```
static data
program code
(run-time) stack
(run-time) heap
```

---

# Static Variables Inside a Function

Static variables declared inside functions:

- initialized **only once**
- retain their value **across function calls**
- accessible **only inside the function**

---

# Example: Static Variable with File Scope

```cpp
#include <iostream> /* File: static-var-file.cpp */
using namespace std;

// Global but static variables can be used
// only in this file (no external linkage)
static int x = 5;

int f()
{
    return ++x;
}

int main()
{
    cout << x << endl;
    cout << f() << endl;
    cout << f() << endl;

    return 0;
}
```

Question:

```
What is the output?
```

---

# Example: Static Variable with Function Scope

```cpp
#include <iostream> /* File: static-var-function.cpp */
using namespace std;

int fibonacci(int n, int& calls)
{
    static int num_calls = 0; // initialized once
    calls = ++num_calls;

    if (n <= 0)
        return 0;
    else if (n == 1 || n == 2)
        return 1;
    else
        return fibonacci(n-2, calls) + fibonacci(n-1, calls);
}

int main()
{
    int n;
    int n_calls;

    cout << "Enter n: ";
    cin >> n;

    cout << "\nfibonacci(" << n << ") = "
         << fibonacci(n, n_calls);

    cout << "\nnumber of fibonacci calls = "
         << n_calls << endl;

    return 0;
}
```

---

# Part I  
## Static Class Data Members

---

# Example: Student Class (Non-Static Memory)

```cpp
#include <iostream> /* File: student-non-static.h */
#include <string>
using namespace std;

const int MAX_MEM {100};

class Student
{
private:
    string name;
    string memory[MAX_MEM];
    int amount_of_memory = 0;

public:
    Student(string s) : name(s) { }

    void do_exam();

    void memorize(string txt)
    {
        if (amount_of_memory >= MAX_MEM)
            cerr << name << " can't memorize anything anymore!\n";
        else
            memory[amount_of_memory++] = txt;
    }
};
```

---

# Student Exam Implementation

```cpp
#include "student-non-static.h" /* File: student-non-static.cpp */

void Student::do_exam()
{
    if (amount_of_memory == 0)
        cout << name << ": Huh???" << endl;
    else
    {
        for (int k = 0; k < amount_of_memory; ++k)
            cout << name << ": " << memory[k] << endl;
    }

    cout << endl;
}
```

---

# Exam Program (Non-Static Version)

```cpp
#include "student-non-static.h" /* File: exam-non-static.cpp */

int main()
{
    Student Jim("Jim");
    Jim.memorize("Data consistency is important");
    Jim.memorize("Copy constructor != operator=");

    Student Steve("Steve");
    Steve.memorize("Overloading is convenient");
    Steve.memorize("Make data members private");
    Steve.memorize("Default constructors have no arguments");

    Student Alan("Alan");

    Jim.do_exam();
    Steve.do_exam();
    Alan.do_exam();

    return 0;
}
```

Compilation:

```
g++ student-non-static.cpp exam-non-static.cpp
```

---

# Output (Normal Case)

```
Jim: Data consistency is important
Jim: Copy constructor != operator=
Steve: Overloading is convenient
Steve: Make data members private
Steve: Default constructors have no arguments
Alan: Huh???
```

Each student has **their own memory**.

---

# Static Memory (Shared by All Students)

```cpp
#include <iostream> /* File: student-static.h */
#include <string>
using namespace std;

const int MAX_MEM {100};

class Student
{
private:
    string name;

    static string memory[MAX_MEM];
    static int amount_of_memory;

public:
    Student(string s) : name(s) { }

    void do_exam();

    void memorize(string txt)
    {
        if (amount_of_memory >= MAX_MEM)
            cerr << name << " can't memorize anything anymore!\n";
        else
            memory[amount_of_memory++] = txt;
    }
};
```

---

# Static Memory Definition

```cpp
#include "student-static.h" /* File: student-static.cpp */

// define + initialize static data
string Student::memory[MAX_MEM] { };
int Student::amount_of_memory {0};

void Student::do_exam()
{
    if (amount_of_memory == 0)
        cout << name << ": Huh???" << endl;
    else
    {
        for (int k = 0; k < amount_of_memory; ++k)
            cout << name << ": " << memory[k] << endl;
    }

    cout << endl;
}
```

---

# Exam Program with Static Data

```cpp
#include "student-static.h" /* File: exam-static.cpp */

int main()
{
    Student Jim("Jim");
    Jim.memorize("Data consistency is important");
    Jim.memorize("Copy constructor != operator=");

    Student Steve("Steve");
    Steve.memorize("Overloading is convenient");
    Steve.memorize("Make data members private");
    Steve.memorize("Default constructors have no arguments");

    Student Alan("Alan");

    Jim.do_exam();
    Steve.do_exam();
    Alan.do_exam();

    return 0;
}
```

Compile:

```
g++ student-static.cpp exam-static.cpp
```

---

# Output with Static Memory

All students share the **same memory**.

```
Jim: Data consistency is important
Jim: Copy constructor != operator=
Jim: Overloading is convenient
Jim: Make data members private
Jim: Default constructors have no arguments

Steve: Data consistency is important
Steve: Copy constructor != operator=
Steve: Overloading is convenient
Steve: Make data members private
Steve: Default constructors have no arguments

Alan: Data consistency is important
Alan: Copy constructor != operator=
Alan: Overloading is convenient
Alan: Make data members private
Alan: Default constructors have no arguments
```

Even Alan (who studied nothing) **knows everything**.

---

# Static Class Data Summary

Static class data members:

- are **global variables within class scope**
- only **one copy exists**
- shared among **all objects**

Properties:

- exist even if **no object exists**
- **not stored inside objects**
- must be **defined outside the class**
- usually in the `.cpp` file

Example definition:

```cpp
int ClassName::static_variable = value;
```

---

# Part II  
## Static Class Member Functions

---

# Example: Class Clock with Static Methods

```cpp
class Clock /* File: clock-w-static-fcn.h */
{
    friend ostream& operator<<(ostream& os, const Clock& c)
    {
        return os << c.hour << " hr. " << c.minute << " min. ";
    }

public:

    Clock() : hour(0), minute(0) { }

    static Clock HHMM(int hhmm)
    {
        return Clock(hhmm/100, hhmm%100);
    }

    static Clock minutes(int m)
    {
        return Clock(m/60, m%60);
    }

private:

    int hour, minute;

    Clock(int h, int m) : hour(h), minute(m) { }
};
```

---

# Testing the Clock Class

```cpp
#include <iostream> /* File: test-clock.cpp */
using namespace std;

#include "clock-w-static-fcn.h"

int main()
{
    Clock c1;
    Clock c2 = Clock::HHMM(123);
    Clock c3 = Clock::minutes(123);

    cout << c1 << endl;
    cout << c2 << endl;
    cout << c3 << endl;

    return 0;
}
```

---

# Static Member Functions

Static methods:

- belong to the **class**, not an object
- behave like **global functions with class scope**

They can be called in two ways:

```
ClassName::function()
```

or

```
object.function()
```

---

# Properties of Static Methods

Static member functions:

- do **not have a `this` pointer**
- can be used **without creating objects**
- can only access **static data members**
- cannot be:

```
const
virtual
```

- cannot overload a non-static member function with the same prototype

---

# Example: Car Class

```cpp
#include <iostream> /* File: car.h */
using namespace std;

class Car
{
public:

    Car() { ++num_cars; }
    ~Car() { --num_cars; }

    void drive(int km)
    {
        total_km += km;
    }

    static int cars_still_running()
    {
        return num_cars;
    }

private:

    static int num_cars;
    int total_km = 0;
};
```

---

# Car Class Implementation

```cpp
#include "car.h"

int Car::num_cars = 0;

int main()
{
    cout << Car::cars_still_running() << endl;

    Car vw;
    vw.drive(1000);

    Car bmw;
    bmw.drive(10);

    cout << Car::cars_still_running() << endl;

    Car *cp = new Car[100];
    cout << Car::cars_still_running() << endl;

    {
        Car kia;
        kia.drive(400);
        cout << Car::cars_still_running() << endl;
    }

    cout << Car::cars_still_running() << endl;

    delete [] cp;

    cout << Car::cars_still_running() << endl;

    return 0;
}
```

---

# Conceptual Analogy

Think of a **class as a factory**.

- Objects → **products**
- Data members → **product data**
- Member functions → **services**

Static members represent:

```
factory-level data and services
```

They exist even if:

```
no products (objects) exist
```

---

# Compilation View

Regular member function:

```cpp
void Car::drive(Car* this, int km)
{
    this->total_km += km;
}
```

Static member function:

```cpp
int Car::cars_still_running()
{
    return Car::num_cars;
}
```

Static functions operate on **class-level data**, not individual objects.

---