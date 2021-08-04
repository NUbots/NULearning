---
author: Cameron Murtagh
date: 27/7/2021
output:
  md_document:
    variant: gfm
title: Modern C++
---

# Struct

`class` has all its members private by default. In c++ a `struct` is a
`class` that has all its members public by default. Typically structs
are chosen for types which are simply collections of other types, while
classes are chosen to represent higher order objects.

# Free

When you (or a library) call `delete` on some heap allocated object,
under the hood C++ is calling a standard C function called `free`.
`free` gives the memory segment back to the operating system, “freeing”
it. The terms `free` and `delete` are often used interchangably by C++
developers, although they are technically distinct.

# Const

The `const` keyword declares a type immutable, that is that the value is
read only and cannot be mutated. The `const`-ness of a variable can be
changed with `const_cast`, but this should be used *exceptionally
rarely*.

## Variables

To declare a variable `const` simply add the keyword to the type.

``` cpp
#include <iostream>
int main(){
    const int x = 1;
    const int& y = x;
    std::cout << "x = " << x << ", y = " << y << std::endl;
}
```

    ## x = 1, y = 1

``` cpp
int main(){
    const int x = 1;
    x ++;
}
```

    ## /home/cameron/Documents/NULearning/cpp/.tmp_c/JYFOG.cpp:3:7: error: cannot assign to variable 'x' with const-qualified type 'const int'
    ##     x ++;
    ##     ~ ^
    ## /home/cameron/Documents/NULearning/cpp/.tmp_c/JYFOG.cpp:2:15: note: variable 'x' declared const here
    ##     const int x = 1;
    ##     ~~~~~~~~~~^~~~~
    ## 1 error generated.

## Parameters

This is usually used when passing parameters by reference, but can be
used with pass by value even though doing so is usually pointless.

**Bonus question:** Why is it usually pointless?

Having a “pass by reference” parameter declared as constant allows the
caller to be sure that the parameter they pass in will not be changed.

``` cpp
#include <iostream>
void cool_print(const int& x, const int& y){
    std::cout << "x " << x << " y " << y << std::endl;
}

void definitely_benign_print(const int& x, int& y){
    std::cout << "x " << x << " y " << y++ << std::endl;
}

int main(){
    const int a = 7;
    int b = 5;
    cool_print(a, b);
    definitely_benign_print(a, b);
    cool_print(a, b);
}
```

    ## x 7 y 5
    ## x 7 y 5
    ## x 7 y 6

## Methods

To add const to a method, add the keyword to the end of the signature.
This means that the method cannot mutate any of the attributes.

If there is an instance of a type declared const it cannot call any
method that is not declared const.

``` cpp
#include <iostream>
struct T {
    int x = 1;
    
    int get_x() const {
        return x;
    }
    
    void set_x(const int& x_) {
        x = x_;
    }
};

int main(){
    T a;
    a.set_x(5);
    std::cout << "a " << a.get_x() << std::endl;
    
    const T b;
    b.set_x(5);
    std::cout << "b " << b.get_x() << std::endl;
}
```

    ## /home/cameron/Documents/NULearning/cpp/.tmp_c/nAthV.cpp:20:5: error: 'this' argument to member function 'set_x' has type 'const T', but function is not marked const
    ##     b.set_x(5);
    ##     ^
    ## /home/cameron/Documents/NULearning/cpp/.tmp_c/nAthV.cpp:9:10: note: 'set_x' declared here
    ##     void set_x(const int& x_) {
    ##          ^
    ## 1 error generated.

## Const Type

The `const` version is actually a distinct type to the original type but
`c++` can implicitly cast to a `const` type from the non-const type.
e.g. `std::array<const int, 1>` and `std::array<int, 1>` are distinct
types, this can be observed by using the subscript operator, which
returns `const int&` and `int&` respectively.

``` cpp
template <typename T>
struct simple_container {
    T x;
};

int main(){
    const simple_container<int> x = {1};
    simple_container<int> y = {2};
    y = x;
    simple_container<const int> z = {3};
    z = y;
}
```

    ## /home/cameron/Documents/NULearning/cpp/.tmp_c/JT1U2.cpp:11:7: error: no viable overloaded '='
    ##     z = y;
    ##     ~ ^ ~
    ## /home/cameron/Documents/NULearning/cpp/.tmp_c/JT1U2.cpp:2:8: note: candidate function (the implicit copy assignment operator) not viable: no known conversion from 'simple_container<int>' to 'const simple_container<const int>' for 1st argument
    ## struct simple_container {
    ##        ^
    ## 1 error generated.

