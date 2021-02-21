# Singletons: Instantiate objects only once

When developing more extensive programs, being aware of different patterns can significantly help us solve problems even before they arise. One of those patterns is the creation of singletons, which are nothing else but objects that can be instantiated only once. In Python, we are exposed to singletons since the beginning, even if we are not aware of them. This article will discuss how singletons permeate our everyday programming and how we can bring them a step further. 

## Every Day Singletons
Before we dive into singletons, it is important to remember how Python behaves when it comes to mutable and immutable data types. A list, for example, is mutable, which means we can change its contents without creating a new object. For example:

```pycon
>>> var1 = [1, 2, 3]
>>> var2 = var1
>>> var1[0] = 0
>>> print(var2)
[0, 2, 3]
```

!!! note
    If the output of the code above puzzles you, I strongly advise you to check out the article on mutable and immutable data types. 

When we have two lists like ``var1`` and ``var2``, we can see whether they have the same content:

```pycon
>>> var1 == var2 
True
```

But we can also see if they are the same object:

```pycon
>>> var1 is var2
True
```

However, we can also do the following: 

```pycon
>>> var1 = [1, 2, 3]
>>> var2 = [1, 2, 3]
>>> var1 == var2
True
>>> var1 is var2
False
```

In this case, both ``var1`` and ``var2`` have the same values ``[1, 2, 3]`` but they are not the same object. That is why ``var1 is var2`` returns ``False``. 

However, Python programmers are likely exposed to the following syntax very early on:

```python
if var is None:
    print('Var is none')
```

At first sight the question that could pop anyone's mind is why can we use ``ìs`` in the example above. And the answer is that ``None`` is a special type of object, one that can be instantiated only once. Let's see some examples:

```pycon
>>> var1 = None
>>> var2 = None
>>> var1 == var2
True
>>> var1 is var2
True
>>> var3 = var2
>>> var3 is var1
True
>>> var3 is None
True
```

It means that through our code, there can be only one ``None``, and any variable that references it will reference the same object, unlike what happened when we created two lists that had the same values. Together with ``None``, the other two common singletons are ``True`` and ``False``:

```pycon
>>> [1, 2, 3] is [1, 2, 3]
False
>>> None is None
True
>>> False is False
True
>>> True is true
True
```

That completes the triad of singletons that Python programmers encounter daily. It explains why we can use the ``is`` syntax when checking if a variable is ``None``, ``True`` or ``False``. The examples above are only the beginning regarding singletons.

## Small integer singletons
Python defines other singletons that are not that obvious and are mostly there because of memory and speed efficiency. Such is the case of small integers in the range -5 to 256. Therefore, we can do something like this:

```pycon
>>> var1 = 1
>>> var2 = 1
>>> var1 is var2
True
```

Or, more interestingly:

```pycon
>>> var1 = [1, 2, 3]
>>> var2 = [1, 2, 3]
>>> var1 is var2
False
>>> for i, j in zip(var1, var2):
...     i is j
... 
True
True
True
```

In the example above, you see two lists with the same elements. They are not the same list as we saw earlier, but each element is the same. If we want to get fancier with Python's syntax (just because we can), we can also run the following:

```pycon
>>> var1 = [i for i in range(250, 260)]
>>> var2 = [i for i in range(250, 260)]
>>> for i, j in zip(var1, var2):
...     print(i, i is j)
... 
250 True
251 True
252 True
253 True
254 True
255 True
256 True
257 False
258 False
259 False
```

And we see that up to 256 integers are the same, but from 257 onwards, they are not. 

## Short strings singletons
Small integers are not the only unexpected singletons in Python. Short strings are, sometimes, also singletons. We can try something like the following:

```pycon
>>> var1 = 'abc'
>>> var2 = 'abc'
>>> var1 is var2
True
```

