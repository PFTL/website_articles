Generators, Iterables, Iterators in Python: When and Where
==========================================================

Generators, Iterables and Iterators are some of the most used tools in Python. However, we don't often stop to think how they work, how we can develop our own generators and iterables. Once you learn what can be done with them, it is possible to expand your toolbox and make your code much more efficient and pythonic. 

## Iterables
One of the very first things you learn about python is that if you have a list, for example, you can go through all the elements with a very handy syntax:

```python
var = [1, 2, 3]
for element in var:
    print(element)

1
2
3
```
Even if you have been working with Python for a while, it is possible that you give this syntax for granted, without stopping to think a lot about it. At some point you may find that it also works with tuples, or dictionaries:

```python
var = {'a': 1, 'b':2, 'c':3}
for key in var:
    print(key)

a
b
c
```

The same syntax works with strings:

```python
var = 'abc'
for c in var:
    print(c)

a
b
c
```

As you can imagine, an iterable is an object in Python that allows to go through its elements one by one. It seems like almost any data type that allows to group information together is iterable. So, the next logical step is to think whether we can define our own iterable. 

### The \_\_getitem\_\_ approach 

As you may know now, to define our own types, we define classes. We could do something like this:

```python
class Arr:
    def __init__(self):
        self.a = 0
        self.b = 1
        self.c = 2

a = Arr()
print(a.a)
0
print(a.b)
1
```
You can see very, very briefly how to get started using classes. It is a very simple example, in which we store three different values, and we then print two of them. However, if we try the following, we would get an error:
```python
for item in a:
    print(item)

Traceback (most recent call last):
  File "<input>", line 1, in <module>
TypeError: 'Arr' object is not iterable

```
Indeed we see that we can't iterate over our object. Python just doesn't know what element comes first, which one later, etc. Therefore, we need to implement it ourselves, and you'll see from the example that it is very clear: 

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
```python
a = Arr()
print(a[0])
0
print(a[1])
1
for element in a:
    print(element)

0
1
2
```