In this example we cannot convert from `simple_container<int>` to
`simple_container<const int>` as they are different types, and thus need
an explicit conversion defined.

## Raw Pointers

There are some caveats with raw pointers (see definition in
[ownership](#raw-pointers)), but we *very rarely* use raw pointers. If
you’re confused about such a case, there are many other resources
explaining what is happening.

## How to Read `const` Specifiers

The rule when reading types in C++ is that you should read “from right
to left”. This manifests as `const` applying to the thing immediately to
its left. If there isn’t anything to the left, it applies to the thing
on its right - please don’t use [“East
const”](https://isocpp.org/wiki/faq/const-correctness#const-ref-alt)
though… e.g. `const int*` and `int const*` are both a pointer to a
constant int, `int* const` is a const pointer to an int and
`const int* const` or `int const* const` is a const pointer to a const
int. ## Task

1.  Add `const` to the following variables that don’t change.

``` cpp
#include <iostream>
int main(){
    int x = 2;
    int y = 7;

    int accumulate = 0;
    for (int i = 0; i < y; i++){
        accumulate += x;
    }
    std::cout << accumulate << std::endl;
}
```

2.  Add `const` to the following methods that don’t mutate the
    attributes.

``` cpp
struct square {
    int x;
    int y;
    int width;
    int height;

    int get_x() {
        return x;
    }

    int get_y() {
        return y;
    }

    int area() {
        return width * height;
    }

    int double_area() {
        width *= 1.4142;
        height *= 1.4142;
        return area();
    }
}
```

# Auto Keyword

The compiler infers the type at compile time.

``` cpp
#include <iostream>
int main() {
    auto x = 5;
    int y = x;
    std::cout << "x = " << x << ", y = " << y << std::endl;
}
```

    ## x = 5, y = 5

The `auto` keyword can also be used to define references and “raw”
pointers, using `auto&` and `auto*`. The `const` keyword also applies as
you would expect to `auto` declarations.

Some C++ people think you should “almost always \[use\] auto”, but we
don’t. Our most common use of `auto` is for the iterator’s object in a
range-based `for` loop, as in the following example:

``` cpp
#include <iostream>
#include <array>
#include <string>
int main(){
    std::array<std::string, 5> letters {"a", "b", "c", "d", "e"};

    // Range-based for loops automatically create iterators for containers
    // which support them, such as std::array, and dereference them for us. 
    // You should usually use `const auto&`, `auto&` (or `auto&&`) for these 
    // variables, because it's convention and it's nice
    for (const auto& letter : letters) {
        std::cout << letter << std::endl;
    }
}
```

    ## a
    ## b
    ## c
    ## d
    ## e

### DO NOT USE `auto` FOR EIGEN TYPES

Eigen’s docs list using the `auto` keyword for their types as a [common
pitfall](https://eigen.tuxfamily.org/dox/TopicPitfalls.html). This is
mostly because Eigen uses intermediate types all the time with many
implicit conversions, which are not at all obvious to the reader.

### Task

1.  Write a program which uses `auto` to define a string, then use auto
    to define a custom class or struct type `T`.

2.  Add a range-based for loop to the program, iterating through an
    array of your class type.

``` cpp
#include <array>
#include <string>
#include <iostream>
struct T {
    int x;
    T (cons int& x_) : x(x_) {}
}

int main() {
    // Define a your string here, call it str
    
    // Define an instance of type T here, call it t

    std::cout << "str: " << str << ", t.x:" << t.x << std::endl;

    std::array<T, 3> a = {T(1), T(2), T(3)};

    // Add the start of your for loop here
    {
        std::cout << i.x << std::endl;
    }
}
```

# Initializer Lists & Namespaces

Initializer lists can be used in constructors to initialise values for
member variables, even if they’re const. Here is an example of the
syntax:

``` cpp
#include "kev/FooLongName.hpp"

namespace bar::baz {
```

This scope defines a namespace `baz` inside of the namespace `bar`.

``` cpp
    using Foo = kev::FooLongName;
```

This is a type alias. These serve the same function as `typedef`
statements, except they don’t declare a new type like `typedef`s do.
These can make things more concise but we don’t use them often, for the
sake of clarity over conciseness.

``` cpp
    using kev::FooLongName::getAlternativeConfig;
```

Adds this name into this namespace. We do use these often, to import
function names, so we don’t have huge, long function names within four
nested namespaces

``` cpp
    class A {
        const int a = 0; // Can still be given another value by a constructor, by using
                         // an initializer list, despite being `const`

        Foo thing_f{};   // Brace-initialisation - calls the associated constructor in-place

        A() = default;   // Makes the compiler define the default constructor for us
                         // even though we defined another constructor, which otherwise
                         // would make the compiler delete the default one
                        
        A(const int& a_) : a(a_), thing_f(getAlternativeConfig()) // We can even call functions in
                                                                  // initializer lists, such as
                                                                  // this previously `using`'d one
        {
            // This code is executed after the initializer list is finished doing things
        }
    };
} // namespace bar::baz
```

(Note that if a member variable or object has no default constructor, it
must be initialised using an initializer list.)

This example makes liberal use of *namespaces*. Namespaces let us
separate functions and objects into different spots where their symbols
won’t conflict. For example, we can have a function in the `bar::baz`
namespace called `run()` with the same signature (besides the namespace)
as `foo::run()`, or even `bar::run()`.

**Anonymous namespaces** are namespaces which don’t have names. These
restrict use of the things inside of the namespace to the things inside
the same file as the things in the anonymous namespace. Anonymous
namespaces [should not be used in
headers](https://wiki.sei.cmu.edu/confluence/display/cplusplus/DCL59-CPP.+Do+not+define+an+unnamed+namespace+in+a+header+file#:~:text=Do%20not%20define%20an%20unnamed%20namespace%20in%20a%20header%20file&text=When%20an%20unnamed%20namespace%20is,used%20within%20that%20translation%20unit.).
Here is an example using an anonymous namespace:

``` cpp
#include "foo.hpp"

// Anonymous namespace - keeps these things accessible only within this file
namespace {
    return_type helper_function() { 
        // does things
    }
}

baz_type foo::baz() {
    // We can call `helper_function` in this file, but we can't call it outside of this file.
    // This keeps it local to this file, without any chance of it being used by anything else.
    const auto bar = helper_function();
    return f(bar);
}
```

# Object Ownership

The model for resource management, e.g. heap memory, we use is called
object ownership. This is where each object has something, usually
another object, that is responsible for freeing the resource.

## Basic Functions

Firstly we’ll create an object that owns some heap memory, then we’ll
print out all the common functions that deal with ownership.

### Attributes

``` cpp
struct A {
    int* x; // Usually you shouldn't use "raw" pointers like this
            // This is just for illustrative purposes
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
    if (x != nullptr) {
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

    ## constructing A pointing to 0x5574fa5f5eb0
    ## destructing A, which pointed to 0x5574fa5f5eb0

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

    ## constructing A pointing to 0x55a5d0e2deb0
    ## copy construction, old 0 other's 0x55a5d0e2deb0 new 0x55a5d0e2eee0
    ## destructing A, which pointed to 0x55a5d0e2eee0
    ## destructing A, which pointed to 0x55a5d0e2deb0

### Copy Assignment

Similar to copy construction, but if this object has ownership of
something we must first clean that up.

``` cpp
// Copy assignment
A& operator=(const A& other) {
    std::cout << "copy assignment, old " << x
    << " other's " << other.x;
    if(x == other.x) {
        return *this;
    }
    if(x != nullptr) {
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
int main() {
    A a{1};
    A b{2};
    b = a;
}
```

    ## constructing A pointing to 0x561a359dfeb0
    ## constructing A pointing to 0x561a359e0ee0
    ## copy assignment, old 0x561a359e0ee0 other's 0x561a359dfeb0 new 0x561a359e0f00
    ## destructing A, which pointed to 0x561a359e0f00
    ## destructing A, which pointed to 0x561a359dfeb0

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

    ## constructing A pointing to 0x5625ff046eb0
    ## move construction, old 0 other's 0x5625ff046eb0 new 0x5625ff046eb0
    ## destructing A, which pointed to 0x5625ff046eb0
    ## destructing A, which pointed to 0

### Move Assignment

Similar to move construction, but if this object has ownership of
something we must first clean that up.

``` cpp
// Move assignment
A& operator=(A&& other) {
    std::cout << "move assignment, old " << x
    << " other's " << other.x;
    if(x == other.x) {
        return *this;
    }
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

    ## constructing A pointing to 0x56423dda6eb0
    ## move assignment, old 0 other's 0x56423dda6eb0 new 0x56423dda6eb0
    ## destructing A, which pointed to 0x56423dda6eb0
    ## destructing A, which pointed to 0

### Task

Create a struct which has the following:

1.  A default constructor which prints the string “Default constructor
    called”
2.  A copy constructor which prints the string “Copy constructor called”
3.  A copy operator which prints the string “Copy operator called”
4.  A move constructor which prints the string “Move constructor called”
5.  A move operator which prints the string “Move operator called”

Once you have defined these, write a program which uses each of them,
printing all 5 strings.

``` cpp
struct ownership_class{
    // Make an attribute here

    // Make a default constructor here
    
    // Make a copy constructor here

    // Make a move constructor here

    // Make a copy assignment operator here

    // Make a move assignment operator here
    
};

ownership_class test_function() {
    return ownership_class();
}

int main() {
    ownership_class a; // Default construct
    ownership_class b{a}; // Copy construct
    ownership_class c{test_function()}; // Move construct
    b = a; // Copy assignment
    c = test_function(); // Move assignment
}
```

#### Post task notes

The copy and move constructors/operators along with the destructor, are
known as the special member functions. They are automatically defined
for all classes and structs, unless you provide a definition for them
yourself.

The **Rule of 5** says that if you define a non-default destructor, you
should define all 5 special member functions explicitly or delete them.
The following example implements a non-default destructor to handle a
class’s file descriptor, abiding by the rule of 5:

``` cpp
A() : fd(-1); // Default file descriptor value
~A() {
    close(fd); // Close the file descriptor on destruction, which is quite typical
}
// Delete our other special member functions, so we don't have to deal with
// transferral of the file descriptor to other `A` instances
A(A& other) = delete;
A(A&& other) = delete;
A& operator=(A& other) = delete;
A& operator=(A&& other) = delete;
```

## l-values and r-values

An l-value is a reference that references a value bound to a variable.
In the function this is a single ampersand `&`.

A r-value is a reference that is not bound to a variable. r-values are
temporary and thus the ownership of the value is yielded to the function
it is passed to. In the function, this is a double ampersand `&&`.

An l-value can be converted to an r-value by using `std::move`. r-values
are returned from functions that return by value.

Move semantics use r-values and copy semantics use l-values.

``` cpp
#include "A.hpp"
A cool() {
    return A(5);
}
int main() {
    A x = cool();
}
```

    ## constructing A pointing to 0x563289f47eb0
    ## destructing A, which pointed to 0x563289f47eb0

## Smart Pointers

To manage memory and ownership we use smart pointers. Smart pointers
handle deletion of our objects for us, with some caveats. A smart
pointer owns the object that it points to.

### Raw Pointers

A raw pointer is a pointer with no wrapping object. These are what you
would have used in **SENG1120**. You should use smart pointers over raw
pointers whenever possible, which is when you have ownership of the
memory, which is *almost* always. The one common counterexample we run
into is pointers to objects the webots API provides us. We can’t call
delete on these, because webots is in charge of managing them, so should
not use smart pointers with them. Being able to (and having the
responsibility to) call delete on something is the defining property of
*ownership*.

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
    // Make a scope so we can see this object get deleted before the one below gets constructed.
    {
        std::unique_ptr<A> b = std::make_unique<A>(5);
    }
    std::unique_ptr<A> c = std::make_unique<A>(5);
}
```

    ## constructing A pointing to 0x560be4268ed0
    ## constructing A pointing to 0x560be4269f20
    ## destructing A, which pointed to 0x560be4269f20
    ## constructing A pointing to 0x560be4269f20
    ## destructing A, which pointed to 0x560be4269f20
    ## destructing A, which pointed to 0x560be4268ed0

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

    ## /home/cameron/Documents/NULearning/cpp/.tmp_c/6zfoT.cpp:5:24: error: call to deleted constructor of 'std::unique_ptr<A>'
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

    ## constructing A pointing to 0x556910789ed0
    ## destructing A, which pointed to 0x556910789ed0

#### Exception Memory Safety

Smart pointers are safer when dealing with exceptions. Here, the smart
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

    ## constructing A pointing to 0x55bce977aed0
    ## constructing A pointing to 0x55bce977bf20
    ## destructing A, which pointed to 0x55bce977bf20

#### Loaning

If another object needs to temporarily have access to an object, you
should pass a reference to the pointed to object. The receiving object
gains “temporary access” and it should assume that the object is freed
after the function has returned.

``` cpp
#include <memory>
#include <iostream>
void borrow_function(const int& x) {
    std::cout << x << std::endl;
}
int main() {
    auto a = std::make_unique<int>();
    // Note that smart pointers have the same de-reference syntax
    borrow_function(*a);
}
```

Webots uses raw pointers to loan objects to us, which is why we do not
use smart pointers with webots code (to be clear, this is bad practice
by the writers of webots).

### Shared

A shared pointer allows for multiple objects to have access to the
pointer. This is done by keeping a count of the number of shared
pointers that point to an object with the object. When this count
reaches zero, both the counter and the object are deleted. This paradigm
of memory management is called “reference counting” and it’s a common
way of preventing leaks. Reference counting is the primary mechanism
used by python’s garbage collector to know what memory it can
deallocate.

Use of shared ownership is rare. Unique pointers are always preferred to
shared pointers.

``` cpp
#include <memory>
#include "A.hpp"
int main(){
    auto a = std::make_shared<A>(1);
    auto b = a;
}
```

    ## constructing A pointing to 0x5621bb045ed0
    ## destructing A, which pointed to 0x5621bb045ed0

### Weak

A weak pointer points to an object that is pointed to by a shared
pointer, but does not count towards the total count. When accessing the
object pointed to the weak pointer is temporarily upgraded to a shared
pointer. The main use of weak pointers is to get rid of circular
references for shared pointers, often found in loops.

``` cpp
#include <memory>
#include <iostream>
struct node_bad {
    std::shared_ptr<node_bad> other;
    node_bad(const std::shared_ptr<node_bad>& other_ = std::shared_ptr<node_bad>()) : other(other_) {
        std::cout << "Constructor " << this << std::endl;
    }
    ~node_bad() {
        std::cout << "Destructor " << this  << std::endl;
    }
};
struct node_good {
    std::weak_ptr<node_good> other;
    node_good(const std::weak_ptr<node_good>& other_ = std::weak_ptr<node_good>()) : other(other_) {
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

    ## Constructor 0x55a659393ec0
    ## Constructor 0x55a659394f00
    ## Constructor 0x55a659394f30
    ## Constructor 0x55a659394f60
    ## Destructor 0x55a659394f60
    ## Destructor 0x55a659394f30

### Deleter

For some legacy or c functions, there may be a custom delete function
for that type.

``` cpp
#include <memory>
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>
int main() {
    addrinfo hints;
    addrinfo* info;
    getaddrinfo("127.0.0.1", "80", &hints, &info);
    auto a = std::unique_ptr<addrinfo, decltype(&freeaddrinfo)>(info, freeaddrinfo);
}
```

### Task

1.  Store an integer in a unique pointer using `make_unique`

``` cpp
#include <memory>

int main(){
    // Put unique pointer here
}
```

# Standard Library Features

## Array

A `std::array` is similar to a c-style array (the “normal” type you
should already know about), but you specify the size of the array at
compile-time. [Reference
Page](https://en.cppreference.com/w/cpp/container/array).

## Vector

This is an array where we pass the size of the array at run-time. We can
also resize the vector by adding elements. [Reference
Page](https://en.cppreference.com/w/cpp/container/vector).

It has both a size and a capacity. Size is the number of current
elements and capacity is the current size of the underlying c array.
When the size reaches capacity, c++ will allocate more memory and
increase the capacity. Make sure that you try to reduce the number of
times that the capacity is changed, to prevent unnecessary re-allocation
performance overhead. One way to do this is to reserve space by
specifying a size in the constructor.

Removing and adding elements near the start of the vector is slow (see
[Reference Page](https://en.cppreference.com/w/cpp/container/vector) for
time complexities). Use a deque when you plan to do this often.

If you know Java, this is similar to `ArrayList`. It’s C++’s basic
dynamically sized array and it’s the first data structure you should
think of in most situations.

## Deque

A `std::deque` implements a deque (double-ended queue). The expansion of
capacity and insertion or removal of elements at the ends is much faster
than a vector, but takes up more memory, and accesses are slower.
Usually, a vector is better as we often do more reads than insertion and
removals.

## Map

This is an associative data structure which associates values to keys.
It can either be a red-black tree
([std::map](https://en.cppreference.com/w/cpp/container/map)), or a hash
map
([std::unordered_map](https://en.cppreference.com/w/cpp/container/unordered_map)).
Maps are bad for cache-locality and they should be avoided where
possible in performance critical code.
