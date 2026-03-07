# Object-Oriented Programming and Data Structures  
## COMP2012: Namespace

Cecia Chan  
Brian Mak  
Pedro Sander  

Department of Computer Science & Engineering  
The Hong Kong University of Science and Technology  
Hong Kong SAR, China  

---

# Motivation

Suppose we want to use **two libraries**, each containing useful classes and functions, but some names are the **same**.

Example:

```cpp
/* File: apple-utils.h */

class Stack { /* incomplete */ };
class Some_Class { /* incomplete */ };

void safari() { cout << "Apple's browser" << endl; };
void app(int x) { cout << "Apple's app: " << x << endl; };
```

```cpp
/* File: ms-utils.h */

class Stack { /* incomplete */ };
class Other_Class { /* incomplete */ };

void edge() { cout << "Microsoft's browser" << endl; };
void app(int x) { cout << "Microsoft's app: " << x << endl; };
```

Problem:

Even if you **do not use** `Stack` or `app`, compilation will fail because:

- Multiple definitions of `Stack`
- Multiple definitions of `app`

---

# Example Showing the Problem

```cpp
#include <iostream> /* File: use-utils.cpp */
using namespace std;

#include "apple-utils.h"
#include "ms-utils.h"

enum class OS { MSWindows, MacOS } choice;

int main()
{
    Some_Class sc;
    Other_Class oc;

    if (choice == OS::MacOS)
        safari();
    else if (choice == OS::MSWindows)
        edge();

    return 0;
}
```

Compiler errors occur due to **name conflicts**.

---

# Solution: Namespace

Namespaces prevent **name conflicts** between libraries.

Example:

```cpp
/* File: apple-utils-namespace.h */

namespace apple
{
    class Stack { /* incomplete */ };
    class Some_Class { /* incomplete */ };

    void safari()
    {
        cout << "Apple's browser" << endl;
    }

    void app(int x)
    {
        cout << "Apple's app: " << x << endl;
    }
}
```

```cpp
/* File: ms-utils-namespace.h */

namespace microsoft
{
    using namespace std;

    class Stack { /* incomplete */ };
    class Other_Class { /* incomplete */ };

    void edge()
    {
        cout << "Microsoft's browser" << endl;
    }

    void app(int x)
    {
        cout << "Microsoft's app: " << x << endl;
    }
}
```

---

# Namespace Alias and Scope Operator `::`

Names inside namespaces are accessed using the **scope resolution operator**.

Example:

```cpp
#include <iostream>
using namespace std;

#include "ms-utils-namespace.h"
#include "apple-utils-namespace.h"

namespace ms = microsoft; // Namespace alias

enum class OS { MSWindows, MacOS } choice;

int main()
{
    apple::Some_Class sc;
    apple::Stack apple_stack;

    ms::Other_Class oc;
    ms::Stack ms_stack;

    ms::app(42);

    cout << "Input your OS choice: ";

    int int_choice;
    cin >> int_choice;

    switch (choice = static_cast<OS>(int_choice))
    {
        case OS::MSWindows:
            ms::edge();
            break;

        case OS::MacOS:
            apple::safari();
            break;

        default:
            cerr << "Unsupported OS" << endl;
    }

    return 0;
}
```

Key syntax:

```
namespace_alias::name
```

---

# Using Declaration

If repeatedly typing namespace prefixes is inconvenient, use **using declarations**.

Example:

```cpp
#include <iostream>
using namespace std;

#include "ms-utils-namespace.h"
#include "apple-utils-namespace.h"

namespace ms = microsoft;

using apple::Some_Class;
using ms::Other_Class;
using apple::Stack;
using ms::app;

int main()
{
    Some_Class sc;
    Other_Class oc;

    Stack apple_stack;
    ms::Stack ms_stack;

    app(2);

    return 0;
}
```

Here:

```
Some_Class → apple::Some_Class
Other_Class → ms::Other_Class
```

---

# Ambiguity with `using namespace`

You can import **all names** from a namespace:

```cpp
using namespace apple;
using namespace ms;
```

But this can cause **ambiguity**.

Example:

```cpp
#include <iostream>
using namespace std;

#include "ms-utils-namespace.h"
#include "apple-utils-namespace.h"

namespace ms = microsoft;

using namespace apple;
using namespace ms;

int main()
{
    Some_Class sc;
    Other_Class oc;

    Stack S; // ERROR: ambiguous

    ms::Stack ms_stack;
    apple::Stack apple_stack;

    return 0;
}
```

The compiler cannot determine which `Stack` to use.

---

# Namespace `std`

The **C++ Standard Library** uses namespace `std`.

Example:

```cpp
#include <iostream>
using namespace std;

int main()
{
    string s;

    cin >> s;
    cout << s << endl;

    s += " is good!";
    cout << s << endl;

    return 0;
}
```

---

# Best Practice for `std`

Although this works:

```cpp
using namespace std;
```

it is **not recommended globally**.

Better approaches:

- import only needed names
- use explicit namespace prefixes

This improves **clarity and safety**.

---

# Using Declarations per Object

Example:

```cpp
#include <iostream>

using std::string;
using std::cin;
using std::cout;
using std::endl;

int main()
{
    string s;

    cin >> s;
    cout << s << endl;

    s += " is good!";
    cout << s << endl;

    return 0;
}
```

Only required symbols are imported.

---

# Explicit Namespace per Usage

Another approach is explicit qualification.

Example:

```cpp
#include <iostream>

int main()
{
    std::string s;

    std::cin >> s;
    std::cout << s << std::endl;

    s += " is good!";
    std::cout << s << std::endl;

    return 0;
}
```

This makes the origin of each function **explicit**.

---

# Namespace Is Expandable

Namespaces can:

- be defined **in multiple steps**
- be **nested**

Example:

```cpp
#include <iostream>

namespace hkust
{
    namespace cse
    {
        int rank()
        {
            return 1;
        }
    }

    void good()
    {
        std::cout << "Good!" << std::endl;
    }
}

namespace hkust
{
    void school()
    {
        std::cout << "School!" << std::endl;
    }
}

int main()
{
    std::cout << "CSE's rank: "
              << hkust::cse::rank()
              << std::endl;

    hkust::good();
    hkust::school();

    return 0;
}
```

Key ideas:

- namespaces support **hierarchies**
- namespaces can be **extended later**

Example call:

```
hkust::cse::rank()
```

---