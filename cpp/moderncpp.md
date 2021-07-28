# Auto Keyword

The compiler infers the type at compile time.

``` cpp
#include <iostream>
int main(){
    auto x = 5;
    int y = x;
    std::cout << x << " " << y << std::endl;
}
```

    ## 5 5

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
    int* x;
};
```

### Constructor and Destructor

We’ll create the constructor and destructor similar as to what you would
have done in **SENG1120**.

``` cpp
// Default Constructor
A():x(nullptr){}

// Constructor
A(const int& _x) : x(new int(_x)){
    std::cout << "constructing A pointing to " << x << std::endl;
}

// Destructor
~A(){
    std::cout << "destructing A, which pointed to " << x << std::endl;
    if (x != nullptr){
        delete x;
        x = nullptr;
    }
}
```

``` cpp
#include "A.hpp"
int main(){
    A a{1};
}
```

    ## constructing A pointing to 0x55c5caf86eb0
    ## destructing A, which pointed to 0x55c5caf86eb0

### Copy Constructor

For copy construction we will create a deep copy of the object. This
means that we will follow any pointers and copy the objects they point
to. We do this because there are no guarantees that the objects pointed
to will not be deconstructed by something else.

We do not gain any ownership over anything relating to the parameter but
we do now own the newly constructed integer.

``` cpp
// Copy constructor
A(const A& other){
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

    ## constructing A pointing to 0x561322cc1eb0
    ## copy construction, old 0 other's 0x561322cc1eb0 new 0x561322cc2ee0
    ## destructing A, which pointed to 0x561322cc2ee0
    ## destructing A, which pointed to 0x561322cc1eb0

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

    ## constructing A pointing to 0x560a5bb45eb0
    ## constructing A pointing to 0x560a5bb46ee0
    ## copy assignment, old 0x560a5bb46ee0 other's 0x560a5bb45eb0 new 0x560a5bb46f00
    ## destructing A, which pointed to 0x560a5bb46f00
    ## destructing A, which pointed to 0x560a5bb45eb0

### Move Constructor

For a move constructor the parameter that is passed has had its
ownership yielded, so we can be sure that nothing else will use or free
the passed in resources. This allows us to do a shallow copy of the
pointed to values, this means that we can just copy the pointers.

``` cpp
// Move constructor
A(A&& other){
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
int main(){
    A a{1};
    A b(std::move(a));
}
```

    ## constructing A pointing to 0x562bd34c5eb0
    ## move construction, old 0 other's 0x562bd34c5eb0 new 0x562bd34c5eb0
    ## destructing A, which pointed to 0x562bd34c5eb0
    ## destructing A, which pointed to 0

### Move Assignment

Similar to move construction, but if this object has ownership of
something we must first clean that up.

``` cpp
// Move assignment
A& operator=(A&& other){
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
int main(){
    A a{1};
    A b{};
    b = std::move(a);
}
```

    ## constructing A pointing to 0x55e1e2e95eb0
    ## move assignment, old 0 other's 0x55e1e2e95eb0 new 0x55e1e2e95eb0
    ## destructing A, which pointed to 0x55e1e2e95eb0
    ## destructing A, which pointed to 0

## Smart Pointers

To manage memory and ownership we use smart pointers, these will delete
our objects for us with some caveats.

### Unique

A unique pointer will manage an object on the heap, deleting it when the
pointer goes out of scope.

Always use `std::make_unique` as this solves some problems, never use
`new`.

``` cpp
#include <memory>
#include "A.hpp"
int main(){
    std::unique_ptr<A> a = std::make_unique<A>(5);
    {
    // Make a scope so we can see this object get deleted before the one below gets constructed.
    std::unique_ptr<A> b = std::make_unique<A>(5);
    }
    std::unique_ptr<A> c = std::make_unique<A>(5);
}
```

    ## constructing A pointing to 0x55a7ae2d3ed0
    ## constructing A pointing to 0x55a7ae2d4f20
    ## destructing A, which pointed to 0x55a7ae2d4f20
    ## constructing A pointing to 0x55a7ae2d4f20
    ## destructing A, which pointed to 0x55a7ae2d4f20
    ## destructing A, which pointed to 0x55a7ae2d3ed0

We cannot copy a unique pointer, we can only move it. This prevents two
separate objects owning the object that is pointed to.

``` cpp
#include <memory>
#include "A.hpp"
int main(){
    std::unique_ptr<A> a = std::make_unique<A>(5);
    std::unique_ptr<A> b = a;
}
```

    ## /home/cameron/Documents/NULearning/cpp/.tmp_c/5bjQw.cpp:5:24: error: call to deleted constructor of 'std::unique_ptr<A>'
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
int main(){
    std::unique_ptr<A> a = std::make_unique<A>(5);
    std::unique_ptr<A> b = std::move(a);
}
```

    ## constructing A pointing to 0x55d446ae4ed0
    ## destructing A, which pointed to 0x55d446ae4ed0

### Shared

### Weak

# Standard Library Features

# C++ Feautres
