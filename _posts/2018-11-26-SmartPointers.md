---
layout: post
title: ! "Smart Pointers: The Wild West Tour(Part 1)"
description: "Motivation, Categorization, What to Choose?, Scoped and unique pointers in-depth including Performance and Microbenchmarks, Thread-Safety, Passing them around and more."
image:
  feature: smartptr/header.jpg
tags: [C++]
---

## Overview
Smart Pointers are a construct in C++ to aid programmers in tackling **memory management**. Beyond motivation and basics, the aim of this post is to understand potential use cases, performance tradeoffs and standard protocols to follow for each smart pointer type.<br>
**The first part introduces smart pointers and discusses single ownership constructs in detail**. If you have some background, feel free to jump to the topic of interest from the table of contents below.

<div class="toc">
<h2 id="outline">Outline</h2>
<li><a href="#motivation">Motivation: System vs. Garbage</a></li>
<li><a href="#categorization">Categorization: The Taxonomy of Smart Pointers</a>
<ul>
<li><a href="#choices">Which to Choose?</a></li>
</ul>
</li>
<li><a href="#scopedptr">Single Ownership Construct #1: Scoped Pointers</a>
<ul>
<li><a href="#scoped_usecase">General Use Case</a></li>
<li><a href="#scoped_size">Performance(size)</a></li>
<li><a href="#scoped_speed">Performance(speed)</a></li>
<li><a href="#scoped_threadsafety">Thread Safety</a></li>
<li><a href="#scoped_movingaround">Passing Them Around</a></li>
<li><a href="#scoped_variants">Standard Variants</a></li>
</ul>
</li>
<li><a href="#uniqptr">Single Ownership Construct #2: Unique Pointers</a>
<ul>
<li><a href="#uniq_usecase">General Use Case</a></li>
<li><a href="#uniq_size">Performance(size)</a></li>
<li><a href="#uniq_speed">Performance(speed)</a></li>
<li><a href="#uniq_threadsafety">Thread Safety</a></li>
<li><a href="#uniq_movingaround">Passing Them Around</a></li>
<li><a href="#uniq_objvariant">Object Variants</a></li>
<li><a href="#uniq_customdeleters">Custom Deleters</a></li>
</ul>
</li>
</div>

---
<center><h2 id="motivation">Motivation: System vs. Garbage</h2></center>
<figure>
    <center><a href="{{ site.url }}/images/smartptr/shootout.jpg"><img style="border-radius:40%;" src="{{ site.url }}/images/smartptr/shootout.jpg" alt="" height="400px" width="400px" align="middle" /></a></center>
</figure>

