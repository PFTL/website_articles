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

We can check whether the modifications to the system environment variables worked by running the same code:

```python
import sys

for path in sys.path:
    print(path)
```

Adding information to the Python Path is a great way of developing a structure on your own computer, with code in different folders, etc. However, it is also important to note that it also makes harder to maintain. The environmental variables in one computer are not the same in another, Python may be loading legacy code from an abscure place on the computer. On the other hand, environmental variables are very useful in contexts like a web server, where the definitions can be loaded before running a program. 

As a quick side-note, it is worth mentioning that Python allows to read environment variables at runtime:

```python
import os

print(os.environ.get('PYTHONPATH'))
```

Note that on Windows, the changes to environment variables are permanent, but on Linux and Mac we need to follow [extra steps](https://stackoverflow.com/questions/3402168/permanently-add-a-directory-to-pythonpath) if we want them to stay.

### PYTHONPATH and Virtual Environments

When we work with virtual environments, we can modify environment variables when we activate or deactivate them. This works seamlessly on Linux and Mac, but Windows users may require some tinkering to adapt the examples below.

If we inspect the **activate** script (located in the folder *venv/bin*), we can get inspiration about what is done with the ``PATH``variable, for example. The first step is to store the old variable, before modifying it, then we append whatever we want. When we deactivate the virtual environment, we set the old variable back.

Virtual environments have three hooks to achieve this behavior. Next to the **activate** script, we can also see three files called *postactivate*, *postdeactivate* and *predeactivate*. We can modify
*postactivate*, which should be empty, and add the following:

```bash
PYTHONPATH_OLD="$PYTHONPATH"
PYTHONPATH=$PYTHONPATH":/home/user"
export PYTHONPATH
export PYTHONPATH_OLD
```

Next time we activate the virtual environment, we will have the directory `/home/user` added to the ``PYTHONPATH``. It is a good practice to go back to the original version of the python path once we deactivate tne environment. We can do it directly in the **predeactivate** file:

```bash
PYTHONPATH="$PYTHONPATH_OLD"
unset $PYTHONPATH_OLD
```

We set the variable to the status it had before activating, and we remove the extra variable we created. Note that in case we don't deactivate the environment, but simply close the terminal, the changes to the `PYTHONPATH` won't be saved. The *predeactivate* script is important if you switch from one environment to another and keep using the same terminal.

### PYTHONPATH and PyCharm

Users of [PyCharm](https://www.jetbrains.com/pycharm/), and probably most other IDE's around will be
similar, can change the environment variables directly from within the program. If we open the
**Run** menu, and select **Edit Configurations** we will be presented with the following menu:

<:image:PyCharm_config.png>

In between the options we can see "Add content roots to PYTHONPATH". This is what makes the imports work out of the box when we are in Pycharm but if we run the same code directly from the terminal may give you some issues. We can also edit the environment variables if we click on the small icon to the right of where it says "environment variables".

Keeping an eye on the environment variables can avoid problems in the long run. Especially if, for example, two developers share the computer. Although strange in many settings, lab computers are normally shared between people, and the software can be edited by multiple users. Perhaps one sets environment variables pointing to specific paths which are not what the second person is expecting.

## Absolute Imports

In the examples of the previous sections, we imported a function *downstream* in the file system. This means, that the function was inside a folder next to the main script file. However, we should also study what happens if we want to import from a sibling package. Imagine we have the following situation:

```bash
├── __init__.py
├── pkg_a
│   ├──  mod_a.py
│   └── __init__.py
├── pkg_b
│   ├── mod_b.py
│   └── __init__.py
└── start.py
```

We have a **start** file in the top-level directory and two packages, **pkg\_a** and **pkg\_b**. Each one has its own **\_\_init\_\_** file. The question is how can we have access to the contents of **mod\_a** from withing **mod\_b**. From the **start** file, the import procedure is easy:

```python
from pkg_a import mod_a
from pkg_b import mod_b
```

We can create some dummy code in order to have a concrete example. First, in the file **mod\_a**, we can create a function:

```python
def simple():
    print('This is simple A')
```

Which, from the **start** file we can use as follows:

```python
from pkg_a.mod_a import simple

simple()
```

If we want to use the same function within the **mod\_b**, the first thing we can try is to simply copy the same line. Thus, in **mod\_b** we can try:

```python
from pkg_a.mod_a import simple


def bsimple():
    print('This is simple B')
    simple()
```

To make it complete, we can trigger it directly from withing the **start** file:

```python
from pkg_b import mod_b

mod_b.bsimple()
```

If we run it, we will get the output we were expecting:

```bash
$ python start.py
This is simple B
This is simple
```

However, and this is very big, **HOWEVER**, sometimes we don't want to run**start**. Instead, we want to run directly **mod\_b**. If we try to run it, the following happens:

```bash
$ python mod_b.py
Traceback (most recent call last):
  File "mod_b.py", line 1, in <module>
    from pkg_a.mod_a import simple
ModuleNotFoundError: No module named 'pkg_a'
```

And here we start to realize the headaches that the importing in Python can generate as soon as the program gets a bit more sophisticated. In the end, the error was expected. When we run ``python mod\_b.py``, Python will try to find ``pkg\_a`` in the same folder, and not one level up. When we trigger ``start`` there is no problem, because from that directory both ``pkg\_a`` and ``pkg\_b`` are visible. 

The same problem will appear if we trigger python from any other location in the computer:

```bash
$ python /path/to/project/start.py
```

What we did in the examples above is called **absolute imports**. It means that we specify the full path to the module we want to import. What we have to remember is that the folder from which you trigger Python is the first place where the program looks for modules. Then it goes to the paths stored in `sys.path`. If we want the code to work, we need to be sure that Python knows where **pkg\_a** and **pkg\_b** are stored.

The proper way would be to include the folder in the **PYTHONPATH** environment variable, as we explained earlier. A *dirtier* way would be to append the folder at runtime, we can add the following lines to **mod\_b.py**:

```python
import os
import sys

BASE_PATH = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
sys.path.append(BASE_PATH)

from pkg_a.mod_a import simple
```

This is very similar to what we have done earlier. It is important to highlight that the definition of ``BASE_PATH`` is the full path to the folder one level above where the current file (**mod\_b.py**) is. Note also that we need to append to the `sys.path` before we try to import ``pkg_a``, or it will fail in the same way it did before.

If we think for a second about this approach, we can quickly notice that it has several drawbacks. The most obvious one is that we should add those lines to every single file you are working with. Another problem is that we are adding a folder that contains many packages, which can give some collisions. Imagine we are a theoretical physicists working on string theory and we develop a module called ``string``. The code would look like this:

```python
from string import m_theory
```

And it will give us problems because `string` belongs to Python's standard library. 

Therefore, it is always better to develop projects in their own folder, even if that forces a bit of a name repetition. In the very simple case we are dealing with here, the structure would be like this:

```bash
code
├── pkg_a
│   ├── docs
│   └── pkg_a
│       ├── __init__.py
│       └── mod_a.py
└── pkg_b
    ├── docs
    └── pkg_b
        ├── __init__.py
        └── mod_b.py
```

In the folder tree above we have a base folder **code**. Inside there are two packages, *a* and *b*. Although the name of the folders repeat (we have twice **pkg_a**, twice **pkg_b**, for example), there are several advantages to working in this way. The most important one is the granularity. We can add ``code/pkg_a`` or ``code/pkg_b`` to the ``PYTHONPATH``. Having control is always better than getting blanket results. 

The most important thing to remember is that in Python, absolute is *relative*. While importing, we are not specifying a path in the file system, but rather an import path. Therefore, the imports are always *relative* to the PYTHONPATH, even if called *absolute*.

## Relative Imports

Another option for importing modules is to define the *relative* path. We can continue building on the example from the previous section. Imagine We have a folder structure like this:

```bash
code
├── pkg_a
│   ├── mod_a.py
│   └── __init__.py
├── pkg_b
│   ├──  mod_b.py
│   ├── __init__.py
│   └── pkg_a
│       ├──  mod_c.py
│       └── __init__.py
└── start.py
```

Let's assume that each `mod_X.py` defines a function called `function_X` (where X is the letter of the file). The function simply prints the name of the function. It should be clear that if we want to import `function_c` from `file_c`, the `start.py` file should look like:

```python
from pkg_b.pkg_a.mod_c import function_c
```

The situation becomes more interesting when we want to import ``function_a`` in ``mod_b``. It is important to pay attention because there are two different `pkg_a` defined in our program. If we add
the following to ``mod_b``:

```python
from pkg_a.mod_c import function_c
```

It would work, regardless of how we run the script:

```bash
$ python pkg_b/mod_b.py
$ cd pkg_b
$ python mod_b.py
```

But this is not what we wanted! We want `function_a` from `mod_a`. If we, however, add the following to `mod_b`:

```python
from pkg_a.mod_a import function_a
```

We would get the following error:

```bash
$ python pkg_b/mod_b.py
Traceback (most recent call last):
  File "pkg_b/mod_b.py", line 1, in <module>
    from pkg_a.mod_a import function_a
ImportError: No module named pkg_a
```

In this case is where relative imports become very handy. From **mod_b**, the module we want to import is one folder up. To indicate that, we can use the ``..`` notation in Python: 

```python
from ..pkg_a.mod_a import function_a


def function_b():
    print('This is simple B')
    function_a()


function_b()
```

Generally speaking, the first ``.`` means *in this directory*, while the second means going one
level up, etc. However, if we run the file, there will be problems. If we run the file, we get the following error:

```bash
$ python3 mod_b.py
Traceback (most recent call last):
  File "mod_b.py", line 1, in <module>
    from ..pkg_a.mod_a import function_a
ValueError: attempted relative import beyond top-level package
```

It doesn't matter if we change folders, if we move one level up, we will get the same problem:

```bash
$ python3 pkg_b/mod_b.py
Traceback (most recent call last):
  File "mod_b.py", line 1, in <module>
    from ..pkg_a.mod_a import function_a
ValueError: attempted relative import beyond top-level package
```

At some point, this becomes nerve-wracking. It doesn't matter if we add folders to the PATH, create **\_\_init\_\_.py** files, etc. It all boils down to the fact that we are not treating our files as a module when we run it. To instruct Python to run the file as part of a package, we would do:

```bash
$ python3 -m code.pkg_b.mod_b
This is function_b
This is function_a
```

Bear in mind that the only way of running the code like this is if python knows where to find the folder `code`. And this brings us back to the discussion of the PYTHONPATH variables. If we are in the folder that contains `code` and run Python from there, we won't see any problems. If we, however, are in any other folder, Python will follow to usual rules to try to understand where `code` is.

There is one more important detail to discuss with relative imports. We can imagine that **mod\_c** has the following code:

```python
from ..mod_b import function_b


def function_c():
    print('This is function c')
    function_b()


function_c()
```

Since **mod\_c** is deeper in the tree, we can try to run it in different ways:

```bash
$ python -m code.pkg_b.pkg_a.mod_c
$ python -m pkg_b.pkg_a.mod_c
```

However, the second option is going to fail. **mod\_c** is importing **mod\_b** which in turn is importing **mod\_a**. Therefore, Python needs to be able to go all the way to the root folder **code**. Therefore, when we plan our code, we should be mindful not only on how to write it, but on how the program is meant to be used. 

The last detail to cover is that we can't mix relative and absolute imports. For example, the following won't work:

```python
from ..pkg_a.mod_a import function_a
from pkg_a.mod_c import function_c


def function_b():
    print('This is function_b')
    function_a()


function_b()
```

We will get the following error:

```bash
$ python -m code.pkg_b.mod_b
Traceback (most recent call last):
[...]
ModuleNotFoundError: No module named 'pkg_a'
```

When we decide to run your code as a module (using the `-m`), then all the imports relative. One way
of solving the problem would be to change the following:

```python
from .pkg_a.mod_c import function_c
```

In this way it becomes clear that we want are importing from `pkg_a` which is in the same folder as **mod\_b**.

## Mixing Absolute and Relative
It is possible mixing relative and absolute imports without any secrets to it. We can change
**mod\_b.py** like this:

```python
from ..pkg_a.mod_a import function_a
from code.pkg_b.pkg_a.mod_c import function_c


def function_b():
    print('This is function_b')
    function_c()


function_b()
```

Mixing relative and absolute is definitely a possibility. The question, as almost always, is why would we do it. The fact that we can does not mean we should. 

## Absolute or Relative: Conclusions

Deciding whether we want to use absolute imports or relative imports is basically up to the taste of the developer or the rules established by the group. If we are developing a package that has a lot of sub-packages and modules with several layers of nesting, using absolute imports can make the code clearer. For example, this is how the same import would look like in the two different cases:

```python
from program.pkg_1.pkg_2.pkg_3.module import my_function
from .module import my_function
```

For some people the first line is much clearer, there are no doubts about what are we importing. But it can get tiresome to type the entire path all the time. However, it is important to consider that typing less is not the only factor at play here. 

If we are planning on allowing some files to run directly we should be mindful about the requirements for the relative imports. If we have many modules with similar names, sometimes the explicit path makes the code much clearer. It is really up to the developer to have enough sensitivity to decide
whether the absolute import or the relative import makes the code clearer and the execution easier.

The example code for this article can be
found [on Github](https://github.com/PFTL/website_example_code/tree/master/pftl_code/code/37_imports)