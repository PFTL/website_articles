Importing is not only a matter of using external libraries, it also allows you to keep your code clean and organized. In
this tutorial, we are going to discuss from the very basics of importing to complex topics such as lazy loading of
modules in packages.

# Introduction to importing

In Python it is important to distinguish between modules and packages in order to have a clear communication. Modules
are, in essence, files with a *.py* extension. They can define variables, functions, classes, and they can also run
code. Packages are collections of modules in a hierarchical structure, which in the end means organizing the module
files in folders.

If we have a file called **module.py**, we can simply use the following syntax to import it:

```python
import module
```

We can also use modules that are bundled with Python, such as ``sys``:

```python
import sys
```

In this case, ``sys`` is a module that belongs to the `Python Standard Library`. It provides functions to interact with
the interpreter itself. For example, we can use it to find out if any arguments where passed while executing a script.
We can create a file, **test\_argv.py**, with the following code:

```python
import sys

print(sys.argv)
```

And we run it to see its output:

```bash
python test_argv.py -b 1
['test_argv.py', '-b', '1']
```

Python comes bundled with plenty of libraries for different tasks. You can find them
all [here](https://docs.python.org/3/library/index.html).

Importing the entire ``sys`` module may not be what we want since we are only using one of its elements. In this case, we can also be selective with the importing procedure while keeping the same output:

```python
from sys import argv

print(argv)
```

Using the full import or just a selection of what we need depends to a great extent on personal preferences, and on how different packages where designed.

## Importing \*

In the examples above, we have either imported one entire module or just one element from ``sys``. To import more elements from the same module, we can specify them on the same line:

```python
from sys import argv, exit

print(argv)
exit()
```

We can import as many things as we need from the same module. To avoid lines becoming too long and hard to read, it is possible to stack the items vertically. For example:

```python
from sys import (api_version,
                 argv,
                 base_exec_prefix,
                 exit,
                 )
```

Note that in this case we must use a set of `(` and `)` to make a clear list of imports. It is also possible to import all the elements from a module by using a ``*`` in the statement, for example: 

```python
from sys import *

print(api_version)
print(argv)
exit()
```

However, this is a **highly discouraged practice**. When we import things without control, some functions may get overwritten without even realizing. The code becomes harder to read and understand. Let's see it with the following example:

```python
from time import *
from asyncio import *

print('Here')
sleep(1)
print('After')
```

Most programmers will be familiar with the ``sleep`` function from the ``time`` module, which halts the execution of a program for a given number of seconds. If we run the script, however, we will notice that there is no delay between the lines `'Here'` and `'After'`. This can be puzzling at first, and for larger projects can indeed become daunting. In this case, both `time` and `asyncio` define a function `sleep` which behaves in very different ways. If in such a simple and compact example the risks of the ``*`` imports become evident, it is easy to understand why almost all developers avoid using the `*` in their programs.

The case of `time` and `asyncio` is special, because both of them belong to the Python Standard Library and are very well documented. However, not all programs are as organized and clean. Therefore, it becomes harder and harder to remember all the modules and functions defined in packages. Moreover, some names are so handy (like ``sleep``), that it is easy to find them defined in different packages and for many purposes.

The script above becomes much clearer with the following syntax:

```python
import time
import asyncio

print('Here')
asyncio.sleep(1)
print('After')
time.sleep(1)
print('Finally')
```

There are no doubts of what is going on, and where problems may be arising, even if we haven't used the *asyncio* library before. 

## Importing As

Sometimes we face the situation in which we want to import specific functions from different modules but their names are the same. For example, both ``time`` and ``asyncio`` define
``sleep`` and we are interested in using them. To avoid a name clash when importing, Python allows us to change the name of what we are importing by doing the following:

```python
from asyncio import sleep as async_sleep
from time import sleep as time_sleep

print('Here')
async_sleep(1)
print('After')
time_sleep(1)
print('Finally')
```

In this way, we can use either ``sleep`` from ``asyncio`` or from ``time``avoiding name clashes. Of course this flexibility must be taken seriously because it can also generate unreadable code. The following code would become very hard to udnerstand for anybody reading it:

```python
from time import sleep as exit

exit(1)
```

The use of the `import as` is not only practical, in some cases it is the de-facto way of working. For example, ``numpy``, ``pandas``, ``matplotlib`` are almost always imported in the same way: 

```python
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
```

The three lines above are ubiquitous in many scientific programs. They are so common that code editors such as Pycharm can suggest you to import numpy if they see a line that includes something like ``np.``. Changing the name of the import can be useful not only to prevent name clashes, but also to shorten the notation. Instead of doing:

```python
matplotlib.pyplot.plot(x, y)
```

We simply have:

```python
plt.plot(x, y)
```

Different packages have different shortcuts. For example `PyQtGraph` is normally shortened as ``pg``, and for sure different fields use different abbreviations. Importing Numpy as ``np`` or Pandas as ``pd`` is not mandatory. However, since it is what the community does, it will make the code much more readable.

> The use of this notation is so widespread that, for example, even in StackOverflow numpy is used as ``np`` without even showing the import statement. 

## Importing your code

So far, we have seen how to import packages and modules developed by other people. Importing, however, is a great tool to structure different parts of the code. It makes it easier to maintan and collaborate. Therefore, sooner or later we are going to find ourselves importing our code. We can start simple and slowly build the complexity. 

We can create a file **first.py**, with following code:

```python
def first_function():
    print('This is the first function')
```

In a file called **second.py**, we can add the following code:

```python
from first import first_function

first_function()
```

And we run it:

```bash
$ python second.py
This is the first function
```

That is as easy as it gets. We define a function in a file, but we use it in another file. Once we have many files, it becomes handier to start creating some structure to organize the program. We can create a folder called **package\_a**, and we add a new file, called **third.py**. The folder structure is like this:

```bash
$ tree
.
├── first.py
├── package_a
│   └── third.py
└── second.py
```

In **third** we create a new function, appropriately called ``third_function``:

```python
def third_function():
    print('This is the third function')
```

The examples are very basic, but they already start to show some patterns and caveats in the importing procedures. If we want to use the new function from the **second.py**, we need to import it:

```python
from first import first_function
from package_a.third import third_function

first_function()
third_function()
```

If we run the code, we'll get the following output:

```bash
This is the first function
This is the third function
```

Pay attention to the notation we used to import the `third_function`. We specified the folder, in this case, `package_a` and then we referred to the file with a dot: `.`. This is the way in which the hyerarchy of folders and files is transformed into packages and modules in Python. In this case, replacing the folder separators by a ``.`` we end up having ``package_a.third``, and we stripped the `.py` extension. 

## The use of the \_\_init\_\_ file

Most likely our code will not be isolated from other projects. When we install packages, they will have dependencies, and very quickly we lose track of what is actually installed. We can understand where the problem arises wtih a very simple example. We can assume that we have **numpy** already installed but we are not aware of that. If we create a new folder, called **numpy**, with a file
called **sleep.py**, the folder structure will end up looking like this:

```bash
.
├── first.py
├── package_a
│   └── third.py
├── numpy
│   └── sleep.py
└── second.py
```

And in the file **sleep.py**, we can add the following function:

```python
def sleep():
    print('Sleep')
```

In the same way we did before, we can update **second.py** to use the new function ``sleep``:

```python
from numpy.sleep import sleep

sleep()
```

Of course the code above will raise many alarms, but the best is to run it to see what happens: 

```bash
Traceback (most recent call last):
  File "second.py", line 3, in <module>
    from numpy.sleep import sleep
ModuleNotFoundError: No module named 'numpy.sleep'
```

The biggest challenge in this case is that the exception is utterly hard to understand. It is telling us that Python tried to look for a module called `sleep` in the *numpy* package. If we open the folder *numpy* we find the module *sleep*. Therefore, there must be something else going on. In this example it is clear that Python is looking for the module *sleep* within the official *numpy* package and not in our folder. 

The quick solution to this problem is to create an empty file called **\_\_init\_\_.py** in our numpy folder:

```bash
.
├── first.py
├── package_a
│   └── third.py
├── numpy
│   ├── __init__.py
│   └── sleep.py
└── second.py
```

We can run the code without problems this time:

```bash
$ python second.py
Sleep
```

It is important to understand what is going on and not just through quick solutions to see whether they work to voercome the immediate problems. The quid is to know how Python looks for packages on the computer. The topic is complex, and Python allows a great deal of customization. The [official documentation](https://docs.python.org/3/reference/import.html) shines some light into the matter once we have experience. 

In short, Python will first look at whether we are trying to import, and check if it belongs to the standard library. If the folder was `time` instead of `numpy`, the behavior would have been different. Adding the **\_\_init\_\_.py** file wouldn't make a difference. Once Python knows the module is not in the standard library, it will check for external modules. First, it starts searching the current directory. Then, it moves to the directories where packages are installed, for example, where numpy ends up after doing `pip install numpy`. 

This raises a very interesting question: why did our code fail in the first attempt and it started working only after adding the **\_\_init\_\_.py** file. In order for Python to consider that a folder is a package, it must contain an **\_\_init\_\_.py** file. This is by design, exactly to prevent unintended name clashes unless we explicitly want them. 

If Python does not find the package within the local or default installation directories, it moves to look into the folders. That is why ``package_a`` works even if we never defined the **\_\_init\_\_.py** file. 

Bear in mind that once Python finds the package, it won't keep searching. Once it finds *numpy* in our local folder, it won't look for another numpy elsewhere. Therefore, we can't mix modules from different packages with the same name. 

### The PATH list of directories
 
A useful thing to do is to check the directories Python uses to look for modules and packages. We can see it by running the following code:

```python
import sys

for path in sys.path:
    print(path)
```

That will list something between 4 and 6 different folders. Most of them are quite logical: where Python is installed, the virtual environment folders, etc.

### Adding Folder to the Path

An easy way of extending the capabilities of Python is to add folders to the list where it looks for packages. The first option is to do it at runtime. We can easily append a directory to the
variable `sys.path`. To add the current directory to the list, we can do the following:

```python
import os
import sys

CURR_DIR = os.path.dirname(os.path.abspath(__file__))
print(CURR_DIR)
sys.path.append(CURR_DIR)
for path in sys.path:
    print(path)
```

We can add any directory, not only the current one. In this approach, we modify the system path only while the program runs. If we run two different programs, each will have its own path.

Another option is to modify a variable in the operating system itself. This has the advantage that it can be made permament and all programs will share the same information. For our application, we have to modify the **PYTHONPATH** environment variable. Environment variables are available on every
operating system, how to set and modify them varies. 

On **Linux** or **Mac**, the command to set these variables is ``export``. We can do the following:

```bash
export PYTHONPATH=$PYTHONPATH':/home/user/'
echo $PYTHONPATH
```

The first line appends the folder ``/home/user`` to the variable ``PYTHONPATH``. Note that we have used ``:`` as a directory separator.

On **Windows**, we need to right-click on "Computer", select "Properties". In the "Advanced System Settings" there is the option "Environment variables". If `PYTHONPATH` exists, we can modify it, if it does not exist, we can create it by clicking on "New". Bear in mind that on Windows, you have to use `;` to separate directories, since ``:`` is part of the folder path (e.g.: ``C:\Users\Test\...``).

Once you modified your Python Path, you can run the following code:

```python
import sys

for path in sys.path:
    print(path)
```

> You will see that `/home/user` appears at the top of the list of
> directories. You can add another directory, for example:

```bash
export PYTHONPATH=$PYTHONPATH':/home/user/test'
```

And you will see it also appearing. Adding information to the Python Path is a great way of developing a structure on
your own computer, with code in different folders, etc. It can also become hard to maintain. As a quick note, Python
allows you to read environment variables at runtime:

```python
import os

print(os.environ.get('PYTHONPATH'))
```

Note that on Windows, the changes to environment variables are permanent, but on Linux and Mac you need to
follow [extra steps](https://stackoverflow.com/questions/3402168/permanently-add-a-directory-to-pythonpath)
if you want them to be kept.

PYTHONPATH and Virtual Environment
----------------------------------

There is a very handy trick when you work with virtual environments which is to modify environment variables when you
activate or deactivate them. This works seamlessly on Linux and Mac, but Windows users may require some tinkering to
adapt the examples below.

If you inspect the **activate** script (located in the folder
*venv/bin*) you can get inspiration about what is done with the `PATH`
variable, for example. The first step is to store the old variable, before modifying it, then we append whatever we
want. When we deactivate the virtual environment, we set the old variable back.

Virtual Environment has three hooks to achieve exactly this. Next to the
**activate** script, you will see three more files, called
*postactivate*, *postdeactivate* and *predeactivate*. Let's modify
*postactivate*, which should be empty if you never used it before. Add the following:

```bash
PYTHONPATH_OLD="$PYTHONPATH"
PYTHONPATH=$PYTHONPATH":/home/user"
export PYTHONPATH
export PYTHONPATH_OLD
```

Next time you activate your virtual environment, you will have the directory `/home/user` added to the PYTHONPATH. It is
a good practice to go back to the original version of the python path once you deactivate your environment. You can do
it editing the **predeactivate** file:

```bash
PYTHONPATH="$PYTHONPATH_OLD"
unset $PYTHONPATH_OLD
```

With this, we set the variable to the status it had before activating and we remove the extra variable we created. Note
that in case you don't deactivate the environment, but simply close the terminal, the changes to the `PYTHONPATH` won't
be saved. The *predeactivate* script is important if you switch from one environment to another and keep using the same
terminal.

PYTHONPATH and PyCharm
----------------------

If you are a user of [PyCharm](https://www.jetbrains.com/pycharm/), and probably most other IDE's around will be
similar, you can change your environment variables directly from within the program. If you open the
**Run** menu, and select **Edit Configurations** you will be presented with the following menu:

<:image:PyCharm_config.png>

In between the options, you can see, for example, "Add content roots to PYTHONPATH". This is what makes the imports work
out of the box when you are in Pycharm but if you run the same code directly from the terminal may give you some issues.
You can also edit the environment variables if you click on the small icon to the right of where it says "environment
variables".

Keeping an eye on the environment variables can avoid problems in the long run. Especially if, for example, two
developers share the computer, which is very often the case in laboratories, where on PC controls the experiment, and
the software can be edited by multiple users. Perhaps one sets environment variables pointing to specific paths which
are not what the second person is expecting.

Absolute Imports
================

In the examples of the previous sections, we imported a function
*downstream* in the file system. This means, that the function was inside of a folder next to the main script file. What
happens if we want to import from a sibling module? Imagine we have the following situation:

```bash
├── __init__.py
├── mod_a
│   ├── file_a.py
│   └── __init__.py
├── mod_b
│   ├── file_b.py
│   └── __init__.py
└── start.py
```

We have a **start** file at the top-level directory, we have two modules, **mod\_a** and **mod\_b**, each with its
own **\_\_init\_\_**
file. Now, imagine that the function you are developing inside of
**file\_b** needs something defined in **file\_a**. Following what we saw earlier, it is easy to import from **start**,
we would do just:

```python
from mod_a import file_a
from mod_b import file_b
```

To have a concrete example, let's create some dummy code. First, in the file **file\_a**, let's develop a simple
function:

```python
def simple():
    print('This is simple A')
```

Which, from the **start** file we can use as follows:

```python
from mod_a.fila_a import simple

simple()
```

If we want to use the same function within the **file\_b**, the first thing we can try is to simply copy the same line.
Thus, open **file\_b**
and add the following:

```python
from mod_a.file_a import simple


def bsimple():
    print('This is simple B')
    simple()
```

> And we can edit **start** to look as follows:

```python
from mod_b import file_b

file_b.bsimple()
```

If we run start, we will get the output we were expecting:

```bash
$ python start
This is simple B
This is simple
```

However, and this is very big, HOWEVER, sometimes we don't want to run
**start**, we want to run directly **file\_b**. If we run it as it is, we are expecting no output, but we can try it
anyway:

```bash
$ python file_b.py
Traceback (most recent call last):
  File "file_b.py", line 1, in <module>
    from mod_a.file_a import simple
ModuleNotFoundError: No module named 'mod_a'
```

And here you start to realize the headaches that the importing in Python can generate as soon as your program gets a bit
more sophisticated. What we are seeing is that depending on where in the file system we run Python, it will understand
what `mod_a` is. If you go back to the previous sections and see what we discussed about the Path used for searching
modules, you will see that the first path is the current directory. When we run **start**, we are triggering Python from
the root of our project and therefore it will find **mod\_a**. If we enter to a sub-directory, then it will no longer
find it.

The same happens if we trigger python from any other folder:

```bash
$ python /path/to/project/start.py
```

Based on what we have discussed earlier, can you think of a solution to prevent the errors?

What we are doing in the examples above is called **absolute imports**. This means that we specify the full path to the
module we want to import. What you have to remember is that the folder from which you trigger Python is the first place
where the program looks for modules. Then it goes to the paths stored in `sys.path`. So, if we want the code above to
work, we need to be sure that Python knows where **mod\_a** and
**mod\_b** are stored.

The proper way would be to include the folder in the **PYTHONPATH**
environment variable, as we explained earlier. A *dirtier* way would be to append the folder at runtime, we can add the
following lines to
**file\_by.py**:

```python
import os
import sys

BASE_PATH = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
sys.path.append(BASE_PATH)

from mod_a.file_a import simple
```

This is very similar to what we have done earlier. The important line is the highlighted, it shows you a way of getting
the full path to the folder one level above where the current file (**file\_b.py**) is. Note that you need to append to
the `sys.path` before you try to import
`mod_a`, or it will fail such as before.

If you think about this approach, you can quickly notice that it has several drawbacks. The most obvious one is that you
should add those lines to every single file you are working with. Imagine that later you develop **mod\_c** which
depends also on **mod\_a**, you will need to append the folder to the path again, etc. This quickly becomes a nightmare.

Another problem with our current approach is that we are specifying the name of the module, but not the package to which
it belongs. This connects back to what we did at the beginning of the article. Modules that belong to packages sometimes
have the same names even if they are very different. Imagine you would like to develop a module called
`string`. Perhaps you are a theoretical physicist working on string theory. If you have code that looks like this:

```python
from string import m_theory
```

It will give you problems because `string` belongs to Python's standard library. It can also happen quite often that you
develop two different packages and both have some modules with the same name. In the end, it is hard to come up with
unique names, and things like *config*, *lib*,
*util*, etc. are quite descriptive.

A better approach is to develop projects always in their own folder. The structure would be like this:

```bash
code
├── pckg_a
│   └── file_a.py
├── pckg_b
│   ├── docs
│   │   ├── conf_complete.py
│   │   └── output
│   │       └── output.txt
│   └── mod_a
│       ├── factorial.py
│       ├── __init__.py
│       └── people.py
└── pckg_c
    ├── another.py
    ├── __init__.py
    ├── mod_a
    │   ├── file_a.py
    │   └── __init__.py
    ├── mod_b
    │   ├── file_b.py
    │   └── __init__.py
    └── start.py
```

In the folder tree above, you can see a base folder called **code**. Inside there are different packages, *a*, *b*,
and *c*. If you check
**pckg\_c** you will notice that it contains the code we were discussing in this tutorial. There are several advantages
to working in this way. First, you can add just the folder **code** to the PYTHONPATH and you will have all your
packages immediately available. The other advantage is that now you can import modules without risking mistakes:

```python
from pckg_b import mod_a as one_module
from pckg_c import mod_a as two_module
```

Now you see that it is very clear what module you are importing, even though they are both called `mod_a`. Remember,
absolute imports mean that you define the full path of what you want to import. However, in Python, full is *relative*.
You are not specifying a path in the file system, but rather an import path. Therefore, it is impossible to think about
absolute imports without also considering the PYTHONPATH.

Relative Imports
================

Another option for importing modules is to define the relative path. Let's continue building on the example from the
previous section. Imagine you have a folder structure like this:

```bash
code
├── mod_a
│   ├── file_a.py
│   └── __init__.py
├── mod_b
│   ├── file_b.py
│   ├── __init__.py
│   └── mod_a
│       ├── file_c.py
│       └── __init__.py
└── start.py
```

Note that in this example, we are placing the files within a folder called **code**, which will be relevant later on.
Each `file_X.py`
defines a function called `function_X` (where X is the letter of the file). The function simply prints the name of the
function. It is a very simple example to show how this structure works. By not it should be clear that if you would like
to import `function_c` from `file_c` in the
`start.py` file, you would simply do the following:

```python
from mod_b.mod_a.file_c import function_c
```

The situation becomes more interesting when you want to import
`function_a` into `file_b`. It is important to pay attention because there are two different `mod_a` defined. If we add
the following to
`file_b`:

```python
from mod_a.file_c import function_c
```

It would work, regardless of how you run the script:

```bash
$ python mod_b/file_b.py
$ cd mod_b
$ python file_b.py
```

But this is not what we wanted! We want `function_a` from `file_a`. If we, however, add the following to `file_b`:

```python
from mod_a.file_a import function_a
```

We would get the following error:

```bash
$ python mod_b/file_b.py
Traceback (most recent call last):
  File "mod_b/file_b.py", line 1, in <module>
    from mod_a.file_a import function_a
ImportError: No module named file_a
```

So, now is where relative imports come into play. From `file_b`, the module we want to import is one folder up. In
principle, it would be enough to write the following:

```python
from ..mod_a.file_a import function_a


def function_b():
    print('This is simple B')
    function_a()


function_b()
```

Most tutorials end at this point. They explain that the first `.` means this directory, while the second means going one
level up, etc. However, if you run the file, there will be problems. If you are still using Python 2 (STRONGLY
discouraged!), you would get the following error:

```bash
$ cd mod_b/
$ python file_b.py
Traceback (most recent call last):
  File "file_b.py", line 1, in <module>
    from ..mod_a.file_a import function_a
ValueError: Attempted relative import in non-package
```

This error means that you can't simply run the file as if it would be a script. You have to tell Python that it is a
package. The way of running a script as if it would be a package is to add a `-m`, you need to be one folder up to work:

```bash
$ python -m mod_b.file_b
```

**Python 3** users can simply run the file as always, and you will get the following error:

```bash
$ python3 file_b.py
Traceback (most recent call last):
  File "file_b.py", line 1, in <module>
    from ..mod_a.file_a import function_a
ValueError: attempted relative import beyond top-level package
```

It doesn't matter if you change folders, if you move one level up, you will get the same problem:

```bash
$ python3 mod_b/file_b.py
Traceback (most recent call last):
  File "mod_b/file_b.py", line 1, in <module>
    from ..mod_a.file_a import function_a
ValueError: attempted relative import beyond top-level package
```

At some point, this becomes nerve-wracking. It doesn't matter if you add folders to the PATH, create **\_\_init\_\_.py**
files, etc. It all boils down to the fact that you are not treating your files as a package.
**Python 2** was showing a different error message that could point into the direction of solving the problem, but
for **Python 3** it became slightly more cryptical. To instruct Python to run your file as part of a package, you would
need to do the following:

```bash
$ python3 -m code.mod_b.file_b
This is function_b
This is function_a
```

Bear in mind that the only way of running the code like this is if python knows where to find the folder `code`. And
this brings us back to the discussion of the PYTHONPATH variables. If you are in the folder that contains `code` and run
Python from there, you won't see any problems. If you, however, are in any other folder on your computer, Python will
follow to usual rules to try to understand where `code` is.

There is one more detail with relative imports. Imagine that **file\_c**
has the following:

```python
from ..file_b import function_b


def function_c():
    print('This is function c')
    function_b()


function_c()
```

Since **file\_c** is deeper, we can try to run it in different ways:

```bash
$ python -m code.mod_b.mod_a.file_c
$ python -m mod_b.mod_a.file_c
```

However, the second option is going to fail. **file\_c** is importing
**file\_b** which in turn is importing **file\_a**. Therefore, Python needs to be able to go all the way to the root
of `code`. This is, however, not always the case. It depends on how the code was developed. Bear in mind that imports
work equally well if you import `code` into another project, provided that Python knows where to find it:

```pycon
>>> from code.mod_b.mod_a.file_c import function_c
```

The last detail is that you can't mix relative imports and what we have done at the beginning of this section. If you
add the following to
**file\_b**:

```python
from ..mod_a.file_a import function_a
from mod_a.file_c import function_c


def function_b():
    print('This is function_b')
    function_a()


function_b()
```

You will get the following error:

```bash
$ python -m code.mod_b.file_b
Traceback (most recent call last):
[...]
ModuleNotFoundError: No module named 'mod_a'
```

When you decide to run your code as a module (using the `-m`), then you have to make all the imports explicit. One way
of solving the problem with our code above would be to change one line:

```python
from .mod_a.file_c import function_c
```

Then it is clear that we want to import from `mod_a` which is in the same folder than the **file\_b**.

Mixing Absolute and Relative
----------------------------

This is possible. There is no secret to it, for example, we can change
**file\_b.py** to look like this:

```python
from ..mod_a.file_a import function_a
from code.mod_b.mod_a.file_c import function_c


def function_b():
    print('This is function_b')
    function_c()


function_b()
```

And if we run the file, there won't be any issues:

```bash
$ python -m code.mod_b.file_b
```

So, now you see that you can easily mix relative and absolute imports.

Absolute or Relative: Conclusions
=================================

Deciding whether you want to use absolute imports or relative imports is up to your own taste. If you are developing a
package that has a lot of modules nested to each other, using relative imports can make your code clearer. For example,
this is how the same import would look like in the two different cases:

```python
from package.module_1.module_2.module_3.file import my_function
from .file import my_function
```

If you would be developing **file\_2** next to **file**, importing
`my_function` can become very different depending on whether you go the absolute or the relative path.

However, it is important to consider that typing less is not the only factor at play here. Remember, that once you use
relative imports, you can't simply run your files using `python file_2.py`, you will need to use the `python -m`
command, and it will mean that you will need to write the full path to the file within the package.

Sometimes bigger programs can be executed in parts, to test the behavior and give a quick feeling of what the code does
without running a full-fledged solution. Therefore, it is really up to the developer to have sensitivity and decide
whether the absolute import or the relative import makes the code clearer and the execution easier.

The example code for this article can be
found [on Github](https://github.com/PFTL/website/tree/master/example_code/37_imports)