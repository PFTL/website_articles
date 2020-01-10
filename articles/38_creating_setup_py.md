# How to create a setup file for your project 

## Why having a setup
When you develop software, at some point you will want other people to be able to use what you have built. Sometimes it is handy if you yourself can quickly use the code you developed in the past on a new machine, or on a different virtual environment. We have already seen that <:Complete Guide to Imports in Python: Absolute, Relative, and More|for importing> to work properly, it is very important to have well defined packages, and that Python knows where to find them. 

With a proper setup file, Python will quickly understand where to find the package, and thus importing from other places becomes trivial. It will also be possible to quickly distribute the code with other people, diminishing the barrier of adoption. Having a **setup.py** file is the first step into being able to switch from scripts to a package on which to rely. 

## Your first setup file
The examples we are going to develop are toy examples, very similar to the ones on <:Complete Guide to Imports in Python: Absolute, Relative, and More|the article about importing>. But they will prove the point. You should start on an empty directory, the name doesn't really matter, but it is important not to mix it with other things, so can follow this article without complications. 

Let's start by creating a file called **first.py**, with the following code inside:

```python
def first_function():
    print('This is the first function')
```

This file is what we want to be able to use from other packages, etc. What we would like to be able to achieve is to run code like this:

```python
from first import first_function

first_function()
# 'This is the first function'
```

Next to the file, let's create a new file called **setup.py**, with the following code:

```python
from setuptools import setup

setup(
    name='My First Setup File',
    version='1.0',
    scripts=['first.py'],
)
```

!!! warning
    The code we are developing is harmless, nevertheless, it is always wise to create a virtual environment to play around, and eventually discard once you understood and polished the details
    
To install your script, you can simply do the following from the console:

```bash
python setup.py install
```

You will see some information printed to the screen. More importantly, you can now use your package from anywhere else on your computer. For example, change your directory to anywhere else on your computer, and to the following:

```python
from first import first_function
first_function()
# This is the first function
```

This is great, now you have a way of using your program from anywhere else, even from other packages. 

### Laying out the project
In the example above we had only one script available, and thus the structure of the program was straightforward. However, this is rarely the case. As soon as your program starts having multiple files, you will need to structure it in such a way that ``setup`` knows what files are needed. Let's evolve the example above. Let's create now a package with tho modules, the folder structure will look like this:

```txt
.
├── my_package
│   ├── __init__.py
│   ├── mod_a
│   │   ├── file_a.py
│   │   └── __init__.py
│   └── mod_b
│       ├── file_b.py
│       └── __init__.py
└── setup.py
```

Let's develop two very simple function inside **file_a** and **file_b**, respectively:

```python
def function_a():
    print('This is function_a')
```

```python
def function_b():
    print('This is function_b')
```

We need to update the **setup.py** file in order to accommodate that the program is more complex than what it was before. Fortunately, the developers at setuptools made our life very easy, we just need to write the following:

```python
from setuptools import setup, find_packages

setup(
    name='My First Setup File',
    version='1.0',
    packages=find_packages()
)
```

We have removed the ``script`` argument, but have added the ``packages`` argument. ``find_packages`` will automatically look for packages in a given directory. If we don't provide any arguments, it will look in the current folder. If you want to see it in action, you can do the following within a python interpreter:

```pycon
>>> from setuptools import find_packages
>>> find_packages()
['my_package', 'my_package.mod_b', 'my_package.mod_a']
>>> find_packages(where='my_package')
['mod_b', 'mod_a']
```

So now you see, find packages returns a list with the names of the packages we will include in the setup. If you want to have a finer control, you can replace ``find_packages`` and type down exactly what you want to include in your install. Also, if your code would be inside of an extra folder, for example ``src``, you would need to specify it: ``find_packages(where='src')``. I, personally, prefer to keep the code directly in a folder next to the setup file, with a name that makes it very clear that it is the entire package inside, just like in the example in this section. Again, if you just install the code, 

```bash
python setup.py install
```

you will be able to use it:

```pycon
>>> from my_package.mod_b.file_b import *
>>> function_b()
This is function_b
```

With everything you know from importing files in Python, it is now dead easy to use relative, absolute imports, etc. because everything is added to the path by the installation procedure. 

### Installing in development mode
One of the important things to point out is that the setup process can be very helpful also while developing, not only when you are releasing the code. This can be achieved by installing the code in **development mode**. The only thing you need to do is to run the setup script with a different argument:

```bash
python setup.py develop
```

Now, if you change any of your files, it will be reflected, on your code. For example, let's edit *file_b.py**:

```python
def function_b():
    print('This is New function_b')
```

If you use the module again, you will see the output was modified:

```pycon
>>> from my_package.mod_b.file_b import *
>>> function_b()
This is New function_b
```

