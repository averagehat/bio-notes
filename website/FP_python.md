###Controlling closures

Closures exist in Python like they do in JavaScript. defining a function within another''s bounds grabs the scope of the outer function.


This is particularly useful when doing reactive or GUI programming, but has other uses as well. 
It''s also useful when we want to *vectorize* our code--here, by using anonymous functions, and "cross-applying" lists to lists. Some examples of vectorization:
In [33]: import numpy as np
In [34]: np.array([1, 2, 3]) + np.array([2, 3, 4])
Out[34]: array([3, 5, 7])

In [35]: [ a + b for a, b in zip([1, 2, 3], [2, 3, 4]) ]
Out[35]: [3, 5, 7]

In [36]: from operator import add 
In [37]: map(add, [1, 2, 3], [2, 3, 4])
Out[37]: [3, 5, 7]
(Notice that map can take any number of sequences as arguments--the number must match the number of parameters in the passed/mapped function.)


i.e. 

    

In [6]: def foo(one, two):
   ...:     def inner(a):
   ...:         print one, a
   ...:     inner('a')

In [7]: foo('one', 'b')
one a


But also:
In [8]: def foo(one, two):
   ...:     def inner(a):
   ...:         print one, a
   ...:     return inner
In [28]: foo('outer', '_')('inner')
outer inner



Note that the function saves the *scope*, it doesn''t copy the variable; so if the scope changes, the variable will change as well. This can lead some unexpected behavior, like below. Let''s try some more vectorization, this time by attempting to use closures. Say we want to do a series of transformations on a list, but each transformation should be unique to its position. We may want to store a list of functions which can be applied (and re-used) to another list (or list of lists).

n [20]: gen  = [lambda c: str(c)*N for N in xrange(5)]

This also demonstrates the [primary?] usefuleness of lambda functions; they can be vectorized. Unfortunately, they are limited by their size--they can only contain one "expression"/statement. Otherwise, they differ little from named functions (declared with `def`). Let''s see how our vectorization worked out:

In [19]: [func(a) for func, a in zip(gen, ['a', 'b', 'c', 'd', 'e'])]
Out[19]: ['aaaa', 'bbbb', 'cccc', 'dddd', 'eeee']
That''s not right. We naively expected a list of characters of varying length--we wanted our lambdas to do different things-- instead, they did the same thing! 
This is because the function stored the *scope*, not the *variable*--and the scope changed. by the time we use our lambdas, N is equal to 4 within that scope! Bummer.
Fortunately, keyword arguments offer a very friendly work-around:
n [20]: gen  [lambda c, N=N: str(c)*N for N in xrange(5)]
In [21]: [func(a) for func, a in zip(gen, ['a', 'b', 'c', 'd', 'e'])]
Out[21]: ['', 'b', 'cc', 'ddd', 'eeee']
Here, we can use the keyword argument to force the lambda expression to store the variable as it is, and we get our intended result!

Another example that demonstrates this same closure hiccup will also help demonstrate "lazy evalutation." The objects (or functions) within an iterator are only "evaluated" (become expressions) when they are retrieved/yielded. If a function is not evaluated, it will again look "above" itself in the scope for its variables/arguments. 
values = (( func(obj) for func in lambdas) for obj in collection)
# Fails
values = (list( func(obj) for func in lambdas) for obj in collection)
# Works

In [56]: matrix = ((func(letter) for func in gen) for letter in ['a', 'b', 'c',])

In [57]: list(matrix)
Out[57]: 
[<generator object <genexpr> at 0x7fab8d305c30>,
 <generator object <genexpr> at 0x7fab8d305cd0>,
 <generator object <genexpr> at 0x7fab8d305d20>]

In [58]: map(list, matrix)
Out[58]: []

In [59]: matrix = [(func(letter) for func in gen) for letter in ['a', 'b', 'c',]]

In [60]: map(list, matrix)
Out[60]: 
[['', 'c', 'cc', 'ccc', 'cccc'],
 ['', 'c', 'cc', 'ccc', 'cccc'],
 ['', 'c', 'cc', 'ccc', 'cccc']]

In [61]: matrix = [[func(letter) for func in gen] for letter in ['a', 'b', 'c',]]

In [62]: map(list, matrix)
Out[62]: 
[['', 'a', 'aa', 'aaa', 'aaaa'],
 ['', 'b', 'bb', 'bbb', 'bbbb'],
 ['', 'c', 'cc', 'ccc', 'cccc']]

Note that this example differs from the one aboves because here we are *evaluating* the function, where as before we were *defining* the functions with in a comprehension

Work things
1. 
qtconsole: have to copy from the front of the line (i.e. [In]) in order to get the [in]/[Out] parts for the whole copy!
