# C++ notes

### Translation unit

A translation unit is a cc file together with all the header files it includes transitively. It is a piece of code that can be compiled in isolation.

### Storage duration

Each object in C++ has a storage duration.

* Automatic: space for the object is allocated at the beginning of the block
  where the object is declared, and deallocated at the end of the block.
* Static: storage is allocated at the beginning of the program, and deallocated
  at the end. This is what happens when you use the static keyword.
* Dynamic: storage is allocated and freed at runtime (using new and delete).
* Thread: for objects declared with thread_local. I have never used this.

You can declare a function-scoped variable as static, and the variable will only
be initialized once, even if the scope declaring the variable is entered many
times! The initialization happens the first time execution reaches the program
point where the variable is declared.

Non-function-scoped names with static storage duration get initialized at the
start of the program, even before main runs. Names in the same translation unit
are initialized top-to-bottom, but it is good style to not rely on that order.
The order for different translation units is undefined.

[More on storage duration and linkage](https://en.cppreference.com/w/cpp/language/storage_duration).

### Linkage

Linkage is some C++ jargon that impacts the scoping rules for a name.

* No linkage: the name is only visible in the scope where it is declared.
* Internal linkage: the name is visible in the whole translation unit.
  E.g.: static variables, names declared in an unnamed namespace.
* External linkage: the name is visible in other translation units.
  E.g.: names declared in a (named) namespace, names declared with the extern
  keyword.

### Inline, constexpr

The original use of inline on a function was a hint to the compiler to inline
it. Currently, it can be used on functions and variables, and it means that
multiple definitions of the name in different translation units are permitted,
as long as they are all identical. In practice, the main use is when
implementing a function in a header file, and that header file is included in
more than one translation unit.

The constexpr specifier on a variable or function means that it can be evaluated
at compile time. On functions, constexpr implies inline.

### Undefined behavior

The spec does not specify the language semantics for every situation. The
unspecified stuff is called undefined behavior (UB). UB is a very complex topic.
I just want to know a few basic cases that are UB, to avoid them in my programs.

* Out of bounds array access.
* Null pointer dereference.
* Accessing an object after it has been destructed.

### Strings

A C string (null terminated sequence of characters) is a different type from a
C++ string. This is important to know because

* Passing a C string to a context that expects C++ string incurs a copy
* C strings need to be explicitly freed, but C++ strings follow the rules of
  other objects and can be destructed, etc.

When you see string literals in a C++ program, those are all C strings! To avoid
copies, when declaring a function that takes a string, make it take
`absl::string_view` instead of `const string&`.

### Casting, RTTI

New syntax for casting. Don't use C-style casts:  
`static_cast <new_type> (expression)`

Runtime type information (RTTI) is the C++ feature that enables us to query the
types of values at runtme.

In non-polymorphic code, we can use `typeid` to query the type of a value; it
returns an object of type `std::type_info`.  
`typeid(123).name()` prints `int`.

In code using inheritance, the closest thing to Java's instanceof in C++ is
dynamic_cast. For example, to cast a type `Foo*` to `Bar*` you do:
```
if (Bar* y = dynamic_cast<Bar*>(x)) {
   y->SomeBarMethod();
}
```
Note that dynamic_cast is more limited than instanceof. The from type needs
to be a pointer (or reference) to a class with a virtual method. You can't use
arbitrary types, e.g., you can't use void* as the from type.

reinterpret_cast always "succeeds" (and gives back garbage in the bad cases).
Don't use it to check type identity.

### References, pointers

A reference to a type Foo (formally called lvalue reference), written `Foo&`,
is an alias of a Foo instance.

* It always aliases the same Foo instance; it can't be reassigned to a
  different Foo.
* You access Foo's members using dot, not arrow.

A reference is not an object. So, you cannot have an array of references, or a
reference of a reference, etc.  
When a function takes a reference, it is common style to also make it const.
If the function mutates the parameter, make the parameter a pointer instead.

### Temporaries, rvalue references, moves

A temporary is a thing that never gets a name, and as a result it can be used
only once, e.g., the return value of a function call `foo_bar()` can be
consumed immediately.

An rvalue reference, written `Foo&&`, is a reference to a temporary of type Foo.
Other than that, it is similar to a normal reference. A subtle point is that
the rvalue reference itself is not a temporary; it just points to a temporary.

`std::move` is a cast: it casts its argument to a temporary.  
By itself, it has no effect. But if you `std::move` a name and pass the result
to a move constructor, or a move assignment, then a move actually happens.  
After a name is moved, you should no longer use that name; the object may be in
some weird state.

`std::unique_ptr<Foo>` is a non-copyable type (that's how we enforce the
uniqueness of Foo), and that is why we often see `std::move` used on unique
pointers.

A move constructor is a constructor that takes an rvalue. A default move
constructor is created for each class, unless the user defines one explicitly.
As its name suggests, when the move constructor is invoked, we avoid a copy.  
```
// This is a move, not a copy
Foo x(std::move(some_other_foo));
// Also a move, because the return value is a temporary.
Foo x(fun_that_rets_foo());
```

Similarly to a move constructor, a move assignment operator avoids a copy. The
compiler defines a default move assignment for each class, unless you define a
custom copy constructor.

### RAII

Use new/delete to manage memory, not malloc/free.  
Actually, avoid explicit allocation/deallocation altogether.
Modern C++ style is to use smart pointers.

RAII (resource acquisition is initialization): Technique used for managing
resources such as files, sockets, and mutexes. When you explicitly do the open
and close of the resource, unexpected control flow in-between can cause the
resource to never be released. RAII creates the resource and associates it with
an object whose lifetime ends when the stack frame no longer exists, so we are
guaranteed that the appropriate destructors will be called to release the
resources.

[MS post on object lifetimes.](https://docs.microsoft.com/en-us/cpp/cpp/object-lifetime-and-resource-management-modern-cpp?view=msvc-160)  
[MS post on smart pointers.](https://docs.microsoft.com/en-us/cpp/cpp/smart-pointers-modern-cpp?view=msvc-160)

The structure of ownership in a program should form a DAG, but this isn't
machine-checked AFAICT. The DAG ensures that resources are released as they are
no longer needed, without using GC.  
Use shared_ptr and make_shared as the default pointer and allocator. Use
weak_ptr to break cycles, do caching, and observe objects without affecting or
assuming anything about their lifetimes.  
Any object created with new should be wrapped in a unique pointer, so its
lifetime can be managed.

Always create smart pointers on a separate line of code, never in a parameter
list, so that a subtle resource leak won't occur due to certain parameter list
allocation rules. (MS tip. Don't understand why yet.)

Shared_ptr: a reference-counted smart pointer.  
Weak_ptr: provides access to an object that is owned by one or more shared_ptr
instances, but does not participate in reference counting.

### Memory management

A memory allocator such as malloc is responsible for allocating and freeing
memory.

A segmentation fault (segfault) happens when a program attempts to read/write
memory that doesn't belong to it, or attempts to write to read-only memory.
Common ways to segfault are use-after-free, null pointer dereference, and
accessing past the end of an array.

When memory is freed, the allocator doesn't return the memory to the OS; it puts
the freed block into a free list.
When it receives allocation requests in the future, it tries to use blocks from
the free list.
First, this means that you can't easily tell what the mem usage of a program is
by looking at the mem usage of the process; this includes the free list.
Second, if you have a pointer `Foo*` and the object becomes dead but the
memory is reused, and then you use the pointer, you are silently accessing
garbage data.
This is a case of use-after-free.
(The more common one is to try to access mem that is still in the free list, and
get a segfault.)

Reminder that in the memory layout of a Linux process, the stack segment starts
at a high address and grows downward, and the heap segment starts at a low
address and grows upward.
It is unlikely
([but not impossible](https://stackoverflow.com/questions/1334055/what-happens-when-stack-and-heap-collide/1335389#1335389))
that the heap grows into the stack.

If I remember correctly, the virtual address space of a process has the same size
as all the RAM available in the machine; the program thinks that's how much mem
it can use.

TODO:  
Read the [asan paper](https://www.usenix.org/system/files/conference/atc12/atc12-final39.pdf)
and the [msan paper](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/43308.pdf).

### Classes

Classes are very different from Java; can't describe the differences succinctly.

Declaring a variable of type Foo (some class) is not the same as Java!
It is an actual object Foo, not a reference! E.g., assignments to such a
variable have complex semantics (they are called copy assignments).

Many syntaxes for constructors and object creation.

Class members are private by default.

Using the scope operator ::, we can put only the header of a method inside the
class, and define the actual method outside the class.

Can't use "new" syntax to create an object; the type is a pointer to an
object.

The "this" keyword exists, but can't access members off of it using dot.
It is a pointer, so you must use arrow.

Friend classes and methods can see the private members of a class.

Must explicitly use public when extending a class, to avoid restricting
visibility of the inherited members.

To get runtime dispatch for method calls, you need to declare the method as
virtual, but you can override a method even if it is not virtual! In the
latter case, the parent method is always called from a parent type.

The equivalent of an abstract method is a pure virtual method. A class Foo
with a pure virtual method is abstract *and can't be instantiated*.
For example, this means that you can't have a field typed Foo in another
class. The syntax to say that a pure virtual method has no implementation
is to assign it to 0, e.g.,  
`virtual int foobar() = 0;`  
To get runtime dispatch for a virtual method of a class Foo, you need to call
it on a pointer or reference, not on a Foo directly!

C++ doesn't have interfaces. A class that has no state and all its methods are
pure virtual is called an interface. You are forced to use these through
pointers, since an abstract class (or interface) can't be instantiated.

For a virtual class, you have to define the destructor explicitly and make it
virtual, and assign it to default, not to 0.

Multiple inheritance for classes.

When a constructor is marked as explicit, then the default constructor isn't
defined. This is encouraged for classes that only define one single-argument
constructor; because then the user won't accidentally forget to provide an
argument when instantiating.

Constructors of subclasses need to say what arguments they are passing to the
super constructor.

### Templates (Generics)

Use `<typename T>` when declaring a template variable instead of the equivalent
`<class T>`.  
When calling a generic function you can omit the instantiation and it is
inferred.

C++ creates a separate class at compile time for each instantiation of a generic
class. You can think of it as macro expansion. As a corollary, generics aren't
erased at runtime; a `Foo<Bar>` is still a `Foo<Bar>`, not a raw Foo.

### Misc notes; must organize

Can use `auto` instead of a type name in a variable declaration.  
Need to qualify the auto with &, otherwise we get unintended copies. Prefer
`const auto&` over `auto&`.

Function arguments are passed by value, and you add & to the formal param name
to pass by reference.  
Formals can be annotated as const.  
Formals can have default values.

Namespace aliasing.

Array declaration syntax is different from Java. The length is carried around
separately :(  
Don't use arrays. Modern C++ style uses `std::vector`.

Pointers and pointer contents can be const.

Arrow operator to access elements from pointers to structures.

All local variables, including objects, live on the stack!!

Besides assignments, copies can happen when passing arguments to a function,
when returning a value from a function, when initializing the fields of a class.
Basically everywhere where a value may be moved from one address to another.

As a useful rule of thumb, if you are writing a function that takes a Foo and
only reads it, pass it by reference, instead of passing a pointer to Foo.  
Only declare an argument that has an object type, instead of a reference or a pointer, when you want to force a copy. Never else.

Don't use `NULL`; its definition is implementation dependent. Use `nullptr`.

RVO (return-value optimization): when a function returns a Foo, in some cases
the callee and the caller don't need to have distinct copies of Foo; the
compiler is guaranteed to optimize the copies away.

* In the caller, the result must either be used directly in an expression, or
  assigned to a newly declared variable.
* In the callee, the return must either be an expression, or a single local
  variable. If there are multiple return statements using different local vars,
  RVO won't happen.

Copy elision. When you have a function that consumes a Foo and inside the body
the Foo is copied, consider changing the function to take the Foo by value
instead of by reference, and skip the copy inside the body by using std::move or
similar. By moving the copy from the body to the call, the compiler may skip it.
Rule of thumb: it's easy to apply copy elision when writing constructors, which
are often short, and it is fine to not try to apply it when writing other
functions.

When debugging, I usually use `LOG(INFO)`. In rare cases when it is not
available, use `std::cerr`, not `std::cout`. The former is unbuffered (flushed
immediately), the latter isn't, and you may lose some prints this way.

Easiest way to turn a vector v to a string is `absl::StrJoin(v, " ")`.

## Learning TODOs

Learn more about [macros](https://gcc.gnu.org/onlinedocs/gcc-4.8.5/cpp/Macros.html#Macros).

