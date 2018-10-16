.. include:: ../../links.txt
.. include:: ../../roles.txt

.. _lec4:

.. contents:: Table of Contents
   :local:
   :depth: 1
   :backlinks: top

.. _lec4_func_basis:

The Basis
=========

.. contents::
   :local:
   :backlinks: top

*Function* is an important concept in programming, because it allows us to
easily modularize our programs and reuse the codes in the future. In this
lecture, I will show you how to write functions in `C++`_.

Fundamental Concepts of Functions in `C++`_
-------------------------------------------

Currently, we write everything inside the :ref:`main <intro_simple_cpp_main>`
function. This is fine for small projects, say homework assignments. However,
for larger projects, this can be very limited. Consider the following
situation.

.. code-block:: cpp

    int flag;
    std::string method;
    // do something with flag
    flag = ...;
    switch (flag) {
    case 0:
        method = "method1";
        break; // don't forget break the switch
    case 1:
        method = "method2";
        break;
    case 2:
        method = "method3";
        break;
    default:
        method = "default";
        break;
    }

    // then run different statements with different "methods"


This code looks fine, but assume you need to determine the "method" twice in
your program, then the intuitive solution would be: "okay, let's just copy
and paste this part." This, of course, works, but what about later on, you need
to add another method, say "method4". You need to add "method4" twice, otherwise
your program may run into troubles.

At this moment, you probably already notice the limitation of this approach,
i.e. not extendable and easy to introduce bugs. A right way is to use a
function:

.. code-block:: cpp
    :linenos:

    std::string chooseMethods(int flag) {
        std::method;
        switch (flag) {
        case 0:
            method = "method1";
            break; // don't forget break the switch
        case 1:
            method = "method2";
            break;
        case 2:
            method = "method3";
            break;
        default:
            method = "default";
            break;
        }
        return method;
    }

Now, in your program, whenever you need to determine the "method", you can
simply **call** the function ``chooseMethods``:

.. code-block:: cpp

    int flag;
    std::cout << "enter method flag integer: ";
    std::cin >> flag;
    std::string method = chooseMethods(flag);

In this way, everytime when you need to add a new method, there is only
once place you need to worry about, i.e. the function ``chooseMethods``.

Now, let's take a closer look at the function above, in which ``chooseMethods``
is its *name*, :code:`std::string` is the *return type*, and :code:`int flag`
is the *parameter list*.

.. tip:: Functions are objects.

Essentially, every function has:

1. a name,
2. a return type, and
3. a parameter list.

.. _lec4_func_empty_return:

Be aware that both 2 and 3 can be empty argument, i.e. ``void`` for empty
return and empty parameter list for the latter.

.. code-block:: cpp

    // an empty param list with empty return
    void printHelloWorld() {
        std::cout << "Hello World\n";
    }

Be aware that *empty value return* doesn't mean there is no return in the
function. The code above is equivalent to:

.. code-block:: cpp
    :emphasize-lines: 3

    void printHelloWorld() {
        std::cout << "Hello World\n";
        return;
    }

The emphasized line demonstrates that this function returns *empty value*.
Returning empty value is not necessary, but sometimes it can be useful.

.. code-block:: cpp

    void doWork(double *data) {
        *data = 1.0;
    }

For the ``doWork`` function, you can simply use it like:

.. code-block:: cpp

    double a;
    doWork(&a); // recall address-of operator to get the pointer
    std::cout << "a=" << a << ".\n"; // print 1

However, we know that if ``data`` is ``nullptr``, then we have a big problem,
i.e. recall dereferencing ``nullptr`` will cause ``seg fault``.

In this case, you can do a *quick return* by checking ``data``, and this
requires returning empty stage.

.. code-block:: cpp

    void doWork(double *data) {
        if (!data) {
            return;
        }
        *data = 1.0;
    }

The Return Type
+++++++++++++++

The return type of a function must be listed explicitly and uniquely, i.e.
you cannot have a function that has multiple return types.

.. code-block:: cpp

    int myFunc() { return 1; } // Ok
    // int, int want2Return2Ints() { return 1, 2; } // ERROR!

However, there are always workarounds to mimic multiple returns that appear
in dynamic languages, e.g. `Python`_. One of them is to use a
:ref:`struct <lec3_stm_loop_struct>`.

.. code-block:: cpp

    struct ComplexNumber {
        double real;
        double imag;
    }; // don't forget the semicolon

    ComplexNumber getDefaultComplexNumber() {
        ComplexNumber a;
        a.real = 0.0;
        a.imag = 0.0;
        return a;
    }

Nonempty return types can be handles or omitted.

.. code-block:: cpp

    bool assign(const int len, double *array, const double value) {
        if (!array || len < 0) {
            return false;
        }
        for (int i = 0; i < len; ++i) {
            array[i] = value;
        }
        return true;
    }

