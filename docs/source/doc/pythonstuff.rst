********************
Python Functionality
********************

Extension Types (Classes)
=========================
Numba supports classes similar to Python classes and extension types.
Currently the methods have static signatures, but in the future they
we will likely support multiple inheritance and autojitting methods.

One can refer to the extension type as a type by accessing its ``exttype``
attribute::

    @jit
    class MyExtension(object):
        ...

    @jit(MyExtension.exttype(double))
    def create_ext(arg):
        return MyExtension(arg)

It is not yet possible to refer to the extension type in the class body or
methods of that extension type.

An example of extension classes and their capabilities and limitations
is shown below:

.. literalinclude:: /../../examples/numbaclasses.py

Closures
========

Numba supports closures (nested functions), and keeps the variable scopes
alive for the lifetime of the closures.
Variables that are closed over by the closures (``cell variables``) have
one consistent type throughout the entirety of the function. This means
differently typed variables can only be assigned if they are unifyable,
such as for instance ints and floats::

    @autojit
    def outer(arg1, arg2):
        arg1 = 0
        arg1 = 0.0      # This is fine
        arg1 = "hello"  # ERROR! arg1 must have a single type

        arg2 = 0
        arg2 = "hello"  # FINE! Not a cell variable

        def inner():
            print arg1

        return inner

Calling an inner function directly in the body of ``outer`` will result in
a direct, native call of the closure. In the future it is likely that passing
around the closure will still result in a native call in other places.

Like Python closures, closures can be arbitrarily nested, and follow the same
scoping rules.

Typed Containers
================
Numba ships implementations of various typed containers, which allow fast
execution and memory-efficient storage.

We hope to support the following container types:

    * list, tuple
    * dict, ordereddict
    * set, orderedset
    * queues, channels
    * <your idea here>

There are many more data structure that can be implemented, but future releases
of numba will make it easier (nearly trivial) for people to implement those
data structure themselves while supporting full data polymorphism.

Currently implemented:

    * typedlist
    * typedtuple

These data structures work exactly like their python equivalents, but take a
first parameter which specifies the element type::

    >>> numba.typedlist(int32, range(10))
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
    >>> numba.typedlist(float32, range(10))
    [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0]

    >>> tlist = numba.typedlist(int32)
    >>> tlist
    []
    >>> tlist.extend([3, 2, 1, 3])
    >>> tlist
    [3, 1, 2, 3]
    >>> tlist.count(3)
    2L
    >>> tlist[0]
    3L
    >>> tlist.pop()
    3L
    >>> tlist.reverse()
    >>> tlist
    [1, 2, 3]

Things that are not yet implemented:

    * Methods ``remove``, ``insert``
    * Slicing

Typed containers can be used from Python or from numba code. Using them from numba code
will result in fast calls without boxing and unboxing.

