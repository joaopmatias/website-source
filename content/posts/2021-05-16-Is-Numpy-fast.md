---
layout: post
date: 2021-05-16
title:  "Is Numpy really faster? - A second look"
---

I recently came accross an interesting blog post by Bjorn Madsen, [Is Numpy really faster](https://root-11.github.io/content/is-numpy-really-faster/index.html).

It is a short post displaying one example where a python snippet using Numpy is much slower than the version written in pure Python due to some overheads associated to Numpy.

Building upon that example, I spent some time writing a function using Numpy that performs similar operations even faster than the pure Python version shown by Bjorn. There is a catch however, I replaced a for loop in the benchmark code by a vector operation, which is the scenario where Numpy really shines.

Let us take look!

```Python
import numpy as np

v1 = [1, 2, 3]
v2 = [2.4, 3, -1]


def f1(v1, v2):  # <--- Using numpy.cross
    return list(np.cross(v1, v2))


def f2(v1, v2):  # <---- Using python
    a1, a2, a3 = v1
    b1, b2, b3 = v2
    return [a2 * b3 - a3 * b2, -(a1 * b3 - a3 * b1), a1 * b2 - a2 * b1]


def x1():  
    for i in range(100000):
        v3 = f1(v1, v2)  # repeated calls for profiling usage of numpy


def x2():
    for i in range(100000):
        v4 = f2(v1, v2)  # # repeated calls for profiling usage of python.

def x3():
    v22 = np.tile(np.array(v2, dtype=np.float64), (100000, 1))
    v11 = np.tile(np.array(v1, dtype=np.float64), (100000, 1))
    np.cross(v11, v22)
```

`x3()` is a new function implementing a vector operation in Numpy. Then, I wrote all these functions in a jupyter notebook and used the `%prun` magic to analyse the results. Here they are.

Original function using Numpy:
```
>>> %prun x1()

         8200004 function calls (7900004 primitive calls) in 5.442 seconds

   Ordered by: internal time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
   100000    1.739    0.000    5.042    0.000 numeric.py:1485(cross)
   600000    0.818    0.000    1.262    0.000 numeric.py:1341(normalize_axis_tuple)
   300000    0.757    0.000    2.354    0.000 numeric.py:1404(moveaxis)
   300000    0.384    0.000    0.384    0.000 {built-in method numpy.array}
   900000    0.252    0.000    0.252    0.000 {built-in method numpy.core._multiarray_umath.normalize_axis_index}
400000/100000    0.211    0.000    5.117    0.000 {built-in method numpy.core._multiarray_umath.implement_array_function}
   100000    0.206    0.000    5.395    0.000 <ipython-input-78-bfb5de128972>:7(f1)
   600000    0.157    0.000    0.296    0.000 numeric.py:1391(<listcomp>)
   300000    0.146    0.000    2.670    0.000 <__array_function__ internals>:2(moveaxis)
  1900000    0.141    0.000    0.141    0.000 {built-in method builtins.len}
   300000    0.107    0.000    0.107    0.000 {built-in method builtins.sorted}
   300000    0.096    0.000    0.096    0.000 {method 'transpose' of 'numpy.ndarray' objects}
   200000    0.058    0.000    0.389    0.000 _asarray.py:23(asarray)
   100000    0.057    0.000    5.189    0.000 <__array_function__ internals>:2(cross)
   600000    0.053    0.000    0.053    0.000 {built-in method _operator.index}
   300000    0.048    0.000    0.048    0.000 numeric.py:1467(<listcomp>)
        1    0.047    0.047    5.442    5.442 <ipython-input-78-bfb5de128972>:17(x1)
   300000    0.046    0.000    0.046    0.000 {method 'insert' of 'list' objects}
   100000    0.036    0.000    0.036    0.000 {built-in method numpy.empty}
   300000    0.034    0.000    0.034    0.000 numeric.py:1400(_moveaxis_dispatcher)
   100000    0.034    0.000    0.034    0.000 {built-in method numpy.promote_types}
   100000    0.014    0.000    0.014    0.000 numeric.py:1481(_cross_dispatcher)
        1    0.000    0.000    5.442    5.442 {built-in method builtins.exec}
        1    0.000    0.000    5.442    5.442 <string>:1(<module>)
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
```

Original function written in pure Python:
```
>>> %prun x2()

         100004 function calls in 0.066 seconds

   Ordered by: internal time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
   100000    0.045    0.000    0.045    0.000 <ipython-input-78-bfb5de128972>:11(f2)
        1    0.021    0.021    0.066    0.066 <ipython-input-78-bfb5de128972>:22(x2)
        1    0.000    0.000    0.066    0.066 {built-in method builtins.exec}
        1    0.000    0.000    0.066    0.066 <string>:1(<module>)
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
```

New function replaced a for loop with Numpy vectorization:
```
>>> %prun x3()

         117 function calls (114 primitive calls) in 0.005 seconds

   Ordered by: internal time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.004    0.004    0.004    0.004 numeric.py:1485(cross)
        2    0.001    0.001    0.001    0.001 {method 'repeat' of 'numpy.ndarray' objects}
        7    0.000    0.000    0.000    0.000 {built-in method numpy.array}
        2    0.000    0.000    0.001    0.001 shape_base.py:1171(tile)
        3    0.000    0.000    0.000    0.000 numeric.py:1404(moveaxis)
        6    0.000    0.000    0.000    0.000 numeric.py:1341(normalize_axis_tuple)
        1    0.000    0.000    0.005    0.005 {built-in method builtins.exec}
      6/3    0.000    0.000    0.005    0.002 {built-in method numpy.core._multiarray_umath.implement_array_function}
        1    0.000    0.000    0.005    0.005 <ipython-input-78-bfb5de128972>:26(x3)
        4    0.000    0.000    0.000    0.000 {method 'reshape' of 'numpy.ndarray' objects}
        3    0.000    0.000    0.000    0.000 {method 'transpose' of 'numpy.ndarray' objects}
        6    0.000    0.000    0.000    0.000 numeric.py:1391(<listcomp>)
        3    0.000    0.000    0.000    0.000 {built-in method builtins.sorted}
        9    0.000    0.000    0.000    0.000 {built-in method numpy.core._multiarray_umath.normalize_axis_index}
        2    0.000    0.000    0.001    0.001 <__array_function__ internals>:2(tile)
       21    0.000    0.000    0.000    0.000 {built-in method builtins.len}
        3    0.000    0.000    0.000    0.000 <__array_function__ internals>:2(moveaxis)
        4    0.000    0.000    0.000    0.000 shape_base.py:1243(<genexpr>)
        1    0.000    0.000    0.005    0.005 <string>:1(<module>)
        6    0.000    0.000    0.000    0.000 {built-in method _operator.index}
        6    0.000    0.000    0.000    0.000 shape_base.py:1253(<genexpr>)
        3    0.000    0.000    0.000    0.000 {method 'insert' of 'list' objects}
        1    0.000    0.000    0.000    0.000 {built-in method numpy.empty}
        3    0.000    0.000    0.000    0.000 numeric.py:1400(_moveaxis_dispatcher)
        2    0.000    0.000    0.000    0.000 _asarray.py:23(asarray)
        2    0.000    0.000    0.000    0.000 {built-in method builtins.all}
        1    0.000    0.000    0.000    0.000 {built-in method numpy.promote_types}
        1    0.000    0.000    0.000    0.000 numeric.py:1481(_cross_dispatcher)
        1    0.000    0.000    0.004    0.004 <__array_function__ internals>:2(cross)
        2    0.000    0.000    0.000    0.000 shape_base.py:1167(_tile_dispatcher)
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
        3    0.000    0.000    0.000    0.000 numeric.py:1467(<listcomp>)
```

In the end, the Numpy vectorization can be faster overall, but always remember to use Numpy wisely!

I hope you enjoyed this short experience and please visit the [original post](https://root-11.github.io/content/is-numpy-really-faster/index.html) by Bjorn Madsen!
