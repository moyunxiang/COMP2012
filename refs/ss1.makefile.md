# Object-Oriented Programming and Data Structures  
## COMP2012: Separate Compilation and Makefile

Cecia Chan  
Brian Mak  
Pedro Sander  

Department of Computer Science & Engineering  
The Hong Kong University of Science and Technology  
Hong Kong SAR, China  

---

# COMP2011 Example: Bulbs and Lamps

Recall the example with two classes:

- `Bulb`
- `Lamp`

Concept:

- A **lamp has at least one light bulb**
- All bulbs on the same lamp share:
  - same **wattage**
  - same **price**

Important rule:

- The **price passed to the Lamp constructor** does **NOT include bulb price**
- Bulbs must be installed separately using:

```
install_bulbs()
```

---

# Example Program: lamp-test.cpp

```cpp
#include "lamp.h" /* File: lamp-test.cpp */

int main()
{
    Lamp lamp1(4, 100.5); // lamp costs HKD100.5 itself; needs 4 bulbs
    Lamp lamp2(2, 200.6); // lamp costs HKD200.6 itself; needs 2 bulbs

    // Install 4 bulbs of 20W costing HKD30.1 each
    lamp1.install_bulbs(20, 30.1);
    lamp1.print("lamp1");

    // Install 2 bulbs of 60W costing HKD50.4 each
    lamp2.install_bulbs(60, 50.4);
    lamp2.print("lamp2");

    return 0;
}
```

Compile command:

```bash
g++ -o lamp-test lamp-test.cpp bulb.cpp lamp.cpp
```

---

# Header File: bulb.h

```cpp
/* File: bulb.h */

class Bulb
{
private:
    int wattage; // bulb power (W)
    float price; // bulb price

public:
    int get_power() const;
    float get_price() const;

    void set(int w, float p);
};
```

Parameters:

```
w = wattage
p = price
```

---

# Implementation: bulb.cpp

```cpp
/* File: bulb.cpp */

#include "bulb.h"

int Bulb::get_power() const
{
    return wattage;
}

float Bulb::get_price() const
{
    return price;
}

void Bulb::set(int w, float p)
{
    wattage = w;
    price = p;
}
```

---

# Header File: lamp.h

```cpp
#include "bulb.h" /* File: lamp.h */

class Lamp
{
private:
    int num_bulbs;   // number of bulbs
    Bulb* bulbs;     // dynamic array of bulbs
    float price;     // lamp price excluding bulbs

public:
    Lamp(int n, float p);
    ~Lamp();

    int total_power() const;
    float total_price() const;

    void print(const char* prefix_message) const;

    void install_bulbs(int w, float p);
};
```

---

# Implementation: lamp.cpp

```cpp
#include "lamp.h"
#include <iostream>
using namespace std;

Lamp::Lamp(int n, float p)
{
    num_bulbs = n;
    price = p;
    bulbs = new Bulb[n];
}

Lamp::~Lamp()
{
    delete [] bulbs;
}

int Lamp::total_power() const
{
    return num_bulbs * bulbs[0].get_power();
}

float Lamp::total_price() const
{
    return price + num_bulbs * bulbs->get_price();
}

void Lamp::print(const char* prefix_message) const
{
    cout << prefix_message
         << ": total power = " << total_power() << "W"
         << " , total price = $" << total_price()
         << endl;
}

void Lamp::install_bulbs(int w, float p)
{
    for (int j = 0; j < num_bulbs; ++j)
        bulbs[j].set(w, p);
}
```

---

# Compilation with Multiple `.cpp` Files

Project structure:

```
bulb.h
lamp.h

bulb.cpp
lamp.cpp

lamp-test.cpp
```

Compile with:

```bash
g++ -o lamp-test lamp-test.cpp bulb.cpp lamp.cpp
```

---

# Separate Compilation

Each file can be compiled individually.

```bash
g++ -c bulb.cpp
g++ -c lamp.cpp
g++ -c lamp-test.cpp
```

This produces:

