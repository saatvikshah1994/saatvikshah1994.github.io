---
layout: post
title: ! "Templates: A 101 Reference"
description: "Basic Reference on Template {Categories, Default parameters, Type Deduction, Specialization(Full/Partial), Overloading, Instantiation, Miscellaneous}"
level: Beginner
image:
  feature: abstract-5.jpg
tags: [C++]
---

<center><h2 id="overview">Overview</h2></center>
Templates are a massive topic in C++ - one that is quite intimidating to beginners and novice programmers alike. This **101 Reference Guide** attempts to briefly cover a number of relevant subtopics with code samples.

<div class="toc">
<h2 id="outline">Outline</h2>
<li><a href="#categories">Template Categories</a></li>
<li><a href="#default">Default Template Parameters</a></li>
<li><a href="#typespec">Template Type Specification/Deduction</a></li>
<li><a href="#specialization">Template Full/Partial Specialization</a></li>
<li><a href="#overloading">Function Template Overloading</a></li>
<li><a href="#instantiation">Template Instantiation</a></li>
<li><a href="#misc">Template "What else?"</a></li>
<li><a href="#refs">References</a></li>
</div>

---
<center><h2 id="categories">Template Categories</h2></center>
The summarizing visual shows the different template categories available as of C++14:
<figure>
    <center><a href="{{ site.url }}/images/templates/TemplateCategories.png"><img src="{{ site.url }}/images/templates/TemplateCategories.png" alt="" height="80%" width="80%" align="middle" /></a></center>