You can use the function above either

.. code-block:: cpp

    if (!assign(len, array, value)) {
        // recall that you should use cerr for error streaming
        // if the inputs are not acceptable, then we know that array is not
        // been touched
        std::cerr << "invalid inputs\n";
    }

or, simply do

.. code-block:: cpp

    assign(len, array, value);


The Parameter List
++++++++++++++++++

The :code:`(int flag)` in ``chooseMethods`` and :code:`(double *data)` in
``doWork`` are called parameter lists. In general, the parameter list of
functions is a *comma-separated* *declaration-like* of parameters. Therefore,
for a parameter list, multiple arguments are, of course, supported.

.. code-block:: cpp

    void demoParList(int a, double b, std::string method, double *data) {
        // do work
    }

.. note::

    The `C++`_ function parameter lists have strict order, i.e. there is
    not *key value pairs* in `C++`_ regarding function inputs.

Return Pointers & References
++++++++++++++++++++++++++++

There is nothing to stop you from returning :ref:`pointers <lec2_ptr>` and
:ref:`references <lec2_ref>`. However, whenever you directly access the memory,
special care must be taken.

Let's first look at the following function that returns a pointer:

.. code-block:: cpp

    int *getPointer() {
        int a = 1;
        int *ptr = &a;
        return ptr;
    }

On return, ``getPointer`` will return a **copy** of ``ptr`` and this behavior
is will defined. Now, let's look at the the *function body* (content inside
the function scope). A **local** integer ``a`` is created as well as a pointer
``ptr`` that points to it. The memory address of ``a`` is copied whiling
returning ``ptr``, but ``a`` is popped out from the stack right after the
return statement. **Therefore, the dereference of the returned pointer is
undefined.**

Typically, returning pointers is used for
:ref:`dynamic memory allocation <lec2_dyn>`:

.. code-block:: cpp

    double *allocArray(unsigned int size) {
        return new double [size];
    }

.. note:: Don't forget to relax the memory.

.. tip::

    If you need to write a function that returns a pointer pointing to
    the head memory, it's general practice that the function name should start
    with ``create``, ``alloc``, etc.

Returning references, on the other hand, is even tricker.

.. code-block:: cpp

    int &getRef() {
        int a;
        return a;
    }

The code above has the legal `C++`_ syntax. However, the returned reference
refers to some local variable that is gone once the function scope ends.
As a result, the refer you get from this function is undefined.

.. note::

    Typically, compilers will warn you for returning local references.

.. note::

    For new `C++`_ programmers, avoid returning references.

Forward Linked List, again, with Functions
++++++++++++++++++++++++++++++++++++++++++

Now, a more structured implementation of our
:ref:`forward linked list example <lec3_stm_loop_struct>` can be found in
:nblec_4:`func_fll`.

.. _lec4_func_pass:

Passing Arguments
-----------------

A very good example to understand argument passing in `C++`_ is the following
``swap`` function. A swapping operation is to exchange the contents between
two objects. Let's take a look at the following pseudo code of swapping:

.. code-block:: matlab

    Inputs: obj1, obj2
    Outputs: obj1 w/ value of obj2, and obj2 w/ value of obj1

    function swap(obj1, obj2)
    do
        obj1 <-> obj2
    end do

Following the pseudo code and with the knowledge in high level programming
languages, you probably simply come with a `C++`_ implementation:

.. code-block:: cpp

    void Swap1(int a, int b) {
        const int temp = a;
        a = b;
        b = temp;
    }

However, this code will not work. The reason is simple, because both ``a`` and
``b`` are copied locally inside the function thus having no effects to the
actual inputted parameters.

The Copy Property
+++++++++++++++++

By default, all arguments are copied by their values thus resulting
locally scoped variables. For instance, let's take a look at the following
usage of our "wrong" swapping function.

.. code-block:: cpp

    int lhs = 1, rhs = 2;
    Swap1(lhs, rhs);
    std::cout << "after swapping, lhs="<< lhs << ", rhs=" << rhs << ".\n";

During the calling of ``Swap1``, ``lhs`` and ``rhs`` are copied as local
variables ``a`` and ``b``, so they have totally different memory addresses
comparing to the original ``lhs`` and ``rhs``. Therefore, any operations
performed on ``a`` and ``b`` have no effects to ``lhs`` and ``rhs``.

Since we have just mentioned memory addresses, you probably can simply come
up a proper implementation like:

.. code-block:: cpp

    void Swap2(int *a, int *b) {
        const int temp = *a;
        *a = *b;
        *b = temp;
    }

Now, the local copies of ``a`` and ``b`` are the pointers that still point to
the input arguments. Therefore, the code above can successfully swap the
contents.

