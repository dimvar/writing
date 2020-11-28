# Python notes

### Sequences: strings, lists, tuples, etc

Strings can be indexed like arrays, and sliced.  
Strings are immutable.  
With triple quotes, a string literal can span multiple lines.  
Two string literals that are next to each other are automatically concatenated!
(as in C++, but still weird to me.)

Lists are what JS calls arrays. They can be indexed and sliced.  
You can assign to a slice to modify a list, and to add/remove items from the
list.  
The built-in `len` function returns the size of a list.  
With the `in` keyword you can check membership in a list.

The built-in functions filter, map, and reduce do the usual thing. I'm a bit 
surprised that there are many built-ins in the global namespace (about 60,
listed [here](https://docs.python.org/2/library/functions.html)).

List comprehension example:
```
>>> [(x, y) for x in [1,2,3] for y in [3,1,4] if x != y]
[(1, 3), (1, 4), (2, 3), (2, 1), (2, 4), (3, 1), (3, 4)]
```

Zip has this name because you can think of the arguments as the separated sides
of a zipper, and the zip function interleaves the teeth. Example:
```
>>> x = [1, 2, 3]
>>> y = [4, 5, 6]
>>> zipped = zip(x, y)
>>> zipped
[(1, 4), (2, 5), (3, 6)]
```

The `del` statement can delete an element or a slice from a list. It can also delete a variable (bad idea, as we know from JS).

Example of creating a tuple and accessing an element:
```
>>> t = 12345, 54321, 'hello!'
>>> t[0]
12345
```

Tuples vs lists: tuples are immutable, and usually contain a heterogeneous
sequence of elements. Lists are mutable, and their elements are usually
homogeneous.

#### Indexing examples:
Get elements at indices 2 through 5  
`a[2:5]`  
Get elements at indices 0 through 5  
`a[:5]`  
Get elements at indices 2 through the end  
`a[2:]`  
Get elements at indices 2 through 15, with stride 3  
`a[2:15:3]`  
Can do multi-dimensional too; dimensions separated by commas:  
`a[2:15:3, :4:2]`

### Sets and dictionaries

Sets are built-in in the language. Operations on sets are very concise to
write:  
Set difference: `a - b`  
Union: `a | b`  
Intersection: `a & b`  
Exclusive or: `a ^ b`  
Python provides set comprehensions, which are similar in spirit to list
comprehensions.

Dictionaries are also built-in. You can think of them as JS object literals
(without the inherited properties). Dictionary comprehensions exist.

Both sets and dictionaries are written using curlies, so be mindful of that.

### Scoping

Inside a function, you can reference a variable from an enclosing non-global
scope, but if you try to write to it, a new variable is actually created in
the current scope. Non-local non-global variables are immutable to you.

### Misc statements

Unlike JS, you don't need to declare a variable, you just assign to it.  
Python has multiple assignment:  
`a, b = 1, 4`

The `pass` statement is a no-op, and it exists because Python doesn't have
semicolon.

### Control flow

IF syntax: with colon and elif
```
if x < 0:
  print 'Negative'
elif x == 0:
  print 'Zero'
elif x == 1:
  print 'One'
else:
  print 'More'
```

FOR is always a for/in, not a classic for:
```
words = ['cat', 'window', 'defenestrate']
for w in words:
  print w, len(w)
```
You use `range()` to create a numeric sequence and iterate over it. Example:
```
squares = []
for x in range(10):
  squares.append(x**2)
```
Wasteful; you need to create the whole sequence when you actually need only
one element at a time. To avoid that, you can use a while loop.

Weird: loops can have an else clause, which gets executed after the loop only
if the loop didn't exit through a break.

### Exceptions

Example:
```
try
  # Your code here.
  # To throw an exception, the Python keyword is “raise”.
except SomeExceptionName as e:
  # exception handling code
finally:
  # cleanup code that runs no matter what
```

### Functions

When defining a function with default values for arguments, the defaults are
evaluated only once, at the function definition site, not at every call to the
function! Therefore, if the default is mutable, its value persists between
calls.

Keyword arguments.  
Sample function definition:  
`def parrot(voltage, state='asdf', action='voom', type='Norwegian Blue'): ...`  
Sample function call:  
`parrot(voltage=1000000, action='VOOOOOM')`

Similar to JS's `arguments`, you can pass arbitrary extra positional
arguments, and refer to them inside the function with a formal named
`*some_name`. And you can pass arbitrary keyword arguments and refer to them
inside the function with a formal named `**some_name`, which is a dictionary.
You can use the `*` and `**` operators at call sites too, to unpack a list or
a dictionary to appropriate arguments for a function call.

You can define a lambda (function expression) like this: `lambda a, b: a+b`  
The body of the lambda must be a single expression!  
In the body, you can reference variables from the enclosing scope.

### Modules

The name of a file without the .py suffix is the name of the module.

```
# Puts the name of the module in the symbol table.
import foo
# Puts the name of the functions from the module in the symbol table.
from foo import f, g
# Imports all definitions from the module
from foo import *
# Import module with a custom name
import foo as my_foo
# Import specific function with a custom name
from foo import f as my_f
```

Modules can be organized in packages. A package is just a series of nested
directories, where each directory defines a file named `__init__.py`. Then you
can say things like:  
`import a.b.c.foo as my_foo`

### Files

Open a file for reading and read a line from it:
```
with open('my_file_name', 'r') as f:
     read_data = f.readline()
```

By using `with`, we ensure that the file is closed after the `with` body is
done.

### Classes

Fields are called attributes in Python lingo.

Example class definition:
```
class Foo:
  # Static attribute
  w = 123
  # The constructor is called __init__
  def __init__(self, x, y):
    # Instance attribute
    self.a = x
  def my_method(self):
    print(“asdf”)

# After the class definition, the class object is available, and its
# name is Foo. Classes are first-class values in Python.
z = Foo(1, 2)
n = Foo.w

# The next two are equivalent:
Foo.my_method(z)
z.my_method()

# Defining a subclass
class Bar(Foo):
  # the code of Bar

issubclass(Bar, Foo) # evaluates to true
isinstance(z, Foo) # also true
```

It is a convention to name the first argument of a method self; it is not
enforced.

You can define a function outside a class, and then assign it to a class
attribute, and after that the first argument of the function will be the
receiver object.

All attributes of a class are public. By convention, you prefix an attribute
with an underscore to say it's private; but the rule is not enforced by the
language.

Multiple inheritance: you can define multiple base classes for a class and
they are searched left to right for method resolution.

### Iterators / Generators

A for loop in Python uses an iterator under the scenes. You can define your own iterator by defining a class with an `__iter__` and a `next` method.

A generator is a compact way of writing an iterator: you write it as a
function instead of as a class, and call `yield` in the body. When the
generator ends, it automatically raises `StopIteration`.

### Coding style

[PEP8](https://www.python.org/dev/peps/pep-0008/) is the full style guide.

The convention is to use CamelCase for classes and lower_case_with_underscores
for functions and methods. Always use self as the name for the first method
argument.

### Libraries

Python is great for scripting; much more manageable than bash. See standard
libraries os, shutil, and re (for regular expressions). The logging library
provides basic logging functionality.

### Misc

Underscore can be used to get the result of the last expression in the repl.

None is a value similar to JS's undefined. A return without an expression
makes a func return None. Finishing the func body w/out a return is also the
same as returning None.

Python provides weak references.
