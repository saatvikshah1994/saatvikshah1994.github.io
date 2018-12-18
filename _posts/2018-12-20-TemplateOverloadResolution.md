---
layout: post
title: ! "Template Function Overload Resolution: A Quick Reference"
description: "Reference on methods for template function overload resolution + Comparison: Manual, SFINAE, Tag Dispatch and Misc. Covers template member and free functions."
level: Intermediate
image:
  feature: abstract-3.jpg
tags: [C++]
---

<div class="toc">
<h2 id="outline">Outline</h2>
<li><a href="#motivation">Motivation and Use Cases</a></li>
<li><a href="#fullspec">Explicit/Full Specialization</a></li>
<li><a href="#tagdispatch">Tag Dispatch</a></li>
<li><a href="#sfinae">SFINAE</a></li>
<li><a href="#refs">References</a></li>
</div>

---
<center><h2 id="motivation">Motivation and Use Cases</h2></center>
<figure>
    <center><a href="{{ site.url }}/images/tol/problem.png"><img src="{{ site.url }}/images/tol/problem.png" alt="" height="90%" width="90%" align="middle" /></a></center>
</figure>
[SFMT and dSFMT](http://www.math.sci.hiroshima-u.ac.jp/~m-mat/MT/SFMT/) provides utilities to fill an array with random numbers using SIMD.  This provides an excellent opportunity to build faster Random number generators than those offered by the STL. The only problem is that the vectorized RNGs offered are only for certain types. The problem is visualized in the figure above. Let's say we have a general purpose templated matrix class and we want our `Matrix<T>::random()` to automatically select the the most efficient implementation based on type `T`. So our aim is to come up with a way to overload either member-function or free-function templates in the best way.

---
<center><h2 id="fullspec">Explicit/Full Specialization</h2></center>
1. **Code Snippet**: ([Try Online!](https://coliru.stacked-crooked.com/a/1af53bd052230fa8))
```c++
template <typename T>
struct Foo{
    void method();
};
//
template <typename T>
void Foo<T>::method() {
    std::cout << "default method " << std::endl;
}
// 
template <>
void Foo<int32_t>::method() {
    std::cout << "int32 method " << std::endl;
}
//
template <>
void Foo<int64_t>::method() {
    std::cout << "int64 method " << std::endl;
}
```
2. **Pros:** Extremely simple to implement.
3. **Cons:** Requires manual repetition if needing to implement the same functionality across different types. For eg. with the problem introduced in the motivation above `fill_array(SFMT)` would have to be rewritten twice for `uint32_t/uint64_t`.

---
<center><h2 id="tagdispatch">Tag Dispatch</h2></center>
1. **Code Snippet:**(Online link in next bullet)
```c++
template <typename T>
struct Foo{
    void unary_td(std::true_type) {
        std::cout << "integral type!" << std::endl;
    }
    void unary_td(std::false_type) {
        std::cout << "non-integral type!" << std::endl;
    }
    
    void unary_td() {
        unary_td(std::is_integral<T>{});
    }
};
```
2. **Variations/Examples**:
    * [Unary Tag Dispatch](https://coliru.stacked-crooked.com/a/acee110d9b8af275)
    * [Binary Tag Dispatch](https://coliru.stacked-crooked.com/a/0ce7f5fbf05222fd)
    * [NAry Tag Dispatch](https://coliru.stacked-crooked.com/a/0f2bbfc9d2435348)
3. **Pros:** For simple overload resolution conditions tag dispatch is easily the most readable and simple while avoiding repetition. In short, the perfect first choice.
4. **Cons:** For complex conditions(as can be seen as we go from Unary to Nary tag dispatch) the complexity of tag dispatch greatly increases. Additionally, based on the template type, conditional disabling of other functions or overloads is not possible. More on this in the references.

---
<center><h2 id="sfinae">SFINAE: Substitution Failure is not an error</h2></center>
When calling a templated function, the compiler will form a set of candidate overloads and try each one in turn from best to worst matching. As suggested in this title, any failure to match any specific overload signature to the function call is not treated as an error. If matches on all overload candidates fail, its an error.
1. **Code Snippet:**(Online link in next bullet)
```c++
template <typename U>
auto sfinae() -> std::enable_if_t<!std::is_same<U, int>::value && !std::is_same<U, float>::value> {
    std::cout << "sfinae default" << std::endl;
}
//
template <typename U>
auto sfinae() -> std::enable_if_t<std::is_same<U, int>::value> {
    std::cout << "sfinae int" << std::endl;
}
// 
template <typename U>
auto sfinae() -> std::enable_if_t<std::is_same<U, float>::value> {
    std::cout << "sfinae float" << std::endl;
}
```
2. **Variations/Examples**:
    * Templated Member Functions
        * [Enable_if in Template](https://coliru.stacked-crooked.com/a/bc08035924d3582e)
        * [Enable_if in Return Type](https://coliru.stacked-crooked.com/a/d3438ee0621dab10)
        * [Enable_if in Function](https://coliru.stacked-crooked.com/a/45dba741ddf2d4a0)
    * Templated Free Functions
        * [Enable_if in Template](https://coliru.stacked-crooked.com/a/2b147e1a2d208881)
        * [Enable_if in Return Type](https://coliru.stacked-crooked.com/a/7edcbcfe5f5a5916)
        * [Enable_if in Function](https://coliru.stacked-crooked.com/a/ba1639ababa9008c)
    * [Catch-All Technique](https://coliru.stacked-crooked.com/a/b5d7ec1e029df65b) - more details [here](https://stackoverflow.com/a/53717698/3656081)
3. **Pros:** Allows conditionally disabling overloads, Potentially simpler when needing overloading resolution on complex conditions.
4. **Cons:** Clearly, it can be tricky to implement.

---
<center><h2 id="refs">References and Further Reading</h2></center>
This post was more of a quick summary on the different techniques involved in template function overload resolution. For more detailed reading you can refer to the following:
1. [**Foonathan Series on Overload Resolution**](https://foonathan.net/blog/2015/10/16/overload-resolution-1.html): Covers all the methods here and more in a lot of detail.
2. **Usefulness of SFINAE over others:** As discussed earlier this is covered in [this](https://foonathan.net/blog/2016/12/21/conditionally-removing-functions.html) and [this](https://blog.rmf.io/cxx11/to-sfinae-or-not-to-sfinae).
3. [Macros for Overload Resolution](https://github.com/shogun-toolbox/shogun/blob/develop/src/shogun/lib/SGVector.cpp#L415) uses Macros for explicit specialization of matrix equality for floating point types while typing substantially lesser.<br>
Also, the final solution for the problem I mentioned above was by using [SFINAE coded up here](https://github.com/shogun-toolbox/shogun/pull/4437).

---
That's all, thanks for reading :) - You can post a comment or reach out to me at *saatvikshah1994@gmail.com*.