.. code-block:: cpp

    int lhs = 1, rhs = 2;
    Swap2(&lhs, &rhs); // be aware that we pass in memory addr
    std::cout << "after swapping, lhs="<< lhs << ", rhs=" << rhs << ".\n";

In programming, this copy property is referred as *pass by values* (PBV).
Notice that with PBV, you duplicate each of the parameters thus doubling
the memory usage.

.. note:: `MATLAB`_, by default, has PBV property.

Download and play around with :nblec_4:`swap`.

*Pass by References* (PBR)
++++++++++++++++++++++++++

`C++`_ allows you pass parameters as their references. This is a convenient
feature that allows one to write efficient code.

.. code-block:: cpp

    void Swap3(int &a, int &b) {
        const int temp = a;
        a = b;
        b = a;
    }

In the code above, instead of creating copies, two references that bind to
the input arguments are created. Recall that references are just alternative
names of their corresponding objects, therefore, any modification will affect
the original variables.

.. tip::

    Use :code:`const` reference with :code:`std::string` whenever possible.

In general, the creation of an :code:`std::string` requires a dynamic memory
allocation (because the size of the string is unknown) and a memory copying.
As a result, passing :code:`std::string` by value can be very inefficient.

.. code-block:: cpp

    // first printing
    void print1(std::string msg) {
        std::cout << msg << '\n';
    }

    // second printing
    void print2(const std::string &msg) {
        std::cout << msg << '\n';
    }

    std::string msg = "hello world!";

    for (int i = 0; i < 10000; ++i) {
        print1(msg);
        print2(msg);
    }

For the example above, without additional efforts, the first version requires
more resources than the second one does due to 10000 additional times of
dynamic memory allocation and data copying.

.. tip::

    Link we have learned in the :ref:`reference <lec2_ref>` section, the same
    rule applies for using references with functions, i.e. use :code:`const`
    whenever possible.

PBV vs. PBR
++++++++++++

In general, if the data is too large so that copying it becomes the bottleneck
of your programs, then you should switch to use PBR.

Considering ``Swap2`` and ``Swap3``, both of them perform the swapping
operation. The former requires copying the pointers, while references are
passed for the latter. For this case, it's hard so say which one is preferred.

.. tip::

    People come converted from C programming usually prefer passing objects
    by their memory addresses (pointers), because it's clear to them that the
    objects will be modified. For native `C++`_ programmers, the latter is
    usually used. But the drawback is that sometime people get confused about
    the function interface, e.g. the interface is identical for ``Swap1`` and
    ``Swap3``.

.. _lec4_func_pro_impl:

Function Prototypes & Implementations
--------------------------------------

Declarations & Definitions of Variables
+++++++++++++++++++++++++++++++++++++++

In :ref:`defining and initialing variables <lec1_define_init>`, we have learned
how to define a variable, say :code:`int a;`. With this simple piece of code,
two steps actually happen: 1) *declaring* ``a`` as an ``int``, and 2)
*defining* it in the stack memory.

We can, of course, explicitly separate these two steps by using the keyword
``extern`` in `C++`_.

.. code-block:: cpp

    extern int a;  // declaration

    int main() {
        std::cout << "a=" << a;
        return 0;
    }

    // define a
    int a = 1;

The separation of declarations and their corresponding definitions is
significant in `C++`_ (as well as in C), because this allows us to structure
libraries (packages) whose declarations go to the *header files* and
definitions stay in the *source files*. These concepts will be taught in the
future lecture.

.. note::

    Declarations can appear as many times as you want, but definitions must
    be unique!

.. code-block:: cpp

    extern int a;  // declaration
    extern int a;  // Ok
    extern int a;  // No problem

    int main() {
        std::cout << "a=" << a;
        return 0;
    }

    // define a
    int a = 1;
    // int a = 2; // ERROR, we already learned

Constant variables can also be declared first.

.. code-block:: cpp

    // declaration
    extern const int b;

    int main() {
        std::cout << "b=" << b;
    }

    const int b = 2; // define it, must be initialized


"Declarations" of Functions
+++++++++++++++++++++++++++

In `C++`_, a function's declaration is called *prototype*.

.. code-block:: cpp

    void myFunc(); // note that the semicolon indicates prototype
    void myFunc(); // you can declare as many times as you want...

Notice that :code:`extern` is optional.

"Definitions" of Functions
++++++++++++++++++++++++++

A function's definition is called *implementation*.

.. code-block:: cpp

    void myFunc() {
        std::cout << "calling myFunc\n";
    }

Notice that the implementation of a function is indicated by the scope
opener (:code:`{`) and closer (:code:`}`).

.. _lec4_func_adv:

Advanced Topics
===============

.. contents::
   :local:
   :backlinks: top

Function Types & Function Pointers
----------------------------------

Default Arguments
-----------------

Function Overloading
--------------------

Function Matching
-----------------