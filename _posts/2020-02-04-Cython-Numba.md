# "Cython Vs Numba"
> "This blog post is going to be a little different to the previous few posts I have made, there will be essentially no mathematics nor code. It is not intended as a "how to" or instructional post, merely a repository for my (current) opinions."
- toc: true
- author: Lewis Cole (2020)
- branch: master
- badges: false
- comments: false
- categories: [performant-python, computation, cython, numba]
- hide: false
- search_exclude: false

## Outline of the Problem
Python at its core is slow for certain things. By being a dynamically typed and interpreted language you incur certain runtime overheads. In some cases these are not much of an issue. At other times they can be critical.

Typically we will try and "vectorize" the code as much as possible (avoiding extraneous loops) and force as much code as we can into NumPy array operations which are typically "quick" (compiled C code). This is fine when it works, but it is not always possible to vectorize the code or, in some cases, the vectorization leads to code that is very hard to read/understand.

Both Numba and Cython (not to be confused with CPython) aim to provide tools to deal with such situations.

## Outline of Cython
Cython is a programming language that is part Python and part C/C++, it can be compiled into a python extension and/or an executable. If you are familiar with Python it is reasonably easy to understand Cython code, it largely just has a few "boiler-plate" code blocks along with a few static type declarations (familiar to those who know C/C++/related languages).

Since there are no dynamic types (in well written Cython code) and it is compiled typically the resulting code is orders of magnitude faster than Python. Compared to vectorised NumPy there may not be a significant improvement but this depends on the exact implementation.

Cython itself is very flexible, if you can express the code in Python it is unlikely you will not be able to express it in Cython. Any arbitrary class structure can work within Cython, as a result it is used for many "high performance" Python packages (e.g. SciPy).

It is possible to parallelize the code or utilise GPU computation using Cython. This normally requires a bit of work but typically does not require nearly as much work as using Cuda in C++ (for example). As usual the normal caveats relating to multi-thread applications also apply to Cython code.

[You can read the Cython documentation here!](https://cython.readthedocs.io/en/latest/)

## Outline of Numba
Numba is a slightly different beast. It uses the concept of a "just in time" compiler (JIT). Essentially this means that code is compiled "on the fly" during runtime instead of requiring compilation prior to execution. Numba compiles the python code using a LLVM compiler.

The syntax is very simple and most of the time just requires a simple decorator on a Python function. It also allows for parallelisation and GPU computation very simply (typically just a "target = 'cuda'" type statement in a decorator). In my experience a lot less thinking is required to set this up compared to Cython.

The downside of Numba (at least for me) is that it is a (comparatively) new package and as such does not have support for absolutely everything you would want, unlike with Cython it is possible to just "hit the wall" where you simply cannot use Numba without a major re-writing of the code. (One such example being you cannot call @guvectorize functions inside the @njit decorator). It is worth checking the github issues log regularly as often these issues are on the docket to be corrected in future releases.

Another downside of Numba is the lack of useful traceback, typically you need to "switch off" Numba and run in regular python to track down an error. This is typically only a minor inconvenience but if the code is particularly slow it can get frustrating trying to find an error without the Numba speed up.

[You can read more about Numba here!](https://numba.pydata.org/)

## Which is better?
From a raw performance perspective I do not see either Cython nor Numba consistently beating the other in all situations. Typically the performance will be comparable and you will rarely find one being many orders of magnitude quicker (assuming you're using both correctly). 

The choice of which to use, in my opinion, comes down to other factors. Convenience being a big one, I typically find Numba easier and quicker to implement when it works. As noted above however it doesn't always work (e.g. if using class structures, custom data types, etc.) With familiarity you do get an instinct as to whether a code will work or not. Cython on the other hand offers much more flexibility.

There is also the issue of how the code will be used. Cython is well established for creating efficient extension modules that sit nicely within the Python eco-system. Numba can be used in a similar way but I have found it a bit more finnicky to deal with (for example through Numba itself changing its API fairly regularly since it's a relatively new module, some code from previous iterations of Numba simply does not work at all with the later versions). 

## What is the catch?
Unfortunately things are not perfect, typically we will still be interfacing Cython/Numba functions via Python and so using repeated calls to these functions we will still incur overheads (typically through the conversion to Python types). This can mean that certain code is still significantly slower than C/C++ equivalents. These packages are therefore most useful for when you have profiled your code and can see that a handful of functions/operations are the real bottleneck. 

These packages may not help if your code is particularly memory intensive, in which case it is better to spend time thinking about memory management instead. In some cases these packages provide some help in that respect also (e.g. NumPy is prone to creating many cached variables for simple operations, if the variables are large arrays this can become a pain.) 

## What about PyPy/etc?
Another option for performant Python code is to use PyPy instead of CPython. I have not used this very much yet, if I get the time to really kick the tyres I may write another blog on my findings. There are some features that appear useful but the eco-system is not as well supported (yet?) and so may require some additional work to recreate some high level functionality.

[You can read more about PyPy here!](https://www.pypy.org/)

## Why not just C/C++?
Ultimately if you require peak performance at all costs these options are still no substitute for well written C/C++. However as I often warn people: computation time is generally cheaper than human time - it is often better to use a slightly sub-optimal (but still respectable) code than devote months to R&D and slow down the development cycle. Since Numba/Cython are so similar to Python (and it is possible to just "tack on" some Python to the end of these codes) you can prototype much more quickly in my experience. All these factors (along with many others such as where the code is to be deployed, what other tools are being used, etc.) need to be weighed up.

## What about Julia?
Some readers may be familiar with the Julia language as an option for high performance scientific computing. I have a little experience (but am far from an expert). As I understand Julia is based around JIT (as with Numba), however being a language to itself it never needs to interface with Python and its limitations. It can therefore create more efficient code for larger scale projects since they never have to worry about Python overheads. Typically the benchmarks seen online are for smaller "toy" problems and so the performance does not appear to be too different from Cython/Numba.

I have not switched to Julia for a few reasons, firstly the popularity of Python - it is typically fairly easy to learn Cython/Numba for somebody who understands Python/NumPy making collaboration easier. Secondly the Python eco-system is well developed there is typically a package available to do almost anything you would want. Thirdly at this point it is fairly easy to get Python to "speak" with other systems if you need to turn something from a prototype to production. These concerns are ultimately just related to uptake however, as more people use Julia I see this becoming less of a concern.

[You can read more about Julia here!](https://julialang.org/)

## Conclusion
Hopefully now we can see that Cython/Numba provide useful tools for bridging the gap between Python and C/C++ runtimes. As the old saying goes "you cannot have your cake and eat it too" and so it may not be possible to get performance as quick using these options. However we can often get performance that is "good enough" in practical terms.