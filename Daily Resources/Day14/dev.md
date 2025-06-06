# C++ Templates and CRTP: A Comprehensive Guide

## Introduction

Templates represent one of C++'s most powerful features, enabling generic programming and compile-time code generation.

## Understanding C++ Templates

### Template Fundamentals

Templates allow you to write code that works with different types without sacrificing type safety or performance. The compiler generates specific instances of template code for each type used, a process called template instantiation.

#### Function Templates

Function templates enable generic functions that can operate on multiple types:

```cpp
template <typename T> //We can choose any name for the type parameter, but `T` is conventional.
T maximum(T a, T b) {
    return (a > b) ? a : b;
}

// Usage
int max_int = maximum<int>(5, 10); //explicity specify type, T = int
double max_double = maximum(3.14, 2.71); // or we can skip it
auto max_double2 = maximum(6, 7.5); //Error: both arguments must be of the same type -> deduction failed
auto max_double3 = maximum<double>(6, 7.5); // T = double, 6 converted to a double
```

The compiler generates a separate version of `maximum` for each type used. This allows for type-safe operations without code duplication.

The code is generated by direct substitution of the required type. If the passed type does not implement some operation required by the template, the compiler will generate an (ugly) error message.

Take a look at this example - 
```cpp
class MyClass {

};

int main() {
    vector<MyClass> v(100);
    sort(v.start, v.end)
}
```

`MyClass` does not implemnt the `<` operator, required by the sort operation. The compiler will substitute the type and then fail to compile with this error message

<details>
    <summary>Error message</summary>