However, with strings, the reality is somewhat different. The process is called **string interning** and is described on [Wikipedia](https://en.wikipedia.org/wiki/String_interning). Python decides whether to allocate memory for strings as singletons based on some rules. First, strings must be defined at *compile-time*. It means that they should not be the output of a formatting task or a function. In the example above, ``var1 = 'abc'`` qualifies. 

Python tries its best at being efficient, and it will intern other strings that it considers will help save memory (and/or time). For example, function names are interned by default:

```pycon
>>> def test_func():
...     print('test func')
... 
>>> var1 = 'test_func'
>>> test_func.__name__ is var1
True
```

Empty strings and some single-character strings are interned by default, as is the case with small integers:

```pycon
>>> var1 = chr(255)
>>> var2 = chr(255)
>>> var3 = chr(256)
>>> var4 = chr(256)
>>> var1 is var2
True
>>> var3 is var4
False
```

The fact that some strings are interned, does not mean we can get too confident about it. For example:

```pycon
>>> var1 = 'Test String'
>>> var2 = 'Test String'
>>> var1 is var2
False
>>> var2 = 'TestString'
>>> var1 = 'TestString'
>>> var1 is var2
True
```

As we can see in the example above, being a short string is not the only requirement. The string must also be made of a limited set of characters, and spaces are not part of it. 

Therefore, the fact that Python is interning strings does not mean we should use the ``is`` syntax instead of the ``==``. It just means that Python is running some optimizations under the hood. Perhaps one day, those optimizations become relevant for our code, but likely they will go completely unnoticed (but enjoyed). 

## Why singletons
What we have seen up to now is interesting, but it does not answer why going through the trouble of defining singletons. One clear answer is that by having singletons, we can save memory. In Python, what we usually call variables, can also be considered labels, pointing to the underlying data. If we have several labels pointing to the same data, it would be very memory efficient. There's no duplication of the information. 

However, if we ask ourselves when to add a singleton to our code, most likely, we won't find good examples. A singleton is a class that is instantiated only once. All other instances refer to the first, and therefore *are* the same. If you think about it, singletons and global variables are easy to mistake one for the other. However, a global variable does not imply anything about how it is instantiated. We could have a global variable pointing to an instance of a class and a local variable pointing to a different instance of the same class. 

Singletons are a programming pattern, and as such, they can be useful, but there's nothing we can do with them that can't be done without them. A standard example of singletons is a **logger**. Several parts of the program share information with the logger. The handler then decides whether to print to the terminal, save to a file, or don't do anything.

## Defining a singleton
The crucial point of singletons is preventing multiple instantiations. Let's start by checking what happens when we instantiate a class twice:

```python
class MySingleton:
    pass


ms1 = MySingleton()
ms2 = MySingleton()
print(ms1 is ms2)
# False
```

As expected, both instances are different objects. To prevent the second instantiation, we must keep track of whether a class was instantiated. We can use a variable in the class itself to do it and return the same object. A possibility is to use the ``__new__`` method of the class:

```python
class MySingleton:
    instance = None
        
    def __new__(cls, *args, **kwargs):
        if not isinstance(cls.instance, cls):
            cls.instance = object.__new__(cls)
        return cls.instance
```

And we can test it:

```pycon
>>> ms1 = MySingleton()
>>> ms2 = MySingleton()
>>> ms1 is ms2
True
```

This approach is relatively straightforward. We only need to check whether the ``instance`` was defined and create it if it does not exist. Sure, we could use ``__instance``, or fancier ways of checking if the variable exists. The result would be the same. 

Singletons are a pattern which, in the end, is hard to justify. So we can see one example on which we open a file more than once. Our singleton class would look like this:

```python
class MyFile:
    _instance = None
    file = None

    def __init__(self, filename):
        if self.file is None:
            self.file = open(filename, 'w')

    def write(self, line):
        self.file.write(line + '\n')

    def __new__(cls, *args, **kwargs):
        if not isinstance(cls._instance, cls):
            cls._instance = object.__new__(cls)

        return cls._instance
```

Note a couple of things. First, we have defined ``file`` as a class attribute. We do this because the ``__init__`` method will be executed as often as the class is instantiated. Therefore, we open the file only once. We can achieve the same behavior directly in the ``__new__`` method after checking the ``_instance`` attribute. Note that we open the file in ``w`` mode, which means that we will overwrite the file's content each time. 

We can use the singleton like this:

```pycon
>>> f = MyFile('test.txt')
>>> f.write('test1')
>>> f.write('test2')

>>> f2 = MyFile('test.txt')
>>> f2.write('test3')
>>> f2.write('test4')
```

And the contents of the file would be:

```
test1
test2
test3
test4
```

It is clear from the example above that it does not matter if we first define ``f`` or ``f2``. The file to which we are going to write is opened only once. The contents get erased only once, and every time we write to it through the program, we will append lines. 

We can also check if it is a singleton:

```pycon
>>> f is f2
True
```

However, in the way we defined our class above, there is one huge problem. What would be the output of the following?

```pycon
>>> f = MyFile('test.txt')
>>> f.write('test1')
>>> f.write('test2')
>>> f2 = MyFile('test2.txt')
>>> f2.write('test3')
>>> f2.write('test4')
```

The code is valid, but the program will create only the first file (``test.txt``) and ignore the second instantiation argument. Another fascinating point is what happens if we removed completely the ``__new__`` method:

```python
class MyFile:
    file = []

    def __init__(self, filename):
        if len(self.file) == 0:
            self.file.append(open(filename, 'w'))

    def write(self, line):
        self.file[0].write(line + '\n')
```

By definition this class is not a singleton, since each time we instantiate it we will get a different object:

```pycon
>>> f = MyFile('test.txt')
>>> f2 = MyFile('test.txt')
>>> f is f2
False
>>> f.write('test1')
>>> f.write('test2')
>>> f2.write('test3')
>>> f2.write('test4')
```

We are slightly cheating now because we changed the ``file`` attribute from ``None`` to an empty list. We do this because lists are mutable. When we append the opened file, the list is still the same and therefore shared among the other instances. In any case, the end effect is the same. The file is opened only once, and the lines get written in the same fashion as before. 

The idea of this quick example is that opening the file only once is not a feature exclusive of singletons. Just by smartly using mutability, we can achieve the same effect in even fewer lines of code. 

## Conclusions
The singleton pattern can be of great help when designing lower-level applications or frameworks. Python uses singletons to speed up its execution and to be more memory efficient. If we compare the time it takes to resolve ``f == f2`` and ``f is f2`` in the singleton and not-singleton cases, we can see some time improvements in the first. Whether the modifications impact the costs and limitations of the implementation will depend on how often we check equality. 

For higher-level solutions, however, the singleton pattern is harder to come by. I have tried to dig examples in other projects, but the only answer that comes back is the *logger* implementation. If anyone has an excellent example of using singletons in higher-level programs, I would be very grateful to hear it. 

### Singletons can break unit tests
A side note that is worth mentioning is that the singleton pattern can easily break unit tests' idea. In the singleton example above, we could have modified the ``MyFile`` object by doing something like ``f.new_file = open('another_file')``. This change would have been persistent and can potentially alter other tests. The spirit behind unit tests is that each one is responsible for one thing and one thing only. If tests can impact each other, they are no longer *unit* tests. 