Now you see that by implementing the ``__getitem__`` method, we can access the attributes of our object like if it would be a list, or a tuple, just by doing ``a[0]``, ``a[1]``. When you start threading this specifically, there are many paths you can follow, many questions you can ask yourself. At this stage may be important to check what [duck typing](https://www.pythonforthelab.com/blog/duck-typing-or-how-to-check-variable-types/) is and how you can work with it. 

You can see that we also raise a ``StopIteration`` exception if we go beyond the first three elements. This exception is very specific to iterables. In the for loop, we get all the elements we wanted without any problems. However, if we try the following the behavior wouldn't be what we expected:
```python
a[3]

Traceback (most recent call last):
  File "<input>", line 1, in <module>
  File "<input>", line 14, in __getitem__
StopIteration
```

If it would have been a list, we would expect to get an ``IndexError`` instead of a ``StopIteration`` exception. Let's see another example, that allows us to construct an iterable from data and not as we did with just 3 elements. We can develop an object called ``Sentence``, which can go word by word within a ``for`` loop, like this:

```python
class Sentence:
    def __init__(self, text):
        self.words = text.split(' ')

    def __getitem__(self, item):
        return self.words[item]
```

This is a very basic example, in which we take some text and split it in the spaces. This result as stored in the attribute ``words``. The ``__getitem__`` in this case is jus returning the appropriate item from the list of words. We can use it as follows:

```python
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

You can very quickly see that the behavior is what we were expecting. Of course, there are some limitations, such as not taking into account punctuation, etc. But we see that the ``for`` loop stops without problems once the list of words runs out of elements, without raising an exception. On the other hand, if we try something like ``s[100]`` we will get the expected ``IndexError``. 

<div class="admonition note">
The class Sentence can be greatly improved if, for instance, we implement a ``__len__`` method. However, this goes beyond the scope of this article.
</div>

The only requisite to create an iterable object such as the ones we showed above, is to have a ``__getitem__`` method that **access elements using a 0-index** approach, i.e., the first element will be s[0], etc. If you are relying on a list, such as the example with ``Sentence`` then there are no problems. If you are developing something more customized, such as our ``Arr`` example, then you need to take care of ensuring the 0 will be the first element to be retrieved. 

### Creating elements on the fly
In the examples above, we could go through the specified elements within a ``for``-loop. However, the elements we were iterating over were specified at the moment of instantiation of the class. This is not necessary. Let's see how it would look like if we create a class that can generate random numbers for as long as we want:

```python
import random 

class RandomNumbers:
    def __getitem__(self, item):
        return random.randint(0, 10)
``` 

And we can use it as follows:

```python
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
And then we don't need to break the ``for`` loop, it will naturally stop once a ``10`` is generated within the class. 

A situation where you may think about applying this method is, for example, if you are acquiring a signal from a device which generates sequential values. It may be frames being acquired from a camera, analog values read by a simple device one after the other. I will use these techniques in a later tutorial focusing on the interfacing with devices. 

Another example of an iterable that generates values only when requested is when we open a file. Python does not load into memory all the contents but instead we can iterate over each line. This is how it looks like with [one of the articles](https://github.com/PFTL/website_articles/blob/master/articles/39_use_arduino_with_python.md) of this website:

```python
>>> with open('39_use_arduino_with_python.md', 'r') as f:
...     for line in f:
...         print(line) 
...
Using Python to communicate with an Arduino
===========================================
    ...
```
This is what allows you to work with files which are much bigger than what it's possible to hold in memory. You can also quickly check it by looking at the sizes:

```python
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
You can see that the size of the ``f`` variable is much smaller than the total size of the contents of the file added up. This is of course not an accurate method to know the size of a file, but it can get you an idea of the areas in which you can start considering iterables instead of loading a wealth of information to the memory of the computer.

## Iterators
In the section above, we have seen that we can create an iterable object just by defining an appropriate ``__getitem__`` method. We have also seen that several objects with which we have probably already interacted such as lists, files, etc. are iterables. However, there is something happening under the hood which is worth discussing, and are called *iterators*. 

The difference between an iterable and an iterator is very subtle. Perhaps it can be thought as the difference between a class and an object. We know we can iterate over a list, or over a custom object. But how is Python dealing with that flow? We can, very quickly, see how it works with a simple list:

```python
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

In the code above, we have constructed an iterator using the built-in function ``iter``. This function takes as an argument an iterable and returns an iterator. The iterator is the object which understands what ``next`` means. As a matter of fact, the iterator is iterable itself: 

```python
>>> it = iter(var)
>>> for element in it:
...     print(element)
...
a
1
0.1
```

If we would like more control on the process, Python gives it to us. An iterator must implement two methods: ``__next__`` and ``__iter__``. If we want our class to use a specific iterator, then it should also specify an ``__iter__`` method. Let's see what would happen with the ``Sentence`` class. To make it more obvious, we will have it iterating through the words but in backwards order. First, we define the iterator:

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

The code above is relatively clear, you only have to keep in mind that the only way of specifying a ``next`` value is by keeping track of the current index being returned. Now that we have our iterator, we must update the original ``Sentence`` class:

```python
class Sentence:
    def __init__(self, text):
        self.words = text.split(' ')
    
    def __iter__(self):
        return SentenceIterator(self.words)
```

And now comes the fun part:

```python
>>> text = "This is a text to test if our iterator returns values backwards"
>>> s = Sentence(text)
>>> s[0]
Traceback (most recent call last):
  File "<input>", line 1, in <module>
TypeError: 'Sentence' object does not support indexing
```
Indeed, we can't access the elements of our ``Sentence`` object because we haven't specified the appropriate ``__getitem__``. However, the ``for`` loop will still work:

```python
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

You see the order in which it is being printed is backwards, as expected. Just the first word is mistaken, because we start in ``0`` and that corresponds also to the last index: ``-12``. You can find a solution to this issue if you want, just requires a bit of thinking. 

The example above also works with ``iter``:

```python
>>> it = iter(s)
>>> next(it)
'This'
>>> next(it)
'backwards'
```

It is important to note that once the iterator is exhausted, there is nothing else we can do with it, while we can always go through the elements in the ``Sentence`` object within a ``for`` loop.

The only missing part now is adding the ability to access the words by index, but in the proper order. If we develop a ``__getitem__``, what do you expect to happen to the ``for`` loop?

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

```python
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

So now you see, we can access the elements through their index, and we can iterate over them in a backwards order. 

### Can Sentence have a \_\_next\_\_ method?
In principle, you can make ``Sentence`` also an iterator, by adding a ``__next__`` method. But this may be a very bad idea. Remember that iterators work until they exhaust themselves, and they maintain an internal index. If you mix ``Sentence`` as both an iterable and an iterator, whenever you encounter a double loop, for example, you will run into problems. 

<div class="admonition note">
Splitting the iterator and the iterable is normally a good idea. You can read a bit more about it on the book Fluent Python, by Luciano Ramalho. Chapter 14 is entirely dedicated to this topic. 
</div>

I do believe that for people working with scientific instruments, splitting the iterable and the iterator can have some benefits. It is plausible to think scenarios where you would want to have an iterator that behaves in different ways, depending on a parameter passed to the iterable. For example, imagine you are reading from a camera, and want to achieve something like:

```python
>>> for frame in camera:
...     analyze(frame)
```

Very simple, you analyze every frame generated by a camera. However, you may be in a situation where frames are generated by an external trigger, and therefore you want to attach the timestamp to each frame, or it may be that the camera acquires a finite number of frames at a given framerate. In principle, it could be the iterator the one taking care of it. 

In many cases, however, we don't need to define a separated iterator, we can use **generators** directly in our ``__iter__`` definition. That is the focus of the next section

## Generators: Using the yield keyword
The last important topic to cover is the role of **generators**, which, in my opinion, are still under-utilized, but they open very interesting doors. A classic example is generating an infinite list of numbers. We know that it is impossible to store a list of infinite numbers in memory.