```
/usr/include/c++/13/bits/predefined_ops.h: In instantiation of ‘constexpr bool __gnu_cxx::__ops::_Iter_less_iter::operator()(_Iterator1, _Iterator2) const [with _Iterator1 = __gnu_cxx::__normal_iterator<MyClass*, std::vector<MyClass> >; _Iterator2 = __gnu_cxx::__normal_iterator<MyClass*, std::vector<MyClass> >]’:
/usr/include/c++/13/bits/stl_algo.h:1819:14:   required from ‘constexpr void std::__insertion_sort(_RandomAccessIterator, _RandomAccessIterator, _Compare) [with _RandomAccessIterator = __gnu_cxx::__normal_iterator<MyClass*, vector<MyClass> >; _Compare = __gnu_cxx::__ops::_Iter_less_iter]’
/usr/include/c++/13/bits/stl_algo.h:1859:25:   required from ‘constexpr void std::__final_insertion_sort(_RandomAccessIterator, _RandomAccessIterator, _Compare) [with _RandomAccessIterator = __gnu_cxx::__normal_iterator<MyClass*, vector<MyClass> >; _Compare = __gnu_cxx::__ops::_Iter_less_iter]’
/usr/include/c++/13/bits/stl_algo.h:1950:31:   required from ‘constexpr void std::__sort(_RandomAccessIterator, _RandomAccessIterator, _Compare) [with _RandomAccessIterator = __gnu_cxx::__normal_iterator<MyClass*, vector<MyClass> >; _Compare = __gnu_cxx::__ops::_Iter_less_iter]’
/usr/include/c++/13/bits/stl_algo.h:4861:18:   required from ‘constexpr void std::sort(_RAIter, _RAIter) [with _RAIter = __gnu_cxx::__normal_iterator<MyClass*, vector<MyClass> >]’
test.cpp:9:9:   required from here
/usr/include/c++/13/bits/predefined_ops.h:45:23: error: no match for ‘operator<’ (operand types are ‘MyClass’ and ‘MyClass’)
   45 |       { return *__it1 < *__it2; }
      |                ~~~~~~~^~~~~~~~
In file included from /usr/include/c++/13/bits/stl_algobase.h:67:
/usr/include/c++/13/bits/stl_iterator.h:1189:5: note: candidate: ‘template<class _IteratorL, class _IteratorR, class _Container> constexpr std::__detail::__synth3way_t<_IteratorR, _IteratorL> __gnu_cxx::operator<=>(const __normal_iterator<_IteratorL, _Container>&, const __normal_iterator<_IteratorR, _Container>&)’ (reversed)
 1189 |     operator<=>(const __normal_iterator<_IteratorL, _Container>& __lhs,
      |     ^~~~~~~~
/usr/include/c++/13/bits/stl_iterator.h:1189:5: note:   template argument deduction/substitution failed:
/usr/include/c++/13/bits/predefined_ops.h:45:23: note:   ‘MyClass’ is not derived from ‘const __gnu_cxx::__normal_iterator<_IteratorL, _Container>’
   45 |       { return *__it1 < *__it2; }
      |                ~~~~~~~^~~~~~~~
/usr/include/c++/13/bits/stl_iterator.h:1208:5: note: candidate: ‘template<class _Iterator, class _Container> constexpr std::__detail::__synth3way_t<_T1> __gnu_cxx::operator<=>(const __normal_iterator<_Iterator, _Container>&, const __normal_iterator<_Iterator, _Container>&)’ (rewritten)
 1208 |     operator<=>(const __normal_iterator<_Iterator, _Container>& __lhs,
      |     ^~~~~~~~
/usr/include/c++/13/bits/stl_iterator.h:1208:5: note:   template argument deduction/substitution failed:
/usr/include/c++/13/bits/predefined_ops.h:45:23: note:   ‘MyClass’ is not derived from ‘const __gnu_cxx::__normal_iterator<_Iterator, _Container>’
   45 |       { return *__it1 < *__it2; }
      |                ~~~~~~~^~~~~~~~
/usr/include/c++/13/bits/predefined_ops.h: In instantiation of ‘constexpr bool __gnu_cxx::__ops::_Val_less_iter::operator()(_Value&, _Iterator) const [with _Value = MyClass; _Iterator = __gnu_cxx::__normal_iterator<MyClass*, std::vector<MyClass> >]’:
/usr/include/c++/13/bits/stl_algo.h:1799:20:   required from ‘constexpr void std::__unguarded_linear_insert(_RandomAccessIterator, _Compare) [with _RandomAccessIterator = __gnu_cxx::__normal_iterator<MyClass*, vector<MyClass> >; _Compare = __gnu_cxx::__ops::_Val_less_iter]’
/usr/include/c++/13/bits/stl_algo.h:1827:36:   required from ‘constexpr void std::__insertion_sort(_RandomAccessIterator, _RandomAccessIterator, _Compare) [with _RandomAccessIterator = __gnu_cxx::__normal_iterator<MyClass*, vector<MyClass> >; _Compare = __gnu_cxx::__ops::_Iter_less_iter]’
/usr/include/c++/13/bits/stl_algo.h:1859:25:   required from ‘constexpr void std::__final_insertion_sort(_RandomAccessIterator, _RandomAccessIterator, _Compare) [with _RandomAccessIterator = __gnu_cxx::__normal_iterator<MyClass*, vector<MyClass> >; _Compare = __gnu_cxx::__ops::_Iter_less_iter]’
/usr/include/c++/13/bits/stl_algo.h:1950:31:   required from ‘constexpr void std::__sort(_RandomAccessIterator, _RandomAccessIterator, _Compare) [with _RandomAccessIterator = __gnu_cxx::__normal_iterator<MyClass*, vector<MyClass> >; _Compare = __gnu_cxx::__ops::_Iter_less_iter]’
/usr/include/c++/13/bits/stl_algo.h:4861:18:   required from ‘constexpr void std::sort(_RAIter, _RAIter) [with _RAIter = __gnu_cxx::__normal_iterator<MyClass*, vector<MyClass> >]’
test.cpp:9:9:   required from here
/usr/include/c++/13/bits/predefined_ops.h:98:22: error: no match for ‘operator<’ (operand types are ‘MyClass’ and ‘MyClass’)
   98 |       { return __val < *__it; }
      |                ~~~~~~^~~~~~~
/usr/include/c++/13/bits/stl_iterator.h:1189:5: note: candidate: ‘template<class _IteratorL, class _IteratorR, class _Container> constexpr std::__detail::__synth3way_t<_IteratorR, _IteratorL> __gnu_cxx::operator<=>(const __normal_iterator<_IteratorL, _Container>&, const __normal_iterator<_IteratorR, _Container>&)’ (reversed)
 1189 |     operator<=>(const __normal_iterator<_IteratorL, _Container>& __lhs,
      |     ^~~~~~~~
/usr/include/c++/13/bits/stl_iterator.h:1189:5: note:   template argument deduction/substitution failed:
/usr/include/c++/13/bits/predefined_ops.h:98:22: note:   ‘MyClass’ is not derived from ‘const __gnu_cxx::__normal_iterator<_IteratorL, _Container>’
   98 |       { return __val < *__it; }
      |                ~~~~~~^~~~~~~
/usr/include/c++/13/bits/stl_iterator.h:1208:5: note: candidate: ‘template<class _Iterator, class _Container> constexpr std::__detail::__synth3way_t<_T1> __gnu_cxx::operator<=>(const __normal_iterator<_Iterator, _Container>&, const __normal_iterator<_Iterator, _Container>&)’ (rewritten)
 1208 |     operator<=>(const __normal_iterator<_Iterator, _Container>& __lhs,
      |     ^~~~~~~~
/usr/include/c++/13/bits/stl_iterator.h:1208:5: note:   template argument deduction/substitution failed:
/usr/include/c++/13/bits/predefined_ops.h:98:22: note:   ‘MyClass’ is not derived from ‘const __gnu_cxx::__normal_iterator<_Iterator, _Container>’
   98 |       { return __val < *__it; }
      |                ~~~~~~^~~~~~~
/usr/include/c++/13/bits/predefined_ops.h: In instantiation of ‘constexpr bool __gnu_cxx::__ops::_Iter_less_val::operator()(_Iterator, _Value&) const [with _Iterator = __gnu_cxx::__normal_iterator<MyClass*, std::vector<MyClass> >; _Value = MyClass]’:
/usr/include/c++/13/bits/stl_heap.h:140:48:   required from ‘constexpr void std::__push_heap(_RandomAccessIterator, _Distance, _Distance, _Tp, _Compare&) [with _RandomAccessIterator = __gnu_cxx::__normal_iterator<MyClass*, vector<MyClass> >; _Distance = long int; _Tp = MyClass; _Compare = __gnu_cxx::__ops::_Iter_less_val]’
/usr/include/c++/13/bits/stl_heap.h:247:23:   required from ‘constexpr void std::__adjust_heap(_RandomAccessIterator, _Distance, _Distance, _Tp, _Compare) [with _RandomAccessIterator = __gnu_cxx::__normal_iterator<MyClass*, vector<MyClass> >; _Distance = long int; _Tp = MyClass; _Compare = __gnu_cxx::__ops::_Iter_less_iter]’
/usr/include/c++/13/bits/stl_heap.h:356:22:   required from ‘constexpr void std::__make_heap(_RandomAccessIterator, _RandomAccessIterator, _Compare&) [with _RandomAccessIterator = __gnu_cxx::__normal_iterator<MyClass*, vector<MyClass> >; _Compare = __gnu_cxx::__ops::_Iter_less_iter]’
/usr/include/c++/13/bits/stl_algo.h:1635:23:   required from ‘constexpr void std::__heap_select(_RandomAccessIterator, _RandomAccessIterator, _RandomAccessIterator, _Compare) [with _RandomAccessIterator = __gnu_cxx::__normal_iterator<MyClass*, vector<MyClass> >; _Compare = __gnu_cxx::__ops::_Iter_less_iter]’
/usr/include/c++/13/bits/stl_algo.h:1910:25:   required from ‘constexpr void std::__partial_sort(_RandomAccessIterator, _RandomAccessIterator, _RandomAccessIterator, _Compare) [with _RandomAccessIterator = __gnu_cxx::__normal_iterator<MyClass*, vector<MyClass> >; _Compare = __gnu_cxx::__ops::_Iter_less_iter]’
/usr/include/c++/13/bits/stl_algo.h:1926:27:   required from ‘constexpr void std::__introsort_loop(_RandomAccessIterator, _RandomAccessIterator, _Size, _Compare) [with _RandomAccessIterator = __gnu_cxx::__normal_iterator<MyClass*, vector<MyClass> >; _Size = long int; _Compare = __gnu_cxx::__ops::_Iter_less_iter]’
/usr/include/c++/13/bits/stl_algo.h:1947:25:   required from ‘constexpr void std::__sort(_RandomAccessIterator, _RandomAccessIterator, _Compare) [with _RandomAccessIterator = __gnu_cxx::__normal_iterator<MyClass*, vector<MyClass> >; _Compare = __gnu_cxx::__ops::_Iter_less_iter]’
/usr/include/c++/13/bits/stl_algo.h:4861:18:   required from ‘constexpr void std::sort(_RAIter, _RAIter) [with _RAIter = __gnu_cxx::__normal_iterator<MyClass*, vector<MyClass> >]’
test.cpp:9:9:   required from here
/usr/include/c++/13/bits/predefined_ops.h:69:22: error: no match for ‘operator<’ (operand types are ‘MyClass’ and ‘MyClass’)
   69 |       { return *__it < __val; }
      |                ~~~~~~^~~~~~~
/usr/include/c++/13/bits/stl_iterator.h:1189:5: note: candidate: ‘template<class _IteratorL, class _IteratorR, class _Container> constexpr std::__detail::__synth3way_t<_IteratorR, _IteratorL> __gnu_cxx::operator<=>(const __normal_iterator<_IteratorL, _Container>&, const __normal_iterator<_IteratorR, _Container>&)’ (reversed)
 1189 |     operator<=>(const __normal_iterator<_IteratorL, _Container>& __lhs,
      |     ^~~~~~~~
/usr/include/c++/13/bits/stl_iterator.h:1189:5: note:   template argument deduction/substitution failed:
/usr/include/c++/13/bits/predefined_ops.h:69:22: note:   ‘MyClass’ is not derived from ‘const __gnu_cxx::__normal_iterator<_IteratorL, _Container>’
   69 |       { return *__it < __val; }
      |                ~~~~~~^~~~~~~
/usr/include/c++/13/bits/stl_iterator.h:1208:5: note: candidate: ‘template<class _Iterator, class _Container> constexpr std::__detail::__synth3way_t<_T1> __gnu_cxx::operator<=>(const __normal_iterator<_Iterator, _Container>&, const __normal_iterator<_Iterator, _Container>&)’ (rewritten)
 1208 |     operator<=>(const __normal_iterator<_Iterator, _Container>& __lhs,
      |     ^~~~~~~~
/usr/include/c++/13/bits/stl_iterator.h:1208:5: note:   template argument deduction/substitution failed:
/usr/include/c++/13/bits/predefined_ops.h:69:22: note:   ‘MyClass’ is not derived from ‘const __gnu_cxx::__normal_iterator<_Iterator, _Container>’
   69 |       { return *__it < __val; }
```

