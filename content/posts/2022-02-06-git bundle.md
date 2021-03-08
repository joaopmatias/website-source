---
layout: post
date: 2021-02-06
title:  "Debugging an issue related with Java and Python interoperability"
draft: true
---

I have been using [JEP](https://github.com/ninia/jep) (Java Embeded Python) to allow calling the code written, in Python, by a team of data scientists in some Java microservices for quite a while now. In general, this solution has been very stable and fast.

However, a while back and after some specific changes in the Python code, we started seeing some segfaults occurring when the code that would run smoothly in Python was executed through JEP. Fortunately, the story ends in success and I would like to share my experience while debugging this issue. I hope some of its aspects ressonate with others.

# The issue 

The issue is documented [here](https://github.com/ninia/jep/issues/241). In broad strokes, running some models from the `pmdarima` or `statsmodels` packages was causing a segfault and due to the sudden crash, a traceback was not available.


Both packages, use Scipy and given the complexity of this library, it was possible that it would be simply impossible to use Jep to run that specific code.



OUT
This is the story of the time when my application that called Python code from within a Java program inexplicably kept crashing. This issue is documented in a github issue of Jep (Java Embedded Python) in https://github.com/ninia/jep/issues/241.

Jep is a project that allows to call Python code from a Java program and to transfer data between the two environments. Moreover, all operations are done in memory so the overall performance is comparable to either languages.

It proved to be useful as a link between Java microservices and the Python code developed by the data science team. Recently, after applying some changes in the Python code, it would cause the Java program to crash suddenly without any stack trace. Since it was using Scipy indirectly and given the complexity of the library, it was possible that it would be simply impossible to use Jep to run that specific code.
OUT



# Writing a minimal example

Naturally, the first instinct was to attempt to pinpoint the part of the Scipy executed immediately before the application crashed. Unfortunately, we didn't find any good debugging tools for this scenario, so we resorted to good old print statements in the middle of the Scipy code that had to be rebuilt and reinstalled everytime after adding more print statements.


pip build

.pyc


The search lead to the routine `cimport scipy.linalg.cython_lapack as lapack` and I wrote a minimal example that reproduced the error.

Soon after, it was pointed out by a JEP developer that the error would not occur if Scipy were built from source beforehand. However, this procedure did not solve the issue completely as it would still occur in some cases other than my minimal example.


OUT
I was clueless again.
OUT



# Reading the source code

As we get deeper in JEP, it is important to be aware of some of its inner gears. In order to run Python code, JEP uses two different bridging technologies: the CPython C-API and the JNI (Java native interface). The first allows to call Python code from C and the second allows to call C code from Java.


BLA BLA Jep has a MainInterpreter in a different thread



OUT
Additionally, Jep creates and runs the Python code in a separate thread. NOT TRUE!!!!
OUT


# Running Scipy tests

Then, I collected a few more code snippets in the Scipy tests that would spark a similar error, to compare in different settings.


pytest.main([...])




So, based on code snippets provided by the Jep developers, I put up an example calling Python code in a separate thread through the CPython C-API. 

Using this snippet it was possible to replicate some segfault errors. However, sometimes the Python code causing these segfaults would run without problems in Jep or vice-versa.






# Is it the threads?



Then, we tried to run Python code running in a separate thread created within the Python code using the `threading` module. Surprisingly, this would work without problems! So, using the `threading` module was causing a behavior different from the Python code not using it. We started looking for more details in the CPython implementation, which eventually lead us to the function `PyThread_start_new_thread`. After some trial and error we got our first functioning example that called it directly and kept trying to peel it more.





In this point it is important to mention the effect of threads in the outcomes of these runs. When Python code is called in the main thread of a C program using the CPython C-API, it would run flawlessly. Moreover, if Python code were called in Java through Jep in a separate thread, it would also run flawlessly.


It so happens that `PyThread_start_new_thread` is generated getting some attributes as input and some clearer than others. One of then is the size of the stack that is put in a helper variable in a hexadecimal format. That was one of the key differences between the use of `create_pthread` that I had in the examples using the CPython C-API and the CPython source code. After searching how to increase the stack size of a java program I crossed my fingers, ran Jep with the examples I gathered and it worked!



PyThread_start_new_thread
attribute
hexadecimal BLA BLA




Despite not having a full grasp of all the parts involved in this issue, some trial and error proved worth-while. Along this challenge I also had the opportinity of looking into many concepts and code that I would not have otherwise.

I hope this may have inspired you in some way. Feel free to reach out if you have any comments!





...

and it failed again BUT







Why?

Cos debugging is cool

Cos Java and Python is cool



What?

You found a blocker and you got through



How?

Many examples