As depicted in the historical picture, there is an ever-ongoing conflict between the systems written by programmers and the garbage they generate. This is especially true for C++ which has given the reins of memory management to the programmers with the promise of efficiency in return. Failure to correctly do so leads to either [memory leaks](https://en.wikipedia.org/wiki/Memory_leak) or [dangling references](https://en.wikipedia.org/wiki/Dangling_pointer). As a result, in the pre-smart pointer era, C++ programs looked something like the image on the left:

<figure>
    <center><a href="{{ site.url }}/images/smartptr/motivation.png"><img src="{{ site.url }}/images/smartptr/motivation.png" alt="" height="100%" width="80%" align="middle" /></a></center>
</figure>

Modern C++(image on right) via smart pointers allows us to get rid of the complete **Cleanup** step. So programmers can now allocate necessary dynamic objects and operate with them without having to worry about cleaning them up. The best part of this is that these smart pointers have **minimum(zero in many cases) overhead** of performing the cleanup.

---
<center><h2 id="categorization">Categorization: The Taxonomy of Pointers</h2></center>
<figure>
    <center><a href="{{ site.url }}/images/smartptr/taxonomy.png"><img src="{{ site.url }}/images/smartptr/taxonomy.png" alt="" height="100%" width="80%" align="middle" /></a></center>
</figure>

1. **Single Owner**: A given object has **ONLY** one owner. Once the owner goes out of scope the object will be deleted. We will discuss: `boost::scoped_ptr`, `std::unique_ptr`.
2. **Multiple Owner**: A single object having sharing/multiple owners. Once all the owners go out of scope the object will be deleted. A **Control Block** is the logical component of such a smart pointer which helps determine when all the owners have gone out of scope so the object can be cleaned. This can be further categorized into:
    1. **Intrusive Control Block**: The control block is inside the object pointed to. We will discuss `boost::intrusive_ptr`.
    2. **Non Intrusive Control Block**: The control block is external to the object pointed to. It can be laid side by side(cache-friendly) with the object or at a completely different location. We will discuss `std::shared_ptr`.

<center><h4 id="choices">Which to Choose?</h4></center>
Surely, you must be scratching your head about which is the right one to choose. Fortunately, Herb Sutter gave an excellent talk on this topic ["Leak Freedom in C++ by Default"](https://www.youtube.com/watch?v=JfmTagWcqoE) accompanied with a poster shown below:
<figure>
    <center><a href="{{ site.url }}/images/smartptr/preference_order.png"><img src="{{ site.url }}/images/smartptr/preference_order.png" alt="" height="100%" width="80%" align="middle" /></a></center>
</figure>
I've augmented the poster with two additional pointer types not covered in the talk at the preference order where I feel they appropriately belong. In summary the order would be:
`locals/members` > `scoped pointer` > `unique pointer` > `shared_ptr` > `intrusive_ptr`<br>
For `locals/members` it is clear since C++ will automatically manage its lifetime by default.
```c++
{ // create a new scope
std::string my_horse = "arabian horse"; // add a local in the scope
}
std::cout << my_horse << std::endl; // not available
```
Now, let's go through the others. As a matter of convenience, I'll be using a sample class:
```c++
class WildWestGang{
    // Note, that std::cout's will be disabled during performance testing.
    public:
        explicit WildWestGang(std::vector<std::string> members): m_members(members) { 
          std::cout << "Birth of the Gang!" << std::endl;
        };
        void Print() {
            for(const auto& mem: m_members) {
                std::cout << mem << " ";
            }
            std::cout << std::endl;
        }
        ~WildWestGang() { std::cout << "Gang is dead!" << std::endl; }
    private:
        std::vector<std::string> m_members;
};
```

---
<center><h3 id="scopedptr">Scoped Pointers</h3></center>
<figure>
    <center><a href="{{ site.url }}/images/smartptr/ScopedPtrHeader.png"><img style="border-radius:40%;" src="{{ site.url }}/images/smartptr/ScopedPtrHeader.png" alt="" height="400px" width="400px" align="middle" /></a></center>
</figure>
Scoped Pointers are meant for *Single Fixed Ownership* - an object can only have a **single owner** which **cannot** be changed. As soon as this owner goes out of scope, its Game Over for both you and the object you owned! Let's see an example([Try Online](https://coliru.stacked-crooked.com/a/e55f55c25a058886)):
```c++
auto my_gang = new WildWestGang({"Dutch", "Arthur", "John"}); // Birth of the Gang!
{
    auto p1 = scoped_ptr<WildWestGang>(my_gang);
    p1->Print(); // Dutch Arthur John
} // Gang is dead!
std::cout << "The End" << std::endl;
```
Now let's try to change ownership - We can try either copying or moving it([Try Online](https://coliru.stacked-crooked.com/a/e0c0c41ea33cfa44)):
```c++
auto p = scoped_ptr<WildWestGang>(new WildWestGang({"Dutch", "Arthur", "John"}));
p->Print(); // Dutch Arthur John 
auto p2 = p; // error
auto p3 = std::move(p); // error
```
The error message depends on the scoped_ptr variant - for example with `boost::scoped_ptr` you would get something like `error: scoped_ptr(scoped_ptr const &); delared private here`.<br>
That said, while the owner can't be changed, the object owned/pointed to can be after deletion of the original object. Let's see an example([Try Online](https://coliru.stacked-crooked.com/a/71077a0b97924a87)):
```c++
{
    auto p = scoped_ptr<WildWestGang>(new WildWestGang({"Dutch", "Arthur", "John"})); // Birth of...
    std::cout << "p currently owns the Wild West Gang: " << std::endl;
    p->Print(); // Dutch Arthur John
    p.reset(new WildWestGang({"Micah", "Bill", "Javier"})); // delete original, point to new
    std::cout << "p now owns the Wild West Gang: " << std::endl;
    p->Print(); // Micah Bill Javier
} // Gang is dead!
```
Let's go over some of its properties now:
1. <b id="scoped_usecase">General Use Case</b>: Stating intent that there is a single owner with no intention to change the owner.(For more, read [here](https://www.boost.org/doc/libs/1_68_0/libs/smart_ptr/doc/html/smart_ptr.html#scoped_ptr_rationale)).
2. <b id="scoped_size">Performance(size)</b>: See below for how a scoped pointer physically looks like(with no custom deleters - we'll come back to this in the `unique_ptr` discussion). There are no extra data members introduced on the end of scoped object, which means that no additional overhead is incurred in terms of size. In case its unclear why the addition of the extra scoped pointer logic/methods is zero size overhead, you can refer to [determining size of a class](https://www.cprogramming.com/tutorial/size_of_class_object.html) and [why methods dont add to size of a class](https://stackoverflow.com/a/8058257/3656081):
    <figure>
        <center><a href="{{ site.url }}/images/smartptr/ScopedPtrDiag.png"><img src="{{ site.url }}/images/smartptr/ScopedPtrDiag.png" alt="" height="20%" width="20%" align="middle" /></a></center>
    </figure>
And here is a simple `sizeof` output([Try Online](https://coliru.stacked-crooked.com/a/4d88106888fb7c8f)):<br>
```c++
auto my_gang = new WildWestGang({"Dutch", "Arthur", "John"});
std::cout << sizeof(my_gang) << std::endl; // 8
auto p = scoped_ptr<WildWestGang>(my_gang);
std::cout << sizeof(p) << std::endl; // 8
```
3. <b id="scoped_speed">Performance(speed)</b>: It is a **Zero Overhead Abstraction**. As long as there are no custom deleters involved, the performance in terms of speed is equal to that of a raw pointer. Here is the Quick-Bench output for Efficiency([Try Online](http://quick-bench.com/ZC6H1D3nXeF8G-qnzfsrHT8Dtbw)):
    <figure>
        <center><a href="{{ site.url }}/images/smartptr/scoped_ptr_performance.png"><img src="{{ site.url }}/images/smartptr/scoped_ptr_performance.png" alt="" height="100%" width="80%" align="middle" /></a></center>
    </figure>
    
4. <b id="scoped_threadsafety">Thread Safety:</b> There are 2 aspects to consider here: 
    1. **Thread safety when accessing the owned object**: A `scoped_ptr` can be thought of as similar to a raw pointer - the dereferencing operators dont add additional mutexes or thread safe mechanisms. Thus access to the owned object will not be thread safe if the same object owned by a `scoped_ptr` is written to by multiple threads.
    2. **Thread safety when writing to the `scoped_ptr`**: i.e. the `reset()` call is also not thread safe. i.e. if the same `scoped_ptr` is written to by different threads, it is undefined behavior.
5. <b id="scoped_movingaround">Passing them around:</b> The diagram below represents the different ways of passing around scoped pointers:
    <figure>
        <center><a href="{{ site.url }}/images/smartptr/ScopedPtrMovingAround.png"><img src="{{ site.url }}/images/smartptr/ScopedPtrMovingAround.png" alt="" height="80%" width="80%" align="middle" /></a></center>
    </figure>
    Note that the **Factory** will not work for scoped pointers unless we use C++17 since we need [guaranteed copy elision](https://en.cppreference.com/w/cpp/language/copy_elision). Also note that **Reseat** also destroys the originally pointed to object. Here are some sample functions to help explain the above choices([Try Online](https://coliru.stacked-crooked.com/a/8d61807120a88c48)):
```c++
scoped_ptr<WildWestGang> createWildWestGang() {
  return scoped_ptr<WildWestGang>(new WildWestGang({"Dutch", "Arthur", "John"}));
}
void useWildWestGang(WildWestGang& g) {
  g.Print();
}
void reseatWildWestGang(scoped_ptr<WildWestGang>& gang_ptr) {
  gang_ptr.reset(new WildWestGang({"Bill", "Lenny", "Charles"}));
}
```
  There could be alternative ways to pass around scoped pointers, but the methods mentioned here adhere strongly to the [CPP Guidelines](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-resource).
6. <b id="scoped_variants">Standard Variants:</b> `boost::scoped_ptr`(pointer type), `const std::unique_ptr`(pointer/dynamic array type) and `boost::scoped_array`(dynamic array type). I've discussed the dynamic array vs. pointer type <a href="#uniq_objvariant">later</a>.  Here is how `const std::unique_ptr` can be correctly applied as a `scoped_ptr`:
```c++
template<typename T>
using scoped_ptr = const std::unique_ptr<T>;
```
If you have the freedom to use `boost::scoped_ptr`, I would recommend preferring it as the `unique_ptr` variant needs to be correctly qualified with `const` or aliased.<br>

**Note:** Well, there is one way to transfer ownership - the use of the `scoped_ptr<T>.swap(scoped_ptr<T> &)` member function which swaps the contents of two scoped pointers. So essentially a **barter system**. That said, the rationale of using scoped pointers to denote **Single Fixed Ownership** still applies.

---
<center><h3 id="uniqptr">Unique Pointers</h3></center>
<figure>
    <center><a href="{{ site.url }}/images/smartptr/UniquePtrHeader.png"><img style="border-radius:40%;" src="{{ site.url }}/images/smartptr/UniquePtrHeader.png" alt="" height="400px" width="400px" align="middle" /></a></center>
</figure>
A `unique_ptr` follows the norm of **Single <strike>Fixed</strike> Ownership**: Yes! I've cut the **Fixed**. It is only guaranteed that the object has a single owner. We don't care who the owner is and can transfer the ownership as need be. However when the current owner goes out of scope, the object will be deleted. Essentially, it is a scoped pointer with three additional properties:
1. Ownership can be transferred using `std::move`.
2. A Custom deleter can be allocated. We will understand custom deleters in detail in another <a href="#uniq_customdeleters">section</a>.
3. A convenience function `std::make_unique<Obj>(params)` to avoid the repeating **Obj** in `smart_ptr<Obj>(new Obj(params))`. This is purely a cosmetic change with no performance implications.

Here, is a code snippet to summarize the above([Try Online](https://coliru.stacked-crooked.com/a/49b0ff67ccf47ea6)):
```c++
auto names = std::vector<std::string>{"Arthur", "Dutch", "John"};
{ // Demonstrate automatic destruction(similar to scoped_ptr)
  auto p = std::unique_ptr<WildWestGang>(new WildWestGang(names)); // constructor
  p->Print(); // Arthur Dutch John
} // automatic destructor call
{ // New unique_ptr features
  auto p = std::make_unique<WildWestGang>(names); // shorter std::make_unique. Yay!
  p->Print(); // Arthur Dutch John
  auto p2 = std::move(p); // Transfer ownership by move ONLY!
  p2->Print();  // Arthur Dutch John
  std::cout << "Is p NULL: " << (p == nullptr) << std::endl; // Yep!
} // automatic destructor call on p2
```
Now, let's have a look at its properties:
1. <b id="uniq_usecase">General Use Case</b>: **Single Transferrable Ownership**
2. <b id="uniq_size">Performance(size)</b>: Below is a diagram showing the physical layout of a `unique_ptr`.
    <figure>
        <center><a href="{{ site.url }}/images/smartptr/unique_ptr_physical.png"><img src="{{ site.url }}/images/smartptr/unique_ptr_physical.png" alt="" height="100%" width="70%" align="middle" /></a></center>
    </figure>
As can be seen, without a custom deleter, the `unique_pointer` physically looks exactly like a `scoped_ptr`. So as expected:
```c++
std::cout << sizeof(int*) << std::endl;                 // 8
std::cout << sizeof(std::unique_ptr<int>) << std::endl; // 8
```
3. <b id="uniq_size">Performance(speed)</b>:It is a **Zero Overhead Abstraction**. As long as there are no custom deleters involved, the performance in terms of speed is equal to that of a raw pointer. Here is the Quick-Bench output for Efficiency([Try Online](http://quick-bench.com/N1c60r4Yr7maBbL2eqBGwbjTYCI)):
    <figure>
        <center><a href="{{ site.url }}/images/smartptr/uniq_ptr_speed.png"><img src="{{ site.url }}/images/smartptr/uniq_ptr_speed.png" alt="" height="100%" width="70%" align="middle" /></a></center>
    </figure>
4. <b id="uniq_threadsafety">Thread Safety:</b> Same rules as `scoped_ptr` - Refer <a href="#scoped_threadsafety">here</a>. The `std::move` ownership transfer op is also not thread safe.
5. <b id="uniq_movingaround">Passing `uniq_ptr` around:</b> The diagram below represents the different ways of passing around unique pointers as provided by the [CPP Guidelines](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-resource):
    <figure>
        <center><a href="{{ site.url }}/images/smartptr/UniquePtrMovingAround.png"><img src="{{ site.url }}/images/smartptr/UniquePtrMovingAround.png" alt="" height="100%" width="100%" align="middle" /></a></center>
    </figure>
Here is sample code demonstrating the same([Try Online](https://coliru.stacked-crooked.com/a/799574a2e87b3b8c)):
```c++
std::unique_ptr<WildWestGang> createWildWestGang() {
    auto names = std::vector<std::string>{"Dutch", "Arthur", "John"};
    return std::make_unique<WildWestGang>(names);
}
void useWildWestGang(WildWestGang& g) {
    g.Print();
}
void reseatWildWestGang(std::unique_ptr<WildWestGang>& gang_ptr) {
    gang_ptr.reset(new WildWestGang({"Bill", "Lenny", "Charles"}));
}
void sink(std::unique_ptr<WildWestGang> gang_ptr) {
    // do additional cleanup ops here
    std::cout << "Time for cleanup!" << std::endl;
    gang_ptr.reset();
}
```
6. <b id="uniq_objvariant">Object variants</b>: `std::unique_ptr` can be applied to either raw pointer(`T*`) or dynamic array(`T[]`). The cool part of this is that the set of operators supported for both are different appropriate to the type used. For example, dereference(`->`) op isnt provided for arrays and index access(`[]`) op isnt provided for pointer types. Here is sample code:([Try Online](https://coliru.stacked-crooked.com/a/8a6e30f1181c9417)):
```c++
  auto smart_ptr = std::make_unique<int>(5);
  std::cout << *smart_ptr << std::endl; // 5
  auto smart_array = std::make_unique<int[]>(5);
  std::cout << smart_array[0] << std::endl; // 0
```
7. <b id="uniq_customdeleters">Custom Deleters</b>: Yes, we are allowed to supply a custom deleter for the owned object. The `std::unique_ptr` type template actually looks like `std::unique_ptr<T,D=std::default_delete>` so the custom deleter defaults to calling `delete obj`. But if needed we can create a custom deleter variant too. Its important to note though that depending on the custom deleter variant used, the **"zero"** cost abstraction no longer stays applicable since the custom deleter occupies some size. First a visualization for how these `unique_ptr` variants look like:
    <figure>
        <center><a href="{{ site.url }}/images/smartptr/unique_ptr_customdeleters.png"><img src="{{ site.url }}/images/smartptr/unique_ptr_customdeleters.png" alt="" height="100%" width="100%" align="middle" /></a></center>
    </figure>
Let's assume we have the following utility functions([Try Online!](https://coliru.stacked-crooked.com/a/a46a89a982585d82)):
```c++
template <typename T, typename D>
auto create_custom_deleter(T val, D deleter) {
    std::cout << "creating ptr with value " << val << std::endl;
    return std::unique_ptr<T, decltype(deleter)>(new T(val), deleter);
}
```
    - **Captureless Lambda:** The reason this ends up occupying the same size as a regular `unique_ptr` is **Empty Base Optimization** explained in the context of unique pointers in this [stackoverflow answer](https://stackoverflow.com/a/42717594/3656081) in reasonable detail. 
```c++
{
    auto captureless_lambda_deleter = [](int *ptr) {
        std::cout << "Killing ptr with value " << *ptr << std::endl;
        delete ptr;
    };
    auto ptr = create_custom_deleter<int, decltype(captureless_lambda_deleter)>(5, captureless_lambda_deleter);
    std::cout << sizeof(ptr) << " " << sizeof(captureless_lambda_deleter) << std::endl; // 8
} // "Killing ptr with value 5"
```
    - **Captured Lambda:** The final size of the `unique_ptr` depends on what's captured.
```c++
{
    auto v = std::vector<int>{1, 2, 3, 4, 5};
    auto capturefull_lambda_deleter = [v](int *ptr) {
        std::cout << "Scaled ptr value: " << (*ptr + v.size()) << std::endl;
        delete ptr;
    };
    auto ptr = create_custom_deleter<int, decltype(capturefull_lambda_deleter)>(5, capturefull_lambda_deleter);
    std::cout << sizeof(ptr) << " " <<  sizeof(capturefull_lambda_deleter) << " " << sizeof(v) << std::endl; // 8 bytes(raw ptr size) + 24 bytes(v)
} // Scaled ptr value: 10
```
    
    - **Function Pointer:** `==` Size of Two pointers
```c++
{
    auto* fn = +[](int *ptr) { // function pointer
        std::cout << "Killing ptr with value " << *ptr << std::endl;
        delete ptr;
    };
    auto ptr = create_custom_deleter<int, decltype(fn)>(5, fn);
    std::cout << sizeof(ptr) << std::endl; // 16(2 pointers)
}
```

That concludes the discussion of both `unique_ptr` and this post. We'll move on to multi-owner constructs in the next post.