</details>

##### Abbreviated Function Template Syntax
Since C++20, you can use auto in parameter name, each parameter will be assigned a different template type - 

```cpp
//these 2 functions are equivalent
template <typename T, typename U>
void print_pair(T first, U second) {
    std::cout << "First: " << first << ", Second: " << second << std::endl;
}

void print_pair(auto first, auto second) {
    std::cout << "First: " << first << ", Second: " << second << std::endl;
}
```

#### Class Templates

Class templates provide blueprints for creating classes that work with different types:

```cpp
template <typename T>
class Vector {
private:
    T* data;
    size_t size;
    size_t capacity;

public:
    Vector() : data(nullptr), size(0), capacity(0) {}
    
    void push_back(const T& value) {
        if (size >= capacity) {
            resize();
        }
        data[size++] = value;
    }
    
    T& operator[](size_t index) {
        return data[index];
    }
    
    const T& operator[](size_t index) const {
        return data[index];
    }
};

// Usage
Vector<int> int_vector;
Vector<std::string> string_vector;
```

### Template Specialization

Template specialization allows you to provide specific implementations for particular types or conditions.

#### Full Specialization

Complete specialization provides a specific implementation for a particular type:

```cpp
template <typename T>
class TypeInfo {
public:
    static const char* name() {
        return "Unknown";
    }
};

// Full specialization for int
template <>
class TypeInfo<int> {
public:
    static const char* name() {
        return "Integer";
    }
};

// Full specialization for double
template <>
class TypeInfo<double> {
public:
    static const char* name() {
        return "Double";
    }
};
```