</figure>
1. **Function Templates([Ref](https://en.cppreference.com/w/cpp/language/function_template), [Try Online!](https://coliru.stacked-crooked.com/a/646a6fb3864edd70))**: 
```c++
template <typename T>
void print(T elem) {
    // __PRETTY_FUNCTION__: Compiler provided pretty printer extension
    std::cout << __PRETTY_FUNCTION__ << std::endl;
}
// Calling Templated Fns
print(3); // void print(T) [with T = int]
print("abc"); // void print(T) [with T = const char*]
```
2. **Class Templates([Ref](https://en.cppreference.com/w/cpp/language/class_template), [Try Online!](https://coliru.stacked-crooked.com/a/0dd2ffb939fa5413))**: 
```c++
template <typename T>
struct Print{
    static void print_pretty_same(T elem) { // simple member function
        std::cout << __PRETTY_FUNCTION__ << std::endl;
        std::cout << elem << std::endl;
    }
    template <typename U>
    static void print_pretty_diff(U elem) { // templated member function
        std::cout << __PRETTY_FUNCTION__ << std::endl;
        std::cout << elem << std::endl;
    }
};
// Calling Template class Member Fns
Print<const char*>::print_pretty_same("123"); // static void Print<T>::print_pretty_same(T) [with T = const char*]
Print<int>::print_pretty_diff("123"); // static void Print<T>::print_pretty_diff(U) [with U = const char*; T = int]
```
Note that the case of class templates can get especially complicated due to further templated member functions. This is very commonly done when attempting to overload template member functions(discussed [here](https://saatvikshah1994.github.io/TemplateOverloadResolution/)).
3. **Variable Templates([Ref](https://en.cppreference.com/w/cpp/language/variable_template), [Try Online!](https://coliru.stacked-crooked.com/a/66f809ce5abfdd65)):**
```c++
template <typename T1, typename T2>
constexpr bool is_same_v = std::is_same<T1, T2>::value;
std::cout << is_same_v<int, int> << std::endl; // 1
```
4. **Alias Templates([Ref](https://en.cppreference.com/w/cpp/language/type_alias), [Try Online!](https://coliru.stacked-crooked.com/a/7c4bb79c642f48c0)):**
```c++
template <typename T>
using MyType = T;
MyType<int> x = 3;
 std::cout << x << std::endl; // 3
```
---
<center><h2 id="default">Default Template Parameters</h2></center>
Templates allow providing a default type to use if none is provided or deducible([Ref](https://en.cppreference.com/w/cpp/language/template_parameters#Default_template_arguments)). Here is an example of a function template([Try Online!](https://coliru.stacked-crooked.com/a/a45c83c7388238a3))
```c++
template <typename T, typename ToType = int>
void typePrinter(T elem) {
    std::cout << (ToType)(5) << std::endl;
}
typePrinter(5.23); // 5
```
Defaults are convenient and allow mixing with non-defaults as can be seen above.<br>
Default template parameters are also allowed for Class Templates([Try Online](https://coliru.stacked-crooked.com/a/0248e7be91c1053f)), Variable Templates([Try Online](https://coliru.stacked-crooked.com/a/2745c1c56d7eb611)) and Alias Templates([Try Online](https://coliru.stacked-crooked.com/a/bf92473600d2d382)). Note though, that for these 3, while calling the template one must append `<>` eg. `TemplateClass<> obj_;`. <br>
Additionally, when attempting to mix defaults with non-defaults for these 3, one must also be aware of certain restrictions which can be found at the start of the reference link above.

---
<center><h2 id="typespec">Template Type Specification/Deduction</h2></center>
Templates can either attempt to **implicitly deduce the types of its parameters**, or be **be explicitly provided those types**.
1. **Implicit Deduction([Ref](https://en.cppreference.com/w/cpp/language/template_argument_deduction), [Try Online](https://coliru.stacked-crooked.com/a/bf0731b3556ce0e6)):** 
```c++
template <typename T>
void adder(T elem1, T elem2) {
    std::cout << __PRETTY_FUNCTION__ << std::endl;
    std::cout << (elem1 + elem2) << std::endl;
}
adder(10, 15); // implicitly deduce T=int
```
The template can in the above example use the parameters passed to the function to deduce the template parameters. For example passing (10, 15) both of type int, helps the template deduce that T=int.
2. **Explicit Specification([Try Online](https://coliru.stacked-crooked.com/a/7a005bc3aec95f38))**:
```c++
template <typename T>
void castNumTo(int num) {
    std::cout << __PRETTY_FUNCTION__ << std::endl;
    std::cout << "Casted: " << static_cast<T>(num) << std::endl;
}
castNumTo<float>(23);
```
In the example here, not explicitly specifying the types by `<float>` will have the compiler unsure which type to cast to - it must be specified. Examples of Non-deducible cases are: `template_fn(var_typ1, var_type2)` or `template_fn()`.
3. **Partial Deduction([Try Online](https://coliru.stacked-crooked.com/a/45ac5176e6357e71)):** 
```c++
 // any types needing explicit specification come first
 // what can be deduced automatically can come second
template <typename ToType, typename T>
void adder(T elem1, T elem2) {
    std::cout << __PRETTY_FUNCTION__ << std::endl;
    std::cout << (ToType)(elem1 + elem2) << std::endl;
}
adder<int>(23.5, 0.23); // void adder(T, T) [with ToType = int; T = double]
```
In the above example, the type `T` can be deduced, but `ToType` must be explicitly specified. When needing to write a signature such that non-deducible/explicitly specified types come first and anything that is to be/can be deduced comes at the end. eg. `Template <typename NonDedType, typename DedType> fn(DedType);`.<br>
Template type deduction is supported for **Function Templates**([Try Online](https://coliru.stacked-crooked.com/a/1a94b1c8c5c815e8)) and **Class Templates**(post C++17, [Try Online](https://coliru.stacked-crooked.com/a/e5d5604d3540c184)). Not for **Variable/Alias Templates**.<br>


---
<center><h2 id="specialization">Template Full/Partial Specialization</h2></center>
This and the next section go into the rules of specifying custom implementations of a template based on certain criteria. Basically the implementations can go from generalized -> specialized. The most general form of the template class/function/etc is known as the **primary template**.<br>
Specializations can be full or partial. The order of preference when matching invocations to signatures is `Full Specialization -> Partial Specialization -> Primary Template`. We'll return to this topic in a later section. That said, let's dive into template specialization. <br>
1. **Template Full Specialization([Ref](https://en.cppreference.com/w/cpp/language/template_specialization)):**
    <figure>
        <center><a href="{{ site.url }}/images/templates/FullSpecialization.png"><img src="{{ site.url }}/images/templates/FullSpecialization.png" alt="" height="100%" width="100%" align="middle" /></a></center>
    </figure>
Basically, when invoking a template function/class/variable, the compiler will first try to match with fully specialized versions only falling back to the primary template when no match is found. In addition to the categories shown above, it is also possible to fully specialize template class member functions([covered here already](https://saatvikshah1994.github.io/TemplateOverloadResolution/)).<br>
Note how full specialization definitions must alwayes start with `template <>`. Let's see some more code samples now:
    - *Function Template Full Specialization*([Try Online](https://coliru.stacked-crooked.com/a/b4c0bb756fb6f772))
    ```c++
template <typename T> // primary template
void print_if_int32(T elem) {
    std::cout << "No int32 No print!" << std::endl;
}
template <>
void print_if_int32(int32_t elem) {
    std::cout << "Good type Me print: " << elem << std::endl;
}
print_if_int32(23); // Good Type
print_if_int32(23L); // No int32
print_if_int32("abc"); // No int32
    ```
    - *Class Template Full Specialization*([Try Online](https://coliru.stacked-crooked.com/a/f1be0ddec41b4743))
    ```c++
template <typename T> // primary
struct MyClassImpl{
    T member_;
    MyClassImpl(T member): member_(member) {}
    void print() { std::cout << "Non Int Type: " << member_ << std::endl; }
};
template <>
struct MyClassImpl<int>{
    int m1_;
    int m2_;
    MyClassImpl(int m1, int m2): m1_(m1), m2_(m2) {}
    void print() { std::cout << "Int Type!: addition is " << (m1_ + m2_) << std::endl; }
};
    ```
    - *Variable Template Full Specialization*([Try Online](https://coliru.stacked-crooked.com/a/f46c2d5920374805))
    ```c++
template<class T>
constexpr auto pi = T(3.1415926535897932385L);  // primary template
template<>
constexpr auto pi<double> = 3.1415926535897932385;  // FS
template<>
constexpr auto pi< const char*> = "Hello Pi!";  // FS
pi<double> // 3.14...
pi<int> // 3
pi<const char*> // Hello Pi
    ```
2. **Template Partial Specialization([Ref](https://en.cppreference.com/w/cpp/language/partial_specialization)):**
    <figure>
        <center><a href="{{ site.url }}/images/templates/PartialSpecialization.png"><img src="{{ site.url }}/images/templates/PartialSpecialization.png" alt="" height="100%" width="100%" align="middle" /></a></center>
    </figure>
    - Note that it will always have `<>` after template name.
    - **Class Template Partial Specialization:([Try Online](https://coliru.stacked-crooked.com/a/0bb482b597846b29))**
    ```c++
template <typename T>
struct MyClassImpl{ // Primary/Base template
    T member_;
    MyClassImpl(T member): member_(member) {}
    void print() { std::cout << member_ << std::endl; }
};
template <typename T>
struct MyClassImpl<T*>{ // Partial specialization: angle brackets after name
    T* member_;
    MyClassImpl(T* member): member_(member) {}
    void print() { std::cout << *member_ << std::endl; }
};
    ```
    - **Variable Template Partial Specialization:([Try Online](https://coliru.stacked-crooked.com/a/32db968c05e4821e))**
    ```c++
// Ref: https://stackoverflow.com/questions/41418432/variable-template-partial-specialization-and-constexpr
template<int M, int N>
const int gcd = gcd<N, M % N>;
template<int M>
const int gcd<M, 0> = M; // Partial specialization
gcd<9, 6> // 3
    ```
    - Not required for function templates because overloading does the trick!
    - No partially specialized member functions allowed too - Can only be done by partially specializing classes.([Ref](https://stackoverflow.com/questions/15374841/c-template-partial-specialization-member-function/15378899#15378899))

---
<center><h2 id="overloading">Function Template Overloading</h2></center>
Functions - be they free, member or templated - can all be overloaded([Reference](https://en.cppreference.com/w/cpp/language/function_template#Function_template_overloading)). In fact, overloading is used to obtain an identical effect as partial specialization in functions. 
<figure>
    <center><a href="{{ site.url }}/images/templates/MixingFunctionTemplateSpecializa.png"><img src="{{ site.url }}/images/templates/FunctionTemplateOverload.png" alt="" height="30%" width="30%" align="middle" /></a></center>
</figure>
Here is the corresponding code snippet([Try Online](https://coliru.stacked-crooked.com/a/b6a13f61dea55a2d)):
```c++
template <typename T>
bool is_pointer(T x) { // (1)
    return false;
}
template <typename Tp>
bool is_pointer(Tp* x) { // (2)
    return true;
}
```
Let's add another layer of complexity - **Mixing Overloads with Full specializations**. What happens during compilation is visualized below:
<figure>
    <center><a href="{{ site.url }}/images/templates/MixingFunctionTemplateSpecialization.png"><img src="{{ site.url }}/images/templates/MixingFunctionTemplateSpecialization.png" alt="" height="50%" width="50%" align="middle" /></a></center>
</figure>
Given this information, can you guess which version would be called for this code snippet:
```c++
template <typename T>
bool is_pointer(T x) { // (1)
    std::cout << "v1" << std::endl;
    return false;
}
template <>
bool is_pointer(int* x) { // (2)
    std::cout << "v2" << std::endl;
    return true;
}
template <typename Tp>
bool is_pointer(Tp* x) { // (3)
    std::cout << "v3" << std::endl;
    return true;
}
int elem = 3;
is_pointer(elem); // O1: ??
is_pointer(&elem); // O2: ??
```
The output of `O1: v1` and `O2: v3` - [Try it here](https://coliru.stacked-crooked.com/a/4eaf776c28290d7d). So basically matching against overloads happens first - here `(3)` is the only available and matching overload. `(2)` is a viable full specialization of `(3)`, but since it appears before `(3)`, it is treated as a full specialization of `(1)`. In summary:
(1) => Primary Template<br>
(2) => Full specialization of (1)<br>
(3) => Overload of (1)<br>
You can read more about this in this amazing [answer](https://stackoverflow.com/a/7108123/3656081) or this [talk](https://www.youtube.com/watch?v=NIDEjY5ywqU).

---
<center><h2 id="instantiation">Template Instantiation</h2></center>
> A class template by itself is not a type, or an object, or any other entity. No code is generated from a source file that contains only template definitions. In order for any code to appear, a template must be instantiated: the template arguments must be provided so that the compiler can generate an actual class (or function, from a function template). - [CPP Reference](https://en.cppreference.com/w/cpp/language/class_template)

In short, if a template is not instantiated you will not see the compiler output any assembly for it. For example this:
```c++
template <typename T>
T add(T num1, T num2) {
    return num1 + num2;
}
```
does not get compiled to any assembly by itself. Check it out [yourself](https://godbolt.org/z/_kwLJs).
<figure>
    <center><a href="{{ site.url }}/images/templates/noinstantiation.png"><img src="{{ site.url }}/images/templates/noinstantiation.png" alt="" height="50%" width="100%" align="middle" /></a></center>
</figure>

There are two ways in which assembly will be generated for this function:
1. **Implicit Instantiation**: The compiler will automatically generate an assembly language definition for the template, when it is invoked in some usage context where the complete type/definition is needed. [Here is an example.](https://godbolt.org/z/nufCYs):
```c++
int main() {
    add(5, 3); // complete definition needed here - implicitly instantiate
    return 0;
}
```
Here is how it looks on Compiler Explorer:
    <figure>
        <center><a href="{{ site.url }}/images/templates/implicitinstantiation.png"><img src="{{ site.url }}/images/templates/implicitinstantiation.png" alt="" height="50%" width="100%" align="middle" /></a></center>
    </figure>
2. **Explicit Instantiation**: An assembly language definition can also be forcefully generated by([Try it here!](https://godbolt.org/z/cM1PjZ)):
```c++
template int add(int, int);
// OR
template int add<>(int, int);
// OR
template int add<int>(int, int);
```
Here is how it looks on Compiler Explorer:
    <figure>
        <center><a href="{{ site.url }}/images/templates/explicitinstantiation.png"><img src="{{ site.url }}/images/templates/explicitinstantiation.png" alt="" height="50%" width="100%" align="middle" /></a></center>
    </figure>
One might wonder - why go about explicit instantiation, when implicit instantiation would happen anyways. There are two scenarios where it comes up:
    - *Separating Template declaration and definition*: This is extremely common for organizing declarations and definitions between header and cpp files to keep things readable. [Here is a code example](https://wandbox.org/permlink/LTFXW82vKJDN1HhM).
    - *Distributing templates in libraries*: Not so common scenario and found from a [MSVC Reference](https://docs.microsoft.com/en-us/cpp/cpp/explicit-instantiation?view=vs-2017) - Explicit Instantiation is useful when you are creating library (.lib) files that use templates for distribution.

---
<center><h2 id="misc">Template "What else?"</h2></center>
There are a few more topics which I haven't covered in this post, but make for useful learning:
1. **Variadic Templates**: Variadic templates allow for creation/evaluation of templates with a variable number of arguments. Eli Bendersky coverts this topic in a great amount of detail in his [blog post](https://eli.thegreenplace.net/2014/variadic-templates-in-c/).
2. **Template Type Deduction Rules**: While I covered a few items about this, I didnt go into a lot of detail about the exact rules behind type deduction. This is covered very well in the first few chapters of [Effective Modern C++](https://www.amazon.com/Effective-Modern-Specific-Ways-Improve/dp/1491903996).
3. **Template Member Function Overload Resolution**: Quite a mouthful huh! As we read, template functions(both simple and class member functions) can be customized to work differently based on the parameter type. I've written a [complete blog post](https://saatvikshah1994.github.io/TemplateOverloadResolution/) on this topic about a month back.
4. **Template Type Maps:** This is a useful trick to create a compile time type mapping which allows something like setting up the correct set of functions/types/etc based on what the template parameter type is. Here is an [example](https://coliru.stacked-crooked.com/a/9e6ef00ced74e5b6).

---
<center><h2 id="refs">References</h2></center>
- **[Template Normal Programming](https://www.youtube.com/watch?v=vwrXHznaYLA)**: Excellent introductory(light-moderate level) talk on templates which covers a whole breadth of topics.
- **[C++ Function Templates: How do they really work?](https://www.youtube.com/watch?v=NIDEjY5ywqU)**: The title says it all.
- **[C++ Template Syntax Patterns](https://eli.thegreenplace.net/2011/04/22/c-template-syntax-patterns)**: Another great introductory blog article.

---
That's all, thanks for reading :) - You can post a comment or reach out to me at *saatvikshah1994@gmail.com*.