```
bulb.o
lamp.o
lamp-test.o
```

These **object files cannot run directly**.

They must be **linked together**:

```bash
g++ -o lamp-test bulb.o lamp.o lamp-test.o
```

---

# Linking

Process:

```
source code (.cpp)
        ↓ compile
object files (.o)
        ↓ link
executable program
```

Example output:

```
a.out
```

Linker:

```
combines compiled object files
```

---

# Dependencies Among Files

Example dependencies:

```
bulb.cpp → bulb.h
lamp.cpp → lamp.h + bulb.h
lamp-test.cpp → lamp.h + bulb.h
```

If a file changes, dependent files must also be recompiled.

Example:

If `bulb.cpp` changes:

```bash
g++ -c bulb.cpp
g++ -o lamp-test bulb.o lamp.o lamp-test.o
```

If `lamp.h` changes:

```bash
g++ -c lamp.cpp
g++ -c lamp-test.cpp
g++ -o lamp-test bulb.o lamp.o lamp-test.o
```

Manual dependency tracking becomes difficult in large projects.

Solution:

```
Makefile
```

---

# Simple Makefile Example

```make
# variables
SRCS = bulb.cpp lamp.cpp lamp-test.cpp
OBJS = bulb.o lamp.o lamp-test.o

# target rule
lamp-test: $(OBJS)
    g++ -o lamp-test $(OBJS)

bulb.o: bulb.cpp bulb.h
    g++ -c bulb.cpp

lamp.o: lamp.cpp lamp.h bulb.h
    g++ -c lamp.cpp

lamp-test.o: lamp-test.cpp lamp.h bulb.h
    g++ -c lamp-test.cpp

clean:
    /bin/rm lamp-test *.o
```

Rule structure:

```
TARGET: DEPENDENCIES
    COMMAND
```

---

# Finding Dependencies Automatically

Use g++ option:

```bash
g++ -MM *.cpp
```

Example output:

```
bulb.o: bulb.cpp bulb.h
lamp-test.o: lamp-test.cpp lamp.h bulb.h
lamp.o: lamp.cpp lamp.h bulb.h
```

These dependencies can be inserted into the Makefile.

---

# Smarter Makefile

```make
SRCS = bulb.cpp lamp.cpp lamp-test.cpp
OBJS = $(SRCS:.cpp=.o)
DEPS = $(OBJS:.o=.d)

EXE = lamp-test

CXXFLAGS = -std=c++11

$(EXE): $(OBJS)
    g++ $(CXXFLAGS) -o $@ $(OBJS)

-include $(DEPS)

.cpp.o:
    g++ $(CXXFLAGS) -MMD -MP -c $<

clean:
    /bin/rm $(EXE) $(OBJS) $(DEPS)
```

Features:

```
automatic dependency generation
automatic object file generation
```

---

# Libraries

A **library** is a collection of object code.

Example:

Standard C++ libraries

```
iostream
string
```

During linking, the linker includes required functions.

Example linking with a library:

```bash
g++ -o myprog myprog.o -lABC
```

This links:

```
libABC.a
```

---

# Static vs Dynamic Linking

## Static Linking

All library code copied into executable.

Pros:

```
faster execution
portable
```

Cons:

```
larger executable
```

---

## Dynamic Linking

Executable references shared libraries.

Pros:

```
smaller file size
shared libraries
```

Cons:

```
slower runtime linking
less portable
```

---

# Preprocessor Directives

Preprocessor runs **before compilation**.

Directives start with:

```
#
```

Example:

```cpp
#include <iostream>
#include "myfile.h"
```

Rules:

```
< >  → standard library headers
" "  → user-defined headers
```

Search path can be changed with:

```
g++ -I
```

---

# Preventing Multiple Inclusion

Nested includes may cause a header file to be included multiple times.

Problems:

```
duplicate definitions
longer compile time
```

Solution:

```cpp
#ifndef LAMP_H
#define LAMP_H

// class declarations

#endif
```

This is called a:

```
header guard
```