An example of this in the STL is `std::vector`, which is defined specialised for `bool`. A single `bool` normally takes 1 byte, but in a `std::vector<bool>`, it is optimised to use a single bit per element. It achieves this by having an int array underneath and using bitwise operations to access individual bits.


#### Partial Specialization

Partial specialization allows specialization for a subset of template parameters:

```cpp
template <typename T, typename U>
class Pair {
    T first;
    U second;
public:
    void print() {
        std::cout << "Generic pair\n";
    }
};

// Partial specialization for pairs where both types are the same
template <typename T>
class Pair<T, T> {
    T first;
    T second;
public:
    void print() {
        std::cout << "Homogeneous pair\n";
    }
};
```


### C++20 Concepts

Concepts is a new feature in C++20 which basically put restrictions on generic types which should be fulfilled before substituting a type in a template. This allows for better error messages and more readable code.

This is an advanced topic and I'll just make a brief mention of it here. If you want to learn more, please refer to the [C++20 Concepts](https://en.cppreference.com/w/cpp/language/constraints) documentation.

Here is an example
```cpp
#include <concepts>

template <std::integral T>
T process_number(T value) {
    return value * 2;
}

template <std::floating_point T>
T process_number(T value) {
    return value * 1.5;
}

// Custom concepts
template <typename T>
concept Arithmetic = std::integral<T> || std::floating_point<T>;

template <Arithmetic T>
T calculate(T a, T b) {
    return a + b;
}

// More complex concept requirements
template <typename T>
concept Container = requires(T container) {
    container.begin();
    container.end();
    container.size();
    typename T::value_type;
};

template <Container C>
void process_container(const C& container) {
    for (const auto& item : container) {
        // Process each item
    }
}
```

