# Object-Oriented Programming and Data Structures  
## COMP2012: Standard Template Library (STL) for Generic Programming

Cecia Chan  
Brian Mak  
Pedro Sander  

Department of Computer Science & Engineering  
The Hong Kong University of Science and Technology  
Hong Kong SAR, China  

---

# The Standard Template Library (STL)

The STL is a collection of powerful, template-based, reusable code libraries.

It provides:

- **Containers** (data structures)
- **Iterators**
- **Algorithms**

These three components form the core of STL.

```
STL
 ├── containers
 ├── iterators
 └── algorithms
```

---

# Part I  
## STL Containers

---

# Container Classes

A **container class** stores a collection of **homogeneous objects** (objects of the same type).

Container classes are usually implemented using **class templates**, since the type of elements may vary.

Example goal:

Design a container similar to an array that:

- stores elements
- supports assignment
- supports copying
- behaves as a first-class object

In practice, STL already provides such containers (e.g., `vector`), but implementing one helps understanding.

---

# Example: Array Container Template

```cpp
/* File: arrayT.h */

template <typename T>
class Array
{
private:
    int _size;
    T* _value;

public:
    Array<T>(int n = 10);        // Default and conversion constructor
    Array<T>(const Array<T>&);   // Copy constructor
    ~Array<T>();

    int size() const { return _size; }
    void init(const T& k);

    Array<T>& operator=(const Array<T>& a); // Copy assignment
    T& operator[](int i) { return _value[i]; }
    const T& operator[](int i) const { return _value[i]; }
};
```

---

# Alternative Syntax

Within templates, the class name may omit `<T>`.

```cpp
template <typename T>
class Array
{
private:
    T* _value;
    int _size;

public:
    Array(int n = 10);
    Array(const Array&);
    ~Array();

    int size() const { return _size; }
    void init(const T& k);

    Array& operator=(const Array&);
    T& operator[](int i) { return _value[i]; }
    const T& operator[](int i) const { return _value[i]; }
};
```

---

# Example Usage

```cpp
#include <iostream>
using namespace std;

#include "array.h"
#include "array-constructors.h"
#include "array-op=.h"
#include "array-op-os.h"

int main()
{
    Array<int> a(3);
    a.init(98); cout << a << endl;

    a = a;
    a[2] = 17;
    cout << a << endl;

    Array<char> b(4);
    b.init('g');
    b[0] = a[1];
    cout << b << endl;

    const Array<char> c = b;
    cout << c << endl;

    Array<int> d;
    d = a;
    cout << d << endl;

    return 0;
}
```

---

# Constructors and Destructor

```cpp
/* File: array-constructors.h */

template <typename T>
Array<T>::Array(int n)
    : _value(new T[n]), _size(n) { }

template <typename T>
Array<T>::Array(const Array<T>& a)
    : Array(a._size)
{
    for (int i = 0; i < _size; ++i)
        _value[i] = a._value[i];
}

template <typename T>
Array<T>::~Array()
{
    delete [] _value;
}

template <typename T>
void Array<T>::init(const T& k)
{
    for (int i = 0; i < _size; ++i)
        _value[i] = k;
}
```

---

# Copy Assignment (Deep Copy)

```cpp
/* File: array-op=.h */

template <typename T>
Array<T>& Array<T>::operator=(const Array<T>& a)
{
    if (&a != this)
    {
        delete [] _value;

        _size = a._size;
        _value = new T[_size];

        for (int j = 0; j < _size; ++j)
            _value[j] = a[j];
    }

    return (*this);
}
```

---

# Stream Output Operator

```cpp
/* File: array-op-os.h */

template <typename T>
ostream& operator<<(ostream& os, const Array<T>& a)
{
    os << "#elements stored = " << a.size() << endl;

    for (int j = 0; j < a.size(); ++j)
        os << a[j] << endl;

    return os;
}
```

---

# Friend Operator Template

```cpp
template <typename T>
class Array
{
    template <typename S>
    friend ostream& operator<<(ostream& os, const Array<S>& x);

private:
    T* _value;
    int _size;

public:
    Array(int n = 10);
    Array(const Array&);
    ~Array();

    int size() const { return _size; }
    void init(const T& k);

    Array& operator=(const Array&);
    T& operator[](int i) { return _value[i]; }
    const T& operator[](int i) const { return _value[i]; }
};
```

---

# Categories of STL Containers

| Category | Containers |
|--------|--------|
| Sequence | vector, list, deque |
| Associative | map, multimap, set, multiset |
| Adaptors | stack, queue, priority_queue |
| Near-containers | bitset, valarray, string |

---

# Sequence Containers

Sequence containers represent **ordered sequences of elements**.

Examples:

- `vector`
- `list`
- `deque`

Characteristics:

| Container | Access | Insert/Delete |
|---|---|---|
| vector | O(1) random access | O(1) at end |
| list | O(n) access | O(1) insert anywhere |
| deque | O(1) random access | O(1) front/back |

---

# Common Sequence Container Operations

Access operations:

```
front()
back()
operator[]
```

Modification:

```
push_back()
pop_back()
push_front()
pop_front()
insert()
erase()
clear()
```

Utility:

```
size()
empty()
resize()
```

---

# Part II  
## Container Adaptors

Two common adaptors:

- **stack**
- **queue**

---

# Stack (LIFO)

