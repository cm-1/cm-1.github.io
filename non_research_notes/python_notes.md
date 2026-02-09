# Library Install Paths
Originally, when writing some Cython code that subclassed a sklearn Cython-written class, I thought I had to get the path of sklearn's .pxd files and pass that in as part of the `include_dirs` in my setup.py, similar to how numpy's .h files have to be obtained via `numpy.get_include()` and passed in. But the Cython compilation works without doing this for sklearn, so ultimately, the code I wrote below for doing so was unneeded. I guess perhaps it's because sklearn has .pxd Cython files rather than .h C files, so it's only needed for numpy because they're .h files? And for .pxd files, simply having sklearn installed in the current environment is sufficient?

Anyway, I'm still saving the code I wrote that gets the library install location, just in case it or similar is needed for a **different** reason in the future.

```py
from pathlib import Path
from sklearn import __path__ as sklearn_path
tree_path = str(Path(sklearn_path[0], "tree"))
print("tree_path:", tree_path)
```

# Cython
A lot of what I learned about Cython can be found in my sklearn decision tree criterion subclass (either implicitly in the code or explicitly in the comments/README), so these notes are not going to be a "complete" summary of what I've learned.

For function definitions, I guess you use **cdef** for functions you only need to call from Cython code, **def** from functions you only need to call from Python code, and **cpdef** for ones you need to call from both. You still place those **def** functions in your Cython .pyx file, but you can't specify a return type or "except" behaviour in the function definition in the same way; that is, you can't say `def int myFunc(...) except -1:`.

Some helpful resources for learning about Cython:

- https://cython.readthedocs.io/en/latest/src/tutorial/cython_tutorial.html

## Step-by-step debugging
I had the error `realloc(): invalid next size` causing a crash in some Cython code that scikit-learn was calling, which meant that I couldn't use "normal" step-by-step Python debugging. The nature of the code also made printf statements implausible as there'd just be too many. So I tried to use cygdb to debug it, [as described in the Cython docs](https://cython.readthedocs.io/en/latest/src/userguide/debugging.html) but was facing issues. I installed `python3-debug` (the Fedora equivalent of python-dbg I guess) and ran `python3-debug setup.py build_ext --inplace`, then ran `cygdb --build-dir ./dir_containing_pyx_code -- --args python3-debug code_calling_sklearn.py` which started gdb, then ran `start`, but wasn't able to set any breakpoints for either my python code or Cython code.

Copilot gave some suggestions that half-worked, and some of them may have been unnecessary, but they at least let me get a backtrace printed.

First, it suggested to build my .pyx in the following manner:
```
export CFLAGS="-g -O0 -fno-omit-frame-pointer"
export CXXFLAGS="$CFLAGS"
export LDFLAGS="-g"
python3-debug setup.py build_ext --inplace
```

It said I should then see my symbols when I run:
```
readelf -s ./*.so | head
nm -D ./*.so | head
```
but I did not see any function names in here, just the main "module" name, so maybe the above doesn't do much/anything.

It then suggested that I start cygdb as I had before and run the following:
```
(gdb) start
(gdb) break malloc_printerr
(gdb) break __libc_message
(gdb) break abort
(gdb) run
```
While the `__libc_message` breakpoint didn't work, the other two did, and by doing this, I was able to then run `(gdb) backtrace` and get my backtrace; it was long, so I had to scroll through with Shift+PageUp. However, other commands, like printing "locals", did not seem to work; maybe this was because the crash, while _caused_ by my Cython code, was not _during_ my Cython code but was instead in the scikit-learn code, so maybe it didn't have debug symbols.

I also was still getting a `warning: .cygdbinit: No such file or directory (gdb)` while trying the above, though I'm not sure how much that impacted functionality.

While I'm not sure that using `python3-debug` instead of `python` did much/anything above, I wanted to use it to be safe, and I also ended up working in a VM and doing the necessary installs, pip installs, etc. in the "main" python environment rather than a virtual environment. While I haven't tested whether it would work with a virutal environment, one thing I know is that, with a virutal environment made with `venv`, no `activate_this.py` file is created, and cygdb expects this file to exist. So you would have to use `virtualenv` or something instead.

## Copies and `__cinit__`
For custom implementations of `__copy__` and `__deepcopy__` in Python, it may be desirable to avoid the class constructor when creating a new object and instead just create it via `__new__` so that you can handle the logic yourself. E.g., if your constructor makes a copy of a list you pass in as a paramter, meaning the object copy made via constructor wouldn't be a shallow copy anymore. However, if you define `__cinit__`, then there is no way to create an object where `__cinit__` does not get called, which can frustrate this attempt.

# Copies and Deepcopies

Suppose that you want to override `__copy__` or `__deepcopy__` in a way where you first get a "default" copy and then modify it.
Is there a good way to do that?

The answers to [the StackOverflow question "How to override the copy/deepcopy operations for a Python object?"](https://stackoverflow.com/questions/1500718/how-to-override-the-copy-deepcopy-operations-for-a-python-object) – asked by user Brent Writes Code (159658/brent-writes-code) on 2009-09-30 – contain some ideas and some hacks, albeit the hacks can have some drawbacks. For example the suggestion by user Peter (851699/peter) written 2014-07-07 suggests to temporarily set the overridden function to None, get a copy that uses the default copying, and then restore the function to something other than None after. Except… then the new copy will just have None as its function and won't have the overriden behaviour if someone tries to make a copy of _it_! Well, then modify the copy to add it back in? Except that means it'll just get the original object's copying function….


## Comparisons and Diffs
Might want to check out the DeepDiff Python library: https://github.com/seperman/deepdiff

# Library-Specific Notes
## IPython/Jupyter
For importing self-written code, might need to run `os.chdir("dirContainingFiles")` first before running files that have `import myOtherFile` in them. 

### Matplotlib + IPython/Jupyter
For getting interactive plots to display (particularly in VS Code), use:

* For interactive + inline, `%matplotlib ipympl` in IPython console.
* For static + inline, `%matplotlib inline` in IPython console.
* For an interactive plot in a separate window, use:
    * `pip install tk` for env (Could also replace "tk" with Qt in this step and below, but I had issues with getting Qt working, and it's big).
    * `%matplotlib` (or maybe `%matplotlib tk`) in IPython console
    * `plt.show(block=True)` 

## Manim
Link to look through at some point: https://slama.dev/manim/groups-transformations-updaters/

### Fading In/Out All a Whole Scene
https://github.com/3b1b/manim/issues/992
  - Basically, you can get a list of a scene's objects via `self.mobjects`.
  - Above seems useful but I didn't use it yet.
  - Original source might be: https://github.com/Elteoremadebeethoven/AnimationsWithManim/blob/master/English/extra/faqs/faqs.md#remove-all-objects-in-screen