There is a way to implement such a "static interface" in old version of C++ using SFINAE (Substitution Failure Is Not An Error) and `std::enable_if`, but it is quite complex and not recommended for beginners. Concepts make this much easier and more readable.

### Compile-time Computation

Templates can also accept compile time constants. This enables compile-time computation.

```cpp
template <int N>
struct Factorial {
    static constexpr int value = N * Factorial<N - 1>::value;
};

template <>
struct Factorial<0> {
    static constexpr int value = 1;
};

constexpr int fact_5 = Factorial<5>::value; // Computed at compile time
```

## The Curiously Recurring Template Pattern (CRTP)

### CRTP Fundamentals

The Curiously Recurring Template Pattern involves a class inheriting from a template base class, with the derived class itself as the template parameter. This pattern enables static polymorphism and compile-time method dispatch.

#### Basic CRTP Structure

```cpp
template <typename Derived>
class Base {
public:
    void interface() {
        static_cast<Derived*>(this)->implementation();
    }
    
    void common_functionality() {
        // Shared code among all derived classes
        std::cout << "Common behavior\n";
    }
};

class Derived : public Base<Derived> {
public:
    void implementation() {
        std::cout << "Derived implementation\n";
    }
};
```

The base class can call methods on the derived class through static_cast, achieving polymorphism without virtual function overhead.

### CRTP vs Virtual Functions

CRTP provides an alternative to virtual functions with different trade-offs:

#### Virtual Function Approach

```cpp
class VirtualBase {
public:
    virtual void process() = 0;
    virtual ~VirtualBase() = default;
};

class VirtualDerived : public VirtualBase {
public:
    void process() override {
        std::cout << "Virtual processing\n";
    }
};

void process_object(VirtualBase& obj) {
    obj.process(); // Runtime dispatch through vtable
}
```

#### CRTP Approach

```cpp
template <typename Derived>
class CRTPBase {
public:
    void process() {
        static_cast<Derived*>(this)->process_impl();
    }
};

class CRTPDerived : public CRTPBase<CRTPDerived> {
public:
    void process_impl() {
        std::cout << "CRTP processing\n";
    }
};

template <typename T>
void process_object(CRTPBase<T>& obj) {
    obj.process(); // Compile-time dispatch
}
```

The CRTP approach avoids the overhead of virtual function calls, and still allows the base class to call derived class methods.

### Further Reading

- [C++ Templates - cppreference.com](https://en.cppreference.com/w/cpp/language/templates)
- [C++20 Concepts - cppreference.com](https://en.cppreference.com/w/cpp/language/constraints)
- [Curiously Recurring Template Pattern (CRTP) - cppreference.com](https://en.cppreference.com/w/cpp/language/crtp)
- [SFINAE - cppreference.com](https://en.cppreference.com/w/cpp/language/sfinae)