!!! note
    Remember that if you change a module in Python, you will need to restart the interpreter. Once you import a module, Python will skip any new import statements of the same packages.
    
One of the best things of this approach is that if you add a new module to your package, it will still work fine. For example, let's create a new module **C**, which uses what is already available in **a** and **b**, using two different import approaches. Create a folder called **mod_c**, with a file **file_c.py**, and add the following:

```python
from ..mod_a.file_a import  function_a
from my_package.mod_b.file_b import function_b

def function_c():
    print('This is function c')
    function_a()
    function_b()
```

And we can simply use it:

```pycon
>>> from my_package.mod_c.file_c import function_c
>>> function_c()
This is function c
This is function_a
This is New function_b
```

Now you see that by having a setup.py file, using relative and absolute imports became trivial. No more worries about the system path, the python path, etc. everything is taken care of. Having a setup.py even when you are just starting makes your life much easier. You stop worrying about where the code is, how to import it, etc.

### But where is the code, actually?
At some point you may be wondering what is happening to your code that makes it work. I will assume you are working within a virtual environment (because that is what you should be doing anyways), and that you know where is your virtual environment located (i.e. the folder where everything is stored). Navigate to the ``site-packages`` folder, in my case, it is located in:

```bash
venv/lib/python3.6/site-packages
```

**venv** is the name of the virtual environment. **python3.6** is the version I am using now, but it can be different for you. Before you install anything else, just after a fresh virtual environment is created, the contents of the folder are very limited. You have pip, setuptools, and not much more. Let's install the package in development mode:

```bash
python setup.py develop
```

And if you look at the contents of the **site-packages** folder, you will notice a new file created: **My-First-Setup-File.egg-link**. Pay attention that the name of the file is associated with the **name** argument we used in the ``setup.py`` file. You can open the file with any text editor, what do you see? Indeed, you see the full path to the directory that contains the **my_package* folder. That files is able to tell the Python interpreter what should be added to its path in order to work with our module. You see that it is not a lot of work, but being able to do it from a single command really pays off. 

If you, on the other hand, install the package:

```bash
python setup.py install
```

