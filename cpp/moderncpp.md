# Auto Keyword

The compiler infers the type at compile time.

``` cpp
#include <iostream>
int main(){
    auto x = 5;
    int y = x;
    std::cout << "x = " << x << ", y = " << y << std::endl;
}
```

    ## x = 5, y = 5

### Task 1.

Write a program which uses `auto` to define a string, then use auto to
define a class or struct type.

# Object Ownership

The model for resource management, e.g. heap memory, we use is called
object ownership. This is where each object has something, usually
another object, that is responsible for freeing the resource.

## Basic Functions

Firstly we’ll create an object that owns some heap memory and will print
out all the common functions that deal with ownership.

### Attributes

``` cpp
struct A{
    int* x; // Usually you shouldn't use "raw" pointers like this
            // this is just for illustrative purposes
};
```

### Constructor and Destructor

We’ll create the constructor and destructor similar as to what you would
have done in **SENG1120**.

``` cpp
// Default Constructor
A() : x(nullptr) {}

// Constructor
A(const int& _x) : x(new int(_x)) {
    std::cout << "constructing A pointing to " << x << std::endl;
}

// Destructor
~A() {
    std::cout << "destructing A, which pointed to " << x << std::endl;
    if (x != nullptr){
        delete x;
        x = nullptr;
    }
}
```

``` cpp
#include "A.hpp"
int main() {
    A a{1};
}
```

    ## constructing A pointing to 0x561b1a0f8eb0
    ## destructing A, which pointed to 0x561b1a0f8eb0

### Copy Constructor

For copy construction we will create a deep copy of the object. This
means that we will follow any pointers and copy the objects they point
to. We do this because there are no guarantees that the objects pointed
to will not be deconstructed by something else.

We do not gain any ownership over anything relating to the parameter but
we do now own the newly constructed integer.

``` cpp
// Copy constructor
A(const A& other) {
    std::cout << "copy constructor, old " << x
    << " other's " << other.x;
    x = new int(*other.x);
    std::cout << " new " << x << std::endl;
}
```

``` cpp
#include "A.hpp"
int main(){
    A a{1};
    A b{a};
}
```

    ## constructing A pointing to 0x563c999e3eb0
    ## copy construction, old 0 other's 0x563c999e3eb0 new 0x563c999e4ee0
    ## destructing A, which pointed to 0x563c999e4ee0
    ## destructing A, which pointed to 0x563c999e3eb0

### Copy Assignment

Similar to copy construction, but if this object has ownership of
something we must first clean that up.

``` cpp
// Copy assignment
A& operator=(const A& other){
    std::cout << "copy assignment, old " << x
    << " other's " << other.x;
    if(x != nullptr){
        delete x;
        x = nullptr;
    }
    x = new int(*other.x);
    std::cout << " new " << x << std::endl;
    return *this;
}
```

``` cpp
#include "A.hpp"
int main(){
    A a{1};
    A b{2};
    b = a;
}
```

    ## constructing A pointing to 0x55c671b3deb0
    ## constructing A pointing to 0x55c671b3eee0
    ## copy assignment, old 0x55c671b3eee0 other's 0x55c671b3deb0 new 0x55c671b3ef00
    ## destructing A, which pointed to 0x55c671b3ef00
    ## destructing A, which pointed to 0x55c671b3deb0

### Move Constructor

For a move constructor the parameter that is passed has had its
ownership yielded, so we can be sure that nothing else will use or free
the passed in resources. This allows us to do a shallow copy of the
pointed to values, this means that we can just copy the pointers.

``` cpp
// Move constructor
A(A&& other) {
    std::cout << "move constructor, old " << x
    << " other's " << other.x;
    x = other.x;
    if (other.x != nullptr){
        other.x = nullptr;
    }
    std::cout << " new " << x << std::endl;
}
```

``` cpp
#include <utility>
#include "A.hpp"
int main() {
    A a{1};
    A b(std::move(a));
}
```

    ## constructing A pointing to 0x55b62901ceb0
    ## move construction, old 0 other's 0x55b62901ceb0 new 0x55b62901ceb0
    ## destructing A, which pointed to 0x55b62901ceb0
    ## destructing A, which pointed to 0

### Move Assignment

Similar to move construction, but if this object has ownership of
something we must first clean that up.

``` cpp
// Move assignment
A& operator=(A&& other) {
    std::cout << "move assignment, old " << x
    << " other's " << other.x;
    if(x != nullptr){
        delete x;
        x = nullptr;
    }
    x = other.x;
    if (other.x != nullptr){
        other.x = nullptr;
    }
    std::cout << " new " << x << std::endl;
    return *this;
}
```

``` cpp
#include <utility>
#include "A.hpp"
int main() {
    A a{1};
    A b{};
    b = std::move(a);
}
```

    ## constructing A pointing to 0x5598c6cbbeb0
    ## move assignment, old 0 other's 0x5598c6cbbeb0 new 0x5598c6cbbeb0
    ## destructing A, which pointed to 0x5598c6cbbeb0
    ## destructing A, which pointed to 0

## Smart Pointers

To manage memory and ownership we use smart pointers. Smart pointers
handle deletion of our objects for us with some caveats. A smart pointer
owns the object that it points to.

### Raw Pointers

A raw pointer is a pointer with no wrapping object, this is exactly what
you would have used in **SENG1120**. You should use smart pointers over
raw pointers whenever possible, which is when you have ownership of the
memory, which is *almost* always. The one common counterexample we run
into is pointers to objects the webots API provides us. We can’t call
delete on these, because webots is in charge of managing them, so should
not use smart pointers with them.

