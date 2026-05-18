---
title: Chapter 12 Dynamic Memory
weight: 12
type: docs
---

## Dynamic Memory and Smart Pointers

- `shared_ptr`, `unique_ptr` and `weak_ptr` are smart pointers defined in `memory`.
- Shared pointers and unique pointers can be used to manage allocated memory for objects:

<img src="../attachments/Pasted image 20240926231120.png">

- `shared_ptr`-specific operations:
    - It can be initialized with another `shared_ptr` or a `new`ed pointer: `p(q)`. **However, it cannot be implicitly converted from a pointer: direct initialization only (no copy initialization/assignment).**
    - A deleter function can be supplied instead of `delete` at construction: `p(q, d)`. It must take a single argument of a pointer type. `q` can be a smart pointer or an address.
    - `p.reset(q, d)` can be used to move the pointer to another address or smart pointer `q`. `d` defaults to `delete`, and `q` defaults to `nullptr`.

<img src="../attachments/Pasted image 20240926231206.png">

<img src="../attachments/Pasted image 20240926231132.png">

- **Smart pointers should be initialized with `make_shared<T>(...)`, which allocates and initializes a dynamic object (arguments are similar to `emplace`).**
- `shared_ptr`s keep records of the reference count. When it is destroyed (goes out of range, or gets lost in assignment), the counter will be decremented (in its destructor).
- **The memory will be released only after the last `shared_ptr` goes away.** Be sure to remove unused `shared_ptr` elements.
- `shared_ptr`s can be used to create dynamic arrays, create dynamic objects and share data between objects (with a pointer member).
- `new` can be used to allocate memory and default/value initialize objects: `new string` (default), `new string()` (value).
    - `new` can be used with `const`: `new const int(1024)`.
    - `new` can be used with `auto`: only a single initializer inside  parentheses is accepted. The type will be inferred: `new auto(...)`.
    - **`new` will throw `bad_alloc` exception if failed.** Using `new (nothrow)` (passing `nothrow` to a placement new, defined in `new`) will suppress the exception and return `nullptr`.
- Do not mix ordinary pointers and smart pointers. This will result in destroying allocated objects by mistake.
- Do not use `get` to initialize another smart pointer. This will result in wrong reference counts and destroying by mistake.
- **Smart pointers can ensure that memory is freed even if the block is exited prematurely (due to exception, etc.).**
- **Deleter can be used to do post-destruction operations. It can be used in non-dynamic situations**: `shared_ptr<T> p(&c, end)` will call `end` at the end of the function (`c` is local).
- `unique_ptr`-specific operations:
    - A `unique_ptr` should always be initialized with `nullptr` or a `new`ed pointer. It does not support ordinary copy or assignment (`p.reset(...)` only). The only exception is returning a `unique_ptr` from a function, in which a special copy will be created.
    - `p.release()` does not free the memory; it only breaks the connection between the `unique_ptr` and the object.
    - **Unlike `shared_ptr`, the type of the deleter is part of the type and should be supplied in the angle brackets**: `unique_ptr<objT, delT> p (new objT, fcn)`. It is called when the `unique_ptr` is destroyed.

<img src="../attachments/Pasted image 20240926231257.png">

- `weak_ptr` does not impact the lifetime of the object. `weak_ptr`-specific operations:
	- `wp.lock()` is used to check if the pointer is valid.

<img src="../attachments/Pasted image 20240926231317.png">

## Dynamic Arrays

- `new` can be used with type aliases of array types.

```cpp
typedef int arrT[42];
int *p = new arrT;
```

- **`begin()` and `end()` cannot be used on dynamic arrays: they use the array dimension to return pointers. Similarly, range `for`s cannot be used to process a dynamic array.**
- Objects are default initialized by default. They are value initialized if followed by an empty pair of parentheses. A braced list of initializers is also supported. A single element initializer is not allowed for dynamic arrays (`auto` not supported).
- **`new int[0]` will return a valid, nonzero pointer.** No special cases are necessary.
- **Elements in an array are destroyed in reverse order.** 
- A special version of `unique_ptr` can be used on arrays allocated by `new` (with a deleter). However, `shared_ptr` can only be used as an ordinary pointer with a customized deleter.

<img src="../attachments/Pasted image 20240927141231.png">

- **`allocator` (defined in `memory`) decouples allocation from construction.** There is no need to initialize a large buffer of objects ahead of input and let most of them go to waste.
	- The allocated memory is unconstructed. We need to use a pointer to somewhere in that memory to `construct` an element at that location, and `destroy` it after use.

<img src="../attachments/Pasted image 20240927142838.png">

```cpp
allocator<string> alloc;           // object that can allocate strings
auto const p = alloc.allocate(n);  // allocate n unconstructed strings

auto q = p;       // q will point to one past the last constructed element
alloc.construct(q++);              // *q is the empty string
alloc.construct(q++, 10, 'c');     // *q is cccccccccc

while (q != p)
	alloc.destroy(--q);            // destroy the strings we actually allocated

alloc.deallocate(p, n);            // free the allocated memory
```

- The allocated memory can be managed with library algorithms defined in `memory`:
	- All these functions take iterators or pointers as parameters.
	- `unitialized_copy` returns the incremented destination iterator like `copy`.

<img src="../attachments/Pasted image 20240927144552.png">