You will see that what gets installed in the **site-packages** folder is radically different, you will find a file called **My_First_Setup_File-1.0-py3.6.egg**. Egg files are just zip files which Python can uncompress and use when required. You can easily open the file with the same program you use to open zip files, and you will find a copy of your code. The folders, the files, etc. plus some metadata of your program, such as the author, license, etc. (which we haven't used yet in your simple setup file). Whenever you change your project files, they won't become available on your other programs, because now Python has its own copy of the source code. Whenever you change a file you will need to run the setup script again:

```bash
python setup.py install
```

And the code will be refreshed and you can use the new development. You have to be careful, because it is very easy to forget to run again the setup script, and you will be banging your head against the wall trying to find why your bugs are still there if you solved them in your code. If you are really developing a solution and trying things out, you should avoid the setup install and keep the setup develop approach. 

### Installing with pip
Just for completeness, it is important to show that once you have your **setup.py** file in place, you can also use pip to install it by running:

```bash
pip install .
```

Note two things: if you used the ``python setup.py`` approach, you should first remove the files created on your ``site-packages`` folder, or pip will fail with a cryptic message. Also note the ``.`` after the install, this is important to tell pip you want to install *this* package. The install always takes the name of the package you would like to install. One of the advantages of using pip for installing, is that you also get the uninstall for free. However, to uninstall you need to use the name you gave to your package:

```bash
pip uninstall My-First-Setup-File
```

Now you see that the name we gave to our package is independent of how we use it. We had to import ``my_package``, while the actual name is ``My-First-Setup-File``. This, in my opinion, can create a lot of headaches. For example, if you want to communicate with a serial device, you install a package called ``PySerial``, while to use it you have to ``import serial``. It may not seem like a big issue at the beginning, until you find two packages with different names which define two different modules with the same name ``serial``.  Imagine we change the name of **my_packe** to **serial**. And we do the following:

``bash
pip install .
pip install pyserial
``

If you explore the folder **serial** within your **site-packages** you will see that both your code and **pyserial** is mixed up. There is no warning of this clash, and the results can be an absolute disaster, especially if the top-level ``__init__.py`` file is used somehow. The example above may seem a bit far-fetched for some. But you would be surprised to know that there are two packages: **pyserial** and **serial** which are used for some very different tasks and both specify the same module name. 

### Installing from Github
One of the advantages of creating **setup.py** files is that you can directly install packages available on github (or on any other online repository). For example, if you would like to install the code of this tutorial, which can be [found here](https://github.com/PFTL/website_example_code), you can run:

```bash
pip install git+https://github.com/PFTL/website_example_code.git#subdirectory=code/38_creating_setup_py
```

There are several things to note on the code above. First, it will not work on Windows as it is written. You will need to enclose the repository address with ``"`` so it understands it is the path to a repository. Note that we have used ``git+https`` because it is the simplest way of making it work. If you have configured your ssh keys properly, you can also use ``pip install git+git``, which can give you access to private repositories as well. We have used ``github.com``, but you can use any other version control platform you like, including gitlab and bitbucket. The path up to ``website_example_code.git`` is simply the location of the repository, but in our case the **setup** file is inside of a subdirectory, that is why we appended the ``#subdirectory=code/38_creating_setup_py`` information. But if it is not the case, you can skip that extra information. 

!!! note
    In most repositories, the **setup.py** is at the top level, and thus you won't need to specify a subdirectory in order to use it.

Now you see that it is very easy to share with others, you just need to put the code in a place that can be accessed by others. However, when you start sharing code, there are other things you need to be aware of. Imagine you update your code. In our case, we changed what **function_c** does. If we upload the code to github, and then you run the same command as above, buried within a lot of information, there will be a line saying:

```bash
Requirement already satisfied (use --upgrade to upgrade)
```

This means that the code was not changed, because pip found that the package was already installed. You can run the same code with the extra ``--upgrade`` argument, and you will see that indeed it updates the code. 

Just to close the discussion with pip installations, there is a very nice feature which allows you to install the package in editable mode, meaning that you can make changes and submit them to version control. This is ideal if you are willing to contribute to a package which is part of the requirements of your current project. In that case, you can run pip like this:

```bash
pip install -e git+git://github.com/PFTL/website_example_code.git#egg=my_test_package-1.0\&subdirectory=code/38_creating_setup_py
```

And if you explore your virtual environment, you will find a folder called **src** in which the package is available. Not only that, but the package itself is a git repository you can use as any other repository. You will also find a file within *site-packages* linking to the directory where the code is, including its subdirectory.  

## Adding an entry point
So far everything is working great, and probably you have plenty of ideas in your head about what can be done with a setup file. But what we have seen up to now is only the beginning. In the examples above, we have always installed a package that could be imported from other packages. This is great for a lot of applications, but in a lot of situations, you would like to be able to run your program directly, without the need to import from another script. That is when entry points become your best friends. 

Let's creat a new file, called **start.py** at the top directory, **my_package**, and let's add the following:

```python
from .mod_c.file_c import function_c


def main():
    function_c()
    print('This is main')
```

This file in itself does not do anything, but it the function ``main`` will generate an output of we run it. If we go back to **setup.py**, we can add the following:

```python
from setuptools import setup, find_packages

setup(
    name='My First Setup File',
    version='1.1',
    packages=find_packages(),
    entry_points={
        'console_scripts': [
            'my_start=my_package.start:main',
        ]
    }
)
```

Note that we have added a new argument, called ``entry_points``. We want to have a console script, meaning something we can run as a command from the terminal. We call it ``my_start``, and then we specify the path to it in the form ``module.file:function``. The last argument must be a callable, something we can execute. If you install our package again:

```bash
python setup.py develop
```

We will be able to run ``my_start`` directly from the terminal:

```bash
$ my_start
This is New function_b
This is main
```

If you want to know where the command is located, you can run (Linux and Mac, not Windows):

```bash
which my_start
[]/venv/bin/my_start
```
**Windows** users need to run:

```
where my_start
[]/venv/Scripts/my_start.exe
```

So, the script is located inside the ``bin`` folder of your virtual environment. One of the nicest things of doing this is that if you are on Windows it will also work. You see that it actually creates an executable ``.exe`` file. The function you run as an entry point can be further extended with arguments, etc. but this is for another discussion. 

If you are building application with a user interface, instead of using ``console_scripts`` you can use ``gui_scripts``. On Windows, this will allow you to run the program without opening a terminal in the background, and this it will look more professional. But is nothing you can't live without. 

Remember that you can add as many entry points as you wish. If your program can perform very different tasks, perhaps you would prefer to have different entry points instead of an endless list of arguments which alter the behavior. It is up to you to choose what you believe is the best approach. 

For consistency, there is one extra thing to mention. Right now, with the entry point, we have a script that runs a specific task, but we can't run our package directly from Python. Thus, let's create a file at the top-level directory of the package, called **__main__.py**, with the following:

```python
from .mod_c.file_c import function_c


def main():
    function_c()
    print('This is main')


if __name__ == '__main__':
    main()
```

It's the same than our start file, but with the extra two lines at the end. Now, we can run our package directly:

```bash
python -m my_package
```

If you look around other packages, you will see that many define a ``__main__`` file and then an entry point using it. We could redefine our entry point, and it will look like:

```python
entry_points={
        'console_scripts': [
            'my_start=my_package.__main__:main',
        ]
    }
```

## Adding dependencies
Right now we have a package which is completely independent from anything else around. Therefore, the next step is to add dependencies to the setup.py file. As an example, let's assume we want to have numpy (regardless of its version) and Django before version 3.0. We can add the following to the **setup.py** file:

```python
install_requires=[
        'numpy',
        'django<3.0'
    ]
```

You will see that if we run the setup script, python will automatically fetch the latest numpy version and Django 2.2 (even though 3.0 is available). Working with dependencies can be a nightmare, and pip is not great at solving conflicts. Imagine you have two packages which require different versions of some libraries, the one you install the latest is the one that prevails. And this is even without considering that those libraries have other requirements, etc. There are other package managers which are more powerful than pip, and take into account all the dependencies in order to find the sweetspot that satisfies all the requirements. 

### Difference with requirements files
A common issue when people start working with setup files is that they wonder what is the role requirement files have if you can specify everything in the setup file. The general answer is that requirement files are normally used to keep track of an environment, and therefore they normally include the specific version of the libraries used. This ensures that you can reproduce almost the exact same environment that the person developing the code had. Setup files, on the other hand, should be more accommodating. In the end, what we want is to make it simple for users to get our code running. Therefore, normally they include the minimum version of libraries to run, or the maximum version, in case future changes of a library would break things. 

The reality is that there are no hard-written rules. You have to put yourself in the shoes of your users and decide what would make their life easier. Is it pin-pointing the versions of every library in the environment, or is it giving flexibility? If it would be a developer looking into your code, one of the options is to always to download the code, install everything from **requirements.txt** and then run setup.py. If things are well configured, the last step won't download anything, since all the dependencies will be present in our environment. And then you know both people have the same exact versions installed while developing. 

There is also the option of not adding any requirements in your setup, and asking users to install everything in the requirements file in order to use your library. This approach is very user-unfriendly, but it also works. So, you see, there is a lot of flexibility for you to decide what approach you think is best in your particular situation. 

## Extra Dependencies
Setup files also give us the option to define extra dependencies, which are not always necessary, but which can enhance the functionality of our program in some cases. We simple define them as an extra argument for the setup function:

```python
from setuptools import setup, find_packages

setup(
    name='My First Setup File',
    version='1.1',
    packages=find_packages(),
    entry_points={
        'console_scripts': [
            'my_start=my_package.__main__:main',
        ]
    },
    install_requires=[
        'numpy>1.18',
        'django<3.0'
    ],
    extras_require={
        'opt1': ['serial'],
        'opt2': ['pyserial'],
    }
)
```
And to install them, we can run a pip command:

```bash
pip install .[opt1]
```

or

```bash
pip install .[opt2]
```

Note that this works also with libraries available on PyPI, for example, ``scikit-learn`` can be installed with extra dependencies:

```bash
pip install scikit-learn[alldeps]
```

Extra dependencies are a great feature when you have dependencies which are hard to install in some systems, or which take a lot of resources, or which are conflicting, just like serial and pyserial above. Note that nothing prevents you from installing both ``opt1`` **and** ``opt2``, thus the conflict can still arise. 

## Where to next
This was an introduction to develop your first **setup.py** file. The topic is far from over, but with what you have up to now, you can go a long way. Something that is very useful is to pay attention to the libraries you use and how they behave. For example, you can check the [setup file of scikit-learn](https://github.com/scikit-learn/scikit-learn/blob/master/setup.py), or of [Django](https://github.com/django/django/blob/master/setup.py). They can be much more complex than what you need, but they can also point you in the right direction if you are looking for inspiration on how to structure your code. If you are curious about more down-to-earth examples, you can see the one I've developed for the [Python for the Lab workshop](https://github.com/PFTL/SimpleDaq/blob/master/setup.py), or for [the startup in which I'm working](https://github.com/aquilesC/DisperPy/blob/master/setup.py). Many of the arguments of setup.py which are available (and almost trivial), such as ``author``, ``website`` were skipped in this tutorial, since they don't bring much to the discussion, and you can quickly add them. 

An obvious step once you have your setup file in order is to release your code through **PyPI**. This topic will be covered soon. Something we didn't discuss, and which I hope grabbed your attention is that when you run ``python setup.py install`` some folders are created, such as ``build`` and ``dist``. Go ahead and explore them, see what is inside and try to understand what is happening when you install the package as we have just done. 

Finally, another very interesting topic is what happens if you want to release your package to be installable through **conda** instead of **pip**. Conda is a great package manager, that allows you install also non-python libraries and manage dependencies much more efficiently than what pip does. However, developing a package which is conda-installable requires some extra steps, which will also be covered later. 