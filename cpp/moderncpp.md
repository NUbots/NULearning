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

std::array<5, std::string> letters {"a", "b", "c", "d", "e"};

// Range-based for loops automatically create iterators for containers
// which support them, such as std::array, and dereference them for us. 
// You should usually use `const auto&`, `auto&` (or `auto&&`) for these 
// variables, because they're huge, ugly, std::iterator types which are 
// hard to read
for (const auto& letter : letters) {
    std::cout << letter << std::endl;
}
```

    ## /home/cameron/Documents/NULearning/cpp/.tmp_c/jG0Tk.cpp:5:12: error: template argument for template type parameter must be a type
    ## std::array<5, std::string> letters {"a", "b", "c", "d", "e"};
    ##            ^
    ## /usr/bin/../lib64/gcc/x86_64-pc-linux-gnu/11.1.0/../../../../include/c++/11.1.0/array:94:21: note: template parameter is declared here
    ##   template<typename _Tp, std::size_t _Nm>
    ##                     ^
    ## /home/cameron/Documents/NULearning/cpp/.tmp_c/jG0Tk.cpp:12:1: error: expected unqualified-id
    ## for (const auto& letter : letters) {
    ## ^
    ## 2 errors generated.

### DO NOT USE `auto` FOR EIGEN TYPES

Eigen’s docs list using the `auto` keyword for their types as a [common
pitfall](https://eigen.tuxfamily.org/dox/TopicPitfalls.html). This is
mostly because Eigen uses intermediate types all the time with many
implicit conversions, which are not at all obvious to the reader.

### Task

1.  Write a program which uses `auto` to define a string, then use auto
    to define a custom class or struct type `T`.
2.  Add a range-based for loop to the program, iterating through a
    `std::vector<T>` of your class type. `std::vector` is a dynamically
    sized array, which is very similar to Java’s `ArrayList`. It’s very
    good and very fast - we use it all the time when we have to use the
    heap (we stick to using the stack whenever possible though). Use
    `emplace_back` to construct the `T` instances as you add them to the
    `std::vector`.

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
These can make things more concise, but we don’t use them often, for the
sake of clarity over conciseness.

``` cpp
    // Adds this name into this namespace. We do use these often, to import function
    // names, so we don't have huge, long function names within four nested namespaces
    using kev::FooLongName::getAlternativeConfig;

    class A {
        const int a = 0; // Can still be given another value by a constructor, by using
                         // an initializer list

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
}
```

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

Firstly we’ll create an object that owns some heap memory and will print
out all the common functions that deal with ownership.

### Attributes

``` cpp
struct A {
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

    ## constructing A pointing to 0x55c4eb97ceb0
    ## destructing A, which pointed to 0x55c4eb97ceb0

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

    ## constructing A pointing to 0x556a5951feb0
    ## copy construction, old 0 other's 0x556a5951feb0 new 0x556a59520ee0
    ## destructing A, which pointed to 0x556a59520ee0
    ## destructing A, which pointed to 0x556a5951feb0

### Copy Assignment

Similar to copy construction, but if this object has ownership of
something we must first clean that up.

``` cpp
// Copy assignment
A& operator=(const A& other) {
    std::cout << "copy assignment, old " << x
    << " other's " << other.x;
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

    ## constructing A pointing to 0x561236639eb0
    ## constructing A pointing to 0x56123663aee0
    ## copy assignment, old 0x56123663aee0 other's 0x561236639eb0 new 0x56123663af00
    ## destructing A, which pointed to 0x56123663af00
    ## destructing A, which pointed to 0x561236639eb0

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

    ## constructing A pointing to 0x56193504feb0
    ## move construction, old 0 other's 0x56193504feb0 new 0x56193504feb0
    ## destructing A, which pointed to 0x56193504feb0
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

    ## constructing A pointing to 0x55a66207eeb0
    ## move assignment, old 0 other's 0x55a66207eeb0 new 0x55a66207eeb0
    ## destructing A, which pointed to 0x55a66207eeb0
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

    ## constructing A pointing to 0x56529c8a3ed0
    ## constructing A pointing to 0x56529c8a4f20
    ## destructing A, which pointed to 0x56529c8a4f20
    ## constructing A pointing to 0x56529c8a4f20
    ## destructing A, which pointed to 0x56529c8a4f20
    ## destructing A, which pointed to 0x56529c8a3ed0

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

    ## /home/cameron/Documents/NULearning/cpp/.tmp_c/Kxu74.cpp:5:24: error: call to deleted constructor of 'std::unique_ptr<A>'
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

    ## constructing A pointing to 0x5594263fced0
    ## destructing A, which pointed to 0x5594263fced0

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

    ## constructing A pointing to 0x55e0ebe23ed0
    ## constructing A pointing to 0x55e0ebe24f20
    ## destructing A, which pointed to 0x55e0ebe24f20

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

    ## constructing A pointing to 0x55edce5bbed0
    ## destructing A, which pointed to 0x55edce5bbed0

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

    ## Constructor 0x559c91e10ec0
    ## Constructor 0x559c91e11f00
    ## Constructor 0x559c91e11f30
    ## Constructor 0x559c91e11f60
    ## Destructor 0x559c91e11f60
    ## Destructor 0x559c91e11f30

### Deleter

# Standard Library Features

# C++ Feautres
