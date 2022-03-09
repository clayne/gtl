
<img src="https://github.com/greg7mdp/gtl/blob/main/html/img/phash.png?raw=true" width="120" align="middle"> 

# Greg's Template Library of useful classes.

  [![License: Apache-2.0](https://img.shields.io/badge/License-Apache-yellow.svg)](https://opensource.org/licenses/Apache-2.0) [![Linux](https://github.com/greg7mdp/gtl/actions/workflows/linux.yml/badge.svg)](https://github.com/greg7mdp/gtl/actions/workflows/linux.yml)  [![MacOS](https://github.com/greg7mdp/gtl/actions/workflows/macos.yml/badge.svg)](https://github.com/greg7mdp/gtl/actions/workflows/macos.yml) [![Windows](https://github.com/greg7mdp/gtl/actions/workflows/windows.yml/badge.svg)](https://github.com/greg7mdp/gtl/actions/workflows/windows.yml) 

## Overview

This repository aims to provide many classes that are commonly needed in substantial C++ projects, but that are either not available in the C++ standard library, or have a specification which makes them slower than they could beslow implementation. In some cases, the C++ standard requirements prevents from providing faster alternatives (for example the  pointer stability requirement for unordered maps or sets prevents providing implementations using open addressing).

Among the many classes offered by [gtl](https://github.com/greg7mdp/gtl), we have a set of excellent **hash map** implementations, as well as a **btree** alternative to `std::map` and `std::set`. These are *drop-in replacements* for the standard C++ classes and provide the same API, but are significantly faster and use less memory. 

We also have a fast `bit_vector` implementation, which is an alternative to `std::vector<bool>` or `std::bitset`, providing both dynamic resizing and a good assortment of bit manipulation primitives, as well as a novel `bit_view` class allowing to operate on subsets of the `bit_vector`.

We are happy to integrate new classes into [gtl](https://github.com/greg7mdp/gtl), provided the license is compatible with ours, and we feel they will be useful to most users. Often, when integrating classes from other sources, we are able to improve their performance both in time and space by using other classes already available in [gtl](https://github.com/greg7mdp/gtl) (such as hash maps, btree, bit_vector, etc...)  instead of the spandard ones.

[gtl](https://github.com/greg7mdp/gtl) requires a C++20 compiler. We currently support:  `Visual Studio 2019 +`, `gcc 11.1 +`, and `clang 11 +` (or Xcode 13.2 + on MacOS).

Because [gtl](https://github.com/greg7mdp/gtl) is a header only library, installation is trivial, just copy the `gtl` directory to your project somewhere in your include path and you are good to go. We also support common package managers such as [Conan](https://conan.io/) and [vcpkg](https://vcpkg.io/en/index.html).

## Installation

Copy the gtl directory to your project. Update your include path. That's all.

If you are using Visual Studio, you probably want to add `gtl/debug_vis/gtl.natvis` to your projects. This will allow for a user friendly display of gtl containers in the debugger. Similar debug visualizers are also provided for gdb and lldb in the \gtl/debug_vis` directory.

> A cmake configuration files (CMakeLists.txt) is provided for building the tests and examples. Command for building and running the tests is: <br>
> `mkdir build && cd build && cmake -DGTL_BUILD_TESTS=ON -DGTL_BUILD_EXAMPLES=ON .. && cmake --build . && make test`


Following is a short look at the various classes available in [gtl](https://github.com/greg7mdp/gtl). In many cases, a more complete description is linked.


## Hash containers

[Gtl](https://github.com/greg7mdp/gtl) provides a set of hash containers (maps and sets) implemented using open addressing (single array of values, very cache friendly), as well as advanced SSE lookup optimizations allowing for excellent performance even when the container is up to 87% full. These containers have the same API as the `unordered` versions from the STL, and are significantly outperforming the unordered version both in terms of speed and space.

The four provided hash containers are:
    - `gtl::flat_hash_map`
    - `gtl::flat_hash_set`
    - `gtl::node_hash_map`
    - `gtl::node_hash_set`

For more information on the hash containers, please see [gtl hash containers](https://github.com/greg7mdp/gtl/tree/main/docs/hmap.md)

> if using Visual Studio, make sure to add the [gtl natvis](https://github.com/greg7mdp/gtl/blob/main/gtl/debug_vis/gtl.natvis) file to your projects, which provides a user-friendly visualization of the content of gtl set and map containers. Debug visualizers are available for gdb and lldb as well.


Here is a very basic example of using the gtl::flat_hash_map:

```c++
#include <iostream>
#include <string>
#include <gtl/phmap.hpp>

using gtl::flat_hash_map;
 
int main()
{
    // Create an unordered_map of three strings (that map to strings)
    flat_hash_map<std::string, std::string> email = 
    {
        { "tom",  "tom@gmail.com"},
        { "jeff", "jk@gmail.com"},
        { "jim",  "jimg@microsoft.com"}
    };
 
    // Iterate and print keys and values 
    for (const auto& n : email) 
        std::cout << n.first << "'s email is: " << n.second << "\n";
 
    // Add a new entry
    email["bill"] = "bg@whatever.com";
 
    // and print it
    std::cout << "bill's email is: " << email["bill"] << "\n";
 
    return 0;
}
```

**Key decision points for hash containers:**

- The `flat` hash containers do not provide pointer stability. This means that when the container resizes, it will move the keys and values in memory. So pointers to something inside a `flat` hash container will become invalid when the container is resized. The `node` hash containers do provide pointer stability, and should be used instead if this is an issue.

- The `flat` hash containers will use less memory, and usually are faster than the `node` hash containers, so use them if you can. the exception is when the values inserted in the hash container are large (say more than 100 bytes [*needs testing*]) and expensive to move.

- The `parallel` hash containers are preferred when you have a few hash containers that will store a very large number of values. The `non-parallel` hash containers are preferred if you have a large number of hash containers, each storing a relatively small number of values.

- The benefits of the `parallel` hash containers are:  
   a. reduced peak memory usage (when resizing), and  
   b. multithreading support (and inherent internal parallelism)

**Acknowledgements** 

Thanks to Google and the "Swiss table" team for the original [implementation](https://github.com/abseil/abseil-cpp), from which ours is derived. 

## Parallel hash containers


The four provided parallel hash containers are:
    - `gtl::parallel_flat_hash_map`
    - `gtl::parallel_flat_hash_set`
    - `gtl::parallel_node_hash_map`
    - `gtl::parallel_node_hash_set`


For a full writeup explaining the design and benefits of the parallel hash containers, [click here](https://greg7mdp.github.io/gtl/).

For more information on the implementation, usage and characteristics of the parallel hash containers, please see [gtl parallel hash containers](https://github.com/greg7mdp/gtl/tree/main/docs/phmap.md)


## Btree containers


The four provided btree containers are:
    - `gtl::btree_map`
    - `gtl::btree_set`
    - `gtl::btree_multimap`
    - `gtl::btree_multiset`

For more information on the hash containers, please see [gtl btree containers](https://github.com/greg7mdp/gtl/tree/main/docs/btree.md)

**Key decision points for btree containers:**

Btree containers are ordered containers, which can be used as alternatives to `std::map` and `std::set`. They store multiple values in each tree node, and are therefore more cache friendly and use significantly less memory.

Btree containers will usually be preferable to the default red-black trees of the STL, except when:
- pointer stability or iterator stability is required
- the value_type is large and expensive to move

When an ordering is not needed, a hash container is typically a better choice than a btree one.


## bit_vector (or dynamic bitset)

[Gtl](https://github.com/greg7mdp/gtl) provides `bit_vector`, which is an alternative to `std::vector<bool>` or `std::bitset`, as it provides both dynamic resizing, and a good assortment of bit manipulation primitives.

I implemented this container because I often needed the functionality it provides, and didn't find an open-source implementation I liked which didn't require pulling in a big library. The `gtl::bit_vector` implementation is self-contained in a single header [file](https://github.com/greg7mdp/gtl/blob/main/gtl/bit_vector.hpp) which can trivially  be added to any project (it currently requires a C++17 compiler). 

In addition, I dreamed of the `gtl::bit_view` functionality, similar to `std::string_view` for strings, to refer and operate on a subset of a full `gtl::bit_vector`, and I thought it would be fun implementing it.

Click [here](https://github.com/greg7mdp/gtl/blob/main/examples/misc/bit_vector.cpp) for an example demonstrating some of the capabilities of `gtl::bit_vector`.

> if using Visual Studio, make sure to add the [gtl natvis](https://github.com/greg7mdp/gtl/blob/main/gtl/debug_vis/gtl.natvis) file to your projects, which provides a user-friendly visualization of the content of a `gtl::bit_vector`.

When printed, converted to `std::string`, or displayed in the debugger with the [gtl natvis](https://github.com/greg7mdp/gtl/blob/main/gtl/debug_vis/gtl.natvis), bits are displayed right to left, so for example a `gtl::bit_vector` of size 16, with the fist two bits set (index 0 and 1) would be displayed as `0x00000003`.

## lru_cache

This is an unordered map container into which you insert (key, value) pairs, but which will keep a maximum number of pairs. When the maximum number of pairs is reached, every insertion of a new key cause the removal of the least recently accessed pair.

This class is useful for implementing caches. 

Thanks to Alexander Ponomarev for the original [implementation](https://github.com/lamerman), from which ours is derived. 

