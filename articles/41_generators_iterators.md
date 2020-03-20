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

As you can imagine, an iterable is an element in Python that allows to go through its elements one by one. It seems like almost data type that allows to group information together is iterable. So, the next logical step is to think whether we can define our own iterable. 

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

However, 



