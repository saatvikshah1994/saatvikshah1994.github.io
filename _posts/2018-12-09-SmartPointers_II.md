---
layout: post
title: ! "A Wild West Tour of Smart Pointers - II"
description: "Sample Smart Pointer Implementation, Shared Pointer Brainstorming, Non-Intrusive and Intrusive shared Pointers including Performance and Microbenchmarks, Thread-Safety, Passing them around and more, References and Further Reading."
level: Beginner
image:
  feature: smartptr/header.jpg
tags: [C++]
---

## Overview
This is the follow up to [Part 1](https://saatvikshah1994.github.io/SmartPointers/) Smart Pointers post. In search of a more satisfying smart pointer experience, we'll move towards the land of Shared ownership constructs which is as dangerous as it is beautiful. Let's go go go!

<figure>
    <center><a href="{{ site.url }}/images/smartptr/next_steps.png"><img src="{{ site.url }}/images/smartptr/next_steps.png" alt="" height="50%" width="50%" align="middle" /></a></center>
</figure>

<div class="toc">
<h2 id="outline">Outline</h2>
<li><a href="#impl">Sample implementation</a></li>
<li><a href="#brainstorming">Brainstorming Shared Pointers</a></li>
<li><a href="#sharedptr_nonintrusive">Non-Intrusive Shared Pointers</a>
<ul>
<li><a href="#shared_usecase">General Use Case</a></li>
<li><a href="#shared_creation">Creation Variants</a></li>
<li><a href="#shared_physicallayout">Physical Layout</a></li>
<li><a href="#shared_performance">Performance</a></li>
<li><a href="#shared_threadsafety">Thread Safety</a></li>
<li><a href="#shared_passingaround">Passing Around</a></li>
<li><a href="#shared_weakptrs">Non-Owners/Weak Pointers</a></li>
<li><a href="#shared_variants">Standard Variants</a></li>
<li><a href="#shared_aliasing">Aliasing Constructor</a></li>
<li><a href="#shared_customdeleter">Custom Deleters</a></li>
</ul></li>
<li><a href="#intrusiveptr">Intrusive Shared Pointers</a>
<ul>
<li><a href="#intrusive_usecase">General Use Case</a></li>
<li><a href="#intrusive_size">Performance(size)</a></li>
<li><a href="#intrusive_speed">Performance(speed)</a></li>
<li><a href="#intrusive_threadsafety">Thread Safety</a></li>
<li><a href="#intrusive_passingaround">Passing Around</a></li>
<li><a href="#intrusive_variant">Standard Variants</a></li>
</ul></li>
<li><a href="#refs">References</a></li>
</div>

---
<center><h2 id="impl">Sample Implementation</h2></center>
One thing we didnt get into in the previous post is that how exactly are smart pointers implemented from scratch. While a production ready implementation of multi-ownership smart pointers might be non-trivial to implement, we can at least see a simple scoped pointer-like implementation. The key points to remember are:
1. Non-copyable: Prevent ownership transfer.
2. Non-movable: Prevent ownership transfer.
3. Automatic destructor call when going out of scope: A variable created on the stack!<br>
Armed with the information above, here is what a simple smart pointer might look like([Try Online!](https://coliru.stacked-crooked.com/a/c6a65d623ce42961)):
```c++
template <typename T>
class smart_ptr {
    public:
    // Enable constructor and deref operator
    smart_ptr(T* obj) {
        std::cout << "Constructing smart_ptr with contents: " << obj << std::endl;
        m_obj = obj;
    }
    T* operator->() { return m_obj; }
    T& operator*() { return *m_obj; }
    // Non-copyable
    // copy constructor
    smart_ptr(const smart_ptr&) = delete;
    // copy assignment operator
    smart_ptr & operator=(const smart_ptr&) = delete;
    // Non-movable
    // move constructor
    smart_ptr(smart_ptr&&) = delete;
    // move assignment operator
    smart_ptr& operator=(smart_ptr&&) = delete;
    // Automatically delete object going out of scope
    ~smart_ptr() {
        std::cout << "Calling destructor" << std::endl;
        delete m_obj;
    }
    private:
    T* m_obj;
};
```
Now let's try it out:
```c++
{
  smart_ptr<int> x(new int(5)); // Constructing smart pointer...
  std::cout << *x << std::endl; // 5
} // Calling destructor
```

---
<center><h2 id="brainstorming">Brainstorming Shared Pointers</h2></center>
Ok single ownership done! Now towards shared ownership constructs. Let's start by discussing a few high level ideas on how one might implement shared ownership pointers.
<figure>
    <center><a href="{{ site.url }}/images/smartptr/SharedOwnerVariants.png"><img src="{{ site.url }}/images/smartptr/SharedOwnerVariants.png" alt="" height="90%" width="90%" align="middle" /></a></center>
</figure>
1. **Reference Counting:** As shown above, each owner points to a block storing a count of people pointing to it. It itself then points to the object. *This is known as the **Control Block** and with some adjustments is the standard way in which shared pointers are imlemented.*
2. **Leader Based:** One of the owners is appointed a leader and is the only one who can delete the object. It does so automatically when it goes out of scope so care must be taken to keep it alive as long as the other owners are alive.
3. **Reference Linking:** Each owner internally maintains two pointers - one to the next alive shared owner and the other to the object. The last owner alive is responsible for deleting the object and does so by detecting it has a self cycle.

---
<center><h2 id="sharedptr_nonintrusive">Non-Intrusive Shared Pointers</h2></center>
A quick recap of **"Non-Intrusive"**: It means that the *Control-Block* we've been discussing is external to the object in question. We'll revisit this once more as we see the physical layout of these non-intrusive smart pointers. Since the standard library `std::shared_ptr` is the most frequently used case of such a pointer, I will use `shared_ptr` and non-intrusive pointer interchangeably. I will also refer to *Control Block* as the shorthand *CB*.<br>
Let's quickstart with some code([Try Online](https://coliru.stacked-crooked.com/a/afe3badbbab208e0)):
```c++
{
    auto p1 = std::shared_ptr<WildWestGang>(new WildWestGang({"Arthur", "Dutch", "Micah"})); // Birth of the Gang
    // use_count gives us the count of owners
    std::cout << p1.use_count() << std::endl; // 1
    auto p2 = p1; // Now p1 and p2 are both owners
    std::cout << p1.use_count() << " " << p2.use_count() << std::endl; // 2 2
} // Gang is dead!
```
1. <b id="shared_usecase">General Use Case</b>: *Shared Ownership of an Object*.
2. <b id="shared_creation">Creation variants</b>: Similar to `std::unique_ptr`, `std::shared_ptr` allows two ways to create a shared pointer `std::shared_ptr<T>(new T(args));` and `std::make_shared<T>(args);`. While with unique pointers the difference was mostly cosmetic, these two constructs have a huge difference with shared pointers in terms of physical layout. We'll see how in the next section. 
3. <b id="shared_physicallayout">Physical Layout</b>: The physical layout of `std::shared_ptr` is slightly more complicated due to the presence of the control block. 
    <figure>
        <center><a href="{{ site.url }}/images/smartptr/shared_ptr_logical.png"><img src="{{ site.url }}/images/smartptr/shared_ptr_logical.png" alt="" height="90%" width="90%" align="middle" /></a></center>
    </figure>
    Let's consume the above diagram bit by bit: 
    - `Number of Owners == Strong Refcount = 2`. When the strong refcount drops to zero the object is destroyed.
    - `Number of Non-Owners = 1`. 
    - Corresponding to that, `Weak Reference count = #Owners + #NonOwners`
    - The CB at the right, apart from the refcounts, holds the custom deleter(or a pointer to it) and a pointer to the object. This pointer is used for deleting the object once the strong refcount drops to zero.
    - Each owner/non-owner pointing to the object take the size of 2 pointers.<br>

    The story doesnt end here though - depending on the way of creating the shared pointer: `std::shared_ptr<T>(new T(args))` vs. `std::make_shared<T>(args)` the physcial layout is different:
    - For `std::shared_ptr<T>(new...` the layout is similar to above. The owners, non-owners, Control block and object lie in different memory locations generally.
    - For `std::make_shared<T>(args)` the object is constructed inside the control block and the control block metadata is placed side by side with the object.
    <figure>
        <center><a href="{{ site.url }}/images/smartptr/shared_ptr_variants.png"><img src="{{ site.url }}/images/smartptr/shared_ptr_variants.png" alt="" height="90%" width="90%" align="middle" /></a></center>
    </figure>
    This has one major implication - Normally if the weak ref count > 0 when the strong reference count = 0, the CB cannot be deleted, but the object can. Here unfortunately both are placed together in continuously allocated memory. Thus the object also must remain alive while the CB is alive and dies with it. The difference in physical layout has some major performance implications discussed in a later section.
  4. <b id="shared_performance">Performance</b>: In terms of size, it is quite clear that shared pointers occupy double the size of raw pointers. This can be easily checked([Try Online](https://coliru.stacked-crooked.com/a/ee291f7b04893cdd)):
```c++
std::cout << sizeof(int*) << std::endl; // 8
std::cout << sizeof(std::shared_ptr<int>) << std::endl; // 16
```
  There's also the additional size of the control block but that is not accessible and implementation dependent.<br>
  Let's move onto speed, because there are many factors to consider here:
  - **Reference Counting:** Shared pointers perform reference counting using atomic operations. While this ensures thread-safe reference counting, it also means that for non multithreaded applications, the overall reference counting strategy is slow. Either way, this ends up slowing down its performance compared to raw pointers. One potential solution is to use [`boost::local_shared_ptr`](https://www.boost.org/doc/libs/1_68_0/libs/smart_ptr/doc/html/smart_ptr.html#local_shared_ptr) which uses non atomic reference counts.
    <figure>
          <center><a href="{{ site.url }}/images/smartptr/shared_refcount.png"><img src="{{ site.url }}/images/smartptr/shared_refcount.png" alt="" height="90%" width="90%" align="middle" /></a></center>
    </figure>
    ([Try Online](http://quick-bench.com/XvsPRc3G3qIp0dWeSipzQ1IS5eg)): In the above, we are benchmarking the speed of creating a deleting shared vs. raw pointers where `*1` refers to number of owners being 1, and `*2` refers to the number of owners being 2.
  - **Cache Hits:** Coming back to the two physical layouts discussed earlier - Can you think of why one might be better for the cache compared to the other. When having to access two memory locations adjacent to each other, there will be a cache hit the second time round. (If your not sure why you should read up on [locality of reference](https://stackoverflow.com/a/16554721/3656081)). Thus the `make_shared` version is preferrable as you potentially reduce your #memory accesses.
    <figure>
          <center><a href="{{ site.url }}/images/smartptr/shared_cache.png"><img src="{{ site.url }}/images/smartptr/shared_cache.png" alt="" height="90%" width="90%" align="middle"/></a></center>
    </figure>
    ([Try Online](http://quick-bench.com/hTgEv273zzFDnn1ymdASFM5ucoo)): In the above, we are benchmarking the speed of creating, accessing and deleting shared vs. raw pointers where `V1` refers to `std::shared_ptr<T>(new..)` variant and `V2` refers to the `std::make_shared` variant.
  - **Memory Load:** You might recall that the `make_shared` version has one problem that the object must be kept alive if the CB is. This can end up being a memory burden if you have huge objects with non-owning pointers pointing to it. **The object must stay alive until the last non-owning pointer goes out of scope.**<br><br>
  **To summarize the rule of thumb should be as follows:**
  - For all sized objects without non-owning pointers prefer `std::make_shared`.
  - Fallback to `std::shared_ptr` based construction in memory exhaustive settings with long-living non-owning pointers.
5. <b id="shared_threadsafety">Thread Safety</b>: The standard library `std::shared_ptr` has the following guarantees regarding thread safety:
  - Writes to the object owned in any way are **not thread safe** eg. ptr->writeMember().
  - Writes to individual owners/non-owners is also **not thread safe** eg. ptr.reset(new...).
  - Changes to the control block are always thread safe i.e. adding or removing owners/non-owners in a multithreaded scenario is thread safe. This is because all reference count changes are atomic.
  - `boost/smart_ptr` offers a number of variants with either less thread safety(for better performance) or more thread safety.
6. <b id="shared_passingaround">Passing Them Around</b>:
    <figure>
          <center><a href="{{ site.url }}/images/smartptr/SharedPtrMovingAround.png"><img src="{{ site.url }}/images/smartptr/SharedPtrMovingAround.png" alt="" height="90%" width="90%" align="middle"/></a></center>
    </figure>
    These are strictly following the [CPP Core guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-resource). Compared to unique pointers, one case might be unclear: **Share vs. May Share** - The *Share* cases increments the ref count by default when calling the function due to the pass by value. The *May Share* case doesnt - the refcount will only increment if sharing happens. This is good for performance. [Let's see some code now](https://coliru.stacked-crooked.com/a/fa270015ed31e590):
    ```c++ 
// 
std::shared_ptr<WildWestGang> createWildWestGang() {
    auto names = std::vector<std::string>{"Dutch", "Arthur", "John"};
    return std::make_shared<WildWestGang>(names);
}
//
void useWildWestGang(WildWestGang& g) {
    // raw ref
    g.Print();
}
//
void reseatWildWestGang(std::shared_ptr<WildWestGang>& gang_ptr) {
    gang_ptr.reset(new WildWestGang({"Bill", "Lenny", "Charles"}));
}
//
void share(std::shared_ptr<WildWestGang> gang_ptr) {
    // the refcount has already been incremented due to pass by value
    std::cout << gang_ptr.use_count() << std::endl; // 2
    auto new_sharer = gang_ptr;
    // when returning the copy "gang_ptr" will be destroyed, so ref_count stays 2
}
//
void may_share(const std::shared_ptr<WildWestGang>& gang_ptr_ref) {
    // refcount has not been incremented
    // will be ONLY if sharing happens(saves us performance on refcount increment)
    // can add more sharers but cant reseat the original
    std::cout << gang_ptr_ref.use_count() << std::endl; // 1
}
    ```

7. <b id="shared_weakptrs">Non-Owners/Weak Pointers</b>:  We've been constantly referring to the term "Non-Owner" pointers above - The technical term for this in standard libraries is Weak pointer. A weak pointer helps to track an object's existence and seeks to use it if it is alive. A number of examples have been suggested in talks given in CppCon:
  - A programmatic cache which should not own owning references to objects it points to. If it does so, the objects will continue to stay alive as long as the cache exists.([Try Example Online](https://coliru.stacked-crooked.com/a/524ed078dbdc8a04))
  ```c++
std::map<int, std::weak_ptr<Foo>> cache;
auto foo_obj = std::shared_ptr<Foo>(new Foo(1));
cache[1] = std::weak_ptr<Foo>(foo_obj);
auto it = cache.find(1)->second;
if(auto x = it.lock()) { // Prints 1
    std::cout << x->m_item << std::endl;
} else {
    std::cout << "Expired!" << std::endl;
}
foo_obj.reset();
if(auto x = it.lock()) { // Prints expired!
    std::cout << x->m_item << std::endl;
} else {
    std::cout << "Expired!" << std::endl;
}
  ```
  - An intermodule callback setup - You only want to use the callback function if the object containing the callback is alive.<br>

    Another critical use of `std::weak_ptr` is that it can make cyclic dependencies possible, which standard shared pointers would never be able to handle.
    > Use std::weak_ptr to break cycles of shared_ptrs - [CPP Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-weak_ptr)
8. <b id="shared_variants">Standard Variants</b>:(Each title is clickable and links to the standard reference) <br>
- [`std::shared_ptr`](https://en.cppreference.com/w/cpp/memory/shared_ptr): C++ Standard Library; 
- [**Boost Smart Pointers**](https://www.boost.org/doc/libs/1_68_0/libs/smart_ptr/doc/html/smart_ptr.html#introduction): [`boost:shared_ptr`](https://www.boost.org/doc/libs/1_68_0/libs/smart_ptr/doc/html/smart_ptr.html#shared_ptr), [`boost::local_shared_ptr`](https://www.boost.org/doc/libs/1_68_0/libs/smart_ptr/doc/html/smart_ptr.html#local_shared_ptr), [`boost::atomic_shared_ptr`](https://www.boost.org/doc/libs/1_68_0/libs/smart_ptr/doc/html/smart_ptr.html#atomic_shared_ptr).
9. <b id="shared_aliasing">Aliasing Constructor</b> The Aliasing constructor is the case where the shared pointer points to one object but owns another. Let's say we have two objects `x1, x2` and their corresponding control blocks `CB1, CB2`. Shared pointer `O` has 2 parts - the object pointer(say `O.obj_ptr`) and the pointer to a control block(`O.cb_ptr`). In the aliasing constructor case: `O.obj_ptr = x1` but `O.cb_ptr=CB2`. You can read more [here](https://stackoverflow.com/a/27109774/3656081).
10. <b id="shared_customdeleter">Custom Deleter</b> As discussed earlier, the custom deleter here is part of the control block - This makes its construction relatively simpler for shared pointers. It can be constructed as `std::shared_ptr<T>(new T(args), deleter)`. ([Try Online!](https://coliru.stacked-crooked.com/a/fa270015ed31e590))
```c++
template <typename T, typename D>
auto create_custom_deleter(T val, D deleter) {
    std::cout << "creating ptr with value " << val << std::endl;
    // D not part of type anymore
    //     std::unique_ptr<T,D> : For unique pointers
    return std::shared_ptr<T>(new T(val), deleter);
}
```
---
<center><h2 id="intrusiveptr">Intrusive Shared Pointers</h2></center>
<figure>
    <center><a href="{{ site.url }}/images/smartptr/intrusive_ptr_intro.png"><img style="border-radius:40%;" src="{{ site.url }}/images/smartptr/intrusive_ptr_intro.png" alt="" height="300px" width="300px" align="middle" /></a></center>
</figure>
As suggested in the figure above, the intrusive pointer allows you to define and embed and completely control the control block inside your object. Let's see an example of how this can be done:
- Step 1: Redefine the original class with an embedded control block and initialize it correctly:
```c++
class WildWestGang{
    public:
        explicit WildWestGang(std::vector<std::string> members): m_cb(),
                                                                 m_members(members) { 
         ...
    struct ControlBlock{ // embedded control block
        int32_t m_refs = 0;
    } m_cb;
    private:
        std::vector<std::string> m_members;
};
```
- Step 2: Add the free functions for managing the control block:
```c++
inline void intrusive_ptr_add_ref(WildWestGang* gang_ptr)
{
    std::cout << "Call intrusive_ptr_add_ref" << std::endl; 
    ++gang_ptr->m_cb.m_refs;
}
//
inline void intrusive_ptr_release(WildWestGang* gang_ptr)
{
    std::cout << "Call intrusive_ptr_release" << std::endl;
    if (--gang_ptr->m_cb.m_refs == 0)
    delete gang_ptr;
}
```

Now let's try using it([Try Online!](https://coliru.stacked-crooked.com/a/b81bacb325664020)):
```c++
{
    auto g1 = boost::intrusive_ptr<WildWestGang>(new WildWestGang({"Arthur", "Dutch", "Lenny"})); // constructor + add_ref
    auto g2 = g1; // add_ref
    g1->Print(); // Arthur Dutch Lenny
    g2->Print(); // Arthur Dutch Lenny
} // 2 release calls + destructor
```
At the inconvenience of having to define your own CB logic, you get a heap of benefits. Essentially you dont end up paying for things you might not need like `weak reference counts`, `atomic reference counting`, `Control block pointer`, etc provided by non-intrusive shared pointer implementations.
1. <b id="intrusive_usecase">General Use Case</b>: Shared Ownership with complete control over the Control block. **Lower memory footprint + Cache Friendly**. Read more [here](https://www.boost.org/doc/libs/1_68_0/libs/smart_ptr/doc/html/smart_ptr.html#intrusive_ptr).
2. <b id="intrusive_size">Performance(size)</b>: As depicted in the figure above, the intrusive pointer is the same size as a raw pointer. On the other hand the control block size depends on how many data members the user defines for it. In the example above its size is `sizeof(int32_t)`. **Benefit**: lower memory usage both for the pointer and the control block.([Try Online!](https://coliru.stacked-crooked.com/a/c68c8b3d4e5479cc))
```c++
std::cout << sizeof(boost::intrusive_ptr<WildWestGang>) << std::endl; // 8
```
3. <b id="intrusive_speed">Performance(speed)</b>: **You only pay for what you need!**
- *Reference Counting*: The speed is directly proportional to how you implement it - using simple variables(fast) or atomics(slower) or some other complex logic(slowest).
- *Cache Access*: Spatial locality(object and counter being together) leads to more cache hits.
4. <b id="intrusive_threadsafety">Thread Safety</b>: Apologies for repeating, but I want to emphasize - You pay for what you need!. Thread safety of **accessing your CB** depends on your CB and the implementation of its control. **Accessing objects** is not thread safe as per the usual.
5. <b id="intrusive_passingaround">Passing Around</b>: The semantics for passing around intrusive shared pointers matches that of <a href="#shared_passingaround">non-intrusive shared pointers</a>.
6. <b id="intrusive_variant">Standard Variants</b>: Boost offers [`boost::intrusive_ptr`](https://www.boost.org/doc/libs/1_68_0/libs/smart_ptr/doc/html/smart_ptr.html#intrusive_ptr).

---
<center><h2 id="refs">References</h2></center>
<figure>
    <center><a href="{{ site.url }}/images/common/refs.jpg"><img style="border-radius:40%;" src="{{ site.url }}/images/common/refs.jpg" alt="" height="300px" width="300px" align="middle" /></a></center>
</figure>
Most of the references I point out to can be read further depending on which factor you want to deep-dive into further:
1. [C++ Core Guidelines on Resource Management](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-resource): Basics, Semantics about passing smart pointers around.
2. **Effective Modern C++**: Basics, The tradeoff between the two physical layouts possible in `std::shared_ptr`, Using `unique_ptr` with PIMPL idiom.
3. **[Fluent CPP](https://www.fluentcpp.com/2017/08/22/smart-developers-use-smart-pointers-smart-pointers-basics/)**: Basics, Lots on Custom deleters, PIMPL idiom with unique_ptr, Covariant return type with unique_ptr.
4. **[Meeting CPP](https://www.meetingcpp.com/blog/items/an-overview-on-smart-pointers.html)**: Basics, Categorization, A nice comparison table with more smart pointer libraries covered.
5. **[OOTips Blog Post](http://ootips.org/yonat/4dev/smart-pointers.html)**: Basics, Different ways of implementing a shared pointer.
6. **[Leak Freedom in C++ by default](https://www.youtube.com/watch?v=JfmTagWcqoE)**: Excellent Talk! Must watch - Covers different ways of thinking about smart pointers and the Pointer -> Owner dependency by looking at various data structures and situations. Demonstrates a [`deferred_ptr`](https://github.com/hsutter/gcpp) possible future smart pointer type which can further automate GC and helps with cyclic dependency clusters.
7. **[GotW #91](https://herbsutter.com/2013/06/05/gotw-91-solution-smart-pointer-parameters/):** Passing them around, `make_shared`/`make_unique` benefits.
8. Standard References on different Smart Pointers linked to in the respective subsections.

Some parts of this post have also been inspired by the Wild West media spree I'm on - **Red Dead Redemption 2** and **The Ballad of Buster Scruggs** :p.

---
That's all, thanks for reading :) - You can post a comment or reach out to me at *saatvikshah1994@gmail.com* for any suggestions or anything else smart pointers.