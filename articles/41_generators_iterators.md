Generators, Iterables, Iterators in Python: When and Where
==========================================================

Generators, Iterables, and Iterators are some of the most used tools in Python. However, we don't often stop to think about how they work, how we can develop our generators and iterables. Once you learn what you can do with them, it is possible to expand your toolbox and make your code much more efficient and pythonic. 

## Iterables
One of the very first things you learn about Python is that if you have a list, for example, you can go through all the elements with a convenient syntax:

```pycon
>>> var = [1, 2, 3]
>>> for element in var:
...     print(element)
... 
1
2
3
```
Even if you have worked with Python for long, you may give this syntax for granted, without stopping to think a lot about it. At some point you may find that it also works with tuples or dictionaries:

```pycon
>>> var = {'a': 1, 'b':2, 'c':3}
>>> for key in var:
...     print(key)
... 
a
b
c
```

The same syntax works with strings:

```pycon
>>> var = 'abc'
>>> for c in var:
...     print(c)
... 
a
b
c
```

As you can imagine, an iterable is an object in Python that allows going through its elements one by one. It seems like almost any data type that allows to group information together is iterable. So, the next logical step is to think about whether we can define our own iterable. 

### The \_\_getitem\_\_ approach 

As you may know now, to define our own types, we define classes. We could do something like this:

```python
class Arr:
    def __init__(self):
        self.a = 0
        self.b = 1
        self.c = 2
```

And to use it:

```pycon
>>> a = Arr()
>>> print(a.a)
0
>>> print(a.b)
1
```

You can see very, very briefly, how to get started using classes. It is a simple example, in which we store three different values, and we then print two of them. However, if we try the following, we would get an error:
```pycon
>>> for item in a:
...     print(item)
... 
Traceback (most recent call last):
  File "<input>", line 1, in <module>
TypeError: 'Arr' object is not iterable

```
Indeed we see that we can't iterate over our object. Python just doesn't know what element comes first, which one later. Therefore, we need to implement it ourselves, and you'll see from the example that it is obvious: 

```python
class Arr:
    def __init__(self):
        self.a = 0
        self.b = 1
        self.c = 2

    def __getitem__(self, item):
        if item == 0:
            return self.a
        elif item == 1:
            return self.b
        elif item == 2:
            return self.c
        raise StopIteration

```

I know it is very convoluted, but let's see how it works:
```pycon
>>> a = Arr()
>>> print(a[0])
0
>>> print(a[1])
1
>>> for element in a:
...     print(element)
... 
0
1
2
```