### Unique

A unique pointer is a smart pointer that will manage an object on the
heap, deleting it when the pointer goes out of scope.

Always use `std::make_unique` as this solves some problems. Never use
`new`.

``` cpp
#include <memory>
#include "A.hpp"
int main() {
    std::unique_ptr<A> a = std::make_unique<A>(5);
    {
    // Make a scope so we can see this object get deleted before the one below gets constructed.
    std::unique_ptr<A> b = std::make_unique<A>(5);
    }
    std::unique_ptr<A> c = std::make_unique<A>(5);
}
```

    ## constructing A pointing to 0x55a50e092ed0
    ## constructing A pointing to 0x55a50e093f20
    ## destructing A, which pointed to 0x55a50e093f20
    ## constructing A pointing to 0x55a50e093f20
    ## destructing A, which pointed to 0x55a50e093f20
    ## destructing A, which pointed to 0x55a50e092ed0

We cannot copy a unique pointer, we can only move it. This prevents two
separate objects owning the object that is pointed to.

``` cpp
#include <memory>
#include "A.hpp"
int main() {
    std::unique_ptr<A> a = std::make_unique<A>(5);
    std::unique_ptr<A> b = a;
}
```

    ## /home/cameron/Documents/NULearning/cpp/.tmp_c/e6erl.cpp:5:24: error: call to deleted constructor of 'std::unique_ptr<A>'
    ##     std::unique_ptr<A> b = a;
    ##                        ^   ~
    ## /usr/bin/../lib64/gcc/x86_64-pc-linux-gnu/11.1.0/../../../../include/c++/11.1.0/bits/unique_ptr.h:468:7: note: 'unique_ptr' has been explicitly marked deleted here
    ##       unique_ptr(const unique_ptr&) = delete;
    ##       ^
    ## 1 error generated.

``` cpp
#include <memory>
#include <utility>
#include "A.hpp"
int main() {
    std::unique_ptr<A> a = std::make_unique<A>(5);
    std::unique_ptr<A> b = std::move(a);
}
```

    ## constructing A pointing to 0x55ff4b67fed0
    ## destructing A, which pointed to 0x55ff4b67fed0

#### Exception Memory Safety

Smart pointers are safer when dealing with exceptions. Here the smart
pointer is deleted whilst the raw pointer is leaked.

``` cpp
#include <memory>
#include "A.hpp"
int main() {
    try {
        auto* a = new A(1);
        auto b = std::make_unique<A>(1);
        // if (error occurs) {
            throw std::runtime_error("Oh no!");
        // }
        delete a;
    } catch (...) {}
}
```

    ## constructing A pointing to 0x55e7152beed0
    ## constructing A pointing to 0x55e7152bff20
    ## destructing A, which pointed to 0x55e7152bff20

#### Loaning

If another object needs to temporally have access to an object, you pass
a reference to the pointed to object as a parameter. The object that
gains temporary access and should assume that the object is freed after
the function has returned.

Webots uses raw pointers to loan objects to us, this is why we do not
use smart pointers with webots code.

``` cpp
#include <memory>
#include <iostream>
void borrow_function(const int& x) {
    std::cout << x << std::endl;
}
int main() {
    auto a = std::make_unique<int>();
    borrow_function(*a);
}
```

    ## 0

### Shared

A shared pointer allows for multiple objects to have access to the
pointer. This is done by keeping a count of the number of shared
pointers that point to an object with the object. When this count
reaches zero both the counter and the object are deleted.

Use of shared ownership is rare and always prefer unique pointers where
you can.

``` cpp
#include <memory>
#include "A.hpp"
int main(){
    auto a = std::make_shared<A>(1);
    auto b = a;
}
```

    ## constructing A pointing to 0x561d72a66ed0
    ## destructing A, which pointed to 0x561d72a66ed0

### Weak

A weak pointer points to an object that is pointed to by a shared
pointer, but does not count towards the total count. When accessing the
object pointed to the weak pointer is temporarily upgraded to a shared
pointer.

``` cpp
#include <memory>
#include <iostream>
struct node_bad {
    std::shared_ptr<node_bad> other;
    node_bad(const std::shared_ptr<node_bad>& _other = std::shared_ptr<node_bad>()) : other(_other) {
        std::cout << "Constructor " << this << std::endl;
    }
    ~node_bad() {
        std::cout << "Destructor " << this  << std::endl;
    }
};
struct node_good {
    std::weak_ptr<node_good> other;
    node_good(const std::weak_ptr<node_good>& _other = std::weak_ptr<node_good>()) : other(_other) {
        std::cout << "Constructor " << this << std::endl;
    }
    ~node_good() {
        std::cout << "Destructor " << this  << std::endl;
    }
};
int main() {
    auto a = std::make_shared<node_bad>();
    auto b = std::make_shared<node_bad>(a);
    a->other = b;

    auto c = std::make_shared<node_good>();
    auto d = std::make_shared<node_good>(c);
    c->other = d;
}
```

    ## Constructor 0x55c21280bec0
    ## Constructor 0x55c21280cf00
    ## Constructor 0x55c21280cf30
    ## Constructor 0x55c21280cf60
    ## Destructor 0x55c21280cf60
    ## Destructor 0x55c21280cf30

### Deleter

# Standard Library Features

# C++ Feautres