Stack follows **Last-In First-Out**.

Operations:

```
top()
push()
pop()
```

Simplified stack implementation:

```cpp
template<typename T, typename Sequence = deque<T>>
class stack
{
protected:
    Sequence c;

public:
    bool empty() const { return c.empty(); }
    size_t size() const { return c.size(); }

    T& top() { return c.back(); }

    void push(const T& x) { c.push_back(x); }

    void pop() { c.pop_back(); }
};
```

---

# Example: Decimal to Binary Conversion

```cpp
#include <iostream>
#include <stack>
using namespace std;

int main()
{
    stack<int> a;
    int x, number;

    while (cin >> number)
    {
        x = number;

        do
        {
            a.push(x % 2);
            x /= 2;
        }
        while (x > 0);

        cout << number << " (base 10) = ";

        while (!a.empty())
        {
            cout << a.top();
            a.pop();
        }

        cout << " (base 2)" << endl;
    }

    return 0;
}
```

---

# Queue (FIFO)

Queue follows **First-In First-Out**.

Operations:

```
front()
push()
pop()
```

Simplified queue:

```cpp
template<typename T, typename Sequence = deque<T>>
class queue
{
protected:
    Sequence c;

public:
    bool empty() const { return c.empty(); }
    size_t size() const { return c.size(); }

    T& front() { return c.front(); }
    T& back() { return c.back(); }

    void push(const T& x) { c.push_back(x); }

    void pop() { c.pop_front(); }
};
```

---

# Part III  
## STL Iterators

Iterators are **generalized pointers** used to traverse containers.

Basic iterator operations:

```
*p        dereference
++p       next element
--p       previous element
p == q    comparison
```

Containers provide:

```
begin()
end()
```

---

# Example: Iterating Through a List

```cpp
#include <iostream>
#include <list>
using namespace std;

int main()
{
    list<int> x;

    for (int j = 0; j < 5; ++j)
        x.push_back(j);

    list<int>::const_iterator p;

    for (p = x.begin(); p != x.end(); ++p)
        cout << *p << endl;

    return 0;
}
```

---

# Iterator-Based Find Function

```cpp
template <class Iterator, class T>
Iterator find(Iterator begin, Iterator end, const T& value)
{
    while (begin != end && *begin != value)
        ++begin;

    return begin;
}
```

This algorithm works for **any container with iterators**.

---

# Part IV  
## STL Algorithms

STL algorithms operate on containers using **iterators**.

Example: `find`

```cpp
template <class Iterator, class T>
Iterator find(Iterator first, Iterator last, const T& value)
{
    while (first != last && *first != value)
        ++first;

    return first;
}
```

---

# Example: Using STL find

```cpp
#include <iostream>
#include <string>
#include <list>
#include <algorithm>

using namespace std;

int main()
{
    list<string> composers;

    composers.push_back("Mozart");
    composers.push_back("Bach");
    composers.push_back("Chopin");

    list<string>::iterator p =
        find(composers.begin(), composers.end(), "Bach");

    if (p == composers.end())
        cout << "Not found." << endl;
    else if (++p != composers.end())
        cout << "Found before: " << *p << endl;
    else
        cout << "Found at the end." << endl;

    return 0;
}
```

---

# find_if Algorithm

More general search using a **predicate**.

```cpp
template <class Iterator, class Predicate>
Iterator find_if(Iterator first, Iterator last, Predicate predicate)
{
    while (first != last && !predicate(*first))
        ++first;

    return first;
}
```

---

# Example Predicate

```cpp
bool greater_than_350(int value)
{
    return value > 350;
}
```

---

# Function Pointers

Functions can be passed as parameters.

Example:

```cpp
int larger(int x, int y)
{
    return (x > y) ? x : y;
}

int smaller(int x, int y)
{
    return (x > y) ? y : x;
}
```

Using function pointer:

```cpp
int (*f)(int, int) = larger;
```

---

# Lambda Version

```cpp
int (*f)(int,int);

f = [](int x, int y)
{
    return (x > y) ? x : y;
};
```

---

# Function Objects (Functors)

A **function object** is an object that behaves like a function.

It overloads `operator()`.

Example:

```cpp
class Greater_Than
{
private:
    int limit;

public:
    Greater_Than(int a) : limit(a) {}

    bool operator()(int value)
    {
        return value > limit;
    }
};
```

Usage:

```cpp
find_if(x.begin(), x.end(), Greater_Than(350));
```

---

# Example: Counting Elements

```cpp
cout << count_if(x.begin(), x.end(), Greater_Than(10));
```

---

# Example: for_each

```cpp
template <class Iterator, class Function>
Function for_each(Iterator first, Iterator last, Function g)
{
    for (; first != last; ++first)
        g(*first);

    return g;
}
```

---

# Example: transform

```cpp
template <class Iterator1, class Iterator2, class Function>
Iterator2 transform(
    Iterator1 first,
    Iterator1 last,
    Iterator2 result,
    Function g)
{
    for (; first != last; ++first, ++result)
        *result = g(*first);

    return result;
}
```

---

# Other STL Algorithms

Common algorithms include:

```
min_element
max_element
equal
generate
remove
remove_if
reverse
rotate
random_shuffle
binary_search
sort
merge
unique
set_union
set_intersection
set_difference
```

---

Source: :contentReference[oaicite:0]{index=0}