Now you see that by implementing the ``__getitem__`` method, we can access the attributes of our object like if it would be a list, or a tuple, just by doing ``a[0]``, ``a[1]``. When you start threading this precisely, there are many paths you can follow, many questions you can ask yourself. At this stage may be essential to check what [duck typing](https://www.pythonforthelab.com/blog/duck-typing-or-how-to-check-variable-types/) is and how you can work with it. 

You can see that we also raise a ``StopIteration`` exception if we go beyond the first three elements. This exception is particular to iterables. In the for loop, we get all the elements we wanted without any problems. However, if we try the following the behavior wouldn't be what we expected:
```pycon
>>> a[3]
Traceback (most recent call last):
  File "<input>", line 1, in <module>
  File "<input>", line 14, in __getitem__
StopIteration
```

If it had been a list, we would expect to get an ``IndexError`` instead of a ``StopIteration`` exception. Let's see another example, that allows us to construct an iterable from data and not as we did with just 3 elements. We can develop an object called ``Sentence``, which can go word by word within a ``for`` loop, like this:

```python
class Sentence:
    def __init__(self, text):
        self.words = text.split(' ')

    def __getitem__(self, item):
        return self.words[item]
```

It is a simple example in which we take some text and split it in the spaces. This result is stored in the attribute ``words``. The ``__getitem__`` in this case is just returning the appropriate item from the list of words. We can use it as follows:

```pycon
>>> s = Sentence('This is some form of text that I want to explore')
>>> for w in s:
        print(w)

This 
is 
some 
form 
of 
text 
that 
I 
want 
to 
explore
```

You can very quickly see that the behavior is what we were expecting. Of course, there are some limitations, such as not taking into account punctuation. However, we see that the ``for`` loop stops without problems once the list of words runs out of elements. On the other hand, if we try something like ``s[100]``, we get the expected ``IndexError``. 

<div class="admonition note">
The class Sentence can be greatly improved if, for instance, we implement a ``__len__`` method. However, this goes beyond the scope of this article.
</div>

The only requisite to create an iterable object such as the ones we showed above, is to have a ``__getitem__`` method that **access elements using a 0-index** approach. The first element is s[0], and so forth. If you are relying on a list, such as in the example with ``Sentence``, then there are no problems. If you are developing something such as our ``Arr`` example, then you need to take care of ensuring the 0 is the first element to be retrieved. 

### Creating elements on the fly
In the examples above, we could go through the specified elements within a ``for``-loop. However, the elements we were iterating over were specified at the moment of instantiation of the class. It is not necessary. Let's see how it would look like if we create a class that can generate random numbers for as long as we want:

```python
import random 

class RandomNumbers:
    def __getitem__(self, item):
        return random.randint(0, 10)
``` 

And we can use it as follows:

```pycon
>>> r = RandomNumbers()
>>> for a in r:
>>>         print(a)
>>>         if a==10:
>>>             break
>>>             
4
1
5
2
5
7
2
7
9
9
3
10
```
Pay attention to the fact that we are stopping the loop as soon as the ``RandomNumber`` object generates a 10. If we don't do this, the for loop would be running forever. You may, or example, use a timer to stop after a certain time, or after a certain number of iterations, or you may simply want to run forever. The choice is yours. You can also limit the generation of values in the class itself:

```python
def __getitem__(self, item):
    num = random.randint(0, 10)
    if num == 10:
        raise StopIteration
    return num
```
And then we don't need to break the ``for`` loop, it naturally stops once a ``10`` is generated within the class. 

A situation where you may think about applying this method is, for example, if you are acquiring a signal from a device that generates sequential values. It may be frames acquired from a camera, or analog values read by a simple device one after the other. I will use these techniques in a later tutorial focusing on the interfacing with devices. 

Another example of an iterable that generates values only when requested is when we open a file. Python does not load into memory all the contents, but instead, we can iterate over each line. It is how it looks like with [one of the articles](https://github.com/PFTL/website_articles/blob/master/articles/39_use_arduino_with_python.md) of this website:

```pycon
>>> with open('39_use_arduino_with_python.md', 'r') as f:
...     for line in f:
...         print(line) 
...
Using Python to communicate with an Arduino
===========================================
    ...
```
This is what allows you to work with files that are much bigger than what it's possible to hold in memory. You can also quickly check it by looking at the sizes:

```pycon
>>> import sys
>>> with open('39_use_arduino_with_python.md', 'r') as f:
>>>     print(sys.getsizeof(f))
>>>     size = 0
...     for line in f:
...         size += sys.getsizeof(line) 
...     print(size)
...
216
56402
```
You can see that the size of the ``f`` variable is much smaller than the total size of the contents of the file added up. It is, of course, not an accurate method to know the size of a file. Still, it can get you an idea of the areas in which you can start considering iterables instead of loading a wealth of information to the memory of the computer.

## Iterators
In the section above, we have seen that we can create an iterable object just by defining an appropriate ``__getitem__`` method. We have also seen that several objects with which we have probably already interacted, such as lists, or files are iterables. However, something is happening under the hood, which is worth discussing and is called *iterators*. 

The difference between an iterable and an iterator is very subtle. Perhaps it can be thought as the difference between a class and an object. We know we can iterate over a list, or over a custom object. But how is Python dealing with that flow? We can, very quickly, see how it works with a simple list:

```pycon
>>> var = ['a', 1, 0.1]
>>> it = iter(var)
>>> next(it)
'a'
>>> next(it)
1
>>> next(it)
0.1
>>> next(it)
Traceback (most recent call last):
  File "<input>", line 1, in <module>
StopIteration
```

In the code above, we have constructed an iterator using the built-in function ``iter``. This function takes an iterable and returns an iterator. The iterator is the object which understands what ``next`` means. The iterator is iterable itself: 

```pycon
>>> it = iter(var)
>>> for element in it:
...     print(element)
...
a
1
0.1
```

If we would like more control over the process, Python gives it to us. An iterator must implement two methods: ``__next__`` and ``__iter__``. If we want our class to use a specific iterator, then it should also specify an ``__iter__`` method. Let's see what would happen with the ``Sentence`` class. To make it more obvious, we have it iterating through the words but in backward order. First, we define the iterator:

```python
class SentenceIterator:
    def __init__(self, words):
        self.words = words
        self.index = 0
    
    def __next__(self):
        try:
            word = self.words[self.index]
        except IndexError:
            raise StopIteration()
        self.index -= 1
        return word

    def __iter__(self):
        return self
```

The code above is relatively clear. You only have to keep in mind that the only way of specifying a ``next`` value is by keeping track of the current index is returned. Now that we have our iterator, we must update the original ``Sentence`` class:

```python
class Sentence:
    def __init__(self, text):
        self.words = text.split(' ')
    
    def __iter__(self):
        return SentenceIterator(self.words)
```

And now comes the fun part:

```pycon
>>> text = "This is a text to test if our iterator returns values backward"
>>> s = Sentence(text)
>>> s[0]
Traceback (most recent call last):
  File "<input>", line 1, in <module>
TypeError: 'Sentence' object does not support indexing
```
Indeed, we can't access the elements of our ``Sentence`` object because we haven't specified the appropriate ``__getitem__``. However, the ``for`` loop will work:

```pycon
>>> for w in s:
...     print(w)
...     
This
backward
values
returns
iterator
our
if
test
to
text
a
is
This
```

You see, the order in which it prints is backward, as expected. Just the first word is mistaken because we start in ``0``, and that also corresponds to the last index: ``-12``. You can find a solution to this issue if you want, it just requires a bit of thinking. 

The example above also works with ``iter``:

```pycon
>>> it = iter(s)
>>> next(it)
'This'
>>> next(it)
'backwards'
```

It is important to note that once the iterator is exhausted, there is nothing else we can do with it. At the same time, we can always go through the elements in the ``Sentence`` object within a ``for`` loop.

The only missing part now is adding the ability to access the words by index but in the proper order. If we develop a ``__getitem__``, what do you expect to happen to the ``for`` loop?

```python
class Sentence:
    def __init__(self, text):
        self.words = text.split(' ')
    
    def __iter__(self):
        return SentenceIterator(self.words)
        
    def __getitem__(self, item):
        return self.words[item]
```

and we use it as follows:

```pycon
>>> s = Sentence(text)
>>> s[0]
'This'
>>> s[1]
'is'
>>> for w in s:
...     print(w)
...     
This
backwards
values
returns
iterator
our
if
test
to
text
a
is
This
```

So now you see, we can access the elements through their index, and we can iterate over them in backward order. 

### Can Sentence have a \_\_next\_\_ method?
In principle, you can make ``Sentence`` also an iterator, by adding a ``__next__`` method. But this may be a terrible idea. Remember that iterators work until they exhaust themselves, and they maintain an internal index. If you mix ``Sentence`` as both an iterable and an iterator, you will run into problems every time you encounter a double loop. 

<div class="admonition note">
Splitting the iterator and the iterable is a good idea. You can read a bit more about it in the book Fluent Python, by Luciano Ramalho. Chapter 14 is entirely dedicated to this topic. 
</div>

For people working with scientific instruments, splitting the iterable and the iterator can have benefits. It is plausible to think scenarios where you want an iterator that behaves in different ways, depending on a parameter. For example, imagine you are reading from a camera, and want to achieve something like:

```pycon
>>> for frame in camera:
...     analyze(frame)
```

Very simple, you analyze every frame generated by a camera. However, you may be in a situation where an external trigger generates frames. Therefore you want to attach the timestamp to each frame, or it may be that the camera acquires a finite number of frames at a given framerate. In principle, it could be the iterator that takes care of it. 

In many cases, however, we don't need to define a separated iterator, we can use **generators** directly in our ``__iter__`` definition. That is the focus of the next section.

## Generators: Using the yield keyword
The last important topic to cover is the role of **generators**, which, in my opinion, are still under-utilized, but they open new doors. A classic example is generating an infinite list of numbers. We know that it is impossible to store a list of infinite numbers in memory. Still, if we know the spacing between them, we can always know what number comes next. It is the core idea with generators. Let's start with a straightforward idea to understand the mechanics:

```python
def make_numbers():
    print('Making a number')
    yield 1
    print('Making a new number')
    yield 2
    print('Making the last number')
    yield 3
```

The first thing that should grab your attention in the code above is that we define a function with the keyword ``def``, but instead of a ``return``, we use a ``yield`` statement. The prints are there for you to understand the flow. Another interesting aspect is that we have three ``yield``, while for a normal function, you would have at most one ``return``. And now, to use it, we can do the following:

```pycon
>>> a = make_numbers()
>>> print(a)
<generator object make_numbers at 0x7f65a8ea8ed0>
``` 

As soon as we invoke the *function* ``make_numbers``, we see no print statement. It means that nothing is running yet. Now, we can ask for the next element of the generator:

```pycon
>>> next(a)
Making a number
1
```

Now you see the print statement is running, because we get the ``making a number`` string on the screen, and we also get the first number. We can keep going:

```pycon
>>> next(a)
Making a new number
2
>>> next(a)
Making the last number
3
>>> next(a)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
>>> 
```

And there you have it, once we run out of ``yield`` statements, the ``StopIteration`` is raised. It is the same exception we saw earlier when developing our iterables. This exception shows us that we could use a generator in the same way we have used an iterable before. For example, we can use it inside a for-loop:

```pycon
>>> b = make_numbers()
>>> for i in b:
...  print(i)
... 
Making a number
1
Making a new number
2
Making the last number
3
```

We can expand the generators to do a plethora of things. For example, let's see how to generate a flexible number of equally spaced integers:

```python
def make_numbers(start, stop, step):
    i = start
    while i<=stop:
        yield i
        i += step
```

And based on what we have done before, we can do, for example:

```pycon
>>> for i in make_numbers(1, 20, 2):
...     print(i)
1
3
5
7
9
11
13
15
17
19
```

What is very important to note here is that every number is generated only when it is needed, and hence the name of generators. Let's see what happens with the amount of memory used by our variables:

```pycon
>>> import sys
>>> z = make_numbers(0, 1000000000, 1)
>>> sys.getsizeof(z)
128
```

The variable ``z`` goes from 0 to 1 billion in steps of 1. However, if we check the amount of memory used, we get just 128 bytes. It is very remarkable and is possible thanks to the generator syntax. 

### Generators in classes
One pattern remaining is mixing generators and iterators. We can rewrite the Sentence class to allow us looping through its elements:

```python
class Sentence:
    def __init__(self, text):
        self.words = text.split(' ')

    def __iter__(self):
        for word in self.words:
            yield word
```

And we can simply use it as before:
```pycon
>>> text = "This is a text to test our iterator"
>>> s = Sentence(text)
>>> for w in s:
...     print(w)
This
is
a
text
to
test
our
iterator
```

Notice that we didn't define an explicit iterator such as ``SentenceIterator``, and we are not defining the ``__next__`` method. However, we can still iterate through our sentence. The example is so basic that maybe we can't see its usefulness. Let's adapt it slightly to be able to iterate through each word of a file. One option would be to read the entire file and split it into words, making a huge list. It can consume too much memory if the file is large enough, and therefore it is not handy. But nothing prevents us from reading line by line:

```python
class WordsFromFile:
    def __init__(self, filename):
        self.filename = filename

    def __iter__(self):
        with open(self.filename, 'r') as f:
            for line in f:
                words = line.split(' ')
                for word in words:
                    yield word
```

For example, we could use it like this:

```pycon
>>> words = WordsFromFile('22_Step_by_step_qt.rst.md')
>>> for w in words:
...     print(w)
```

What you can see from the example is that when we instantiate the ``WordsFromFile`` class, we only store the filename, we don't open any file. However, when we iterate through the file, we open it, and instead of reading it all to memory, we iterate through each line. It is something that Python allows us to do with files. Then, we split the words on each line, just as we did before, and we ``yield`` each word on each line. 

Another exciting result that arises from using a generator as we just did is that we can nest loops, for example:

```pycon
>>> for w in words:
...     for c in words:
...         print(w, c)
```

It is a ridiculous example, but we would get an output like the following:

```pycon
considerations very
considerations simple
considerations way.
considerations You
considerations could
considerations find
considerations better
considerations solutions,
considerations of
considerations course,
considerations but
considerations this
```

If we had defined an index in the class and used it to grab the following element, the nesting would not have worked. You can go ahead and try it by yourself. 

## Generators, Iterators, Iterables?
It may happen that after reading the entire guide, you are still struggling to understand the difference between generators, iterators, and iterables. Iterables are the easiest to separate because it refers to objects on which is possible to iterate. Without the formality, if given an element of your object, you know how to get the next element, then your object is iterable. 

To iterate through objects, however, you create a new type of class, called an iterator. Iterators are the ones responsible for keeping track of the index, and knowing when you reached the end. Generators are, in essence, the same, but they just use the ``yield`` syntax. 

There are some subtleties regarding whether iterators can be generators or vice-versa. Still, the distinction is useful when you want to transmit a clear message of what your code is doing. An iterator can be thought of as a class that goes through given elements. For example, a file has lines, and an iterator would go through all those lines. A generator, on the other hand, can generate values under demand. For example, we could generate numbers one after the other while they are being requested. 

The main difference would be that an iterator just returns values as they are, while generators can modify values on the fly. Is this a strict distinction? I would claim that it is not, and the gray area is so large, that it doesn't make much sense to stress about the specificities. However, I do like making the distinction if I have to explain my code to someone. I do believe that the words generator and iterator make a big difference when understanding the logic. 

For example, if I am acquiring images from a Camera, I would say I am using a generator. The images are not available beforehand, nor I know how they are going to look like. On the other hand, if I am reviewing the frames of data stored on the hard drive, I would try to use the word iterator. The data is already there, and I am not altering it. 

## Conclusions
Iterators and Generators are two tools that are very handy to keep in your repository of strategies when planning code. They are especially handy when you are dealing with streams of data that may exceed the memory you have available. In the past, we have seen how to create [your classes supporting context managers](https://www.pythonforthelab.com/blog/the-with-command-and-custom-classes/), with generators and iterators now you can support loops and iterations. 

Understanding generators is essential not only to be able to add it to your code but also to be able to understand the logic behind other packages. If you quickly search around, Django uses [plenty of generators in its code base](https://github.com/django/django/search?q=yield&unscoped_q=yield). Once you can understand generators, it will give you a hint to how the developer was expecting you to use their code. 

